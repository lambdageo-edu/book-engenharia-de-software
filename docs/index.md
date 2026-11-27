# Desenvolvimento de Software

Este é um livro aberto sobre como construir, entregar e manter software. Em oito capítulos, ele reúne fundamentos clássicos da engenharia de software, consolidados por autores como Sommerville e Pressman, e as práticas atuais: métodos ágeis, arquitetura, testes automatizados e integração e entrega contínuas (CI/CD).

---

## 🧭 Antes de começar

O livro pressupõe lógica de programação e experiência com pelo menos uma linguagem. Os exemplos usam Python (pytest, FastAPI) e GitHub Actions.

Os capítulos seguem uma ordem progressiva, mas também podem ser consultados separadamente.

---

## 🎯 Estrutura do Livro

### 📘 Fundamentos (Capítulos 1-2)
O que diferencia software de outros produtos, por que a engenharia de software nasceu como disciplina, e como as organizações estruturam seu trabalho — do modelo em cascata, passando pelo desenvolvimento incremental, até o Manifesto Ágil, Scrum, Kanban e a cultura DevOps.

### 📙 Ciclo de desenvolvimento (Capítulos 3-6)
Como transformar uma necessidade de negócio em software funcionando: elicitação e especificação de requisitos (em formato clássico e como histórias de usuário, ou *user stories*), projeto de arquitetura e padrões de projeto, práticas de implementação e qualidade de código, e a disciplina de testes — de unitários a testes de sistema, além de testes de aceitação e TDD.

### 📗 Prática moderna e sustentação (Capítulos 7-8)
Como entregar software com frequência e confiança: pipelines de integração e entrega contínua com GitHub Actions, contêineres, e como o software evolui depois de estar em produção — manutenção, débito técnico, refatoração e observabilidade.

---

## 🧪 Hands-on

Ao longo do livro, os capítulos 6 e 7 incluem um projeto prático guiado: uma API REST em **FastAPI**, testada com **pytest** e publicada por um pipeline de **GitHub Actions**, usada como fio condutor para os conceitos de testes automatizados e CI/CD.

---

## ⚖️ Licença

© Sergio Souza Costa. Este livro, incluindo os exemplos de código, é distribuído sob a licença [Creative Commons Atribuição-NãoComercial-CompartilhaIgual 4.0 Internacional (CC BY-NC-SA 4.0)](https://creativecommons.org/licenses/by-nc-sa/4.0/deed.pt-br). Você pode copiar e adaptar o material para fins não comerciais, desde que dê o crédito e distribua as adaptações sob a mesma licença.

---

## 📚 Bibliografia e Referências

* SOMMERVILLE, Ian. *Engenharia de Software*. 10. ed. Pearson, 2019.
* PRESSMAN, Roger S.; MAXIM, Bruce R. *Engenharia de Software: Uma Abordagem Profissional*. 9. ed. AMGH, 2021.
* Manifesto para Desenvolvimento Ágil de Software — [agilemanifesto.org](https://agilemanifesto.org/iso/ptbr/manifesto.html)
* FOWLER, Martin. *Refactoring: Improving the Design of Existing Code*. 2. ed. Addison-Wesley, 2018.
* Documentação oficial: [pytest](https://docs.pytest.org/), [FastAPI](https://fastapi.tiangolo.com/), [GitHub Actions](https://docs.github.com/actions)
