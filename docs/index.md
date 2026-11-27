# Desenvolvimento de Software

Este é um livro aberto sobre como construir, entregar e manter software. Em oito capítulos, ele reúne fundamentos clássicos da engenharia de software, consolidados por autores como Sommerville e Pressman, e as práticas atuais: métodos ágeis, arquitetura, testes automatizados e integração e entrega contínuas (CI/CD).

---

## 🧭 Antes de começar

O livro pressupõe lógica de programação e experiência com pelo menos uma linguagem. Os exemplos usam Python (pytest, FastAPI) e GitHub Actions.

Os capítulos seguem uma ordem progressiva, mas também podem ser consultados separadamente.

---

## 🎯 Estrutura do Livro

### 📘 Parte I: Fundamentos (Capítulos 1-2)
O que diferencia software de outros produtos, por que a engenharia de software nasceu como disciplina e como as equipes organizam o trabalho: cascata, desenvolvimento incremental, Manifesto Ágil, Scrum, Kanban e a cultura DevOps.

### 📙 Parte II: Requisitos e projeto (Capítulos 3-4)
Como descobrir e registrar o que o sistema precisa fazer, modelar o seu comportamento e a sua estrutura e tomar decisões de arquitetura, inclusive as de segurança.

### 📗 Parte III: Construção (Capítulos 5-6)
Código legível e seguro, revisão, controle de versão e testes, dos unitários aos de aceitação, com TDD.

### 📕 Parte IV: Entrega e evolução (Capítulos 7-8)
Pipelines de integração e entrega contínuas, contêineres e a vida do software em produção: manutenção, dependências, débito técnico e observabilidade.

### 🧰 Apêndice
Os comandos de [Git e contêineres](apendices/git_conteineres.md) usados ao longo do livro.

---

## 🧪 O projeto condutor: TaskFlow

Os exemplos giram em torno de um gerenciador de tarefas. Nos Capítulos 6 e 7, ele é uma pequena API em **FastAPI**, com os dados em memória, testada com **pytest** e verificada por um pipeline do **GitHub Actions**. Dela nasce o [TaskFlow](https://github.com/lambdageo-edu/taskflow): a mesma ideia para uma equipe, com banco SQLite, *login*, projetos, membros e permissões. É só a API, sem interface web.

O repositório tem uma etiqueta (*tag*) para cada etapa:

| Tag | Etapa |
|---|---|
| `v1-memoria` | API em memória ([Capítulo 6](capitulos/06_testes_validacao.md)) |
| `v2-pipeline` | pipeline, imagem e verificações de segurança ([Capítulo 7](capitulos/07_integracao_entrega_continua.md)) |
| `v3-sqlite` | persistência |
| `v4-login` | usuários e sessões |
| `v5-equipe` | projetos, membros, responsável e estados da tarefa |

```bash
git clone https://github.com/lambdageo-edu/taskflow.git
cd taskflow
git checkout v3-sqlite   # vê o projeto como estava nesta etapa
```

As etiquetas são retratos do passado e não recebem correções posteriores. Para usar ou modificar o projeto, parta da branch `main`.

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
