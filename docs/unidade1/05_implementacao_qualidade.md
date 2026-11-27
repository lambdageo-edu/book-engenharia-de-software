# Implementação e Qualidade de Código

## ⌨️ Da arquitetura ao código

Implementação é o processo de traduzir um projeto em um programa executável. Parece a etapa mais "mecânica" do processo de software, mas é aqui que a maior parte das decisões do dia a dia acontece — e onde a qualidade acumulada (ou a falta dela) determina o quão fácil (ou doloroso) será manter o sistema nos próximos anos.

## 🧹 Clean code: por que importa

O termo *clean code* foi popularizado pelo livro homônimo de Robert C. Martin (2008), mas a ideia é anterior e simples: **código é lido muito mais vezes do que é escrito**. Um desenvolvedor gasta boa parte do seu tempo lendo e entendendo código existente — próprio ou de outros — antes de conseguir modificá-lo com segurança. Código difícil de ler tem um custo real e recorrente, mesmo quando "funciona".

Algumas práticas amplamente aceitas de código limpo:

* **Nomes que revelam intenção**: `diasDesdeUltimoLogin` comunica mais que `d`. O tempo economizado por nomes ruins na hora de escrever é pago, com juros, por quem lê depois.
* **Funções pequenas e com um único propósito**: uma função que faz "uma coisa" é mais fácil de testar, entender e reutilizar do que uma função de 200 linhas que faz cinco coisas.
* **Evitar duplicação (princípio DRY — *Don't Repeat Yourself*)**: lógica duplicada em vários lugares significa que uma correção precisa ser lembrada e replicada em todos eles.
* **Comentários que explicam o "porquê", não o "o quê"**: se o código precisa de um comentário para explicar o que ele faz, muitas vezes é sinal de que ele poderia ser reescrito de forma mais clara. Comentários são mais valiosos quando explicam uma decisão não óbvia ("usamos X em vez de Y por causa da limitação Z da API externa").

!!! warning "Clean code não é dogma"
    Regras de estilo (como "funções devem ter no máximo N linhas") são heurísticas úteis, não leis absolutas. O objetivo final é sempre a legibilidade e a manutenibilidade — se seguir uma regra à risca torna o código pior, a regra perde.

## 👀 Revisão de código (code review)

Revisão de código é a prática de um ou mais colegas lerem e comentarem uma mudança de código antes que ela seja integrada ao projeto principal. Hoje é quase universal em times profissionais, tipicamente formalizada através de *pull requests* (GitHub, GitLab) ou *merge requests*.

Benefícios bem documentados da revisão de código:

* **Detecção de defeitos** antes que cheguem à produção — mais barata do que corrigir depois.
* **Disseminação de conhecimento** sobre o sistema entre os membros do time, reduzindo o risco de conhecimento concentrado em uma única pessoa (*bus factor* baixo).
* **Consistência de estilo e de decisões de projeto** ao longo do código.
* **Aprendizado mútuo**, especialmente entre desenvolvedores mais e menos experientes.

Boas práticas de revisão:

* Mudanças pequenas e focadas são mais fáceis (e mais rápidas) de revisar bem do que um *pull request* gigante que mistura várias funcionalidades.
* Comentários devem ser específicos e, idealmente, sugerir uma alternativa — não apenas apontar um problema.
* Separar comentários bloqueantes ("isso precisa mudar antes de aprovar") de sugestões ("isso é uma ideia, mas não bloqueio por causa disso") evita fricção desnecessária.

## 🤖 Revisando código gerado por IA

Assistentes e agentes de codificação (GitHub Copilot, Claude Code, Cursor e equivalentes) já fazem parte do dia a dia de boa parte dos times de desenvolvimento, gerando desde trechos pontuais até *pull requests* inteiros. Isso não elimina a necessidade de revisão de código — pelo contrário, muda o que a revisão precisa verificar:

* **O volume de código por *pull request* tende a aumentar** quando parte dele é gerada por IA, o que reforça (não enfraquece) a boa prática de manter mudanças pequenas e focadas.
* **Código gerado pode "parecer" correto sem ser**: sintaticamente limpo, bem nomeado, mas resolvendo um problema ligeiramente diferente do pedido, ou ignorando uma restrição de negócio que não estava explícita no prompt. Revisar significa validar a lógica contra o requisito real, não apenas a legibilidade do código.
* **Rastreabilidade da autoria não substitui responsabilidade**: quem abre o *pull request* — humano ou agente supervisionado por um humano — continua responsável por entender e defender a mudança na revisão, não apenas por tê-la solicitado a uma ferramenta.
* **Análise estática (próxima seção) se torna ainda mais importante** como primeira linha de defesa mecânica, liberando a revisão humana para focar em design, lógica de negócio e nas armadilhas acima — a mesma divisão entre máquina e pessoa explicada a seguir, agora com mais volume de código passando por ela.

!!! tip "IA muda o \"como\", não o \"porquê\" da revisão"
    Os benefícios de revisão de código listados no início desta seção — detecção de defeitos, disseminação de conhecimento, consistência, aprendizado — continuam os mesmos independentemente de quem (ou o quê) escreveu o código. O que muda é o volume e o tipo de atenção que o revisor precisa dedicar a cada mudança.

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
    Times sob pressão de prazo frequentemente cortam revisão de código, testes ou análise estática como forma de "ir mais rápido". Na prática, dados de pesquisas como o *State of DevOps* mostram o oposto: times de alto desempenho entregam **mais rápido e com mais qualidade simultaneamente** — a disciplina de qualidade reduz o retrabalho que, no médio prazo, é o que realmente desacelera um time.

---

## 🎯 Atividades

1. Pegue uma função de um projeto seu com mais de 30 linhas. Reescreva-a aplicando pelo menos dois princípios de clean code discutidos aqui (nomes melhores, quebra em funções menores, remoção de duplicação).
2. Configure um linter (`ruff` ou similar) em um projeto Python pequeno e corrija os três primeiros avisos que ele reportar.
3. Escreva um checklist de code review com pelo menos 6 itens que você usaria como revisor em um projeto real.

## 📚 Para saber mais

* MARTIN, Robert C. *Clean Code: A Handbook of Agile Software Craftsmanship*. Prentice Hall, 2008.
* FORSGREN, Nicole; HUMBLE, Jez; KIM, Gene. *Accelerate: The Science of Lean Software and DevOps*. IT Revolution, 2018.
* Documentação do `ruff` (linter/formatador Python) — [docs.astral.sh/ruff](https://docs.astral.sh/ruff/)
* Guia de código semântico de commits — [conventionalcommits.org](https://www.conventionalcommits.org/pt-br/)
