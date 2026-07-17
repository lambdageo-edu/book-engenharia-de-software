# Especificação de Requisitos

## 📋 O que são requisitos

Requisitos descrevem **o que** um sistema deve fazer e **sob quais restrições** ele deve operar. Eles são, em geral, o produto mais difícil de acertar em um projeto de software: requisitos errados ou incompletos custam caro para corrigir quanto mais tarde forem descobertos, e a maioria dos problemas de projetos de software tem raiz em requisitos malentendidos, não em código malfeito.

Requisitos costumam existir em diferentes níveis de abstração, e diferentes públicos leem diferentes níveis:

* **Requisitos de usuário**: enunciados em linguagem natural (e às vezes diagramas), descrevendo os serviços que o sistema deve fornecer e as restrições sob as quais deve operar, voltados a clientes e usuários finais.
* **Requisitos de sistema**: descrições mais detalhadas das funções, serviços e restrições do sistema, servindo como contrato entre cliente e desenvolvedores.

## 🏷️ Classificação dos requisitos

### Requisitos funcionais

Descrevem serviços que o sistema deve fornecer, como o sistema deve reagir a entradas específicas, e como deve se comportar em situações particulares. Podem também declarar explicitamente o que o sistema **não** deve fazer.

*Exemplo:* "O sistema deve permitir que o usuário redefina sua senha por e-mail."

### Requisitos não funcionais

Restringem os serviços ou funções oferecidos pelo sistema — não dizem o que o sistema faz, mas **como** ele deve fazer. Costumam ser classificados em:

| Categoria | Exemplos |
|---|---|
| **Requisitos de produto** | Desempenho, usabilidade, confiabilidade, portabilidade |
| **Requisitos organizacionais** | Padrões de processo da empresa, políticas de segurança da informação |
| **Requisitos externos** | Regulamentações legais (LGPD, GDPR), interoperabilidade com sistemas de terceiros |

*Exemplo:* "O sistema deve responder a 95% das requisições em menos de 300ms sob carga de 1000 requisições/segundo."

!!! warning "Requisitos não funcionais são frequentemente subestimados"
    É comum um projeto ter uma lista extensa de requisitos funcionais e apenas uma frase vaga sobre desempenho ou segurança. Na prática, requisitos não funcionais mal especificados são uma das causas mais comuns de retrabalho tardio e caro — descobrir que o sistema não escala, ou que não atende a uma exigência de conformidade, depois de já estar em produção.

## 🔎 A engenharia de requisitos

A engenharia de requisitos é o processo estruturado de descobrir, analisar, documentar e verificar esses serviços e restrições. Ela se organiza, tradicionalmente, em quatro grandes atividades:

1. **Elicitação de requisitos** — descoberta dos requisitos através da interação com stakeholders, documentos existentes e análise do domínio.
2. **Classificação e organização** — agrupar requisitos relacionados e estruturá-los de forma coerente.
3. **Priorização e negociação** — resolver conflitos entre requisitos e stakeholders, decidindo o que entra em cada versão.
4. **Especificação** — documentar os requisitos de forma clara, verificável e (idealmente) testável.

### Elicitação: como descobrir requisitos

A elicitação de requisitos combina várias técnicas, cada uma com pontos fortes e limitações:

=== "Entrevistas"
    Conversas estruturadas ou semiestruturadas com stakeholders. Boas para explorar contexto e motivação, mas dependem da capacidade do entrevistado de articular necessidades que muitas vezes são implícitas.

=== "Observação (etnografia)"
    Observar usuários realizando seu trabalho real revela necessidades que eles não saberiam verbalizar — hábitos, atalhos informais, exceções ao processo "oficial".

=== "Workshops e dinâmicas colaborativas"
    Sessões com múltiplos stakeholders (ex: *Event Storming*, *Story Mapping*) ajudam a alinhar visões divergentes e descobrir requisitos nas lacunas entre departamentos.

=== "Protótipos"
    Mostrar algo tangível — mesmo que simples — costuma gerar feedback muito mais preciso do que discutir requisitos de forma puramente verbal ou textual.

### User stories: o formato ágil de requisito

Em processos ágeis (Capítulo 2), requisitos funcionais são frequentemente escritos como **user stories**, um formato deliberadamente leve que adia detalhes de especificação para o momento em que a funcionalidade será de fato construída:

```
Como <tipo de usuário>,
quero <realizar uma ação>,
para que <benefício/objetivo>.
```

*Exemplo:* "Como assinante do plano gratuito, quero ver quantos créditos me restam no mês, para que eu possa planejar meu uso sem ser surpreendido."

Uma user story sozinha costuma ser incompleta demais para guiar o desenvolvimento — por isso ela é normalmente acompanhada de **critérios de aceitação**, que funcionam como uma especificação mínima e testável:

```
Dado que o assinante está logado no plano gratuito,
quando ele acessa o painel principal,
então o sistema exibe o número de créditos restantes no mês corrente.
```

Esse formato *Dado/Quando/Então* (Gherkin) tem a vantagem de já nascer no formato usado por ferramentas de teste de aceitação automatizado (como Cucumber ou Behave) — aproximando requisito e teste, um tema que retomaremos no Capítulo 6.

!!! tip "User story não é 'requisito ágil de baixa qualidade'"
    Um erro comum é achar que user stories dispensam análise. Na verdade, elas deslocam o detalhamento para o momento certo (*just in time*), em vez de tentar prever tudo antecipadamente — mas ainda exigem elicitação cuidadosa, critérios de aceitação claros e refinamento contínuo do backlog.

## ✅ Boas práticas de especificação

Independente do formato (documento formal ou user story), um bom requisito deve ser:

* **Claro e não ambíguo** — evitar termos vagos como "rápido", "fácil de usar", "sempre" sem defini-los quantitativamente.
* **Verificável/testável** — deve ser possível checar, objetivamente, se o requisito foi atendido.
* **Rastreável** — deve ser possível ligar o requisito à sua origem (stakeholder, regra de negócio) e ao código/teste que o implementa.
* **Consistente** — não pode contradizer outro requisito do mesmo sistema.

Um critério útil e amplamente usado para avaliar requisitos, especialmente em contexto ágil, é o acrônimo **INVEST**: Independente, Negociável, Valioso, Estimável, Pequeno (*Small*) e Testável.

## 🔁 Validação e gerenciamento de requisitos

**Validar** requisitos significa checar se eles de fato representam o que o cliente precisa — não apenas se estão bem escritos. Técnicas comuns incluem revisões estruturadas com stakeholders, protótipos e a construção de casos de teste diretamente a partir dos requisitos (se é difícil escrever um teste para um requisito, é sinal de que ele está mal especificado).

**Gerenciar** requisitos é reconhecer que eles mudam — por entendimento incompleto do domínio, por mudança do próprio negócio, ou por decisões de escopo. Um processo básico de gerência de requisitos inclui:

* identificação única de cada requisito (rastreabilidade);
* um processo definido para propor e avaliar mudanças;
* uma matriz de rastreabilidade ligando requisitos a componentes de código e casos de teste;
* versionamento do documento (ou backlog) de requisitos.

Ferramentas modernas de gestão de produto (Jira, Linear, Azure Boards) e de documentação viva (Notion, Confluence) cumprem hoje boa parte desse papel de forma mais leve do que os documentos de especificação formal (como o padrão IEEE 830) usados historicamente — mas o problema que resolvem é o mesmo: manter requisitos rastreáveis, atualizados e visíveis para todo o time.

---

## 🎯 Atividades

1. Escreva três requisitos funcionais e três requisitos não funcionais para um aplicativo de biblioteca universitária (empréstimo de livros).
2. Reescreva um dos requisitos funcionais do item anterior como uma user story com critérios de aceitação no formato Dado/Quando/Então.
3. Avalie uma user story real (sua ou de um colega) usando o critério INVEST. Ela falha em algum dos critérios? Como você a melhoraria?

## 📚 Para saber mais

* SOMMERVILLE, Ian. *Engenharia de Software*. 10. ed. — Capítulo 4.
* COHN, Mike. *User Stories Applied: For Agile Software Development*. Addison-Wesley, 2004.
* Formato Gherkin (Cucumber) — [cucumber.io/docs/gherkin](https://cucumber.io/docs/gherkin/)
