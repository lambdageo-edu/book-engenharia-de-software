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
4. **Empacotamento**: construir um artefato de deploy — no ecossistema moderno, quase sempre uma imagem de contêiner.
5. **Deploy**: publicar o artefato em um ambiente (homologação, produção), com estratégia de rollout controlada.

## 🧑‍💻 Hands-on: pipeline de CI para a API de tarefas

Vamos construir um pipeline de GitHub Actions para a API de tarefas do Capítulo 6, cobrindo lint, formatação, verificação de tipos, testes com cobertura mínima obrigatória e build de uma imagem de contêiner.

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
fastapi==0.165.0
uvicorn[standard]==0.54.0
pydantic==2.14.4
```

`requirements-dev.txt`:

```
-r requirements.txt
pytest==9.2.0
bandit==1.9.4
pip-audit==2.10.1
pytest-cov==7.1.0
httpx2==2.16.2
ruff==0.17.6
mypy==2.4.1
```

As versões estão fixadas (`==`) para que o pipeline instale hoje o mesmo que instalará amanhã. São as versões testadas em novembro de 2026. Fixar não significa congelar: atualize de propósito, com os testes rodando, por exemplo a partir dos *pull requests* do Dependabot. Para fixar também as dependências indiretas, use um arquivo de *lock*, gerado por ferramentas como `uv` ou `pip-tools`.

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
        uses: actions/checkout@3d3c42e5aac5ba805825da76410c181273ba90b1 # v7.0.1

      - name: Configurar Python
        uses: actions/setup-python@6d4fd22b20465ccc0f7354476588e8df7c9ca0e0 # v7.1.0
        with:
          python-version: "3.14"
          cache: "pip"
          cache-dependency-path: requirements*.txt

      - name: Instalar dependências
        run: pip install -r requirements-dev.txt

      - name: Rodar linter (ruff)
        run: ruff check .

      - name: Conferir formatação
        run: ruff format --check .

      - name: Verificar tipos
        run: mypy app

      - name: Rodar testes com cobertura
        run: pytest -v --cov=app --cov-report=term-missing --cov-fail-under=80

  imagem:
    needs: test
    runs-on: ubuntu-latest

    steps:
      - name: Checkout do código
        uses: actions/checkout@3d3c42e5aac5ba805825da76410c181273ba90b1 # v7.0.1

      - name: Construir a imagem
        run: docker build -t api-tarefas .
```

Repare na estrutura: `on` define os gatilhos (aqui, todo `push` ou `pull_request` para `main`); `jobs.test.steps` é a sequência de passos executados em uma máquina virtual limpa do GitHub a cada execução. O job `imagem` declara `needs: test`, portanto só roda se o job de testes passar. A *action* `actions/setup-python` com `cache: "pip"` acelera execuções futuras reaproveitando o cache de dependências entre pipelines.

O parâmetro `--cov-fail-under=80` faz o próprio comando de teste **falhar** se a cobertura cair abaixo de 80% — transformando um limite de qualidade em um portão automático do pipeline, em vez de depender de alguém checar o relatório manualmente.

### Empacotando em um contêiner: `Dockerfile`

```dockerfile
FROM python:3.14-slim

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

O job `imagem` apenas constrói a imagem, para garantir que o `Dockerfile` continua funcionando. O passo seguinte seria usar `docker/build-push-action` para publicá-la em um registro (GitHub Container Registry, Docker Hub, ECR) a cada merge na branch principal — o primeiro passo em direção à entrega contínua.

Para ir além deste `Dockerfile`, veja o tutorial [Otimização de imagens: desenvolvimento e produção](https://lambdageo-edu.github.io/lambdageo-tutorials/docker/otimizacao-imagens-dev-vs-prod/). Os comandos básicos de Git e Docker estão no [apêndice](../apendices/git_conteineres.md).

=== "Continuous Delivery"
    Após o build da imagem, o pipeline aguarda uma aprovação manual (ex: um ambiente protegido do GitHub com *reviewers* obrigatórios) antes do deploy em produção — útil quando há razões de negócio (janelas de manutenção, aprovação de compliance) para controlar o momento do deploy.

=== "Continuous Deployment"
    Após o build da imagem, o deploy em produção acontece automaticamente, sem intervenção humana — depende de uma suíte de testes muito confiável e, tipicamente, de estratégias de mitigação de risco como *feature flags* e *rollout* gradual (abaixo).

## 🚦 Estratégias de deploy com baixo risco

Automatizar o deploy não significa "publicar tudo de uma vez para todo mundo". Estratégias comuns para reduzir o risco de um deploy automatizado:

* **[Blue-Green Deployment](https://www.martinfowler.com/bliki/BlueGreenDeployment.html)**: mantém dois ambientes de produção idênticos (azul e verde); o tráfego é roteado para um deles enquanto o outro recebe a nova versão, e a troca de tráfego é uma operação quase instantânea — com rollback igualmente rápido, se necessário.
* **[Canary Release](https://www.martinfowler.com/bliki/CanaryRelease.html)**: a nova versão é liberada para uma pequena fatia do tráfego real (ex: 5%) antes de ser expandida gradualmente para 100%, monitorando métricas de erro a cada passo.
* **[Feature Flags](https://martinfowler.com/articles/feature-toggles.html)**: o código de uma nova funcionalidade já está em produção, mas desligado por uma configuração — permitindo separar o **deploy** do código (uma operação técnica) do **lançamento** da funcionalidade (uma decisão de produto), e habilitar a funcionalidade gradualmente ou apenas para usuários específicos.

!!! tip "Deploy frequente reduz risco, não aumenta"
    É comum a intuição de que "fazer deploy com mais frequência é mais arriscado". A evidência das pesquisas da DORA (relatórios *State of DevOps*, até 2024) aponta o contrário: deploys pequenos e frequentes são mais fáceis de testar, mais fáceis de reverter e mais fáceis de depurar (menos mudanças acumuladas por deploy) do que grandes lançamentos espaçados — que concentram risco em vez de distribuí-lo.

## 🛡️ Segurança no pipeline

O pipeline é o lugar natural para as verificações que ninguém lembra de fazer à mão. Cada uma procura um tipo de problema ([Capítulo 5](05_implementacao_qualidade.md)):

| Verificação | Ferramenta no exemplo | O que procura |
|---|---|---|
| Código (SAST) | [Bandit](https://bandit.readthedocs.io/) | Padrões inseguros no seu código Python |
| Dependências (SCA) | [pip-audit](https://github.com/pypa/pip-audit) | Bibliotecas com vulnerabilidades conhecidas |
| Segredos | [Gitleaks](https://github.com/gitleaks/gitleaks) | Chaves e tokens no histórico do repositório |
| Imagem | [Trivy](https://trivy.dev/) | Pacotes vulneráveis dentro da imagem |

Acrescente ao `ci.yml` o job `seguranca`, no mesmo nível de `test`:

```yaml
  seguranca:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout do código
        uses: actions/checkout@3d3c42e5aac5ba805825da76410c181273ba90b1 # v7.0.1
        with:
          fetch-depth: 0 # o Gitleaks examina todo o histórico

      - name: Procurar segredos
        run: >
          docker run --rm -v "$PWD:/repo:ro"
          zricethezav/gitleaks:v8.30.1@sha256:c00b6bd0aeb3071cbcb79009cb16a60dd9e0a7c60e2be9ab65d25e6bc8abbb7f
          git /repo --redact

      - name: Configurar Python
        uses: actions/setup-python@6d4fd22b20465ccc0f7354476588e8df7c9ca0e0 # v7.1.0
        with:
          python-version: "3.14"

      - name: Instalar dependências
        run: pip install -r requirements-dev.txt

      - name: Analisar o código (SAST)
        run: bandit -r app

      - name: Auditar as dependências
        run: pip-audit -r requirements-dev.txt
```

E, ao final do job `imagem`, depois de construí-la:

```yaml
      - name: Procurar vulnerabilidades na imagem
        run: >
          docker run --rm -v /var/run/docker.sock:/var/run/docker.sock
          aquasec/trivy:0.76.2@sha256:8f6bb947dbc528902e11f2f5be52f7b519f6a39d5d148f41166f2040499768fb
          image --scanners vuln --severity HIGH,CRITICAL --ignore-unfixed
          --exit-code 1 api-tarefas
```

`--exit-code 1` faz o job falhar quando há vulnerabilidade alta ou crítica, e `--ignore-unfixed` ignora as que ainda não têm correção. Uma falha aqui pode vir da imagem base, e não do seu código: nesse caso, reconstrua com a base atualizada.

Um teste simples mostra por que o Gitleaks precisa do histórico completo: um token gravado em um *commit* e apagado no seguinte continua sendo apontado.

!!! warning "As ferramentas do pipeline também são dependências"
    Em março de 2026, atacantes publicaram uma versão maliciosa do Trivy e reescreveram quase todas as *tags* da `trivy-action`, passando a roubar segredos de pipelines que as usavam ([comunicado da Aqua Security](https://www.aquasec.com/blog/trivy-supply-chain-attack-what-you-need-to-know/)). Uma *tag* como `v7` ou `0.76.2` pode ser movida; o SHA de um *commit* e o *digest* de uma imagem, não. Por isso os exemplos acima fixam os dois, como [recomenda o GitHub](https://docs.github.com/en/actions/reference/security/secure-use). Fixar não torna a versão confiável: apenas garante que ela não troca sem você ver. A imagem base do `Dockerfile` (`python:3.14-slim`) continua sendo uma *tag*, de propósito, para receber correções a cada reconstrução; fixá-la por *digest* é a alternativa quando reproduzir o build importa mais.

Dê também ao pipeline o **privilégio mínimo**. No topo do `ci.yml`, antes de `jobs`:

```yaml
permissions:
  contents: read
```

## 🤖 Agentes de codificação e o pipeline

Uma tendência recente é a de agentes de codificação que não apenas sugerem trechos de código, mas abrem *pull requests* completos de forma autônoma — recebendo uma tarefa (ex: uma *issue* do GitHub), explorando o repositório, implementando a mudança, e submetendo-a para revisão. Do ponto de vista do pipeline de CI/CD, isso não muda a arquitetura discutida neste capítulo: um *pull request* aberto por um agente passa pelos mesmos portões de qualidade — lint, testes, cobertura mínima — que um *pull request* aberto por uma pessoa.

O que muda é a importância desses portões automatizados como linha de defesa: se o volume de *pull requests* aumenta porque parte deles é gerada por agentes (Capítulo 5), o pipeline de CI se torna a garantia consistente de que nenhuma mudança — humana ou não — chega à branch principal sem passar pelos mesmos critérios mínimos de qualidade.

Um agente com acesso ao repositório traz um risco novo. Ele lê *issues*, comentários, páginas e arquivos, e um texto malicioso nesses lugares pode ser interpretado como instrução (*prompt injection*). Se o agente tiver permissões demais (*excessive agency*), o estrago é maior. Esses dois riscos estão entre os três primeiros do [OWASP Top 10 para aplicações com LLM, edição de 2026](https://genai.owasp.org/resource/owasp-genai-llm-top-10-2026/). Medidas práticas:

* conceder apenas as permissões que a tarefa exige, sem segredos de produção;
* exigir revisão humana antes do merge, com regras de proteção na branch principal;
* tratar o conteúdo de *issues* e *pull requests* de terceiros como entrada não confiável;
* revisar com atenção mudanças do agente em workflows, dependências e permissões.

---

## 🎯 Atividades

1. Adicione o workflow `ci.yml` deste capítulo a um repositório GitHub contendo a API de tarefas do Capítulo 6 e confirme que ele roda com sucesso a cada `push`.
2. Modifique o pipeline para falhar propositalmente (ex: aumente `--cov-fail-under` para 100, acima da cobertura atual do seu código) e observe o comportamento do GitHub Actions.
3. Construa a imagem Docker localmente (`docker build -t api-tarefas .`) e rode um contêiner a partir dela (`docker run -p 8000:8000 api-tarefas`), acessando `http://localhost:8000/docs` para ver a documentação interativa gerada automaticamente pelo FastAPI.
4. Descreva, para um sistema hipotético que você conheça, qual estratégia de deploy (blue-green, canary, feature flags) você usaria para lançar uma mudança arriscada, e por quê.
5. Acrescente o job `seguranca` ao seu repositório. Depois, em uma branch de teste, grave um token **falso** em um arquivo e confira se o pipeline o aponta.

## 📚 Para saber mais

* FOWLER, Martin. *Continuous Integration* — [martinfowler.com/articles/continuousIntegration.html](https://martinfowler.com/articles/continuousIntegration.html)
* HUMBLE, Jez; FARLEY, David. *Continuous Delivery*. Addison-Wesley, 2010.
* GitHub, *Secure use reference* (Actions) — [docs.github.com](https://docs.github.com/en/actions/reference/security/secure-use)
* Aqua Security. *Trivy supply chain attack: what you need to know*, 2026 — [aquasec.com](https://www.aquasec.com/blog/trivy-supply-chain-attack-what-you-need-to-know/)
* *OWASP GenAI Top 10 for LLM Applications*, 2026 — [genai.owasp.org](https://genai.owasp.org/resource/owasp-genai-llm-top-10-2026/)
* Documentação oficial: [GitHub Actions](https://docs.github.com/actions), [Docker](https://docs.docker.com/)
* Relatórios anuais da DORA (Google Cloud): *State of DevOps* até 2024 e *State of AI-assisted Software Development* em 2025 — [dora.dev](https://dora.dev/).
* FOWLER, Martin. *Blue Green Deployment*, 2010 — [martinfowler.com](https://www.martinfowler.com/bliki/BlueGreenDeployment.html)
* SATO, Danilo. *Canary Release*, 2014 — [martinfowler.com](https://www.martinfowler.com/bliki/CanaryRelease.html)
* HODGSON, Pete. *Feature Toggles (aka Feature Flags)*, 2017 — [martinfowler.com](https://martinfowler.com/articles/feature-toggles.html)
