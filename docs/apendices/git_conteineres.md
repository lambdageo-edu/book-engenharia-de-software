# Apêndice: Git e contêineres na prática

Os capítulos deste livro supõem que você versiona o código com Git e, a partir do Capítulo 7, que sabe construir uma imagem. Este apêndice reúne o mínimo para acompanhar, sem substituir a documentação.

## 🌱 Antes do primeiro commit

O Git guarda a história do projeto como uma sequência de *commits*, cada um com autor, data e mensagem (CHACON; STRAUB, 2014). Configure sua identidade uma vez e crie o repositório:

```bash
git config --global user.name "Seu Nome"
git config --global user.email "voce@exemplo.com"

mkdir api-tarefas && cd api-tarefas
git init -b main
```

Antes de adicionar arquivos, crie o `.gitignore`. Ele lista o que **não** deve ser versionado: ambiente virtual, arquivos gerados e, principalmente, segredos.

```text
.venv/
__pycache__/
.coverage
.env
```

!!! warning "O `.gitignore` não apaga o passado"
    Ele só vale para arquivos que o Git ainda não rastreia. Um arquivo já commitado continua sendo versionado.

## 🔁 O ciclo básico

```bash
echo '# API de tarefas' > README.md

git status                 # o que mudou?
git add README.md .gitignore
git diff --staged          # revise exatamente o que vai entrar
git commit -m "Adiciona README e .gitignore"
git log --oneline          # a história até aqui
```

Prefira commits pequenos, com **um propósito** cada. A mensagem deve dizer o que a mudança faz; o porquê, quando não for óbvio, vai no corpo da mensagem.

## 🌿 Branches e conflitos

Uma *branch* é uma linha de trabalho independente. Crie uma para cada mudança e mantenha a `main` sempre funcionando.

```bash
git switch -c titulo-do-projeto
echo '# API de tarefas da equipe' > README.md
git commit -am "Ajusta o título"

git switch main
echo '# Gerenciador de tarefas' > README.md
git commit -am "Renomeia o projeto"

git merge titulo-do-projeto
# CONFLICT (content): Merge conflict in README.md
```

As duas branches mudaram a mesma linha, e o Git não escolhe por você. O arquivo fica assim:

```text
<<<<<<< HEAD
# Gerenciador de tarefas
=======
# API de tarefas da equipe
>>>>>>> titulo-do-projeto
```

Entre `<<<<<<<` e `=======` está a versão da branch atual; depois, a da outra. Edite o arquivo até ele ficar como deve, **sem os marcadores**, e conclua:

```bash
echo '# Gerenciador de tarefas da equipe' > README.md   # ou use o editor
git add README.md
git commit -m "Resolve o conflito no título"
```

Se preferir desistir no meio do caminho, `git merge --abort` volta ao estado anterior ao *merge*. Depois de resolver, rode os testes: um *merge* sem conflito de texto ainda pode quebrar o comportamento.

## 🤝 Pull request

Um *pull request* é uma proposta de incorporar as mudanças de uma branch em outra, aberta a discussão, revisão e verificações automáticas antes do *merge* (GITHUB, *About pull requests*). Com o repositório criado no GitHub:

```bash
git remote add origin https://github.com/SEU_USUARIO/api-tarefas.git
git push -u origin main

git switch -c prioridade-da-tarefa
# ... edite, teste e faça commits ...
git push -u origin prioridade-da-tarefa
```

Abra o *pull request* pela interface do GitHub. Um bom *pull request*:

* é pequeno o bastante para ser revisado com atenção;
* explica o que muda, por quê e como foi testado;
* só é incorporado depois que o [pipeline](../capitulos/07_integracao_entrega_continua.md) passa e alguém revisa — inclusive o código escrito com ajuda de IA.

## 🔑 Commitei um segredo. E agora?

1. **Revogue ou troque o segredo.** É o primeiro passo, antes de mexer no histórico.
2. Só então avalie reescrever a história. Isso sozinho não basta: o commit pode continuar em clones, *forks*, visualizações em cache e *pull requests* (GITHUB, *Removing sensitive data*).

Apagar o arquivo em um commit novo não remove o segredo dos commits anteriores. Por isso o [Capítulo 7](../capitulos/07_integracao_entrega_continua.md) procura segredos no histórico inteiro.

## 📦 Contêineres: o mínimo

Uma **imagem** é o pacote somente leitura com a aplicação e suas dependências; um **contêiner** é uma execução dessa imagem. Com o `Dockerfile` do [Capítulo 7](../capitulos/07_integracao_entrega_continua.md):

```bash
docker build -t api-tarefas .
docker run -d --name api -p 127.0.0.1:8000:8000 api-tarefas
docker logs api            # primeira parada quando algo falha
docker stop api && docker rm api
```

O `127.0.0.1` publica a porta só para a sua máquina: a API deste livro não tem HTTPS.

O que é gravado dentro do contêiner se perde quando ele é removido; dados que precisam sobreviver vão para um *volume*.

Para praticar passo a passo (rede, volumes, Docker Compose e otimização de imagens), siga os [tutoriais de Docker da coleção LambdaGeo](https://lambdageo-edu.github.io/lambdageo-tutorials/docker/).

## 🎯 Atividades

1. Reproduza o conflito acima e resolva-o com o editor.
2. Abra um *pull request* em um repositório seu e peça a um colega para revisar.
3. Grave um token **falso** em um arquivo, faça commit, apague o arquivo em outro commit e encontre o token com `git log -p`.

## 📚 Para saber mais

* CHACON, Scott; STRAUB, Ben. *Pro Git*. 2. ed. Apress, 2014 — [git-scm.com/book/pt-br/v2](https://git-scm.com/book/pt-br/v2)
* GitHub. *About pull requests* — [docs.github.com](https://docs.github.com/en/pull-requests/get-started/about-pull-requests)
* GitHub. *Removing sensitive data from a repository* — [docs.github.com](https://docs.github.com/en/authentication/keeping-your-account-and-data-secure/removing-sensitive-data-from-a-repository)
* Documentação oficial do Docker — [docs.docker.com](https://docs.docker.com/)
