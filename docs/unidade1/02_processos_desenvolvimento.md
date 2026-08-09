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

O modelo em cascata representa as atividades fundamentais do processo — especificação, projeto, implementação, validação e evolução — como **fases sequenciais e distintas**: análise e definição de requisitos, projeto do sistema e do software, implementação e teste unitário, integração e teste de sistema, operação e manutenção. É considerado o primeiro modelo de processo de desenvolvimento de software publicado (Royce, 1970), derivado de processos mais gerais de engenharia de sistemas.

**Vantagem:** o processo é visível — a documentação produzida em cada fase facilita o acompanhamento do progresso por gerentes, de forma similar a outras engenharias.

**Desvantagem:** a divisão rígida em estágios exige compromissos assumidos cedo no processo, dificultando a adaptação a mudanças de requisitos. Por isso, o cascata funciona melhor quando os requisitos são bem compreendidos e pouco sujeitos a mudança — o que é raro em software de produto, mas ainda comum em contratos de sistemas críticos ou regulados (aeroespacial, defesa, dispositivos médicos).

### Desenvolvimento incremental

Intercala especificação, desenvolvimento e validação: o sistema é construído como uma série de versões (incrementos), cada uma adicionando funcionalidade à anterior. A ideia é desenvolver uma implementação inicial, expô-la a comentários dos usuários, e refiná-la em versões sucessivas até chegar a um sistema adequado.

O desenvolvimento incremental é a base de todas as abordagens ágeis modernas, e Sommerville destaca três vantagens sobre o cascata:

1. O custo de acomodar mudanças de requisitos é menor — há muito menos documentação e análise a refazer.
2. É mais fácil obter feedback real dos clientes, que enxergam software funcionando em vez de documentos de projeto.
3. É possível entregar valor rapidamente, mesmo antes de toda a funcionalidade estar pronta.

Como desvantagens, o processo é menos visível para fins de gestão (entregas rápidas dificultam medir progresso via documentação), e a estrutura do sistema tende a se degradar a cada novo incremento, a menos que tempo seja investido em refatoração.

### Engenharia orientada a reúso

Baseia-se em um conjunto significativo de componentes reutilizáveis — o desenvolvimento se concentra em integrar componentes e sistemas já existentes, em vez de construir tudo do zero. Esse modelo é ainda mais relevante hoje do que era: bibliotecas *open source*, serviços gerenciados de nuvem (bancos de dados, filas, autenticação) e SaaS de terceiros (pagamento, e-mail transacional) tornaram o reúso a norma, não a exceção, no desenvolvimento moderno.

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

Scrum é o framework ágil mais adotado no mercado. Organiza o trabalho em **sprints** (iterações de tempo fixo, tipicamente 1 a 4 semanas), com três papéis (*Product Owner*, *Scrum Master*, time de desenvolvimento), três artefatos (*Product Backlog*, *Sprint Backlog*, Incremento) e cinco eventos (*Sprint Planning*, *Daily Scrum*, *Sprint Review*, *Sprint Retrospective*, e a própria Sprint).

### Kanban

Kanban tem origem no sistema de produção da Toyota e foi adaptado ao desenvolvimento de software por David J. Anderson. Em vez de iterações de tempo fixo, o trabalho flui continuamente por colunas em um quadro (ex: *A Fazer → Em Progresso → Em Revisão → Concluído*), com **limites explícitos de trabalho em progresso (WIP)** para evitar sobrecarga e destacar gargalos.

=== "Scrum"
    * Trabalho organizado em sprints de duração fixa
    * Papéis formais e cerimônias definidas
    * Boa opção quando o time precisa de ritmo e previsibilidade de entrega
    * Exige maturidade para não virar "cascata em fatias de 2 semanas"

=== "Kanban"
    * Fluxo contínuo, sem iterações de tempo fixo
    * Foco em limitar trabalho em progresso e visualizar gargalos
    * Boa opção para times de suporte/manutenção com fluxo de demanda irregular
    * Métricas centrais: *lead time* e *throughput*

Na prática, muitas equipes usam híbridos (às vezes chamados *Scrumban*): sprints para planejamento e alinhamento, com um quadro Kanban e limites de WIP para gerenciar o fluxo do dia a dia.

!!! note "Scrum não é a única resposta a processos ágeis"
    Scrum continua sendo o framework ágil mais adotado no mercado, mas não é unânime: é comum times relatarem "fadiga de cerimônias" quando o ritmo de sprints, *dailies* e retrospectivas vira ritual repetitivo em vez de ferramenta útil — o que alguns autores chamam de "Scrum de fachada" (todas as cerimônias, nenhuma agilidade real). Como resposta, abordagens mais leves como o **Shape Up** (criado na Basecamp) vêm ganhando adoção real em alguns times e empresas: substituem sprints curtos e cerimônias diárias por ciclos mais longos (tipicamente 6 semanas) de trabalho focado, com um período de "resfriamento" entre ciclos. Isso não torna Scrum obsoleto — continua sendo uma escolha sólida, especialmente com múltiplos times que precisam de sincronização — mas reforça o ponto já feito neste capítulo: não existe processo universalmente superior, e vale conhecer alternativas ao framework mais popular.

## 🛠️ DevOps: além do processo de desenvolvimento

Enquanto o Ágil trata principalmente de como uma equipe planeja e executa o trabalho de *desenvolvimento*, o **DevOps** ataca a fronteira histórica entre times de desenvolvimento (Dev) e operações/infraestrutura (Ops). O termo surgiu por volta de 2009, popularizado por conferências como a *DevOpsDays*, como resposta a um problema recorrente: times de desenvolvimento otimizados para entregar mudanças rápido, e times de operação otimizados para manter sistemas estáveis — com incentivos frequentemente opostos.

DevOps propõe tratar essa fronteira como uma responsabilidade compartilhada, apoiada por automação e cultura, não apenas por ferramentas. Três ideias centrais:

* **"Você constrói, você opera" (*you build it, you run it*)**: o mesmo time responsável por escrever o software é responsável por operá-lo em produção, o que cria incentivo direto para escrever software fácil de operar (bom logging, boas métricas, *rollback* simples).
* **Automação de ponta a ponta**: infraestrutura como código, pipelines de integração e entrega contínua (Capítulo 7), testes automatizados (Capítulo 6) — tudo para reduzir trabalho manual e erro humano em processos repetitivos.
* **Cultura de aprendizado sem culpa (*blameless postmortems*)**: falhas em produção são tratadas como oportunidades de melhorar o sistema e o processo, não como motivo para punir indivíduos — o que incentiva as pessoas a reportar problemas cedo, em vez de escondê-los.

Um pilar prático do DevOps é a métrica **DORA** (*DevOps Research and Assessment*), que identifica quatro indicadores fortemente correlacionados com o desempenho de engenharia de uma organização: frequência de deploy, *lead time* para mudanças, taxa de falha em mudanças, e tempo de restauração após incidente. Times de alto desempenho fazem deploy sob demanda (múltiplas vezes ao dia), com *lead time* de horas, baixa taxa de falha e recuperação em minutos — contra deploys mensais/trimestrais, *lead time* de meses e recuperação em dias, típicos de organizações menos maduras.

!!! tip "Ágil e DevOps não competem — se complementam"
    Ágil resolve "como planejamos e priorizamos o trabalho de desenvolvimento". DevOps resolve "como levamos esse trabalho até produção, com segurança e frequência". Um time pode ser ágil sem ser DevOps (sprints organizados, mas deploys manuais e raros) — e o contrário também é possível, embora menos comum na prática.

## 📊 Escolhendo um processo

Não existe processo universalmente superior — a escolha depende de fatores como:

| Fator | Favorece processos dirigidos a planos | Favorece processos ágeis |
|---|---|---|
| Estabilidade dos requisitos | Requisitos bem compreendidos e estáveis | Requisitos incertos ou mutáveis |
| Criticidade do sistema | Sistemas de segurança crítica, regulados | Produtos digitais, sistemas internos |
| Tamanho e distribuição do time | Times grandes, múltiplas organizações | Times pequenos e colocados |
| Necessidade de auditoria/contrato | Contratos de escopo fechado | Parcerias com clientes colaborativos |

A maioria das organizações de produto digital hoje adota variantes ágeis com forte automação (DevOps), reservando processos mais dirigidos a planos para componentes de alto risco ou exigências regulatórias específicas dentro do mesmo sistema.

---

## 🎯 Atividades

1. Descreva um cenário de projeto de software em que o modelo em cascata ainda seria a escolha mais razoável hoje, e justifique.
2. Compare Scrum e Kanban usando um exemplo de equipe que você conhece (trabalho, estágio, projeto acadêmico). Qual se encaixaria melhor e por quê?
3. Explique, com suas palavras, a diferença entre "ser ágil" e "fazer DevOps". Dê um exemplo de time que seria um, mas não o outro.

## 📚 Para saber mais

* SOMMERVILLE, Ian. *Engenharia de Software*. 10. ed. — Capítulo 2.
* Manifesto para Desenvolvimento Ágil de Software — [agilemanifesto.org/iso/ptbr/manifesto.html](https://agilemanifesto.org/iso/ptbr/manifesto.html)
* Scrum Guide (oficial) — [scrumguides.org](https://scrumguides.org/)
* FORSGREN, Nicole; HUMBLE, Jez; KIM, Gene. *Accelerate: The Science of Lean Software and DevOps*. IT Revolution, 2018.
* Relatório *State of DevOps* (DORA/Google Cloud), publicado anualmente.
