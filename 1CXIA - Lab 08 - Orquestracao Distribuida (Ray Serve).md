# Lab 08: Orquestração Distribuída (Serving com Ray Serve)

## 🎯 Objetivo
Elevar o nível da discussão de MLOps: em vez de gerenciar containers individuais, vamos gerenciar uma **Plataforma de Inferência Distribuída**. Aprenderemos a escalar réplicas, compor serviços e realizar Canary Releases via código (Python-native).

---

## 🏗️ Ambiente de Simulação (Killercoda)
Para este laboratório, utilizaremos um ambiente Ubuntu limpo no Killercoda.
*   **Acesso:** [Killercoda Playgrounds - Ubuntu](https://killercoda.com/playgrounds/scenario/ubuntu)
*   **Tempo de Sessão:** 60 minutos (Suficiente para o setup e execução).

---

## 🧠 O Cenário: O Cérebro Distribuído da CAIXA
Na Semana 10, vocês verão como o Spark processa dados em massa. Hoje, veremos como o **Ray Serve** (utilizado pela OpenAI para o ChatGPT) gerencia a inteligência em tempo real. O objetivo é servir modelos BERT de forma resiliente, sem depender de arquivos de configuração de infraestrutura (YAML/Nginx).

---

## 🚀 Passo 1: Preparação do Ambiente no Killercoda
No terminal do Killercoda, vamos instalar o suporte a ambientes virtuais e isolar nossa plataforma de IA dos pacotes protegidos do sistema Ubuntu.

```bash
# 1. Instalação do suporte a venv (Ubuntu standard)
apt update && apt install -y python3-venv

# 2. Criação e Ativação do Ambiente Virtual (venv)
python3 -m venv ~/venv-ray
source ~/venv-ray/bin/activate

# 3. Instalação do Ray Serve e dependências (Agora sem restrições)
pip install "ray[serve]" fastapi uvicorn

# 4. Criação do diretório do Lab
mkdir ~/lab08-ray && cd ~/lab08-ray
```

---

## 🚀 Passo 2: Criando o Script de Deploy (`deploy_caixa.py`)
Vamos criar um único script que define toda a nossa infraestrutura distribuída. Utilize o editor ou o comando `cat` abaixo:

```bash
cat <<EOF > deploy_caixa.py
import ray
from ray import serve
import random
from fastapi import FastAPI

app = FastAPI()

# 1. Modelo Versão 1.0.0 (Estável - 2 Réplicas)
# ray_actor_options={"num_cpus": 0.1} permite rodar vários modelos em 1 CPU
@serve.deployment(num_replicas=2, ray_actor_options={"num_cpus": 0.1})
class SentimentV1:
    def __call__(self):
        return {"model": "BERT-CAIXA", "version": "1.0.0", "engine": "Ray-Node-Alpha"}

# 2. Modelo Versão 2.0.0 (Canary - 1 Réplica)
@serve.deployment(num_replicas=1, ray_actor_options={"num_cpus": 0.1})
class SentimentV2:
    def __call__(self):
        return {"model": "BERT-CAIXA", "version": "2.0.0-canary", "engine": "Ray-Node-Beta"}

# 3. O Roteador Inteligente (Ingress Router)
@serve.deployment(ray_actor_options={"num_cpus": 0.1})
@serve.ingress(app)
class IngressRouter:
    def __init__(self, v1, v2):
        self.v1 = v1
        self.v2 = v2

    @app.get("/")
    async def route(self):
        # Lógica de Plataforma: 90/10 para o Canary
        if random.random() < 0.1:
            return await self.v2.remote()
        return await self.v1.remote()

# 4. Bind das peças do Grafo
v1 = SentimentV1.bind()
v2 = SentimentV2.bind()
deployment = IngressRouter.bind(v1, v2)
EOF
```

---

## 🚀 Passo 3: Executando o Cluster e o Deploy (Modo Estabilidade)
No Killercoda (2GB RAM), iniciar o Ray com as configurações padrão causará erro de memória (OOM). Vamos iniciar o cluster manualmente com o Dashboard desativado e limite de memória.

```bash
# 1. Parar qualquer instância anterior (Limpeza de Guerra)
ray stop -f

# 2. Iniciar o Cluster Ray com limites de RAM e SEM Dashboard
# --object-store-memory: Limita o cache de memória do Ray
# --include-dashboard=false: Poupa ~200MB de RAM vital
ray start --head --include-dashboard=false --object-store-memory 200000000

# 3. Executar o Deploy no cluster já iniciado
serve run deploy_caixa:deployment --address auto
```

*Nota: O parâmetro `--address auto` conecta o Serve ao cluster que acabamos de configurar.*


---

## 🚀 Passo 4: Validação (A Prova dos 10%)
Abra um **segundo terminal** no Killercoda (clique no ícone de '+' acima do terminal) e **ative o ambiente virtual novamente**:

```bash
# 1. Ativar o venv no NOVO terminal
source ~/venv-ray/bin/activate

# 2. Simular 20 requisições para ver o balanceamento automático
for i in {1..20}; do curl -s http://localhost:8000; echo ""; done
```
*Note que a maioria das respostas virá da versão 1.0.0, mas a v2 (Canary) aparecerá ocasionalmente.*

---

## 🚀 Passo 5: Monitoramento do Cluster
Para garantir que o Ray não sofra com a falta de memória (OOM) no Killercoda, desativamos o **Dashboard**. Isso faz com que comandos de alta abstração (como `serve status` e `ray list actors`) falhem, pois eles dependem da API do Dashboard (porta 8265).

A fonte de verdade para a saúde do seu cluster será o `ray status`.

```bash
# Ver o uso de recursos, CPUs alocadas e saúde dos nós
ray status
```



---

## 🧪 Desafio: Escala a Quente
Como você faria para aumentar a `num_replicas` da `SentimentV1` para 3 sem derrubar o serviço? 
*(Dica: Altere o arquivo `deploy_caixa.py` e o `serve run` detectará a mudança automaticamente!)*

---

## 🏁 Por que Ray Serve e não Nginx?
- **Abstração:** Você escala modelos, não containers.
- **Python-First:** Toda a infraestrutura é código versionável.
- **Futuro:** É a base para entender os Sistemas Distribuídos que vocês verão na próxima semana com Spark.

---

## 🏆 Gabarito do Desafio: Escala a Quente

### 1. Alteração do Código (`deploy_caixa.py`)
A infraestrutura é o código. Basta alterar o parâmetro `num_replicas`:

```python
@serve.deployment(num_replicas=3, ray_actor_options={"num_cpus": 0.1})
class SentimentV1:
    ...
```

### 2. Aplicação da Mudança
No terminal com o `venv` ativo, execute novamente:
`serve run deploy_caixa:deployment --address auto`

O Ray detectará o "diff" e subirá as 3 novas réplicas sem interromper as antigas (Zero Downtime).

