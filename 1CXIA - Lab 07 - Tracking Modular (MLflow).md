# 🧪 Lab 07: Tracking Modular (MLflow & Model Registry)

Neste laboratório, você aprenderá a organizar a "bagunça" da experimentação. Sairemos de scripts que imprimem métricas no terminal para uma governança de nível sênior, utilizando o **MLflow** para rastrear parâmetros, métricas e registrar o modelo vencedor.

---

## 🎯 Objetivo do Lab
1.  Implementar o **Tracking** de parâmetros e métricas no código (Fase Lite).
2.  Visualizar e comparar experimentos via **MLflow UI** (Fase Visual).
3.  Promover um modelo para o **Model Registry** (Fase Full).
4.  Simular a **Aprovação de Governança** no Azure DevOps.

---

## 🖥️ Acesso ao Ambiente
Utilizaremos o playground de **Ubuntu** do Killercoda:
👉 **Acesse aqui:** [https://killercoda.com/playgrounds/scenario/ubuntu](https://killercoda.com/playgrounds/scenario/ubuntu)

---

## 🚀 Atividade 0: Bootstrap (Restauração de Ambiente)

Se a sua sessão expirou, execute o bloco abaixo para restaurar o projeto completo (incluindo o DVC do Lab 06):

```bash
# 1. Reconstrução do Ambiente (Código + DVC + Git + Agente)
cd ~ && rm -rf ~/lab04 && mkdir ~/lab04 && cd ~/lab04
cat <<EOF > model.py
def predict_credit_score(income, age, debt):
    if income < 0 or age < 0 or debt < 0: return None
    score = (income / 10000) + (age / 100) - (debt / 5000)
    return max(0, min(1, score))
EOF

# 2. Setup de Dependências
sudo apt update && sudo apt install -y python3-venv --quiet
python3 -m venv venv && source venv/bin/activate
pip install pytest dvc mlflow --quiet

# 3. Restaurar Git e DVC
git init --quiet && dvc init --quiet
git branch -M main
git config user.email "aluno@caixa.gov.br" && git config user.name "Aluno CAIXA"
mkdir data && echo "id,valor,data,fraude\n1,100,2026-04-01,0" > data/transacoes.csv
dvc add data/transacoes.csv --quiet
git add . && git commit -m "feat: restore environment for Lab 07" --quiet

echo "✅ Ambiente Pronto! MLflow instalado."
```

---

## 🟢 Atividade 1: Fase 1 - Tracking "Ghost" (No Code)

Vamos ensinar o seu script a "falar" com o MLflow sem precisar de um servidor ligado agora.

**Missão:** Crie o arquivo `train.py` para simular o treinamento do modelo de crédito:

```bash
cat <<EOF > train.py
import mlflow
import random

# Forçar persistência em banco local (Senior Governance)
mlflow.set_tracking_uri("sqlite:///mlflow.db")

# Iniciar o rastro (Tracking)
with mlflow.start_run(run_name="Experimento_BERT_V1"):
    # 1. Logar Parâmetros (Configurações)
    mlflow.log_param("modelo_base", "bert-base-multilingual")
    mlflow.log_param("epochs", 5)
    
    # 2. Simular métricas (Onde a mágica acontece)
    accuracy = random.uniform(0.85, 0.98)
    mlflow.log_metric("accuracy", accuracy)
    mlflow.log_metric("loss", 1 - accuracy)
    
    print(f"Treino Finalizado! Acurácia: {accuracy:.4f}")
EOF

# Rodar o treino
python3 train.py
```
*Observe que o arquivo mlflow.db foi criado. O MLflow salvou tudo em uma base local.*

---

## 🟡 Atividade 2: Fase 2 - Consultando Resultados via CLI (Headless)

Como a interface Web pode ser pesada, aprenderemos a extrair o rastro diretamente pelo terminal — exatamente como um Agente de Automação faz.

1.  No terminal, liste os experimentos registrados:
    ```bash
    mlflow experiments search
    ```
2.  Liste as rodadas (Runs) e veja as métricas de Acurácia:
    ```bash
    mlflow runs list --experiment-id 0
    ```
3.  **Desafio:** Rode o `train.py` novamente. Note que um novo `run_id` será gerado com uma acurácia diferente. A "memória" do seu modelo está sendo construída na base `mlflow.db`.

---

## 🔴 Atividade 3: Fase 3 - Model Registry (OPCIONAL)

**⚠️ ALERTA DE ESTABILIDADE:** Esta etapa envolve o registro de artefatos binários. Em sessões do Killercoda com muitos alunos, o consumo de disco e RAM pode travar ou encerrar sua sessão ("Killed"). Execute apenas se o seu ambiente estiver estável.

Um modelo só é "Sênior" se ele estiver registrado e versionado estruturalmente.

**Missão:** Crie e execute o script para registrar o modelo vencedor. 
*(Abra um NOVO TERMINAL no Killercoda e ative o venv: `source ~/lab04/venv/bin/activate`)*.

```bash
# 1. Criar o script de registro (Versão Sklearn Simulado)
cat <<EOF > register_model.py
import mlflow
import mlflow.sklearn
from sklearn.linear_model import LogisticRegression

# Forçar persistência em banco local (Senior Governance)
mlflow.set_tracking_uri("sqlite:///mlflow.db")

# Iniciar uma run para registro
with mlflow.start_run(run_name="Registro_Oficial_CAIXA"):
    # Criar um modelo fictício para satisfazer o MLflow
    model_dummy = LogisticRegression()
    
    # Salvar o modelo estruturado (Isso cria a pasta 'model')
    mlflow.sklearn.log_model(model_dummy, "model")
    
    # Obter o Run ID atual
    run_id = mlflow.active_run().info.run_id
    
    # Registrar o modelo no Registry oficial
    model_uri = f"runs:/{run_id}/model"
    mlflow.register_model(model_uri, "Modelo_Credito_CAIXA")
    
    print(f"✅ Modelo registrado com sucesso!")
    print(f"📍 Run ID: {run_id}")
EOF

# 2. Instalar dependência necessária para o sabor sklearn
pip install scikit-learn --quiet

# 3. Executar o registro
python3 register_model.py
```
4.  **Verificação via CLI:** Em vez de usar a interface pesada, valide o registro diretamente pelo terminal usando este comando Python:
    ```bash
    # Listar todos os modelos registrados no Registry oficial
    python3 -c "import mlflow; [print(f'📍 Modelo: {m.name}') for m in mlflow.search_registered_models()]"
    ```
    *Note que o 'Modelo_Credito_CAIXA' aparecerá na lista como oficialmente registrado.*

---

## ☁️ Atividade 4: Governança no Azure DevOps

Agora, vamos simular o **Portão de Aprovação (Manual Gate)**.

1.  No Azure DevOps, vá em **Pipelines -> Environments**.
2.  Crie um Environment chamado `Producao`.
3.  Clique nos `...` -> **Approvals and checks**.
4.  Adicione **Approvals** e coloque o seu e-mail.
5.  **Reflexão:** Agora, mesmo que o MLflow diga que o modelo é ótimo, ele só vai para o ar se você (o gestor) clicar em **Approve** no ADO.

---

## 💡 Lição de Casa (Cenário de Caos)
No MLflow UI, observe a acurácia. Se ela cair abaixo de 90%, como você usaria o **Model Registry** para fazer um **Rollback** para a Versão anterior? 

## ☁️ Atividade 5: Rastro de Auditoria no Azure Pipelines

Para a CAIXA, o mais importante não é o gráfico, mas a **Linhagem**. Como este repositório é novo, vamos criar a "receita" do Pipeline para automatizar o treino e carimbar o ID do experimento.

1.  **Criar o arquivo de Pipeline:** No seu terminal, crie o arquivo `azure-pipelines.yml` (**copie o bloco inteiro**):
```bash
cat <<'EOF' > azure-pipelines.yml
trigger:
- main

pool:
  name: 'default'

steps:
- script: |
    # Criar venv isolado para o Agente (Evita erro de permissão do root)
    python3 -m venv venv_ci
    source venv_ci/bin/activate
    pip install mlflow scikit-learn
    
    # Executar o treino (O arquivo train.py deve estar no repo)
    python3 train.py
  displayName: 'Executar Treino e Tracking'

- script: |
    # Capturar o Run ID do MLflow usando Python (Sênior e Independente de Backend)
    source venv_ci/bin/activate
    RUN_ID=$(python3 -c "import mlflow; from mlflow.tracking import MlflowClient; client = MlflowClient(); run = client.search_runs(experiment_ids=['0'], max_results=1, order_by=['attribute.start_time DESC'])[0]; print(run.info.run_id)")
    echo "##vso[task.setvariable variable=MLFLOW_RUN_ID]$RUN_ID"
    echo "Modelo Validado! MLflow Run ID: $RUN_ID"
  displayName: 'Registrar Linhagem de Auditoria'
EOF
```

2.  **Enviar para o Azure:**
```bash
# Vincular ao seu repositório (Use a URL do Caixa_DVC criada no Lab 06)
git remote add origin <URL_DO_REPOSITORIO_CAIXA_DVC>

# ADICIONAR TUDO (Inclusive o train.py e o YAML)
git add train.py azure-pipelines.yml
git commit -m "ci: add training script and audit pipeline"
git push -f origin main
```

3.  **Bootstrap do Agente (Obrigatório):** Se você não tem o Agente rodando, execute o bloco abaixo integralmente para instalar e conectá-lo:
```bash
# 1. Instalação e Permissões

sudo useradd -m agentuser || true
echo "agentuser ALL=(ALL) NOPASSWD:ALL" | sudo tee /etc/sudoers.d/agentuser
sudo mkdir -p /home/agentuser/myagent && cd /home/agentuser/myagent
sudo curl -fkSL -o agent.tar.gz https://download.agent.dev.azure.com/agent/4.270.0/vsts-agent-linux-x64-4.270.0.tar.gz
sudo tar zxvf agent.tar.gz
sudo ./bin/installdependencies.sh
sudo chown -R agentuser:agentuser /home/agentuser/myagent && sudo chmod -R 777 /home/agentuser/myagent

sudo rm -f /etc/resolv.conf
sudo tee /etc/resolv.conf <<EOF
nameserver 8.8.8.8
nameserver 8.8.4.4
EOF

# 2. Configuração (Substitua os valores <...>)
sudo chmod -R 777 /home/agentuser/myagent
sudo -u agentuser bash -c "cd ~/myagent && ./config.sh --unattended \
  --url https://dev.azure.com/<SUA_ORGANIZACAO> \
  --auth pat \
  --token <SEU_PAT> \
  --pool default \
  --agent killercoda-agent \
  --replace"

# 3. Iniciar o Agente
sudo -u agentuser bash -c "cd ~/myagent && ./run.sh"
```

4.  **Verificar no Portal:** No Azure DevOps, vá em **Pipelines** -> **Create Pipeline** (aponte para o seu repo `Caixa_DVC` se ele não detectar automaticamente).
5.  Abra a execução do Pipeline e veja o ID do experimento registrado nos logs. **Seu deploy agora tem rastro oficial.**

---

## 🎁 Conteúdo Extra: MLflow UI (Interface Visual)

**⚠️ REQUISITO:** Use apenas se o seu ambiente estiver rápido e você estiver sozinho na sessão. A interface Web consome muita RAM.

1.  No terminal, inicie o servidor com o comando de liberação:
    ```bash
    mlflow ui --host 0.0.0.0 --port 5000 --workers 1 --allowed-hosts "*"
    ```
2.  No topo da interface do Killercoda, clique no ícone **Traffic/Ports**.
3.  Digite **5000** e clique em **Access**.
4.  Explore os gráficos e compare as métricas de forma visual.

### ✅ Lab 07 Finalizado!
Amanhã: **Deploy em Escala (Aula 05)**.
