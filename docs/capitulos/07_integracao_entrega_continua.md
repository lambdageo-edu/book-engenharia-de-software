# Integração e Entrega Contínua (CI/CD)

## 🔗 Por que integração contínua

**Integração contínua (CI — *Continuous Integration*)** é a prática de integrar mudanças de código na branch principal com frequência (idealmente, várias vezes ao dia), com cada integração validada automaticamente por um pipeline que compila, analisa e testa o código.

A motivação original, descrita por Martin Fowler há mais de duas décadas, é simples: quanto mais tempo uma mudança fica isolada em uma branch separada, mais ela diverge do resto do código, e mais doloroso (e arriscado) se torna integrá-la depois — o clássico problema de "*merge hell*". Integrar cedo e com frequência transforma um grande risco ocasional em um risco pequeno e constante, muito mais fácil de gerenciar.

**Entrega contínua (CD — *Continuous Delivery*)** estende essa ideia: se toda mudança validada no pipeline está, por definição, em um estado potencialmente pronto para produção, o processo de colocá-la em produção também pode — e deve — ser automatizado. A diferença entre *Continuous Delivery* e *Continuous Deployment* é sutil, mas importante:

* **Continuous Delivery**: toda mudança aprovada pelo pipeline está pronta para ir a produção a qualquer momento, mas o deploy em si ainda pode exigir uma aprovação manual (ex: um ambiente protegido do GitHub com revisores obrigatórios), útil quando há razões de negócio, como janelas de manutenção ou *compliance*, para controlar o momento.
* **Continuous Deployment**: toda mudança aprovada pelo pipeline vai para produção **automaticamente**, sem intervenção humana — o nível mais alto de automação, e o que exige a suíte de testes mais confiável, além de formas de reduzir o risco, como *feature flags* e liberação gradual (veja adiante).

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

# Correções de segurança do sistema que a imagem base ainda não traz.
RUN apt-get update && apt-get upgrade -y && rm -rf /var/lib/apt/lists/*

COPY requirements.txt .
# O pip só serve para instalar: removê-lo tira da imagem as bibliotecas que ele embute.
RUN pip install --no-cache-dir -r requirements.txt && pip uninstall -y pip

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
* **Atualizar os pacotes do sistema**: a imagem base pode demorar a incorporar uma correção já publicada. A troca é que dois builds em datas diferentes podem produzir imagens diferentes, e o cache do Docker pode reaproveitar uma camada antiga (`docker build --pull --no-cache` refaz tudo).
* **Remover o `pip`**: o que não é necessário para rodar também não precisa ser corrigido depois.
* **`.dockerignore`**: evita enviar ao *build* o ambiente virtual, o histórico do Git e arquivos locais como `.env`, que podem conter segredos.

O job `imagem` apenas constrói a imagem, para garantir que o `Dockerfile` continua funcionando. O passo seguinte é publicá-la em um registro a cada merge na branch principal, o que se faz [adiante](#publicar-a-imagem), depois das verificações de segurança.

!!! example "Quando há banco de dados"
    O sistema de arquivos de um contêiner some junto com ele. O [`Dockerfile` do TaskFlow](https://github.com/lambdageo-edu/taskflow/blob/main/Dockerfile) cria a pasta `/dados`, entrega-a ao usuário sem privilégios, declara um `VOLUME` e aponta o banco para lá com a variável `TAREFAS_DB`:

    ```bash
    docker build -t taskflow .
    docker run -p 127.0.0.1:8000:8000 -v taskflow-dados:/dados taskflow
    ```

    Com o volume nomeado, o contêiner pode ser removido e recriado sem perder os dados. Volume não é cópia de segurança. E publicar a porta em `127.0.0.1` evita expor na rede uma API sem HTTPS.

Para ir além deste `Dockerfile`, veja o tutorial [Otimização de imagens: desenvolvimento e produção](https://lambdageo-edu.github.io/lambdageo-tutorials/docker/otimizacao-imagens-dev-vs-prod/). Os comandos básicos de Git e Docker estão no [apêndice](../apendices/git_conteineres.md).

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

## 🎯 Testar a aplicação em execução: DAST e teste de invasão

As verificações acima examinam arquivos: código, dependências, histórico e imagem. O **DAST** (*Dynamic Application Security Testing*) faz o contrário: envia requisições à aplicação **em execução**, como alguém de fora, e observa as respostas. Pode encontrar problemas que só aparecem com tudo montado, como cabeçalhos de segurança ausentes ou erros que expõem detalhes internos.

Uma ferramenta livre é o [ZAP](https://www.zaproxy.org/docs/docker/api-scan). Para uma API que publica a descrição OpenAPI, como as feitas com FastAPI, com a aplicação rodando na porta 8000 (comando para Linux; a imagem está fixada por *digest* na versão 2.17.0):

```bash
docker run --rm --network host \
  zaproxy/zap-stable@sha256:9624669db72f54d86c4b46a7064518bd9bf79d8c61a4c31befa9923489f328af \
  zap-api-scan.py -t http://localhost:8000/openapi.json -f openapi
```

!!! danger "Só em ambiente seu"
    Essa varredura é **ativa**: envia ataques de verdade e pode criar ou alterar dados. Rode-a em um ambiente de teste, com banco descartável. Varrer um sistema alheio sem autorização por escrito pode ser crime.

Ler o resultado exige cuidado. Em um teste com o TaskFlow, em novembro de 2026, o ZAP não apontou falha, apenas um aviso sobre o cabeçalho `X-Content-Type-Options`. Mas, das 1.316 requisições, só 3 receberam 200: cerca de metade foi recusada com 401 e quase todo o resto, por dados inválidos (422). Sem um *token*, a varredura não passou da porta nas rotas protegidas. "Nenhum alerta" não quer dizer "sem vulnerabilidades". Para ir além, é preciso configurar a autenticação na ferramenta, e a autorização entre usuários continua dependendo de testes escritos por quem conhece as regras ([Capítulo 6](06_testes_validacao.md)).

| | Examina | Quando é mais útil | Limite |
|---|---|---|---|
| **SAST** | O código | A cada *commit* | Falsos alarmes; não vê o sistema montado |
| **DAST** | A aplicação rodando | Em um ambiente de teste | Só alcança o que consegue acessar |
| **Teste de invasão** | O sistema, com uma pessoa especializada | Antes de lançamentos e mudanças importantes | Caro, e retrata um momento |

O **teste de invasão** (*pentest*) é feito por pessoas autorizadas que combinam falhas e exploram a lógica do negócio, o que ferramentas automáticas fazem mal. O escopo, o período e o que é proibido são acertados antes, por escrito.

## 📦 Publicar, versionar e implantar

Até aqui, o pipeline confere a imagem e a descarta. Falta colocá-la em uso.

### Publicar a imagem

Um **registro** guarda imagens para que outras máquinas as baixem. Para usar o do GitHub (GHCR), ajuste o início do job `imagem`, para que ele dependa também das verificações de segurança e possa gravar pacotes:

```yaml
  imagem:
    needs: [test, seguranca]
    runs-on: ubuntu-latest
    permissions:
      contents: read
      packages: write
```

E acrescente, depois do passo do Trivy:

```yaml
      - name: Publicar a imagem
        if: github.event_name == 'push'
        env:
          TOKEN: ${{ secrets.GITHUB_TOKEN }}
          IMAGEM: ghcr.io/${{ github.repository }}:${{ github.sha }}
        run: |
          echo "$TOKEN" | docker login ghcr.io -u "${{ github.actor }}" --password-stdin
          docker tag api-tarefas "$IMAGEM"
          docker push "$IMAGEM"
```

Assim, só é publicada a imagem que passou pelos testes e pela varredura, e apenas em `push` para a `main`. A etiqueta com o identificador do *commit* diz de que código ela saiu. O nome da imagem precisa estar em minúsculas.

### Versões e releases

Quem usa o sistema precisa saber o que mudou. O [versionamento semântico](https://semver.org/lang/pt-BR/) usa três números, `MAIOR.MENOR.CORREÇÃO`:

| Mudança | Número que sobe | Exemplo em uma API |
|---|---|---|
| Incompatível com o uso anterior | MAIOR | Remover um campo da resposta |
| Funcionalidade nova, compatível | MENOR | Acrescentar um *endpoint* |
| Correção compatível | CORREÇÃO | Consertar um código de resposta errado |

Enquanto a versão começa com `0.`, tudo ainda pode mudar. Uma versão publicada não se altera: corrige-se com outra. No Git, uma etiqueta anotada marca o ponto:

```bash
git tag -a v1.2.0 -m "Filtro por responsável"
git push origin v1.2.0
```

Registre as novidades em um [`CHANGELOG.md`](https://keepachangelog.com/) ou nas *releases* do GitHub, escrito para quem usa, não como cópia da lista de *commits*. (As etiquetas do TaskFlow marcam etapas didáticas, e não versões semânticas.)

### Do contêiner ao servidor

É comum ter ambientes separados: o de desenvolvimento, o de **homologação**, parecido com o real, para conferir antes, e o de **produção**. A imagem é a mesma em todos; o que muda é a configuração, recebida do ambiente.

Para publicar uma API na rede falta o **HTTPS**. Um caminho simples é colocar à frente dela um ***proxy* reverso**, que recebe as conexões, cifra o tráfego e repassa os pedidos. O [Caddy](https://caddyserver.com/docs/quick-starts/reverse-proxy) obtém e renova os certificados sozinho. Com o Docker Compose, um arquivo `compose.yaml` na raiz do TaskFlow descreve os dois serviços:

```yaml
services:
  api:
    build: .
    image: ${IMAGEM:-taskflow:local}
    restart: unless-stopped
    volumes:
      - dados:/dados

  proxy:
    image: caddy:2
    restart: unless-stopped
    command: caddy reverse-proxy --from ${DOMINIO:-localhost} --to api:8000
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - certificados:/data
    depends_on:
      - api

volumes:
  dados:
  certificados:
```

```bash
docker compose up -d --build   # na sua máquina: constrói a imagem a partir do código
curl -k https://localhost/docs
```

Repare que a API não publica porta: só o *proxy* é alcançável de fora. Em `localhost`, o certificado é emitido por uma autoridade local, e por isso o `curl` precisa do `-k`. Com um domínio real, o DNS deve apontar para o servidor e as portas 80 e 443 devem estar abertas à internet. Logo depois de subir, o *proxy* pode responder 502 por alguns segundos, enquanto a API inicia. A imagem `caddy:2` é uma *tag*, pelo mesmo motivo da imagem base do `Dockerfile`.

No servidor não se reconstrói: usa-se a imagem que o pipeline testou, varreu e publicou. Por isso o serviço `api` tem também `image`, cujo nome vem da variável `IMAGEM`. Um arquivo `.env` ao lado do `compose.yaml`, fora do Git, a define no servidor:

```text
IMAGEM=ghcr.io/SEU_USUARIO/taskflow:IDENTIFICADOR_DO_COMMIT
DOMINIO=tarefas.exemplo.org
```

```bash
docker compose pull
docker compose up -d --no-build
```

Um pacote novo no GHCR é [privado](https://docs.github.com/en/packages/working-with-a-github-packages-registry/working-with-the-container-registry). Para baixá-lo, o servidor precisa de `docker login ghcr.io` com um *personal access token (classic)* com a permissão `read:packages`, a menos que você torne o pacote público.

Voltar atrás (*rollback*) é pôr em `IMAGEM` o identificador anterior e repetir os dois comandos, o que só funciona se o banco continuar compatível com ela ([Capítulo 8](08_evolucao_manutencao.md)). E o HTTPS não resolve tudo: continuam faltando [cópias de segurança](08_evolucao_manutencao.md), limite de requisições e monitoramento.

### A proteção está ligada?

Uma proteção pode estar no repositório e não estar no sistema em uso. Dois casos comuns:

* uma classe com o nome certo, como um limitador de requisições, que é só um `TODO` ou nunca foi registrada na aplicação;
* um limite configurado no *proxy* de um arquivo do Compose, enquanto o arquivo usado por padrão publica a porta da API direto, sem *proxy*.

Nos dois casos, quem lê o repositório por alto acredita que há proteção. Confira o comportamento na configuração que de fato é implantada: envie pedidos além do limite e espere a recusa, tente a porta da API de fora do servidor. O que puder, transforme em teste automatizado. É o que a OWASP classifica como [configuração incorreta de segurança](https://owasp.org/Top10/2025/0x00_2025-Introduction/) quando o padrão fica aberto.

### Infraestrutura como código

O `compose.yaml` é um exemplo de **infraestrutura como código**: o ambiente é descrito em um arquivo versionado e revisado como o restante, em vez de montado à mão. Ferramentas como [OpenTofu](https://opentofu.org/) ou Terraform descrevem também servidores, redes e bancos; Ansible configura as máquinas. O ganho é poder recriar o ambiente e saber quem mudou o quê. O cuidado: segredos não vão para esses arquivos.

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
2. Modifique o pipeline para falhar propositalmente (ex: aumente `--cov-fail-under` para um valor acima da cobertura atual; se ela já é de 100%, acrescente antes uma função sem teste) e observe o comportamento do GitHub Actions.
3. Construa a imagem Docker localmente (`docker build -t api-tarefas .`) e rode um contêiner a partir dela (`docker run -p 127.0.0.1:8000:8000 api-tarefas`), acessando `http://localhost:8000/docs` para ver a documentação interativa gerada automaticamente pelo FastAPI.
4. Descreva, para um sistema hipotético que você conheça, qual estratégia de deploy (blue-green, canary, feature flags) você usaria para lançar uma mudança arriscada, e por quê.
5. Acrescente o job `seguranca` ao seu repositório. Depois, em uma branch de teste, grave um token **falso** em um arquivo e confira se o pipeline o aponta.
6. Rode o ZAP contra a sua API, em um ambiente de teste. Quantas rotas ele realmente exercitou? O que mudaria se ele tivesse um *token*?
7. Suba o TaskFlow com o `compose.yaml`. Confira que o HTTP redireciona para HTTPS e que a porta 8000 não responde fora do contêiner. Depois execute `docker compose down` e suba de novo: os dados continuam? (Cuidado: com `-v`, os volumes são apagados.)
8. Para três mudanças no TaskFlow (uma correção, uma funcionalidade e uma quebra de compatibilidade), diga qual seria a próxima versão a partir de `1.4.2` e escreva a entrada do *changelog*.
9. Escolha uma proteção de um projeto seu (limite, permissão, cabeçalho, HTTPS) e demonstre, na configuração implantada, que ela está ativa. Se houver mais de um arquivo de implantação, repita em cada um.

## 📚 Para saber mais

* FOWLER, Martin. *Continuous Integration* — [martinfowler.com/articles/continuousIntegration.html](https://martinfowler.com/articles/continuousIntegration.html)
* HUMBLE, Jez; FARLEY, David. *Continuous Delivery*. Addison-Wesley, 2010.
* GitHub, *Secure use reference* (Actions) — [docs.github.com](https://docs.github.com/en/actions/reference/security/secure-use)
* ZAP, *API Scan* — [zaproxy.org](https://www.zaproxy.org/docs/docker/api-scan)
* Aqua Security. *Trivy supply chain attack: what you need to know*, 2026 — [aquasec.com](https://www.aquasec.com/blog/trivy-supply-chain-attack-what-you-need-to-know/)
* *OWASP GenAI Top 10 for LLM Applications*, 2026 — [genai.owasp.org](https://genai.owasp.org/resource/owasp-genai-llm-top-10-2026/)
* *Versionamento Semântico 2.0.0* — [semver.org](https://semver.org/lang/pt-BR/)
* *Keep a Changelog* — [keepachangelog.com](https://keepachangelog.com/)
* GitHub, *Working with the Container registry* — [docs.github.com](https://docs.github.com/en/packages/working-with-a-github-packages-registry/working-with-the-container-registry)
* Caddy, *Reverse proxy quick-start* — [caddyserver.com](https://caddyserver.com/docs/quick-starts/reverse-proxy)
* GitHub, *Publishing and installing a package with GitHub Actions* — [docs.github.com](https://docs.github.com/en/packages/managing-github-packages-using-github-actions-workflows/publishing-and-installing-a-package-with-github-actions)
* Documentação oficial: [GitHub Actions](https://docs.github.com/actions), [Docker](https://docs.docker.com/)
* Relatórios anuais da DORA (Google Cloud): *State of DevOps* até 2024 e *State of AI-assisted Software Development* em 2025 — [dora.dev](https://dora.dev/).
* FOWLER, Martin. *Blue Green Deployment*, 2010 — [martinfowler.com](https://www.martinfowler.com/bliki/BlueGreenDeployment.html)
* SATO, Danilo. *Canary Release*, 2014 — [martinfowler.com](https://www.martinfowler.com/bliki/CanaryRelease.html)
* HODGSON, Pete. *Feature Toggles (aka Feature Flags)*, 2017 — [martinfowler.com](https://martinfowler.com/articles/feature-toggles.html)
