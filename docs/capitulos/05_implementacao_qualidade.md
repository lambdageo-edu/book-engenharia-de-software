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
* **Disseminação de conhecimento** sobre o sistema entre os membros do time, reduzindo o risco de conhecimento concentrado em uma única pessoa (*bus factor* baixo). Também apoia a **propriedade coletiva** do código, prática do XP ([Capítulo 2](02_processos_desenvolvimento.md)).
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

O marcador (`?` no `sqlite3`) varia conforme a biblioteca. A mesma ideia vale para comandos do sistema operacional e para HTML gerado a partir de dados, o próximo assunto.

### XSS: dado que vira código no navegador

No **XSS** (*cross-site scripting*), um texto enviado por um usuário é interpretado como código no navegador de outro. Imagine uma tela que lista as tarefas de uma API:

```javascript
item.innerHTML = tarefa.titulo;    // vulnerável: o título é lido como HTML
item.textContent = tarefa.titulo;  // seguro: o título aparece como texto
```

Com o título `<img src=x onerror="alert('XSS')">`, a primeira linha executa o código do atributo `onerror` no navegador de quem abrir a lista. No lugar do `alert`, um atacante faria pedidos em nome dessa pessoa.

As defesas, em ordem de importância, seguem a [folha da OWASP sobre XSS](https://cheatsheetseries.owasp.org/cheatsheets/Cross_Site_Scripting_Prevention_Cheat_Sheet.html):

* **tratar o dado na saída, conforme o lugar onde entra** (corpo do HTML, atributo, URL, JavaScript). *Frameworks* de interface e motores de *templates* costumam fazer isso, mas oferecem saídas que desligam a proteção, como o `dangerouslySetInnerHTML` do React ou o filtro `safe` do Jinja. Cada uso delas merece revisão;
* **higienizar com uma biblioteca mantida**, como o DOMPurify, quando o usuário precisa mesmo escrever HTML;
* **usar uma [*Content Security Policy*](https://developer.mozilla.org/en-US/docs/Web/HTTP/Guides/CSP) como segunda barreira**: um cabeçalho que restringe de onde o navegador aceita *scripts*. Ela reduz o dano, mas não substitui as anteriores.

Validar a entrada ajuda, mas não resolve: um título pode legitimamente conter `<` ou aspas, e o mesmo dado pode ser exibido em lugares com regras diferentes.

O TaskFlow não tem interface: aceita esse título e o devolve em JSON, o que sozinho não executa nada. A responsabilidade passa para cada tela que consumir a API.

#### CSRF e *cookies*

No **CSRF** (*cross-site request forgery*), uma página maliciosa faz o navegador da vítima enviar um pedido a um sistema em que ela está conectada. Isso funciona quando a credencial é enviada automaticamente, como um *cookie* de sessão. A [folha da OWASP sobre CSRF](https://cheatsheetseries.owasp.org/cheatsheets/Cross-Site_Request_Forgery_Prevention_Cheat_Sheet.html) recomenda um *token* anti-CSRF, com o atributo `SameSite` do *cookie* como camada adicional, e lembra que um XSS derruba essas defesas. No TaskFlow, o *token* vai no cabeçalho `Authorization`, que o navegador não acrescenta sozinho a um pedido vindo de outra página. Para um *cookie* de sessão, use também `HttpOnly`, que impede a leitura por JavaScript, e `Secure`, que o restringe ao HTTPS.

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

O tamanho mínimo não basta: quinze letras iguais passam pela regra. O NIST SP 800-63B também exige comparar a senha nova com uma lista de senhas comuns, esperadas ou já vazadas, e recusá-la se estiver nela. Por outro lado, ele não recomenda impor regras de composição, como obrigar maiúscula, número e símbolo. O TaskFlow confere apenas o tamanho.

#### Sessão no servidor ou token assinado

Depois do *login*, cada pedido precisa dizer quem é o usuário. Há dois caminhos comuns:

| Caminho | Como o servidor reconhece o usuário | Ganho | Custo |
|---|---|---|---|
| **Sessão no servidor** | O token é um número aleatório, procurado no banco a cada pedido | Apagar a sessão vale na hora | Uma consulta por pedido |
| **Token assinado** (como o [JWT](https://www.rfc-editor.org/rfc/rfc7519)) | O token traz os dados, e o servidor confere a assinatura e a validade | Dispensa a consulta | O que está no token vale até ele expirar |

O custo do segundo caminho aparece quando algo muda. Se o papel do usuário é lido apenas do token, rebaixar ou desativar uma conta não tem efeito enquanto o token não expirar: com validade de 30 minutos, a pessoa desligada continua entrando por até 30 minutos. Respostas possíveis são validade curta, uma [lista de tokens revogados](https://cheatsheetseries.owasp.org/cheatsheets/JSON_Web_Token_Cheat_Sheet.html) ou a consulta ao banco em cada pedido, o que devolve o custo que se queria evitar. Quanto atraso é aceitável é um requisito, e deve ser escrito e testado: "uma conta desativada recebe 401 no pedido seguinte".

O TaskFlow guarda as sessões no servidor. Ele ainda não tem como desativar uma conta; se ganhar, as sessões dela precisam ser apagadas ou o estado conferido a cada pedido.

#### Limite de tentativas

Sem limite, um programa pode testar senhas sem parar. O NIST pede no máximo 100 falhas consecutivas por conta, e permite um limite menor. Duas decisões vêm junto:

* **o que contar:** por conta, protege contra quem tenta muitas senhas em um usuário; por endereço de origem, contra quem tenta uma senha em muitos usuários. Um não substitui o outro;
* **o que fazer ao atingir o limite:** bloquear a conta de vez entrega a um atacante o poder de [trancar a conta de outra pessoa](https://cheatsheetseries.owasp.org/cheatsheets/Authentication_Cheat_Sheet.html). Esperas crescentes ou bloqueio temporário reduzem esse efeito.

O limite pode ficar na aplicação, que conhece a conta, ou no *proxy* à frente dela, que barra o pedido antes de gastar processamento. O TaskFlow ainda não tem nenhum dos dois.

Nada disso dispensa HTTPS. Em um sistema real, considere também delegar a autenticação a um provedor de identidade.

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

## 🔀 Concorrência

Um servidor atende vários pedidos ao mesmo tempo. No FastAPI, por exemplo, uma rota declarada com `def` roda em um [conjunto de *threads*](https://fastapi.tiangolo.com/async/), e dois pedidos podem executar a mesma função lado a lado. Há uma **condição de corrida** quando a **correção** do resultado depende de como se intercalam os passos de pedidos simultâneos sobre um dado compartilhado: em algumas ordens dá certo, em outras não.

Suponha que um membro só possa assumir uma tarefa que ainda não tem responsável. O código óbvio consulta e depois grava. No programa abaixo, `corrida.py`, os passos de Ana e de Bruno são intercalados à mão, para o problema aparecer sempre:

```python
import sqlite3

banco = sqlite3.connect(":memory:", isolation_level=None)
banco.execute("CREATE TABLE tarefas (id INTEGER PRIMARY KEY, responsavel TEXT)")
banco.execute("INSERT INTO tarefas VALUES (1, NULL)")


def esta_livre():
    linha = banco.execute("SELECT responsavel FROM tarefas WHERE id = 1").fetchone()
    return linha[0] is None


def gravar(quem):
    banco.execute("UPDATE tarefas SET responsavel = ? WHERE id = 1", (quem,))


ana_viu_livre = esta_livre()
bruno_viu_livre = esta_livre()  # Ana ainda não gravou

if ana_viu_livre:
    gravar("ana")
if bruno_viu_livre:
    gravar("bruno")  # apaga a escolha de Ana

print(banco.execute("SELECT responsavel FROM tarefas").fetchone()[0])  # bruno
```

As duas consultas acharam a tarefa livre, as duas gravações deram certo e Ana acredita que a tarefa é dela. Entre **verificar** e **usar** o resultado, o dado mudou. Essa falha é conhecida como [TOCTOU](https://cwe.mitre.org/data/definitions/367.html) (*time-of-check to time-of-use*) e, quando está numa regra de permissão, é um problema de segurança.

A correção é fazer a verificação e a gravação em **um só comando**, que o banco executa sem intercalar outra escrita, e depois perguntar quantas linhas mudaram:

```python
def assumir(quem):
    cursor = banco.execute(
        "UPDATE tarefas SET responsavel = ? WHERE id = 1 AND responsavel IS NULL",
        (quem,),
    )
    return cursor.rowcount == 1


banco.execute("UPDATE tarefas SET responsavel = NULL")  # libera de novo
print(assumir("ana"))    # True
print(assumir("bruno"))  # False: já tem responsável
```

O [TaskFlow](https://github.com/lambdageo-edu/taskflow) faz isso em `app/tarefas.py`: quando quem pede não é o dono do projeto, a condição entra no próprio `UPDATE` e, se nenhuma linha muda, a resposta é um erro. A mudança de estado da tarefa usa a mesma técnica.

Algumas regras práticas:

* **Desconfie de "consulta, decide, grava"** sobre um dado que outro pedido pode mudar.
* **Deixe o banco garantir a regra** quando ele puder: um `UPDATE` condicional ou uma restrição como `UNIQUE`.
* **Se a operação precisa de vários comandos**, use uma transação com o isolamento adequado. No SQLite, [`BEGIN IMMEDIATE`](https://www.sqlite.org/lang_transaction.html) reserva a escrita antes da primeira consulta, e quem chega depois pode receber um erro de banco ocupado, que o código precisa tratar.
* **Um *lock* na memória** (`threading.Lock`) só protege um processo. Com duas instâncias da aplicação, cada uma tem o seu.
* **Não conte com os testes comuns** para achar o problema: um pedido depois do outro não reproduz a corrida, e disparar vários ao mesmo tempo pode passar muitas vezes antes de falhar.

## 🐞 Depuração

Um teste ou um usuário mostram que algo falhou. **Depurar** é encontrar o defeito que causou a falha ([Capítulo 6](06_testes_validacao.md)) e corrigi-lo. Mudar linhas ao acaso até o sintoma sumir costuma esconder o problema. Um roteiro:

1. **Reproduzir.** Ache os passos e os dados que provocam a falha, de preferência no menor exemplo possível.
2. **Ler a mensagem inteira.**
3. **Formular uma hipótese** e testá-la, mudando uma coisa de cada vez.
4. **Corrigir a causa**, e não só o sintoma.
5. **Deixar um teste** que falhava antes da correção e passa depois.

### Ler o *traceback*

Este programa, em `relatorio.py`, tem um defeito:

```python
def media_por_membro(tarefas, membros):
    return len(tarefas) / len(membros)


def resumo(projeto):
    media = media_por_membro(projeto["tarefas"], projeto["membros"])
    return f"{media:.1f} tarefas por membro"


if __name__ == "__main__":
    print(resumo({"tarefas": ["Publicar"], "membros": []}))
```

Com Python 3.14, a saída é esta (o caminho do arquivo varia):

```text
Traceback (most recent call last):
  File "/app/relatorio.py", line 11, in <module>
    print(resumo({"tarefas": ["Publicar"], "membros": []}))
          ~~~~~~^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
  File "/app/relatorio.py", line 6, in resumo
    media = media_por_membro(projeto["tarefas"], projeto["membros"])
  File "/app/relatorio.py", line 2, in media_por_membro
    return len(tarefas) / len(membros)
           ~~~~~~~~~~~~~^~~~~~~~~~~~~~
ZeroDivisionError: division by zero
```

Comece pela última linha, que diz **o que** aconteceu. Acima dela está o **caminho das chamadas**, com a mais recente embaixo. A linha 2 é onde o erro apareceu, mas a decisão que falta é outra: o que o relatório deve dizer de um projeto sem membros? Respondida a pergunta, escreve-se o teste, em `test_resumo.py`:

```python
from relatorio import resumo


def test_projeto_sem_membros():
    projeto = {"tarefas": ["Publicar"], "membros": []}
    assert resumo(projeto) == "projeto sem membros"
```

Ele falha com o código atual. A correção entra em `resumo`, e o teste passa a protegê-la.

### Parar o programa e olhar

Quando a leitura não basta, o **depurador** interrompe a execução para examinar as variáveis. Em Python, escreva `breakpoint()` na linha onde quer parar e use o [pdb](https://docs.python.org/3.14/library/pdb.html):

| Comando | O que faz |
|---|---|
| `p expressão` | Mostra um valor |
| `n` | Executa até a próxima linha da função atual |
| `s` | Avança entrando na função chamada |
| `w` | Mostra onde se está na pilha de chamadas |
| `c` | Continua a execução |
| `q` | Encerra |

Com `pytest --pdb`, o depurador abre no ponto em que um teste falha. Editores costumam oferecer os mesmos recursos com uma interface gráfica. Remova os `breakpoint()` antes do *commit*. Em produção não se depura parando o programa: usam-se os *logs* ([Capítulo 8](08_evolucao_manutencao.md)).

### Quando começou?

Se funcionava e deixou de funcionar, o [`git bisect`](https://git-scm.com/docs/git-bisect/2.52.0) procura, por busca binária, o *commit* que introduziu o problema:

```bash
git bisect start HEAD v1.2.0 --   # HEAD tem o defeito; v1.2.0 não tinha
git bisect run pytest -q test_resumo.py
git bisect reset                  # volta para onde você estava
```

O teste precisa continuar disponível nos *commits* antigos. Um arquivo novo, ainda não adicionado ao Git, permanece no lugar durante a busca. *Commits* pequenos fazem o resultado apontar para poucas linhas.

Um assistente de IA pode ajudar a interpretar um *traceback*. Antes de colá-lo, retire segredos e dados pessoais, e confirme a explicação reproduzindo o erro.

## 🤖 Revisando código gerado por IA

Assistentes e agentes de codificação (GitHub Copilot, Claude Code, Cursor e equivalentes) já fazem parte do dia a dia de boa parte dos times de desenvolvimento, gerando desde trechos pontuais até *pull requests* inteiros. Isso não elimina a necessidade de revisão de código — pelo contrário, muda o que a revisão precisa verificar:

* **O volume de código por *pull request* tende a aumentar** quando parte dele é gerada por IA, o que reforça (não enfraquece) a boa prática de manter mudanças pequenas e focadas.
* **Código gerado pode "parecer" correto sem ser**: sintaticamente limpo, bem nomeado, mas resolvendo um problema ligeiramente diferente do pedido, ou ignorando uma restrição de negócio que não estava explícita no prompt. Revisar significa validar a lógica contra o requisito real, não apenas a legibilidade do código.
* **Rastreabilidade da autoria não substitui responsabilidade**: quem abre o *pull request* — humano ou agente supervisionado por um humano — continua responsável por entender e defender a mudança na revisão, não apenas por tê-la solicitado a uma ferramenta.
* **Análise estática (próxima seção) se torna ainda mais importante** como primeira linha de defesa mecânica, liberando a revisão humana para focar em design, lógica de negócio e nas armadilhas acima — a mesma divisão entre máquina e pessoa explicada a seguir, agora com mais volume de código passando por ela.

!!! tip "IA muda o \"como\", não o \"porquê\" da revisão"
    Os benefícios listados na seção [Revisão de código](#revisao-de-codigo-code-review) — detecção de defeitos, disseminação de conhecimento, consistência, aprendizado — continuam os mesmos independentemente de quem (ou o quê) escreveu o código. O que muda é o volume e o tipo de atenção que o revisor precisa dedicar a cada mudança.

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

## 📏 Medir a qualidade do código

Ler o código continua sendo a principal forma de julgá-lo, mas algumas propriedades podem ser medidas. Os números ajudam a decidir **onde olhar primeiro**, sobretudo quando o volume de código cresce.

| Métrica | O que responde | Cuidado |
|---|---|---|
| **Complexidade ciclomática** ([McCabe, 1976](https://doi.org/10.1109/TSE.1976.233837)) | Quantos caminhos independentes passam por uma função | Não mede nomes nem clareza |
| **Tamanho** de funções e módulos | Quanto é preciso ler de uma vez | Dividir sem critério só espalha a complexidade |
| **Duplicação** | Quanto código se repete | Trechos parecidos podem ter razões diferentes para mudar |
| **Dependências** entre módulos | Quem depende de quem, e se há ciclos ([acoplamento](04_projeto_arquitetura.md)) | O que importa é a direção, não só a quantidade |
| **Cobertura** ([Capítulo 6](06_testes_validacao.md)) | O que os testes executam | Executar não é verificar |
| **Teste de mutação** | Se os testes percebem um defeito introduzido de propósito | É lento, e alguns mutantes se comportam como o original |
| **CRAP** (*Change Risk Anti-Patterns*) | Onde há muita complexidade com pouco teste | O limite adotado é uma convenção |

O CRAP, proposto por Alberto Savoia e Bob Evans, combina a complexidade `c` de uma função com a fração dela coberta por testes, `cov`: `c² × (1 − cov)³ + c`. Com cobertura total, sobra apenas a complexidade.

Em Python, a complexidade pode ser limitada pelo próprio `ruff` (regra [`C901`](https://docs.astral.sh/ruff/rules/complex-structure/)) e listada pelo [`radon`](https://radon.readthedocs.io/).

### Cobertura total e testes fracos

Em `prazo.py`:

```python
def situacao(dias_restantes):
    if dias_restantes < 0:
        return "atrasada"
    if dias_restantes <= 2:
        return "urgente"
    return "normal"
```

E em `tests/test_prazo.py`:

```python
from prazo import situacao


def test_situacoes():
    assert situacao(-5) == "atrasada"
    assert situacao(1) == "urgente"
    assert situacao(10) == "normal"
```

O teste passa e cobre 100% das linhas e dos desvios. O teste de mutação faz pequenas alterações no código, os **mutantes**, e roda os testes contra cada uma. Se os testes falham, o mutante foi morto; se passam, sobreviveu. Com o [`mutmut`](https://github.com/boxed/mutmut) 3.8.1, quatro dos dez mutantes deste exemplo sobrevivem, entre eles:

```diff
-    if dias_restantes <= 2:
+    if dias_restantes < 2:
```

Os quatro mexem nos limites das faixas, que o teste não visita. Acrescentando os casos de fronteira ([Capítulo 6](06_testes_validacao.md)), os dez são mortos:

```python
def test_limites():
    assert situacao(-1) == "atrasada"
    assert situacao(0) == "urgente"
    assert situacao(2) == "urgente"
    assert situacao(3) == "normal"
```

Para repetir o experimento, indique o arquivo no `pyproject.toml` e rode `mutmut run` e `mutmut results`:

```toml
[tool.pytest.ini_options]
pythonpath = ["."]

[tool.mutmut]
source_paths = ["prazo.py"]
```

O `mutmut` depende de um recurso do Unix (`fork`): no Windows, é preciso usar o [WSL](https://learn.microsoft.com/windows/wsl/).

### Métricas e código gerado por IA

Quando agentes escrevem muito mais código do que alguém consegue ler, quanto confiar nos números? Há quem discorde de forma frontal. Segundo [reportagem da Startup Fortune](https://startupfortune.com/uncle-bob-martin-says-he-no-longer-reads-ai-generated-code-and-the-developer-world-is-split/) (julho de 2026), sobre publicações em rede social:

* **Robert C. Martin** diz que já não lê o código dos seus agentes e confia nas barreiras que ele precisa atravessar: testes unitários e de aceitação, procedimentos de qualidade, limites de complexidade e de tamanho de módulo, análise de dependências, cobertura e teste de mutação.
* **Grady Booch** discorda e revisa todo o código gerado. Para ele, métricas não revelam vulnerabilidades, código morto nem oportunidades de refatoração perdidas.

A posição de Martin pressupõe testes e limites que alguém soube definir e que são cobrados de fato. A pesquisa ainda é recente. Um [estudo não revisado por pares](https://arxiv.org/abs/2603.24755), com tarefas em Python, observou que a repetição e a concentração de complexidade tendem a crescer quando agentes estendem várias vezes o próprio código. Um [relatório da GitClear](https://gitclear.com/the_ai_code_quality_maintainability_gap), empresa que vende ferramentas de análise, registra mais blocos duplicados e menos código movido por refatoração entre 2023 e 2026; são tendências simultâneas à adoção da IA, e não prova de que ela as causou.

!!! warning "Métrica que vira meta"
    Quando um número passa a ser cobrado por si só, pessoas e agentes aprendem a satisfazê-lo sem melhorar o código: testes sem asserção sobem a cobertura, e funções fatiadas sem critério baixam a complexidade. É a ideia conhecida como **lei de Goodhart**. Use as métricas para fazer perguntas, acompanhe a tendência e continue revisando o que é crítico.

## 🌳 Controle de versão colaborativo

**Gerência de versões** é o controle do que mudou, quando, por quem e por quê, de modo que se possa voltar a um estado anterior e combinar o trabalho de várias pessoas (CHACON; STRAUB, 2014). Há três organizações:

* **local**: o histórico fica só na máquina de quem trabalha;
* **centralizada**, como no Subversion: um servidor guarda o histórico, e cada pessoa tem uma cópia dos arquivos;
* **distribuída**, como no Git: cada clone tem o histórico inteiro. Dá para fazer *commits* e consultar o passado sem rede, e a equipe combina qual repositório serve de referência.

No Git, um *commit* registra um estado do projeto; uma *branch* é um ponteiro que avança a cada *commit*; e uma **etiqueta** (*tag*) marca um *commit* que não deve mudar de lugar, como o de uma versão publicada ([Capítulo 7](07_integracao_entrega_continua.md)). Os comandos estão no [apêndice](../apendices/git_conteineres.md).

### Fluxos de trabalho

Um **fluxo de trabalho** é o combinado da equipe sobre quais *branches* existem, quanto duram e como o trabalho chega à principal. Três são muito citados:

| Fluxo | Como funciona | Quando faz sentido |
|---|---|---|
| [**GitHub Flow**](https://docs.github.com/en/get-started/using-github/github-flow) | Uma *branch* por mudança, *pull request*, revisão, *merge* na `main` e exclusão da *branch* | Equipes pequenas, com uma versão em uso e entrega frequente |
| [***Trunk-based development***](https://dora.dev/capabilities/trunk-based-development/) | Integração na principal pelo menos uma vez por dia, com no máximo três *branches* ativas | Testes confiáveis, muita automação e revisão rápida |
| [**Git Flow**](https://nvie.com/posts/a-successful-git-branching-model) (Driessen, 2010) | Duas *branches* permanentes, `master` (o que está publicado) e `develop`, e outras temporárias para funcionalidades, preparação de versão e correções urgentes | Versões planejadas ou várias versões mantidas ao mesmo tempo |

Os três não são opostos perfeitos: um GitHub Flow com *branches* que duram poucas horas pode ser uma forma de *trunk-based*. No *trunk-based*, o que ainda está incompleto entra na principal desligado, por exemplo com [*feature flags*](07_integracao_entrega_continua.md). O próprio autor do Git Flow acrescentou ao texto, em 2020, que para entrega contínua sugere um fluxo bem mais simples, e que o modelo dele ainda pode servir a quem mantém várias versões.

Para escolher, pergunte: quantas versões precisam de correção ao mesmo tempo? Com que frequência se entrega? Quanto se confia nos testes? Quanto tempo uma revisão espera? O [TaskFlow](https://github.com/lambdageo-edu/taskflow), com uma só versão e um pipeline que roda a cada *pull request*, é bem atendido pelo GitHub Flow.

### Hábitos que valem em qualquer fluxo

* **Branches de curta duração**: quanto mais uma *branch* vive, mais diverge, e mais difícil fica o *merge*.
* **Mensagens de commit significativas**: descrever o "porquê" de uma mudança, não apenas o "o quê". O *diff* já mostra o que mudou.
* **Convenção de commits semânticos** (ex: `feat:`, `fix:`, `docs:`, `refactor:`), útil para gerar *changelogs* e comunicar a natureza de uma mudança.

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
7. Reproduza o erro de `relatorio.py`, pare nele com `breakpoint()` e examine `membros`. Corrija e confira que o teste passa.
8. Em um repositório de teste, introduza um defeito no meio de uma sequência de *commits* e encontre-o com `git bisect run`.
9. Em `corrida.py`, troque o trecho intercalado por chamadas a `assumir` e confira que só a primeira devolve `True`. Depois procure, em um projeto seu, um "consulta, decide, grava" e diga o que acontece se dois pedidos chegarem juntos.
10. Rode o exemplo de `prazo.py` com cobertura e com teste de mutação. Depois peça a um assistente de IA testes para uma função sua e veja quantos mutantes eles matam.
11. Em um sistema seu, desative um usuário que está com a sessão aberta. Quanto tempo ele ainda consegue usar o sistema? Escreva o teste que expressa o prazo que você considera aceitável.
12. Em um projeto seu com interface, procure `innerHTML`, `dangerouslySetInnerHTML` ou o filtro `safe`. Para cada uso, de onde vem o dado? Em um ambiente local, teste com o título do exemplo de XSS.
13. Descubra o fluxo usado em um projeto que você conhece (ou em um repositório público): que *branches* existem e há quanto tempo? Ele combina com a forma como o projeto é entregue?

## 📚 Para saber mais

* OWASP *Cross Site Scripting Prevention Cheat Sheet* — [cheatsheetseries.owasp.org](https://cheatsheetseries.owasp.org/cheatsheets/Cross_Site_Scripting_Prevention_Cheat_Sheet.html)
* OWASP *Cross-Site Request Forgery Prevention Cheat Sheet* — [cheatsheetseries.owasp.org](https://cheatsheetseries.owasp.org/cheatsheets/Cross-Site_Request_Forgery_Prevention_Cheat_Sheet.html)
* MDN, *Content Security Policy (CSP)* — [developer.mozilla.org](https://developer.mozilla.org/en-US/docs/Web/HTTP/Guides/CSP)
* NIST SP 800-63B-4, *Authentication and Authenticator Management* — [pages.nist.gov](https://pages.nist.gov/800-63-4/sp800-63b.html)
* RFC 7519, *JSON Web Token (JWT)* — [rfc-editor.org](https://www.rfc-editor.org/rfc/rfc7519)
* OWASP *Authentication Cheat Sheet* — [cheatsheetseries.owasp.org](https://cheatsheetseries.owasp.org/cheatsheets/Authentication_Cheat_Sheet.html)
* OWASP *JSON Web Token Cheat Sheet* — [cheatsheetseries.owasp.org](https://cheatsheetseries.owasp.org/cheatsheets/JSON_Web_Token_Cheat_Sheet.html)
* CHACON, Scott; STRAUB, Ben. *Pro Git*. 2. ed. Apress, 2014 — [git-scm.com/book/pt-br/v2](https://git-scm.com/book/pt-br/v2)
* GitHub, *GitHub flow* — [docs.github.com](https://docs.github.com/en/get-started/using-github/github-flow)
* DORA, *Trunk-based development* — [dora.dev](https://dora.dev/capabilities/trunk-based-development/)
* DRIESSEN, Vincent. *A successful Git branching model*, 2010, com nota de 2020 — [nvie.com](https://nvie.com/posts/a-successful-git-branching-model)
* MARTIN, Robert C. *Clean Code: A Handbook of Agile Software Craftsmanship*. Prentice Hall, 2008.
* MEYER, Bertrand. *Object-Oriented Software Construction*. 2. ed. Prentice Hall, 1997.
* FORSGREN, Nicole; HUMBLE, Jez; KIM, Gene. *Accelerate: The Science of Lean Software and DevOps*. IT Revolution, 2018.
* Documentação do `ruff` (linter/formatador Python) — [docs.astral.sh/ruff](https://docs.astral.sh/ruff/)
* Documentação do Python, *pdb* — [docs.python.org](https://docs.python.org/3.14/library/pdb.html)
* Git, *git-bisect* — [git-scm.com](https://git-scm.com/docs/git-bisect/2.52.0)
* MITRE, *CWE-367: Time-of-check Time-of-use (TOCTOU) Race Condition* — [cwe.mitre.org](https://cwe.mitre.org/data/definitions/367.html)
* SQLite, *Transaction* — [sqlite.org](https://www.sqlite.org/lang_transaction.html)
* FastAPI, *Concurrency and async / await* — [fastapi.tiangolo.com](https://fastapi.tiangolo.com/async/)
* ZELLER, Andreas. *Why Programs Fail: A Guide to Systematic Debugging*. 2. ed. Morgan Kaufmann, 2009.
* OWASP *Error Handling Cheat Sheet* — [cheatsheetseries.owasp.org](https://cheatsheetseries.owasp.org/cheatsheets/Error_Handling_Cheat_Sheet.html)
* *OWASP Top 10:2025* — [owasp.org/Top10/2025](https://owasp.org/Top10/2025/0x00_2025-Introduction/)
* OWASP *Secrets Management Cheat Sheet* — [cheatsheetseries.owasp.org](https://cheatsheetseries.owasp.org/cheatsheets/Secrets_Management_Cheat_Sheet.html)
* PERRY, Neil; SRIVASTAVA, Megha; KUMAR, Deepak; BONEH, Dan. *Do Users Write More Insecure Code with AI Assistants?* ACM CCS, 2023 — [arxiv.org/abs/2211.03622](https://arxiv.org/abs/2211.03622)
* SPRACKLEN, Joseph et al. *We Have a Package for You! A Comprehensive Analysis of Package Hallucinations by Code Generating LLMs*. USENIX Security, 2025 — [usenix.org](https://www.usenix.org/conference/usenixsecurity25/presentation/spracklen)
* Guia de código semântico de commits — [conventionalcommits.org](https://www.conventionalcommits.org/pt-br/)
* HUNT, Andrew; THOMAS, David. *The Pragmatic Programmer*. Addison-Wesley, 1999.
* BACCHELLI, Alberto; BIRD, Christian. *Expectations, Outcomes, and Challenges of Modern Code Review*. ICSE, 2013 — [microsoft.com/research](https://www.microsoft.com/en-us/research/?p=164195)
* MCCABE, Thomas J. *A Complexity Measure*. IEEE Transactions on Software Engineering, 1976 — [doi.org](https://doi.org/10.1109/TSE.1976.233837)
* ORLANSKI, Gabriel et al. *SlopCodeBench: Benchmarking How Coding Agents Degrade Over Long-Horizon Iterative Tasks*. 2026, não revisado por pares — [arxiv.org/abs/2603.24755](https://arxiv.org/abs/2603.24755)
