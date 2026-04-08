# 🚀 Lab 05b: O Pipeline de Confiança (YAML)

Neste laboratório, você orquestrará a sua primeira esteira de **CI (Continuous Integration)**. O Azure DevOps enviará o comando e o seu Agente no **Killercoda** executará os testes unitários do modelo automaticamente.

---

## 🏗️ Atividade 1: Criando o Repositório e os Arquivos

Para que o pipeline funcione, o Azure DevOps precisa "enxergar" o seu código.

1.  No Azure DevOps, vá em **Repos** -> **Files**.
2.  Como o repositório está vazio, clique em **Initialize** (no final da página) para criar um branch `main` com um arquivo README.
3.  Agora, vamos adicionar os arquivos do **Lab 04** (Pytest) diretamente pelo portal:
    *   Clique em **+ New** -> **File**.
    *   **Nome:** `model.py`. Cole o conteúdo do Lab 04.
    *   **Nome:** `test_model.py`. Cole o conteúdo do Lab 04.
    *   Clique em **Commit** para salvar.

---

## 📜 Atividade 2: Escrevendo a Receita (YAML)

O arquivo YAML define o que o pipeline deve fazer.

1.  Vá em **Pipelines** -> **Create Pipeline**.
2.  Selecione **Azure Repos Git**.
3.  Selecione o seu repositório.
4.  Selecione **Starter pipeline**.
5.  **IMPORTANTE:** Apague todo o conteúdo e cole o código abaixo:

```yaml
# azure-pipelines.yml
trigger:
- main

# DIRECIONANDO PARA O KILLERCODA
pool:
  name: 'default' 

steps:
- script: |
    # Garantir que o ambiente de testes esteja pronto no Runner
    sudo apt update && sudo apt install -y python3-venv
    python3 -m venv venv
    source venv/bin/activate
    pip install pytest
    
    # Rodar os testes que estão no repositório
    pytest test_model.py
  displayName: 'Executar Testes Unitários de ML (Pytest)'
```

---

## 🚀 Atividade 3: O "Disparo" e a Validação

1.  Clique em **Save and run**.
2.  **⚠️ ATENÇÃO:** Na primeira execução, o Azure DevOps pode exibir um aviso: **"This pipeline needs permission to access a resource before this run can continue to Executar Testes..."**. 
    *   Clique em **View** -> **Permit** -> **Permit**. (Isso autoriza o Pipeline a usar o seu Agente Privado).
3.  Clique no **Job** para ver os logs em tempo real.
3.  **Assista à Mágica:** Você verá o Azure DevOps enviando os arquivos para o Killercoda e o terminal do Killercoda executando o `pytest`.
4.  Se as "bolinhas verdes" aparecerem no dashboard da Azure, sua primeira esteira de MLOps está **HOMOLOGADA**!

---

## 🧪 Atividade 4: O Cenário de Caos (Falha de Build)

Um bom pipeline deve impedir que código ruim chegue na frente.

1.  Vá em **Repos** -> **Files**.
2.  Edite o arquivo `model.py` e altere o cálculo do score para algo errado (ex: `return 1.5`).
3.  Faça o **Commit**.
4.  Vá em **Pipelines** e observe o disparo automático.
5.  **O que aconteceu?** O pipeline deve ficar **VERMELHO** (Fail). O Pytest detectou que o modelo quebrou o contrato de sanidade (Score > 1.0).

### 🎉 Conclusão
Você acaba de implementar uma **Governança Automatizada**. O modelo só "vive" se passar pelos seus testes de confiança!

---

## 🧹 Atividade 5: Limpeza e Segurança (Higiene de MLOps)

Ao final do treinamento, é uma boa prática de segurança (Compliance) remover os acessos concedidos:

1.  **Remover o Agente:** No Killercoda, pare a execução do agente (`Ctrl + C`) e execute `./config.sh remove --auth pat --token <SEU_PAT>`. No portal Azure DevOps, vá em **Agent pools** -> **Default** -> **Agents** e exclua o `killercoda-agent`.
2.  **Revogar o PAT:** Vá em **User Settings** -> **Personal Access Tokens** e clique no ícone de lixeira (Revoke) ao lado do token `KillercodaAgent`.
3.  **Excluir o Pipeline:** Vá em **Pipelines**, clique nos `...` ao lado do seu pipeline e selecione **Delete**.
4.  **Excluir o Projeto (Opcional):** Se desejar remover tudo, vá em **Project Settings** -> **Overview** e clique no botão **Delete** no final da página.
