# Plano da Disciplina — Engenharia de Software

## Identificação

| | |
|---|---|
| **Carga horária** | 40 horas (8 aulas de 5h, ou 16 aulas de 2h30) |
| **Pré-requisitos** | Lógica de programação; experiência com pelo menos uma linguagem imperativa ou orientada a objetos |
| **Stack de exemplo** | Python 3.12+, pytest, FastAPI, Git/GitHub, GitHub Actions |

## Objetivos de aprendizagem

Ao final desta disciplina, o estudante deverá ser capaz de:

1. **Explicar** o que caracteriza software como produto de engenharia e por que a disciplina de Engenharia de Software existe.
2. **Comparar** modelos de processo de software (cascata, incremental, ágil) e justificar quando cada um é adequado.
3. **Elicitar e especificar** requisitos funcionais e não funcionais, tanto no formato clássico quanto como *user stories*.
4. **Projetar** a arquitetura de um sistema simples, aplicando padrões de projeto e arquitetura reconhecidos.
5. **Escrever código com qualidade**, aplicando práticas de *clean code*, revisão de código e análise estática.
6. **Escrever e executar testes automatizados** (unitários, integração, aceitação), aplicando a pirâmide de testes e TDD.
7. **Construir um pipeline de CI/CD** com GitHub Actions que valida, testa e publica um software automaticamente.
8. **Analisar** a evolução de um sistema em produção, identificando débito técnico e propondo refatorações.

## Cronograma

| Aula | Conteúdo | Capítulo |
|:---:|---|---|
| 1 | O que é software; a crise do software; qualidades de um bom software | [Introdução à Engenharia de Software](01_introducao_engenharia_software.md) |
| 2 | Modelos de processo: cascata, incremental, orientado a reúso; Manifesto Ágil, Scrum, Kanban, DevOps | [Processos: do Cascata ao Ágil e DevOps](02_processos_desenvolvimento.md) |
| 3 | Tipos de requisitos, elicitação, user stories, validação e gerência de requisitos | [Especificação de Requisitos](03_especificacao_requisitos.md) |
| 4 | Princípios de projeto, arquitetura em camadas, microsserviços, padrões de projeto | [Projeto e Arquitetura de Software](04_projeto_arquitetura.md) |
| 5 | Clean code, revisão de código, análise estática, controle de versão colaborativo | [Implementação e Qualidade de Código](05_implementacao_qualidade.md) |
| 6 | Defeito/erro/falha, pirâmide de testes, TDD; **hands-on**: testes com pytest em uma API FastAPI | [Testes e Validação](06_testes_validacao.md) |
| 7 | Integração contínua, entrega contínua, containers; **hands-on**: pipeline com GitHub Actions | [Integração e Entrega Contínua (CI/CD)](07_integracao_entrega_continua.md) |
| 8 | Tipos de manutenção, débito técnico, refatoração, observabilidade; revisão geral | [Evolução, Manutenção e Débito Técnico](08_evolucao_manutencao.md) |

## Metodologia

Aulas expositivas dialogadas intercaladas com demonstrações práticas de código. As aulas 6 e 7 são majoritariamente práticas, com um projeto guiado (API REST em FastAPI) usado como fio condutor para testes automatizados e CI/CD. Recomenda-se que o estudante reproduza os exemplos de código em seu próprio ambiente.

## Avaliação

- **Prova 1 (25%)** — após o Capítulo 3: fundamentos, processos e requisitos.
- **Prova 2 (25%)** — após o Capítulo 5: projeto, arquitetura e qualidade de código.
- **Trabalho prático (35%)** — implementação de uma API com suíte de testes (pytest) e pipeline de CI/CD (GitHub Actions) funcional, entregue ao final do Capítulo 7.
- **Prova 3 (15%)** — Capítulo 8: evolução e manutenção de software.

## Bibliografia da disciplina

- SOMMERVILLE, Ian. *Engenharia de Software*. 10. ed. Pearson, 2019.
- PRESSMAN, Roger S.; MAXIM, Bruce R. *Engenharia de Software: Uma Abordagem Profissional*. 8. ed. McGraw Hill, 2016.
- Manifesto para Desenvolvimento Ágil de Software. Disponível em: [agilemanifesto.org](https://agilemanifesto.org/iso/ptbr/manifesto.html).
- FOWLER, Martin. *Refactoring: Improving the Design of Existing Code*. 2. ed. Addison-Wesley, 2018.
