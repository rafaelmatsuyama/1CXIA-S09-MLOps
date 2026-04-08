# 🛠️ Lab 05a: Setup do Private Agent (Killercoda)

Neste laboratório, você transformará o seu ambiente do **Killercoda** em um "Data Center Privado" da CAIXA. Configuraremos um **Self-hosted Agent** que se conectará ao seu Azure DevOps para executar os pipelines de MLOps com total controle e segurança.

---

## 🌐 Acesso ao Ambiente

1.  **Killercoda (Ubuntu):** [https://killercoda.com/playgrounds/scenario/ubuntu](https://killercoda.com/playgrounds/scenario/ubuntu)
2.  **Azure DevOps (SaaS):** [https://dev.azure.com](https://dev.azure.com)
    *   *Nota: Se você ainda não tem uma organização, crie uma gratuitamente com o seu e-mail da Microsoft.*

---

## 🔑 Atividade 1: Gerando o seu Passaporte (PAT)

O **PAT (Personal Access Token)** é a chave que permite ao Agent se autenticar na sua Organização Azure DevOps.

1.  No seu portal **Azure DevOps**, clique no ícone de "User Settings" (engrenagem com perfil) no canto superior direito.
2.  Selecione **Personal Access Tokens**.
3.  Clique em **+ New Token**.
4.  **Nome:** `KillercodaAgent`
5.  **Expiration:** 30 days (ou menos).
6.  **Scopes:** Selecione **Custom defined** e clique em **Show all scopes**.
7.  Localize **Agent Pools** e marque as caixas **Read & Manage**.
8.  Clique em **Create** e **COPIE o token gerado**. (Você não conseguirá vê-lo novamente!).

---

## 🚀 Atividade 2: Bootstrap do Agente (Killercoda)

Agora vamos "instalar" o Agente no Killercoda. Como o Agente **NÃO** pode rodar como `root`, vamos criar um usuário comum chamado `agentuser`.

**Missão:** Execute o comando abaixo no terminal para criar o script de setup:

```bash
cat <<EOF > setup_agent.sh
#!/bin/bash
# 1. Criar usuário e pasta (Agente não roda como root)
sudo useradd -m agentuser
echo "agentuser ALL=(ALL) NOPASSWD:ALL" | sudo tee /etc/sudoers.d/agentuser
sudo mkdir -p /home/agentuser/myagent
cd /home/agentuser/myagent

# 2. Baixar o Agent v4 (Oficial Microsoft)
sudo curl -fkSL -o vsts-agent-linux-x64-4.270.0.tar.gz https://download.agent.dev.azure.com/agent/4.270.0/vsts-agent-linux-x64-4.270.0.tar.gz
sudo tar zxvf vsts-agent-linux-x64-4.270.0.tar.gz

# 3. Instalar dependências e dar permissão total ao usuário na pasta
sudo ./bin/installdependencies.sh
sudo chown -R agentuser:agentuser /home/agentuser/myagent

echo "--- Agent Pronto para Configuração (com agentuser) ---"
EOF

# Dar permissão de execução e rodar
chmod +x setup_agent.sh
./setup_agent.sh
```

---

## ⚙️ Atividade 3: Configuração e Registro

Agora vamos conectar o seu Killercoda à sua conta da Azure através do novo usuário.

**Missão:** Execute o comando abaixo, substituindo os valores entre `< >` pelas suas informações:

```bash
# Entrar no contexto do usuário agentuser para configurar
sudo -u agentuser bash -c "cd ~/myagent && ./config.sh --unattended \
  --url https://dev.azure.com/<SUA_ORGANIZACAO> \
  --auth pat \
  --token <SEU_PAT_COPIADO> \
  --pool default \
  --agent killercoda-agent \
  --replace"

# Iniciar o Agente (O terminal ficará preso aqui, 'ouvindo' jobs)
sudo -u agentuser bash -c "cd ~/myagent && ./run.sh"
```

---

## ✅ Atividade 4: Validação no Portal

1.  No Azure DevOps, vá em **Project Settings** (canto inferior esquerdo).
2.  Clique em **Agent pools** (sob a aba Pipelines).
3.  Selecione o pool **Default**.
4.  Clique na aba **Agents**.
5.  Você deverá ver o **killercoda-agent** com o status **Online** (Bolinha Verde).

### 🎉 Parabéns! 
Seu "Data Center" no Killercoda agora é um executor de tarefas da sua Cloud Azure.
