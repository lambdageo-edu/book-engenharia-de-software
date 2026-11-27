# Apêndice: HTTP e APIs em poucas linhas

Os exemplos deste livro são APIs: programas que outros programas usam. Uma **API web** recebe pedidos e devolve respostas pelo **HTTP**, o protocolo da web. Este apêndice reúne o vocabulário necessário para acompanhar os capítulos.

## 📨 Pedido e resposta

Um **cliente** (navegador, aplicativo, script) envia um pedido, e o **servidor** responde. O pedido tem um **método**, que diz a intenção; um **caminho**, que identifica o **recurso**; **cabeçalhos**, com informações sobre o pedido; e, às vezes, um **corpo** com dados. A resposta tem um **código de estado**, cabeçalhos e, em geral, um corpo.

```http
POST /tarefas HTTP/1.1
Host: localhost:8000
Content-Type: application/json

{"titulo": "Estudar HTTP"}
```

```http
HTTP/1.1 201 Created
content-type: application/json

{"titulo":"Estudar HTTP","concluida":false,"id":1}
```

O formato do corpo, neste caso, é o **JSON**. O **HTTPS** é o mesmo HTTP dentro de um canal cifrado (TLS): sem ele, senhas e *tokens* podem ser lidos no caminho.

## 🔧 Métodos

| Método | Uso comum | Seguro | Idempotente |
|---|---|---|---|
| `GET` | Ler um recurso | Sim | Sim |
| `POST` | Criar um recurso ou disparar uma ação | Não | Não |
| `PUT` | Criar ou substituir no endereço indicado | Não | Sim |
| `PATCH` | Alterar parte de um recurso | Não | Não necessariamente |
| `DELETE` | Remover | Não | Sim |

Um método **seguro** não pede mudança no servidor. Um método **idempotente** produz o mesmo efeito pretendido se for repetido (RFC 9110). Isso importa quando a rede falha e o cliente não sabe se o pedido chegou: repetir um `PUT` do responsável não muda nada além da primeira vez; repetir `POST /tarefas` cria duas tarefas. São convenções que o código precisa respeitar: o protocolo não impede um `GET` que apaga dados.

## 🔢 Códigos de estado

O primeiro dígito indica a família: `2xx`, deu certo; `3xx`, redirecionamento; `4xx`, o problema está no pedido; `5xx`, o problema está no servidor. Os que aparecem neste livro:

| Código | Nome | Quando |
|---|---|---|
| 200 | *OK* | Pedido atendido |
| 201 | *Created* | Um recurso foi criado |
| 204 | *No Content* | Deu certo e não há o que devolver |
| 401 | *Unauthorized* | Falta autenticação válida (apesar do nome, em inglês, "não autorizado") |
| 403 | *Forbidden* | O servidor entendeu e recusa |
| 404 | *Not Found* | O recurso não existe, ou o servidor não quer revelar que existe |
| 409 | *Conflict* | O pedido não cabe no estado atual do recurso |
| 422 | *Unprocessable Content* | O formato é compreendido, mas o conteúdo é inválido |
| 500 | *Internal Server Error* | Erro não previsto no servidor |
| 502 | *Bad Gateway* | Um intermediário, como um *proxy*, não obteve resposta válida do servidor à sua frente |

## 🧱 REST

**REST** é um estilo de arquitetura descrito por Roy Fielding em sua tese (2000). Entre as suas restrições estão a separação entre cliente e servidor; a comunicação **sem estado de sessão** no servidor, em que cada pedido traz tudo o que é preciso para entendê-lo; a possibilidade de *cache*; e uma **interface uniforme**, em que recursos têm identificadores, são manipulados por meio de representações e as respostas indicam, por *links*, o que se pode fazer em seguida.

Na prática, muitas APIs chamadas de REST seguem só parte disso: recursos nomeados por substantivos (`/projetos/1/tarefas`), métodos HTTP e códigos de estado. O TaskFlow é um exemplo: guarda as sessões no servidor, não devolve *links* e tem uma rota de ação (`POST .../tarefas/2/concluir`). Isso não o torna errado, mas convém saber o que o nome promete.

## 📄 Contrato da API

Quem usa uma API precisa saber caminhos, campos e códigos. O padrão **OpenAPI** descreve isso em um arquivo que ferramentas entendem. O FastAPI o gera a partir do código, em `/openapi.json`, com uma página interativa em `/docs`. Mudar um campo ou um código pode quebrar quem já usa a API: trate o contrato como parte do produto.

## 🎯 Atividades

1. Rode a API do [Capítulo 6](../capitulos/06_testes_validacao.md) e use `curl -i` para criar, listar e concluir uma tarefa. Anote método, caminho e código de cada resposta.
2. Escolha três erros possíveis nessa API e diga qual código cada um deveria devolver, e por quê.

## 📚 Para saber mais

* FIELDING, R.; NOTTINGHAM, M.; RESCHKE, J. (eds.). *RFC 9110: HTTP Semantics*, 2022 — [rfc-editor.org](https://www.rfc-editor.org/info/rfc9110)
* FIELDING, Roy T. *Architectural Styles and the Design of Network-based Software Architectures*. Tese, University of California, Irvine, 2000, cap. 5 — [ics.uci.edu](https://ics.uci.edu/~fielding/pubs/dissertation/rest_arch_style)
* OpenAPI Initiative — [openapis.org](https://www.openapis.org/)
