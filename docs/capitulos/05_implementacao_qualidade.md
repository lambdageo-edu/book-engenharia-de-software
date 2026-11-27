# Implementação e Qualidade de Código

## ⌨️ Da arquitetura ao código

Implementação é o processo de traduzir um projeto em um programa executável. Parece a etapa mais "mecânica" do processo de software, mas é aqui que a maior parte das decisões do dia a dia acontece — e onde a qualidade acumulada (ou a falta dela) determina o quão fácil (ou doloroso) será manter o sistema nos próximos anos.

## 🧹 Clean code: por que importa

O termo *clean code* foi popularizado pelo livro homônimo de Robert C. Martin (2008), mas a ideia é anterior e simples: **código é lido muito mais vezes do que é escrito**. Um desenvolvedor gasta boa parte do seu tempo lendo e entendendo código existente — próprio ou de outros — antes de conseguir modificá-lo com segurança. Código difícil de ler tem um custo real e recorrente, mesmo quando "funciona".

Algumas práticas amplamente aceitas de código limpo:

* **Nomes que revelam intenção**: `dias_desde_ultimo_login` comunica mais que `d`. O tempo economizado por nomes ruins na hora de escrever é pago, com juros, por quem lê depois.
* **Funções pequenas e com um único propósito**: uma função que faz "uma coisa" é mais fácil de testar, entender e reutilizar do que uma função de 200 linhas que faz cinco coisas.
* **Evitar duplicação (princípio DRY — *Don't Repeat Yourself*, de Hunt e Thomas)**: lógica duplicada em vários lugares significa que uma correção precisa ser lembrada e replicada em todos eles.
* **Comentários que explicam o "porquê", não o "o quê"**: se o código precisa de um comentário para explicar o que ele faz, muitas vezes é sinal de que ele poderia ser reescrito de forma mais clara. Comentários são mais valiosos quando explicam uma decisão não óbvia ("usamos X em vez de Y por causa da limitação Z da API externa").

### Efeitos colaterais

Uma função surpreende quando faz mais do que o nome promete:

```python
def senha_confere(usuario, senha):
    if usuario.senha_correta(senha):
        usuario.iniciar_sessao()  # quem só queria conferir também fez login
        return True
    return False
```

Quem chama `senha_confere` para confirmar uma operação não espera abrir uma sessão. Separe a pergunta da ação, deixando uma função que apenas responde e outra, chamada de forma explícita, que inicia a sessão. Bertrand Meyer chamou essa regra de **separação entre comandos e consultas**: um método muda o estado ou devolve uma informação, não os dois. É uma orientação, com exceções conhecidas, como retirar um item de uma fila.

### A regra do escoteiro

Robert C. Martin propõe deixar o código um pouco mais limpo do que o encontrou: um nome melhor, uma duplicação removida, um comentário desatualizado apagado. Assim a limpeza não fica para um grande dia que não chega. Mantenha o passo pequeno e coberto por testes, e separe-o em um *commit* próprio: misturar limpeza ampla com mudança de comportamento dificulta a revisão. O [Capítulo 8](08_evolucao_manutencao.md) trata de refatoração e débito técnico.

!!! warning "Clean code não é dogma"
    Regras de estilo (como "funções devem ter no máximo N linhas") são heurísticas úteis, não leis absolutas. O objetivo final é sempre a legibilidade e a manutenibilidade — se seguir uma regra à risca torna o código pior, a regra perde.

## 👀 Revisão de código (code review)

Revisão de código é a prática de um ou mais colegas lerem e comentarem uma mudança de código antes que ela seja integrada ao projeto principal. Hoje é quase universal em times profissionais, tipicamente formalizada através de *pull requests* (GitHub, GitLab) ou *merge requests*.

Benefícios da revisão de código (um estudo na Microsoft mostra que, além de achar defeitos, ela serve à troca de conhecimento: [Bacchelli e Bird, 2013](https://www.microsoft.com/en-us/research/?p=164195)):

* **Detecção de defeitos** antes que cheguem à produção — mais barata do que corrigir depois.
* **Disseminação de conhecimento** sobre o sistema entre os membros do time, reduzindo o risco de conhecimento concentrado em uma única pessoa (*bus factor* baixo).
* **Consistência de estilo e de decisões de projeto** ao longo do código.
* **Aprendizado mútuo**, especialmente entre desenvolvedores mais e menos experientes.

Boas práticas de revisão:

* Mudanças pequenas e focadas são mais fáceis (e mais rápidas) de revisar bem do que um *pull request* gigante que mistura várias funcionalidades.
* Comentários devem ser específicos e, idealmente, sugerir uma alternativa — não apenas apontar um problema.
* Separar comentários bloqueantes ("isso precisa mudar antes de aprovar") de sugestões ("isso é uma ideia, mas não bloqueio por causa disso") evita fricção desnecessária.

## 🔐 Código seguro

Boa parte das vulnerabilidades nasce de decisões comuns de programação. O [OWASP Top 10](https://owasp.org/Top10/2025/0x00_2025-Introduction/), levantamento dos riscos mais importantes em aplicações web, tem estas categorias na edição de 2025:

| Código | Categoria | Em poucas palavras |
|---|---|---|
| A01 | *Broken Access Control* | Alguém acessa ou altera o que não deveria |
| A02 | *Security Misconfiguration* | Configuração padrão, aberta ou esquecida |
| A03 | *Software Supply Chain Failures* | Dependência, ferramenta ou processo de build comprometido |
| A04 | *Cryptographic Failures* | Criptografia ausente, fraca ou mal usada |
| A05 | *Injection* | Dado do usuário interpretado como comando |
| A06 | *Insecure Design* | A falha está no projeto, não só no código ([Capítulo 4](04_projeto_arquitetura.md)) |
| A07 | *Authentication Failures* | Identificação de usuários que pode ser enganada |
| A08 | *Software or Data Integrity Failures* | Código ou dados aceitos sem verificar a origem |
| A09 | *Security Logging and Alerting Failures* | O ataque acontece e ninguém percebe |
| A10 | *Mishandling of Exceptional Conditions* | Erros e situações anormais tratados de forma insegura |

É uma lista para conscientização, não uma lista completa: para requisitos verificáveis, use o ASVS ([Capítulo 3](03_especificacao_requisitos.md)).

### Validar a entrada na fronteira

Tudo o que vem de fora deve ser conferido quando entra no sistema: tipo, tamanho e formato. Com Pydantic, a regra fica no próprio modelo:

=== "Sem limite"
    ```python
    class TarefaEntrada(BaseModel):
        titulo: str
    ```

=== "Com limite"
    ```python
    from typing import Annotated

    from pydantic import BaseModel, StringConstraints

    Titulo = Annotated[
        str,
        StringConstraints(strip_whitespace=True, min_length=1, max_length=200),
    ]


    class TarefaEntrada(BaseModel):
        titulo: Titulo
    ```

O [Capítulo 6](06_testes_validacao.md) aplica essa correção na API de tarefas, escrevendo o teste primeiro.

### Separar comando de dado

A injeção acontece quando um texto vindo do usuário é concatenado a um comando. Em SQL, a defesa é usar parâmetros:

```python
# Vulnerável: o título passa a fazer parte do comando SQL
cursor.execute(f"SELECT * FROM tarefas WHERE titulo = '{titulo}'")

# Seguro: o driver trata o título apenas como dado
cursor.execute("SELECT * FROM tarefas WHERE titulo = ?", (titulo,))
```

O [TaskFlow](https://github.com/lambdageo-edu/taskflow) usa parâmetros em todas as consultas que recebem dados e tem um teste que cadastra uma tarefa com o título `x'); DROP TABLE tarefas; --`: ele é guardado como texto, e nada é apagado.

O marcador (`?` no `sqlite3`) varia conforme a biblioteca. A mesma ideia vale para comandos do sistema operacional e para HTML gerado a partir de dados.

### Segredos fora do código

Senhas, tokens e chaves não pertencem ao repositório, nem à imagem de contêiner. A aplicação os recebe do ambiente, por exemplo por variáveis de ambiente ou por um cofre de segredos:

```python
import os

chave_api = os.environ["CHAVE_API"]  # falha logo se não estiver definida
```

Para desenvolvimento, um arquivo `.env` local funciona, desde que esteja no `.gitignore` e no `.dockerignore`.

!!! warning "Vazou um segredo? Apagar o arquivo não resolve"
    O histórico do Git guarda as versões anteriores, e cópias do repositório podem já ter o segredo. A ordem recomendada pela [OWASP](https://cheatsheetseries.owasp.org/cheatsheets/Secrets_Management_Cheat_Sheet.html) é: **revogar** o segredo exposto, **substituí-lo** por um novo, **removê-lo** de onde apareceu e **registrar** o que aconteceu e quem teve acesso.

### Senhas e sessões

Senha não se guarda em texto puro, nem cifrada de forma reversível: guarda-se um *hash* feito para esse fim, lento e com *sal*, como o Argon2id ([OWASP](https://cheatsheetseries.owasp.org/cheatsheets/Password_Storage_Cheat_Sheet.html)). No [TaskFlow](https://github.com/lambdageo-edu/taskflow), a biblioteca `argon2-cffi` cuida disso (trecho de `app/auth.py`):

```python
from argon2 import PasswordHasher

hasher = PasswordHasher()  # Argon2id, com sal aleatório

hash_guardado = hasher.hash(senha)       # no cadastro
hasher.verify(hash_guardado, tentativa)  # no login; levanta exceção se não conferir
```

Outras decisões do mesmo arquivo merecem ser lidas:

* a senha tem no mínimo 15 caracteres, como pede o [NIST SP 800-63B](https://pages.nist.gov/800-63-4/sp800-63b/passwords) para senhas usadas como único fator;
* o erro de *login* é o mesmo para usuário inexistente e senha errada (o cadastro, porém, ainda revela que um *login* já existe);
* o token de sessão é aleatório, expira, pode ser invalidado e só o seu *hash* vai para o banco.

Isso não dispensa o que está fora do código, como HTTPS e limite de tentativas. Em um sistema real, considere também delegar a autenticação a um provedor de identidade.

## 🧯 Tratamento de erros

Há erros **esperados**, parte do funcionamento normal, como um *login* já usado, e erros **inesperados**, como um defeito ou um disco cheio. Os primeiros merecem uma resposta específica; os segundos, uma resposta genérica ao cliente e um registro detalhado para quem mantém o sistema. Tratar mal as situações anormais é a categoria A10 da lista acima.

O pior tratamento é o que esconde:

```python
def salvar(banco, titulo):
    try:
        banco.execute("INSERT INTO tarefas (titulo) VALUES (?)", (titulo,))
    except Exception:
        pass  # o erro some, e quem chamou acha que gravou
```

Capture apenas o que você sabe tratar. O TaskFlow captura a violação de unicidade no cadastro e a traduz para o vocabulário da API (trecho de `app/auth.py`):

```python
try:
    cursor = banco.execute(
        "INSERT INTO usuarios (login, nome, senha_hash) VALUES (?, ?, ?)",
        (dados.login, dados.nome, hasher.hash(dados.senha)),
    )
except sqlite3.IntegrityError:
    raise HTTPException(409, "Login já utilizado") from None
```

Qualquer outra exceção segue adiante e vira um 500. No FastAPI, fora do modo de depuração, o cliente recebe apenas `Internal Server Error`, e o detalhe vai para o terminal ou o *log* do servidor.

Algumas regras práticas:

* **Na dúvida, negue.** Se a verificação de permissão falhar por um erro, o acesso não deve ser liberado.
* **Não vaze detalhes.** Pilha de execução, consulta SQL e caminhos de arquivos ajudam um atacante; guarde-os no *log*, sem senhas nem *tokens* ([Capítulo 8](08_evolucao_manutencao.md)).
* **Libere os recursos**, com `with` ou `finally`. Atenção: no `sqlite3`, `with conexao` encerra a transação, mas não fecha a conexão.
* **Não deixe gravação pela metade.** Use transações quando a operação tem vários passos ([Capítulo 4](04_projeto_arquitetura.md)).
* **Defina um prazo** (*timeout*) para chamadas a outros serviços, e decida o que fazer quando ele estoura.
* **Teste o caminho do erro.** É o que menos roda no dia a dia e o que menos se conhece.

O primeiro item, em código (exemplo fora do TaskFlow, com um serviço de permissões externo):

```python
import logging

log = logging.getLogger(__name__)


def pode_concluir(servico, usuario_id, tarefa_id):
    try:
        return servico.permite(usuario_id, tarefa_id)
    except TimeoutError:
        log.exception("serviço de permissões não respondeu")
        return False
```

## 🤖 Revisando código gerado por IA

Assistentes e agentes de codificação (GitHub Copilot, Claude Code, Cursor e equivalentes) já fazem parte do dia a dia de boa parte dos times de desenvolvimento, gerando desde trechos pontuais até *pull requests* inteiros. Isso não elimina a necessidade de revisão de código — pelo contrário, muda o que a revisão precisa verificar:

* **O volume de código por *pull request* tende a aumentar** quando parte dele é gerada por IA, o que reforça (não enfraquece) a boa prática de manter mudanças pequenas e focadas.
* **Código gerado pode "parecer" correto sem ser**: sintaticamente limpo, bem nomeado, mas resolvendo um problema ligeiramente diferente do pedido, ou ignorando uma restrição de negócio que não estava explícita no prompt. Revisar significa validar a lógica contra o requisito real, não apenas a legibilidade do código.
* **Rastreabilidade da autoria não substitui responsabilidade**: quem abre o *pull request* — humano ou agente supervisionado por um humano — continua responsável por entender e defender a mudança na revisão, não apenas por tê-la solicitado a uma ferramenta.
* **Análise estática (próxima seção) se torna ainda mais importante** como primeira linha de defesa mecânica, liberando a revisão humana para focar em design, lógica de negócio e nas armadilhas acima — a mesma divisão entre máquina e pessoa explicada a seguir, agora com mais volume de código passando por ela.

!!! tip "IA muda o \"como\", não o \"porquê\" da revisão"
    Os benefícios de revisão de código listados no início desta seção — detecção de defeitos, disseminação de conhecimento, consistência, aprendizado — continuam os mesmos independentemente de quem (ou o quê) escreveu o código. O que muda é o volume e o tipo de atenção que o revisor precisa dedicar a cada mudança.

### O que a pesquisa já mostrou

* Num experimento com 47 participantes, a maioria estudantes, publicado em 2023, quem usou um assistente de IA escreveu código **menos seguro** em quatro de cinco tarefas e, ao mesmo tempo, tendeu a **confiar mais** na segurança do que escreveu ([Perry et al., 2023](https://arxiv.org/abs/2211.03622)). O modelo usado é antigo, e os números não valem automaticamente para as ferramentas atuais, mas o alerta sobre excesso de confiança continua útil.
* Modelos que geram código às vezes recomendam **pacotes que não existem**. Um estudo com 16 modelos mediu, em média, ao menos 5,2% de pacotes inventados nos modelos comerciais e 21,7% nos abertos ([Spracklen et al., 2025](https://www.usenix.org/conference/usenixsecurity25/presentation/spracklen)). Alguém pode publicar um pacote malicioso com um desses nomes. Antes de instalar uma dependência sugerida, confirme que ela existe, é a que você pretendia e tem manutenção.

### O que não entregar à ferramenta

Antes de colar algo num assistente, ou de dar a um agente acesso a uma pasta, verifique as regras da organização e deixe de fora:

* senhas, tokens, chaves e arquivos `.env`;
* dados pessoais reais, inclusive em *logs* e cópias de banco de dados usadas para depuração;
* código ou documentos sigilosos sem autorização para esse uso.

## 🔍 Análise estática de código

Ferramentas de **análise estática** examinam o código sem executá-lo, procurando por padrões problemáticos: bugs comuns, padrões inseguros, violações de estilo, complexidade excessiva. Elas complementam — não substituem — a revisão humana, automatizando a detecção de problemas mecânicos e liberando revisores para focar em design e lógica de negócio. Quando o foco é segurança, esse tipo de análise é chamado de **SAST** (*Static Application Security Testing*).

Categorias comuns de ferramentas, com exemplos do ecossistema Python:

| Categoria | O que verifica | Exemplos (Python) |
|---|---|---|
| **Linters** | Estilo, convenções, erros óbvios | `ruff`, `flake8`, `pylint` |
| **Formatadores** | Formatação automática e consistente | `black`, `ruff format` |
| **Verificadores de tipo** | Consistência de tipos (em linguagens com tipagem gradual) | `mypy`, `pyright` |
| **Análise de segurança do código (SAST)** | Padrões inseguros no seu próprio código | `bandit` |

Duas outras verificações automáticas complementam a análise do código e não devem ser confundidas com ela:

| Verificação | O que procura | Exemplos |
|---|---|---|
| **Análise de dependências (SCA)** | Vulnerabilidades conhecidas nas bibliotecas que o projeto usa | `pip-audit`, GitHub Dependabot |
| **Varredura de segredos** | Senhas, tokens e chaves gravados no repositório | `gitleaks`, GitHub secret scanning |

A maioria dos times integra essas ferramentas diretamente no pipeline de CI (Capítulo 7), rejeitando automaticamente mudanças que violem regras mínimas de qualidade — tirando a discussão de estilo (tabs vs. espaços, aspas simples vs. duplas) da revisão humana e deixando-a para a ferramenta decidir de forma consistente.

## 🌳 Controle de versão colaborativo

Git se tornou, de longe, o sistema de controle de versão dominante na indústria. Além de rastrear o histórico de mudanças, ele viabiliza os fluxos de trabalho colaborativos que sustentam revisão de código e integração contínua:

* **Branches de curta duração**: criar uma branch a partir da branch principal para cada mudança, integrá-la via *pull request* após revisão e testes, e apagá-la em seguida. Branches de longa duração tendem a acumular divergência e a tornar a integração (*merge*) mais dolorosa quanto mais tempo demoram.
* **Trunk-based development**: uma prática, adotada por times de alta performance segundo pesquisas como o *Accelerate* (Forsgren, Humble, Kim), de integrar mudanças pequenas e frequentes diretamente (ou quase diretamente) na branch principal, evitando branches de longa duração e grandes *merges* conflituosos.
* **Mensagens de commit significativas**: descrever o "porquê" de uma mudança, não apenas o "o quê" — o diff já mostra o que mudou; a mensagem de commit é o lugar certo para registrar contexto que o diff sozinho não conta.
* **Convenção de commits semânticos** (ex: `feat:`, `fix:`, `docs:`, `refactor:`), útil para gerar changelogs automaticamente e para comunicar rapidamente a natureza de uma mudança.

!!! tip "Qualidade de código é um investimento, não um luxo"
    Times sob pressão de prazo frequentemente cortam revisão de código, testes ou análise estática como forma de "ir mais rápido". Na prática, as pesquisas da DORA (relatórios *State of DevOps*, até 2024) indicam o oposto: times de alto desempenho entregam **mais rápido e com mais qualidade simultaneamente** — a disciplina de qualidade reduz o retrabalho que, no médio prazo, é o que realmente desacelera um time.

---

## 🎯 Atividades

1. Pegue uma função de um projeto seu com mais de 30 linhas. Reescreva-a aplicando pelo menos dois princípios de clean code discutidos aqui (nomes melhores, quebra em funções menores, remoção de duplicação).
2. Configure um linter (`ruff` ou similar) em um projeto Python pequeno e corrija os três primeiros avisos que ele reportar.
3. Escreva um checklist de code review com pelo menos 6 itens que você usaria como revisor em um projeto real.
4. Acrescente ao checklist três itens de segurança, cada um ligado a uma categoria do OWASP Top 10.
5. Peça a um assistente de IA o código de um *login* simples e revise-o: como a senha é armazenada? Há segredo no código? As dependências sugeridas existem?
6. Escreva um teste para `pode_concluir` com um dublê que levanta `TimeoutError`. Depois procure, em um projeto seu, um `except` que esconda um erro.

## 📚 Para saber mais

* MARTIN, Robert C. *Clean Code: A Handbook of Agile Software Craftsmanship*. Prentice Hall, 2008.
* MEYER, Bertrand. *Object-Oriented Software Construction*. 2. ed. Prentice Hall, 1997.
* FORSGREN, Nicole; HUMBLE, Jez; KIM, Gene. *Accelerate: The Science of Lean Software and DevOps*. IT Revolution, 2018.
* Documentação do `ruff` (linter/formatador Python) — [docs.astral.sh/ruff](https://docs.astral.sh/ruff/)
* OWASP *Error Handling Cheat Sheet* — [cheatsheetseries.owasp.org](https://cheatsheetseries.owasp.org/cheatsheets/Error_Handling_Cheat_Sheet.html)
* *OWASP Top 10:2025* — [owasp.org/Top10/2025](https://owasp.org/Top10/2025/0x00_2025-Introduction/)
* OWASP *Secrets Management Cheat Sheet* — [cheatsheetseries.owasp.org](https://cheatsheetseries.owasp.org/cheatsheets/Secrets_Management_Cheat_Sheet.html)
* PERRY, Neil; SRIVASTAVA, Megha; KUMAR, Deepak; BONEH, Dan. *Do Users Write More Insecure Code with AI Assistants?* ACM CCS, 2023 — [arxiv.org/abs/2211.03622](https://arxiv.org/abs/2211.03622)
* SPRACKLEN, Joseph et al. *We Have a Package for You! A Comprehensive Analysis of Package Hallucinations by Code Generating LLMs*. USENIX Security, 2025 — [usenix.org](https://www.usenix.org/conference/usenixsecurity25/presentation/spracklen)
* Guia de código semântico de commits — [conventionalcommits.org](https://www.conventionalcommits.org/pt-br/)
* HUNT, Andrew; THOMAS, David. *The Pragmatic Programmer*. Addison-Wesley, 1999.
* BACCHELLI, Alberto; BIRD, Christian. *Expectations, Outcomes, and Challenges of Modern Code Review*. ICSE, 2013 — [microsoft.com/research](https://www.microsoft.com/en-us/research/?p=164195)
