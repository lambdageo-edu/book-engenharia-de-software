# Processos de Desenvolvimento: do Cascata ao Ágil e DevOps

## 🔄 O que é um processo de software

Segundo Sommerville, um **processo de software** é um conjunto de atividades relacionadas que levam à produção de um produto de software — seja um sistema construído do zero, seja a extensão de um sistema já existente. Pressman define de forma similar: um processo é uma coleção de atividades, ações e tarefas executadas sempre que algum produto de trabalho é criado.

Todo processo de software, independente de qual modelo segue, inclui quatro atividades fundamentais:

1. **Especificação** — definir a funcionalidade do software e as restrições ao seu funcionamento.
2. **Projeto e implementação** — produzir o software que atenda às especificações.
3. **Validação** — garantir que o software atenda de fato às necessidades do cliente.
4. **Evolução** — fazer o software evoluir para acompanhar as mudanças dessas necessidades.

Além das atividades, todo processo envolve **produtos** (os resultados de cada atividade, como um documento de requisitos ou um modelo de arquitetura), **papéis** (as responsabilidades das pessoas envolvidas — gerente de projeto, desenvolvedor, QA) e **pré/pós-condições** (afirmações que devem ser verdadeiras antes e depois de uma atividade).

Não existe um processo "ideal" universal: cada organização aprende e adapta o seu, considerando a natureza do sistema. O desenvolvimento de um aplicativo de agenda pessoal é bem diferente do desenvolvimento do software de controle de uma usina nuclear — e essa diferença de risco e criticidade é o que leva a processos mais rígidos ou mais leves.

## ⚖️ Duas grandes categorias de processo

Sommerville categoriza os processos de software em duas grandes famílias:

* **Processos dirigidos a planos** (também chamados preditivos ou prescritivos): todas as atividades são planejadas com antecedência, e o progresso é medido comparando-se com esse plano inicial.
* **Processos ágeis**: o planejamento é incremental e gradual, e é mais fácil ajustar o processo para refletir mudanças nas necessidades do cliente.

Essas categorias não são mutuamente excludentes — a maioria das equipes reais usa uma combinação das duas, com mais planejamento em partes críticas e mais agilidade onde a incerteza é maior.

## 🌊 Modelos tradicionais dirigidos a planos

### Modelo em cascata

O modelo em cascata representa as atividades fundamentais do processo — especificação, projeto, implementação, validação e evolução — como **fases sequenciais e distintas**: análise e definição de requisitos, projeto do sistema e do software, implementação e teste unitário, integração e teste de sistema, operação e manutenção. É derivado de processos mais gerais de engenharia de sistemas e costuma ser associado ao artigo de Royce (1970). A associação pede cuidado: o artigo não usa o termo "cascata" e aponta como arriscada a sequência pura, com testes apenas no final, propondo passos adicionais para reduzir esse risco.

**Vantagem:** o processo é visível — a documentação produzida em cada fase facilita o acompanhamento do progresso por gerentes, de forma similar a outras engenharias.

**Desvantagem:** a divisão rígida em estágios exige compromissos assumidos cedo no processo, dificultando a adaptação a mudanças de requisitos. Por isso, o cascata funciona melhor quando os requisitos são bem compreendidos e pouco sujeitos a mudança — o que é raro em software de produto, mas ainda comum em contratos de sistemas críticos ou regulados (aeroespacial, defesa, dispositivos médicos).

### Desenvolvimento incremental

Intercala especificação, desenvolvimento e validação: o sistema é construído como uma série de versões (incrementos), cada uma adicionando funcionalidade à anterior. A ideia é desenvolver uma implementação inicial, expô-la a comentários dos usuários, e refiná-la em versões sucessivas até chegar a um sistema adequado.

O desenvolvimento incremental é a base de todas as abordagens ágeis modernas, e Sommerville destaca três vantagens sobre o cascata:

1. O custo de acomodar mudanças de requisitos é menor — há muito menos documentação e análise a refazer.
2. É mais fácil obter feedback real dos clientes, que enxergam software funcionando em vez de documentos de projeto.
3. É possível entregar valor rapidamente, mesmo antes de toda a funcionalidade estar pronta.

Como desvantagens, o processo é menos visível para fins de gestão (entregas rápidas dificultam medir progresso via documentação), e a estrutura do sistema tende a se degradar a cada novo incremento, a menos que tempo seja investido em refatoração.

### Integração e configuração

Também chamado de desenvolvimento orientado a reúso, baseia-se em um conjunto significativo de componentes reutilizáveis — o desenvolvimento se concentra em integrar componentes e sistemas já existentes, em vez de construir tudo do zero. Esse modelo é ainda mais relevante hoje do que era: bibliotecas *open source*, serviços gerenciados de nuvem (bancos de dados, filas, autenticação) e SaaS de terceiros (pagamento, e-mail transacional) tornaram o reúso a norma, não a exceção, no desenvolvimento moderno.

## 🚀 O Manifesto Ágil

No início dos anos 2000, a insatisfação com processos pesados e burocráticos levou um grupo de desenvolvedores a formalizar princípios que já vinham sendo praticados informalmente (Extreme Programming, Scrum, Crystal). O resultado foi o **Manifesto para Desenvolvimento Ágil de Software** (2001), que estabelece quatro valores centrais:

> Estamos descobrindo maneiras melhores de desenvolver software, fazendo-o nós mesmos e ajudando outros a fazerem o mesmo. Através deste trabalho, passamos a valorizar:
>
> * **Indivíduos e interações** mais que processos e ferramentas
> * **Software em funcionamento** mais que documentação abrangente
> * **Colaboração com o cliente** mais que negociação de contratos
> * **Responder a mudanças** mais que seguir um plano

!!! note "Um detalhe importante"
    O manifesto não diz que processos, documentação, contratos e planos não têm valor — diz que os itens à esquerda são **mais valorizados** que os da direita. Ágil não é sinônimo de "sem processo" ou "sem documentação"; é uma priorização diferente diante da incerteza.

Esses valores se desdobram em 12 princípios, dos quais vale destacar alguns que resumem bem a mudança de mentalidade: entregar software funcionando com frequência (de poucas semanas a poucos meses, preferindo os períodos mais curtos); receber bem mudanças de requisito, mesmo tardias no desenvolvimento; medir progresso principalmente pelo software funcionando; e manter um ritmo sustentável de trabalho.

### Scrum

Scrum é o framework ágil mais adotado no mercado. Organiza o trabalho em **sprints**, iterações de duração fixa de um mês ou menos. A versão de 2020 do [Guia do Scrum](https://scrumguides.org/) define um *Scrum Team* com três **responsabilidades** (*accountabilities*): *Product Owner*, *Scrum Master* e *Developers*. São cinco eventos (a própria Sprint, *Sprint Planning*, *Daily Scrum*, *Sprint Review* e *Sprint Retrospective*) e três artefatos, cada um com um **compromisso** que dá direção e transparência:

| Artefato | Compromisso |
|---|---|
| *Product Backlog* | Meta do produto (*Product Goal*) |
| *Sprint Backlog* | Meta da sprint (*Sprint Goal*) |
| Incremento | Definição de pronto (*Definition of Done*) |

#### Definição de pronto

A **definição de pronto** (*Definition of Done*) descreve o estado que um incremento precisa atingir para atender às medidas de qualidade do produto. Um item que não a cumpre não é entregue nem apresentado na revisão da *sprint*: volta para o *backlog* (SCHWABER; SUTHERLAND, 2020). Para a API de tarefas, o [projeto condutor](../index.md) do livro, construída no [Capítulo 6](06_testes_validacao.md), uma primeira versão poderia ser:

* os critérios de aceitação têm testes, e todos passam;
* o pipeline está verde, incluindo as verificações de segurança (Capítulo 7);
* outra pessoa revisou o *pull request*;
* a documentação afetada foi atualizada;
* nenhum segredo ou dado real foi incluído no repositório.

Não confunda com os **critérios de aceitação** (Capítulo 3): eles mudam a cada história e dizem o que *aquela* funcionalidade deve fazer; a definição de pronto vale para *todas*. A ideia é útil mesmo fora do Scrum.

### Kanban

Kanban tem origem no sistema de produção da Toyota e foi adaptado ao desenvolvimento de software por David J. Anderson (2010). Em vez de iterações de tempo fixo, o trabalho flui continuamente por colunas em um quadro (ex: *A Fazer → Em Progresso → Em Revisão → Concluído*), com **limites explícitos de trabalho em progresso (WIP)** para evitar sobrecarga e destacar gargalos.

=== "Scrum"
    * Trabalho organizado em sprints de duração fixa
    * Responsabilidades e eventos definidos
    * Boa opção quando o time precisa de ritmo e previsibilidade de entrega
    * Exige maturidade para não virar "cascata em fatias de 2 semanas"

=== "Kanban"
    * Fluxo contínuo, sem iterações de tempo fixo
    * Foco em limitar trabalho em progresso e visualizar gargalos
    * Boa opção para times de suporte/manutenção com fluxo de demanda irregular
    * Métricas centrais: *lead time* e *throughput*

Na prática, muitas equipes usam híbridos (às vezes chamados *Scrumban*): sprints para planejamento e alinhamento, com um quadro Kanban e limites de WIP para gerenciar o fluxo do dia a dia.

!!! note "Scrum não é a única resposta a processos ágeis"
    A adoção ampla não o torna unânime: é comum times relatarem "fadiga de cerimônias" quando o ritmo de sprints, *dailies* e retrospectivas vira ritual repetitivo em vez de ferramenta útil — o que se costuma chamar de "Scrum de fachada" (todas as cerimônias, nenhuma agilidade real). Como resposta, abordagens mais leves como o [**Shape Up**](https://basecamp.com/shapeup) (criado na Basecamp e descrito por Ryan Singer) propõem outro ritmo: substituem sprints curtos e cerimônias diárias por ciclos mais longos (tipicamente 6 semanas) de trabalho focado, com um período de "resfriamento" entre ciclos. Isso não torna Scrum obsoleto, que continua sendo uma escolha sólida, especialmente com múltiplos times que precisam de sincronização.

## 🛠️ DevOps: além do processo de desenvolvimento

Enquanto o Ágil trata principalmente de como uma equipe planeja e executa o trabalho de *desenvolvimento*, o **DevOps** ataca a fronteira histórica entre times de desenvolvimento (Dev) e operações/infraestrutura (Ops). O termo surgiu por volta de 2009, popularizado por conferências como a *DevOpsDays*, como resposta a um problema recorrente: times de desenvolvimento otimizados para entregar mudanças rápido, e times de operação otimizados para manter sistemas estáveis — com incentivos frequentemente opostos.

DevOps propõe tratar essa fronteira como uma responsabilidade compartilhada, apoiada por automação e cultura, não apenas por ferramentas. Três ideias centrais:

* **"Você constrói, você opera" (*you build it, you run it*)**, expressão de Werner Vogels, da Amazon, em [entrevista de 2006](https://queue.acm.org/detail.cfm?id=1142065): o mesmo time responsável por escrever o software é responsável por operá-lo em produção, o que cria incentivo direto para escrever software fácil de operar (bom logging, boas métricas, *rollback* simples).
* **Automação de ponta a ponta**: infraestrutura como código, pipelines de integração e entrega contínua (Capítulo 7), testes automatizados (Capítulo 6) — tudo para reduzir trabalho manual e erro humano em processos repetitivos.
* **Cultura de aprendizado sem culpa (*blameless postmortems*)**, prática [descrita por John Allspaw](https://www.etsy.com/codeascraft/blameless-postmortems): falhas em produção são tratadas como oportunidades de melhorar o sistema e o processo, não como motivo para punir indivíduos — o que incentiva as pessoas a reportar problemas cedo, em vez de escondê-los.

Um instrumento prático do DevOps são as [métricas de entrega de software da DORA](https://dora.dev/guides/dora-metrics/history/) (*DevOps Research and Assessment*). Desde 2024 são cinco, em dois grupos:

| Grupo | Métrica | O que mede |
|---|---|---|
| Vazão | Tempo de espera das mudanças (*change lead time*) | Do *commit* até a mudança em produção |
| Vazão | Frequência de implantação | Com que frequência se implanta |
| Vazão | Tempo de recuperação de implantação com falha | Quanto se leva para recuperar uma implantação que falhou |
| Instabilidade | Taxa de falha das mudanças | Proporção de implantações que causam falha em produção |
| Instabilidade | Taxa de retrabalho de implantação | Proporção de implantações não planejadas, feitas para corrigir problemas |

Relatórios anteriores a 2025 agrupavam as organizações em faixas de desempenho (de baixo a elite). O [relatório de 2025](https://redmonk.com/rstephens/2025/12/18/dora2025/), renomeado para *State of AI-assisted Software Development*, trocou essas faixas por sete perfis de equipe. Use as métricas para acompanhar a evolução do seu próprio time, e não como um placar entre times.

!!! tip "Ágil e DevOps não competem — se complementam"
    Ágil resolve "como planejamos e priorizamos o trabalho de desenvolvimento". DevOps resolve "como levamos esse trabalho até produção, com segurança e frequência". Um time pode ser ágil sem ser DevOps (sprints organizados, mas deploys manuais e raros) — e o contrário também é possível, embora menos comum na prática.

## 📊 Escolhendo um processo

Não existe processo universalmente superior — a escolha depende de fatores como:

| Fator | Favorece processos dirigidos a planos | Favorece processos ágeis |
|---|---|---|
| Estabilidade dos requisitos | Requisitos bem compreendidos e estáveis | Requisitos incertos ou mutáveis |
| Criticidade do sistema | Sistemas de segurança crítica, regulados | Produtos digitais, sistemas internos |
| Tamanho e distribuição do time | Times grandes, múltiplas organizações | Times pequenos e co-localizados |
| Necessidade de auditoria/contrato | Contratos de escopo fechado | Parcerias com clientes colaborativos |

A maioria das organizações de produto digital hoje adota variantes ágeis com forte automação (DevOps), reservando processos mais dirigidos a planos para componentes de alto risco ou exigências regulatórias específicas dentro do mesmo sistema.

---

## 🎯 Atividades

1. Descreva um cenário de projeto de software em que o modelo em cascata ainda seria a escolha mais razoável hoje, e justifique.
2. Compare Scrum e Kanban usando um exemplo de equipe que você conhece (trabalho, estágio, projeto acadêmico). Qual se encaixaria melhor e por quê?
3. Explique, com suas palavras, a diferença entre "ser ágil" e "fazer DevOps". Dê um exemplo de time que seria um, mas não o outro.
4. Escreva a definição de pronto de um projeto em que você participa. Quais itens podem ser verificados automaticamente?

## 📚 Para saber mais

* SOMMERVILLE, Ian. *Engenharia de Software*. 10. ed. — Capítulo 2.
* Manifesto para Desenvolvimento Ágil de Software — [agilemanifesto.org/iso/ptbr/manifesto.html](https://agilemanifesto.org/iso/ptbr/manifesto.html)
* Scrum Guide (oficial) — [scrumguides.org](https://scrumguides.org/)
* SCHWABER, Ken; SUTHERLAND, Jeff. *The Scrum Guide*, 2020 — [scrumguides.org](https://www.scrumguides.org/scrum-guide.html)
* FORSGREN, Nicole; HUMBLE, Jez; KIM, Gene. *Accelerate: The Science of Lean Software and DevOps*. IT Revolution, 2018.
* Relatórios anuais da DORA (Google Cloud): *State of DevOps* até 2024 e *State of AI-assisted Software Development* em 2025 — [dora.dev](https://dora.dev/).
* ROYCE, Winston W. *Managing the Development of Large Software Systems*. Proceedings of IEEE WESCON, 1970.
* ANDERSON, David J. *Kanban: Successful Evolutionary Change for Your Technology Business*. Blue Hole Press, 2010.
* SINGER, Ryan. *Shape Up: Stop Running in Circles and Ship Work that Matters*. Basecamp, 2019 — [basecamp.com/shapeup](https://basecamp.com/shapeup)
* VOGELS, Werner. *A Conversation with Werner Vogels* (entrevista a Jim Gray). ACM Queue, 4(4), 2006 — [queue.acm.org](https://queue.acm.org/detail.cfm?id=1142065)
* ALLSPAW, John. *Blameless PostMortems and a Just Culture*. Etsy, 2012 — [etsy.com/codeascraft](https://www.etsy.com/codeascraft/blameless-postmortems)
