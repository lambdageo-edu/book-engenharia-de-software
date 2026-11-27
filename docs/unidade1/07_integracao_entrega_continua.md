# Integração e Entrega Contínua (CI/CD)

## 🔗 Por que integração contínua

**Integração contínua (CI — *Continuous Integration*)** é a prática de integrar mudanças de código na branch principal com frequência (idealmente, várias vezes ao dia), com cada integração validada automaticamente por um pipeline que compila, analisa e testa o código.

A motivação original, descrita por Martin Fowler há mais de duas décadas, é simples: quanto mais tempo uma mudança fica isolada em uma branch separada, mais ela diverge do resto do código, e mais doloroso (e arriscado) se torna integrá-la depois — o clássico problema de "*merge hell*". Integrar cedo e com frequência transforma um grande risco ocasional em um risco pequeno e constante, muito mais fácil de gerenciar.

**Entrega contínua (CD — *Continuous Delivery*)** estende essa ideia: se toda mudança validada no pipeline está, por definição, em um estado potencialmente pronto para produção, o processo de colocá-la em produção também pode — e deve — ser automatizado. A diferença entre *Continuous Delivery* e *Continuous Deployment* é sutil, mas importante:

* **Continuous Delivery**: toda mudança aprovada pelo pipeline está pronta para ir a produção a qualquer momento, mas o deploy em si ainda pode exigir uma aprovação manual (ex: um clique para liberar).
* **Continuous Deployment**: toda mudança aprovada pelo pipeline vai para produção **automaticamente**, sem intervenção humana — o nível mais alto de automação, e o que exige a suíte de testes mais confiável.

## 🏗️ Anatomia de um pipeline

Um pipeline de CI/CD típico encadeia estágios, cada um servindo de "portão de qualidade" para o próximo — se um estágio falha, o pipeline para ali, e a mudança não avança:

1. **Build**: compilar o código (ou, em linguagens interpretadas como Python, instalar dependências e validar que o pacote pode ser construído).
2. **Análise estática**: rodar linters, formatadores e verificadores de tipo (Capítulo 5).
3. **Testes automatizados**: rodar a suíte de testes (Capítulo 6), tipicamente com um limite mínimo de cobertura.
4. **Empacotamento**: construir um artefato de deploy — no ecossistema moderno, quase sempre uma imagem de container.
5. **Deploy**: publicar o artefato em um ambiente (homologação, produção), com estratégia de rollout controlada.

## 🧑‍💻 Hands-on: pipeline de CI para a API de tarefas

Vamos construir um pipeline de GitHub Actions para a API de tarefas do Capítulo 6, cobrindo lint, testes com cobertura mínima obrigatória, e build de uma imagem de container.

### Estrutura do projeto

```
api-tarefas/
├── app/
│   ├── __init__.py
│   └── main.py
├── test_main.py
├── requirements.txt
├── requirements-dev.txt
├── Dockerfile
├── .dockerignore
└── .github/
    └── workflows/
        └── ci.yml
```

### Dependências

Separe o que a aplicação precisa para **rodar** do que só é necessário para **desenvolver e testar**. Assim, as ferramentas de teste não vão parar na imagem de produção.

`requirements.txt`:

```
fastapi>=0.115
uvicorn[standard]>=0.32
pydantic>=2.9
```

`requirements-dev.txt`:

```
-r requirements.txt
pytest>=8.3
pytest-cov>=6.0
httpx2>=2.9
ruff>=0.7
```

### O workflow: `.github/workflows/ci.yml`

```yaml
name: CI

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  test:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout do código
        uses: actions/checkout@v7

      - name: Configurar Python
        uses: actions/setup-python@v7
        with:
          python-version: "3.12"
          cache: "pip"
          cache-dependency-path: requirements*.txt

      - name: Instalar dependências
        run: pip install -r requirements-dev.txt

      - name: Rodar linter (ruff)
        run: ruff check .

      - name: Rodar testes com cobertura
        run: pytest -v --cov=app --cov-report=term-missing --cov-fail-under=80
```

!!! note "Validação deste workflow"
    O YAML acima foi validado neste material por *parsing* automatizado (`yaml.safe_load`), garantindo que sua sintaxe está correta. Ao adaptá-lo, atenção a um detalhe clássico de YAML: a chave `on:` pode ser interpretada por alguns parsers (não pelo GitHub Actions) como o booleano `true` — um problema puramente de ferramentas de terceiros que leem o arquivo, sem efeito no funcionamento real do workflow.

Repare na estrutura: `on` define os gatilhos (aqui, todo `push` ou `pull_request` para `main`); `jobs.test.steps` é a sequência de passos executados em uma máquina virtual limpa do GitHub a cada execução. A *action* `actions/setup-python` com `cache: "pip"` acelera execuções futuras reaproveitando o cache de dependências entre pipelines.

O parâmetro `--cov-fail-under=80` faz o próprio comando de teste **falhar** se a cobertura cair abaixo de 80% — transformando um limite de qualidade em um portão automático do pipeline, em vez de depender de alguém checar o relatório manualmente.

### Empacotando em um container: `Dockerfile`

```dockerfile
FROM python:3.12-slim

WORKDIR /app

COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

COPY app/ ./app/

RUN useradd --create-home appuser
USER appuser

EXPOSE 8000
CMD ["uvicorn", "app.main:app", "--host", "0.0.0.0", "--port", "8000"]
```

`.dockerignore`:

```
.venv/
.git/
__pycache__/
.pytest_cache/
.coverage
.env
```

Alguns cuidados de boas práticas embutidos neste `Dockerfile`:

* **Imagem base `slim`**: reduz o tamanho da imagem final e a superfície de ataque, evitando pacotes desnecessários presentes na imagem completa.
* **Copiar `requirements.txt` antes do código da aplicação**: aproveita o cache de camadas do Docker — se apenas o código muda (e não as dependências), a camada de instalação de dependências não precisa ser refeita a cada build.
* **`--no-cache-dir`**: evita que o `pip` guarde um cache de pacotes dentro da imagem, mantendo-a menor.
* **Usuário sem privilégios (`USER appuser`)**: por padrão o processo roda como `root` dentro do contêiner. Trocar o usuário limita o estrago caso a aplicação seja comprometida.
* **Só as dependências de execução**: a imagem instala `requirements.txt`, sem `pytest` nem `ruff`.
* **`.dockerignore`**: evita enviar ao *build* o ambiente virtual, o histórico do Git e arquivos locais como `.env`, que podem conter segredos.

Para estender o pipeline com o passo de build da imagem, adicionaríamos um job (ou passo) adicional usando `docker/build-push-action`, publicando a imagem em um registro (GitHub Container Registry, Docker Hub, ECR) a cada merge na branch principal — o primeiro passo em direção à entrega contínua.

=== "Continuous Delivery"
    Após o build da imagem, o pipeline aguarda uma aprovação manual (ex: um ambiente protegido do GitHub com *reviewers* obrigatórios) antes do deploy em produção — útil quando há razões de negócio (janelas de manutenção, aprovação de compliance) para controlar o momento do deploy.

=== "Continuous Deployment"
    Após o build da imagem, o deploy em produção acontece automaticamente, sem intervenção humana — depende de uma suíte de testes muito confiável e, tipicamente, de estratégias de mitigação de risco como *feature flags* e *rollout* gradual (abaixo).

## 🚦 Estratégias de deploy com baixo risco

Automatizar o deploy não significa "publicar tudo de uma vez para todo mundo". Estratégias comuns para reduzir o risco de um deploy automatizado:

* **Blue-Green Deployment**: mantém dois ambientes de produção idênticos (azul e verde); o tráfego é roteado para um deles enquanto o outro recebe a nova versão, e a troca de tráfego é uma operação quase instantânea — com rollback igualmente rápido, se necessário.
* **Canary Release**: a nova versão é liberada para uma pequena fatia do tráfego real (ex: 5%) antes de ser expandida gradualmente para 100%, monitorando métricas de erro a cada passo.
* **Feature Flags**: o código de uma nova funcionalidade já está em produção, mas desligado por uma configuração — permitindo separar o **deploy** do código (uma operação técnica) do **lançamento** da funcionalidade (uma decisão de produto), e habilitar a funcionalidade gradualmente ou apenas para usuários específicos.

!!! tip "Deploy frequente reduz risco, não aumenta"
    É comum a intuição de que "fazer deploy com mais frequência é mais arriscado". A evidência de pesquisas como o *State of DevOps* aponta o contrário: deploys pequenos e frequentes são mais fáceis de testar, mais fáceis de reverter e mais fáceis de depurar (menos mudanças acumuladas por deploy) do que grandes lançamentos espaçados — que concentram risco em vez de distribuí-lo.

## 🤖 Agentes de codificação e o pipeline

Uma tendência recente é a de agentes de codificação que não apenas sugerem trechos de código, mas abrem *pull requests* completos de forma autônoma — recebendo uma tarefa (ex: uma *issue* do GitHub), explorando o repositório, implementando a mudança, e submetendo-a para revisão. Do ponto de vista do pipeline de CI/CD, isso não muda a arquitetura discutida neste capítulo: um *pull request* aberto por um agente passa pelos mesmos portões de qualidade — lint, testes, cobertura mínima — que um *pull request* aberto por uma pessoa.

O que muda é a importância desses portões automatizados como linha de defesa: se o volume de *pull requests* aumenta porque parte deles é gerada por agentes (Capítulo 5), o pipeline de CI se torna a garantia consistente de que nenhuma mudança — humana ou não — chega à branch principal sem passar pelos mesmos critérios mínimos de qualidade.

---

## 🎯 Atividades

1. Adicione o workflow `ci.yml` deste capítulo a um repositório GitHub contendo a API de tarefas do Capítulo 6 e confirme que ele roda com sucesso a cada `push`.
2. Modifique o pipeline para falhar propositalmente (ex: aumente `--cov-fail-under` para 100, acima da cobertura atual do seu código) e observe o comportamento do GitHub Actions.
3. Construa a imagem Docker localmente (`docker build -t api-tarefas .`) e rode um container a partir dela (`docker run -p 8000:8000 api-tarefas`), acessando `http://localhost:8000/docs` para ver a documentação interativa gerada automaticamente pelo FastAPI.
4. Descreva, para um sistema hipotético que você conheça, qual estratégia de deploy (blue-green, canary, feature flags) você usaria para lançar uma mudança arriscada, e por quê.

## 📚 Para saber mais

* FOWLER, Martin. *Continuous Integration* — [martinfowler.com/articles/continuousIntegration.html](https://martinfowler.com/articles/continuousIntegration.html)
* HUMBLE, Jez; FARLEY, David. *Continuous Delivery*. Addison-Wesley, 2010.
* Documentação oficial: [GitHub Actions](https://docs.github.com/actions), [Docker](https://docs.docker.com/)
* Relatório *State of DevOps* (DORA/Google Cloud), publicado anualmente.
