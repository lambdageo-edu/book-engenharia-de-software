# Introdução à Engenharia de Software

Antes de falar em processos, requisitos ou arquitetura, é preciso responder a uma pergunta aparentemente simples: **o que é software, afinal, e por que ele exige uma disciplina de engenharia própria?**

---

## 💻 A natureza do software

Hoje é difícil encontrar uma atividade humana que não dependa de software: da distribuição de energia elétrica ao entretenimento, passando por sistemas bancários, hospitalares e de transporte. Mas, diferente do hardware — algo físico, palpável — o software é **abstrato e intangível**.

Segundo Sommerville, os sistemas de software se diferenciam de outros produtos de engenharia porque:

* são abstratos e intangíveis;
* não são restringidos pelas propriedades dos materiais;
* não são governados pelas leis da física;
* nem pelos processos de manufatura tradicionais.

Essa natureza muda completamente a forma como o software "envelhece". Hardware sofre desgaste físico: no início de sua vida útil há falhas associadas a defeitos de fabricação (mortalidade infantil); depois um longo período estável; e por fim falhas por desgaste dos componentes. Software não se desgasta fisicamente — em teoria, uma vez corrigido, um defeito não deveria voltar a aparecer sozinho. Na prática, porém, a curva de falhas do software tende a **crescer** ao longo do tempo, não pela deterioração do código em si, mas porque cada mudança e cada novo requisito introduzem risco de novos defeitos. É esse fenômeno — mudança constante gerando risco constante — que torna a manutenção de software (Capítulo 8) tão central para a disciplina.

!!! tip "Por que isso importa"
    Se o software não se desgasta fisicamente, por que sistemas antigos ficam tão difíceis de manter? A resposta não está no material, mas na **estrutura**: cada mudança malfeita degrada a arquitetura, aumenta o acoplamento e cria débito técnico. Vamos voltar a esse ponto várias vezes ao longo do livro.

## 🧩 Software não é apenas "escrever programas"

Sommerville destaca uma diferença importante entre **escrever um programa** e **desenvolver software profissional**. Muitas pessoas escrevem programas — para automatizar uma tarefa do dia a dia, processar dados de um experimento, ou por hobby. O desenvolvimento de software profissional é diferente: é uma atividade de engenharia em que o software é construído para um propósito de negócio específico, será usado por pessoas além de quem o escreveu, é normalmente produzido por equipes (não indivíduos), e precisa ser mantido e evoluído durante toda a sua vida útil — que pode durar décadas.

Pressman complementa com uma definição mais operacional. Um software é composto por:

1. **Instruções** (programas) que, executadas, produzem a função e o desempenho desejados;
2. **Estruturas de dados** que permitem aos programas manipular informação adequadamente;
3. **Informação descritiva**, impressa ou digital, que documenta a operação e o uso do software.

Repare que documentação, testes e infraestrutura de deploy fazem parte do "software" tanto quanto o código-fonte — um ponto que ferramentas modernas de "tudo como código" (*infrastructure as code*, *docs as code*) levam ainda mais a sério hoje do que nos anos 1990.

## 📦 Tipos de produtos e aplicações

Sommerville classifica produtos de software em duas categorias:

* **Produtos genéricos**: desenvolvidos por uma organização e vendidos a qualquer cliente interessado — editores de texto, ferramentas de banco de dados, SaaS de mercado (Slack, Notion, GitHub).
* **Produtos sob encomenda**: desenvolvidos para um cliente específico — sistemas de controle de tráfego aéreo, software bancário proprietário, sistemas internos de uma empresa.

Essa distinção segue válida, mas o mercado atual borrou bastante a linha entre as duas categorias: produtos SaaS "genéricos" são fortemente configuráveis por cliente, e produtos "sob encomenda" hoje quase sempre são construídos sobre plataformas genéricas (nuvem, frameworks, serviços gerenciados) em vez de do zero.

Quanto aos tipos de aplicação, os exemplos clássicos de Sommerville continuam válidos, mas vale atualizá-los ao cenário atual:

=== "Classificação clássica (Sommerville)"
    1. Aplicações *stand-alone* (executadas localmente)
    2. Aplicações interativas baseadas em transações (cliente-servidor)
    3. Sistemas embarcados
    4. Sistemas de processamento em lote
    5. Sistemas de entretenimento
    6. Sistemas de modelagem e simulação
    7. Sistemas de coleta de dados do ambiente
    8. Sistemas de sistemas

=== "Panorama atual"
    1. Aplicações *web* e *SaaS* multi-tenant na nuvem
    2. Aplicativos móveis nativos e híbridos
    3. APIs e microsserviços que compõem sistemas maiores
    4. Sistemas embarcados e IoT (agora frequentemente conectados à nuvem)
    5. *Data pipelines* e sistemas de processamento em lote/streaming (ex: Spark, Kafka)
    6. Sistemas de entretenimento e mídia sob demanda
    7. Sistemas de IA/ML embutidos em produtos (recomendação, visão computacional, LLMs)
    8. Plataformas internas (*internal developer platforms*) que orquestram outros sistemas

## ✅ Qualidades de um bom software

As qualidades esperadas variam conforme o tipo de sistema — um jogo e um sistema bancário têm prioridades diferentes — mas, de forma geral, Sommerville destaca quatro atributos essenciais:

| Atributo | O que significa |
|---|---|
| **Manutenibilidade** | O software deve poder evoluir para atender a novas necessidades — mudança é inevitável em qualquer negócio vivo. |
| **Confiabilidade e segurança** | Não deve causar prejuízo físico ou econômico em caso de falha; usuários maliciosos não devem conseguir acessar ou corromper o sistema. |
| **Eficiência** | Não deve desperdiçar recursos — memória, tempo de processamento, banda, energia. |
| **Aceitabilidade** | Deve ser compreensível, usável e compatível com os sistemas que os usuários já utilizam. |

!!! note "Atributos que ganharam peso nas últimas décadas"
    A lista de Sommerville é atemporal, mas hoje colocamos peso adicional em atributos que antes eram secundários:

    * **Observabilidade** — a capacidade de entender o que um sistema em produção está fazendo (logs estruturados, métricas, *tracing*), essencial em sistemas distribuídos.
    * **Escalabilidade** — a capacidade de crescer horizontalmente para atender picos de uso, um requisito quase padrão em produtos que nascem na nuvem.
    * **Segurança por padrão (*secure by design*)** — deixou de ser uma etapa final de revisão e passou a ser parte do processo de projeto, especialmente após décadas de vazamentos de dados de alto perfil.
    * **Sustentabilidade e custo operacional (*FinOps*)** — eficiência deixou de ser só sobre desempenho técnico e passou a incluir o custo financeiro e ambiental de rodar o software em nuvem.

## 🏛️ O nascimento da Engenharia de Software

No final da década de 1960, computadores se tornaram mais acessíveis a grandes empresas e instituições, impulsionados pelo avanço da microeletrônica. A demanda por software cresceu rapidamente — e a capacidade da indústria de entregá-lo com qualidade, no prazo e dentro do orçamento, não acompanhou esse crescimento. O fenômeno ficou conhecido como a **crise do software**, marcada por estimativas de prazo e custo sistematicamente imprecisas, produtividade insuficiente e qualidade abaixo do esperado.

Um marco histórico dessa época foi a **Conferência da OTAN sobre Engenharia de Software** (NATO Software Engineering Conference), realizada em outubro de 1968 em Garmisch, Alemanha. O encontro reuniu cientistas da computação para discutir como tornar o desenvolvimento de software tão rigoroso quanto as demais engenharias, e é considerado hoje o marco de nascimento da disciplina. O relatório produzido, com mais de 130 páginas, defendia que o software deveria ser construído sobre princípios práticos e teóricos sólidos — não apenas sobre a habilidade individual de bons programadores.

De acordo com a definição do IEEE, engenharia de software é a aplicação de uma abordagem:

* **sistemática** — existe um processo definido, com atividades explícitas;
* **disciplinada** — os processos definidos são de fato seguidos;
* **quantificável** — há métricas extraídas do processo para orientar decisões;

ao **desenvolvimento, operação e manutenção** de software — as três grandes fases do ciclo de vida.

Sommerville complementa: *"a engenharia de software é uma disciplina de engenharia cujo foco está em todos os aspectos da produção de software, desde os estágios iniciais da especificação do sistema até sua manutenção, quando o sistema já está sendo usado."* Em outras palavras, engenharia tem a ver com obter resultados de qualidade dentro de cronograma e orçamento, através de uma abordagem sistemática e organizada — escolhendo o método mais adequado para cada contexto, e não aplicando a mesma receita a todo problema.

!!! warning "Um mito comum"
    "Engenharia de software" não significa que exista uma única forma "certa" de desenvolver software, tampouco que processos rígidos sejam sempre superiores a processos ágeis. Como veremos no Capítulo 2, a escolha do processo é, ela própria, uma decisão de engenharia — depende do domínio, da criticidade do sistema e da estabilidade dos requisitos.

---

## 🎯 Atividades

1. Escolha um software que você usa diariamente (aplicativo, site, jogo). Liste evidências de que ele foi projetado pensando em manutenibilidade, confiabilidade, eficiência e aceitabilidade — ou evidências do contrário.
2. Explique, com suas palavras, por que a curva de falhas do software tende a aumentar com o tempo, mesmo o software não se desgastando fisicamente.
3. Compare um "produto genérico" e um "produto sob encomenda" que você conheça. Identifique elementos de configuração que tornam essa fronteira menos nítida hoje.

## 📚 Para saber mais

* SOMMERVILLE, Ian. *Engenharia de Software*. 10. ed. — Capítulo 1.
* PRESSMAN, Roger S.; MAXIM, Bruce R. *Engenharia de Software: Uma Abordagem Profissional*. 9. ed. AMGH, 2021. — Capítulo 1.
* Relatório da Conferência da OTAN de 1968 sobre Engenharia de Software (documento histórico, domínio público) — disponível em [homepages.cs.ncl.ac.uk/brian.randell/NATO](http://homepages.cs.ncl.ac.uk/brian.randell/NATO/).
