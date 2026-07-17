# Engenharia de Software

Bem-vindo ao material didático da disciplina de **Engenharia de Software**! Este livro reúne, em oito capítulos, os fundamentos clássicos da disciplina — consolidados por autores como Sommerville e Pressman — atualizados com as práticas, ferramentas e o vocabulário que o mercado usa hoje: metodologias ágeis, arquitetura de software moderna, testes automatizados e integração/entrega contínua (CI/CD).

---

## 📋 Plano da Disciplina

| | |
|---|---|
| **Carga horária** | 40 horas (8 capítulos, ~5h de estudo/prática cada) |
| **Estrutura** | Unidade única, sequencial, com avaliações ao final de cada bloco |
| **Stack de exemplo** | Python (pytest, FastAPI) e GitHub Actions |
| **Pré-requisitos** | Lógica de programação e experiência com pelo menos uma linguagem de programação |

**Ementa:** O que é engenharia de software e por que ela existe. Processos e modelos de desenvolvimento, do cascata ao ágil e ao DevOps. Especificação e gerência de requisitos. Projeto de arquitetura e padrões de projeto. Implementação, qualidade de código e revisão de código. Testes automatizados e a pirâmide de testes. Integração e entrega contínua (CI/CD). Evolução, manutenção e débito técnico.

Consulte o [plano detalhado da disciplina](unidade1/00_plano_unidade1.md) para o cronograma aula a aula, metodologia e critérios de avaliação.

---

## 🎯 Estrutura do Curso

### 📘 Fundamentos (Capítulos 1-2)
O que diferencia software de outros produtos, por que a engenharia de software nasceu como disciplina, e como as organizações estruturam seu trabalho — do modelo em cascata, passando pelo desenvolvimento incremental, até o Manifesto Ágil, Scrum, Kanban e a cultura DevOps.

### 📙 Ciclo de desenvolvimento (Capítulos 3-6)
Como transformar uma necessidade de negócio em software funcionando: elicitação e especificação de requisitos (em formato clássico e como *user stories*), projeto de arquitetura e padrões de projeto, práticas de implementação e qualidade de código, e a disciplina de testes — de unitários a testes de aceitação, passando por TDD.

### 📗 Prática moderna e sustentação (Capítulos 7-8)
Como entregar software com segurança e frequência: pipelines de integração e entrega contínua com GitHub Actions, containers, e como o software evolui depois de estar em produção — manutenção, débito técnico, refatoração e observabilidade.

---

## 🧪 Hands-on

Ao longo do livro, os capítulos 6 e 7 incluem um projeto prático guiado: uma API REST em **FastAPI**, testada com **pytest** e publicada por um pipeline de **GitHub Actions**, usada como fio condutor para os conceitos de testes automatizados e CI/CD.

---

## 📚 Bibliografia e Referências

* SOMMERVILLE, Ian. *Engenharia de Software*. 10. ed. Pearson, 2019.
* PRESSMAN, Roger S.; MAXIM, Bruce R. *Engenharia de Software: Uma Abordagem Profissional*. 8. ed. McGraw Hill, 2016.
* Manifesto para Desenvolvimento Ágil de Software — [agilemanifesto.org](https://agilemanifesto.org/iso/ptbr/manifesto.html)
* FOWLER, Martin. *Refactoring: Improving the Design of Existing Code*. 2. ed. Addison-Wesley, 2018.
* Documentação oficial: [pytest](https://docs.pytest.org/), [FastAPI](https://fastapi.tiangolo.com/), [GitHub Actions](https://docs.github.com/actions)
