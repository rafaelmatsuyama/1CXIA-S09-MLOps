# 🧪 Lab 04: O "Teste de DNA" do Modelo (Pytest)

Neste laboratório, você aprenderá a criar a primeira camada de defesa de um sistema de MLOps: **Testes Unitários para Modelos de ML**. Validaremos se a lógica de inferência é robusta e livre de vieses básicos antes de qualquer deploy.

---

## 🌐 Acesso ao Ambiente

Utilizaremos o playground de **Ubuntu** do Killercoda para garantir um ambiente limpo e isolado:
👉 **Acesse aqui:** [https://killercoda.com/playgrounds/scenario/ubuntu](https://killercoda.com/playgrounds/scenario/ubuntu)

---

## 🛠️ Atividade 1: Preparação do Ambiente

No terminal do **Killercoda**, prepare o ambiente e instale as dependências:

```bash
# Garantir que o gerenciador de pacotes esteja atualizado
sudo apt update

# Instalar o suporte a ambientes virtuais (venv) e o Pytest
sudo apt install -y python3-venv

# Criar pasta isolada para o lab e o ambiente virtual
mkdir ~/lab04 && cd ~/lab04
python3 -m venv venv
source venv/bin/activate

# Instalar o Pytest no ambiente virtual
pip install pytest
```

---

## 🔬 Atividade 2: O "Modelo" (Lógica de Inferência)

Imagine que treinamos um modelo de **Análise de Crédito**. Vamos criar um script que simula a saída desse modelo.

**Missão:** Execute o comando abaixo no terminal para criar o arquivo `model.py`:

```bash
cat <<EOF > model.py
# model.py
def predict_credit_score(income, age, debt):
    """
    Simula um modelo de ML que retorna a probabilidade de aprovação (0 a 1).
    """
    if income < 0 or age < 0 or debt < 0:
        return None  # Erro de entrada
    
    # Lógica simplificada (Simulando o modelo treinado)
    score = (income / 10000) + (age / 100) - (debt / 5000)
    
    # Garantir que o resultado esteja entre 0 e 1
    return max(0, min(1, score))
EOF
```

---

## ✅ Atividade 3: Criando os Testes de Confiança

Agora, vamos escrever os testes que garantirão que nosso "modelo" não tem comportamentos absurdos.

**Missão:** Execute o comando abaixo no terminal para criar o arquivo `test_model.py`:

```bash
cat <<EOF > test_model.py
# test_model.py
from model import predict_credit_score

def test_sanity_range():
    """Teste de Sanidade: O score deve estar sempre entre 0 e 1."""
    result = predict_credit_score(5000, 30, 1000)
    assert 0 <= result <= 1

def test_negative_input():
    """Teste de Robustez: O modelo deve lidar com entradas negativas."""
    result = predict_credit_score(-100, 30, 1000)
    assert result is None

def test_high_debt_low_income():
    """Teste de Lógica: Muita dívida e pouca renda deve dar score baixo."""
    result = predict_credit_score(1000, 20, 10000)
    assert result < 0.2

def test_bias_check():
    """Teste de Viés: O modelo não deve aprovar todo mundo (False Positive)."""
    # Simulando um perfil de alto risco
    result = predict_credit_score(1200, 18, 8000)
    assert result < 0.5
EOF
```

---

## 🚀 Atividade 4: Execução e Validação

No terminal, execute o comando para rodar os testes:

```bash
pytest test_model.py
```

### 🎯 O que observar:
1.  **Bolinhas Verdes (`....`):** Significa que o DNA do seu modelo está saudável.
2.  **Falha (`F`):** Se um teste falhar, o pipeline que criaremos no Azure DevOps **bloqueará o deploy**.

---

## 💡 Desafio Extra (Cenário de Caos)
Altere a lógica no `model.py` para que ele ignore a dívida (`debt`). Rode o `pytest` novamente. O teste `test_high_debt_low_income` falhou? É assim que pegamos bugs de ML antes da produção!
