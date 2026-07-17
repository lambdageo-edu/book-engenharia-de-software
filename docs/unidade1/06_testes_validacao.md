# Testes e Validação

## 🧭 Defeito, erro e falha

Antes de falar em testes, vale distinguir três termos que costumam ser usados de forma intercambiável, mas que têm significados técnicos distintos:

* **Defeito (*fault*/*bug*)**: uma característica estática e incorreta do código-fonte — uma linha escrita errada, uma condição invertida.
* **Erro (*error*)**: um estado interno incorreto do sistema em tempo de execução, causado por um defeito ao ser exercitado.
* **Falha (*failure*)**: o comportamento observável e incorreto do sistema, visível para o usuário ou para outro sistema — a consequência externa de um erro.

Um defeito pode existir no código por anos sem nunca causar uma falha, se o caminho de código que o contém nunca for exercitado com as condições certas. Testes automatizados existem justamente para aumentar a chance de descobrir defeitos **antes** que eles se tornem falhas em produção.

## 🔬 Inspeções e revisões

Nem todo defeito é encontrado executando o código. **Inspeções** e **revisões** (Capítulo 5) são técnicas de verificação estática — analisar o código sem executá-lo — e são complementares aos testes: uma inspeção pode encontrar um problema de lógica que um teste mal escrito não cobriria, e vice-versa. Times maduros combinam as duas abordagens: revisão de código para design e lógica, análise estática (Capítulo 5) para padrões mecânicos, e testes automatizados para comportamento em tempo de execução.

## 🔺 A pirâmide de testes

A **pirâmide de testes**, popularizada por Mike Cohn, organiza os testes automatizados em camadas conforme seu escopo, velocidade e custo de manutenção:

```
        /\
       /  \      Testes de E2E / Aceitação
      /----\     (poucos, lentos, alto valor de confiança)
     /      \
    /--------\   Testes de Integração
   /          \  (moderados, verificam componentes juntos)
  /------------\
 /              \ Testes Unitários
/________________\ (muitos, rápidos, isolados)
```

* **Testes unitários** verificam uma unidade de código isolada (uma função, um método), sem dependências externas reais (banco de dados, rede). São rápidos, numerosos e baratos de manter — a base da pirâmide.
* **Testes de componente/integração** verificam a colaboração entre algumas unidades reais, ou entre a aplicação e uma dependência externa real (um banco de dados de teste, por exemplo).
* **Testes de sistema/aceitação** verificam o sistema completo, muitas vezes através de suas interfaces externas (API, UI), simulando um uso real de ponta a ponta. São os mais caros de escrever e manter, e os mais lentos de executar — por isso devem ser relativamente poucos, cobrindo os fluxos mais críticos.

!!! warning "O antipadrão do 'sorvete invertido'"
    Um erro comum é ter muitos testes de ponta a ponta (lentos, frágeis, caros de manter) e poucos testes unitários — o oposto da pirâmide. Isso é às vezes chamado de "cone de sorvete invertido": a suíte de testes fica lenta, instável e cara de manter, exatamente o oposto do que se espera de uma boa suíte de testes.

## 🧪 Tipos de teste no ciclo de desenvolvimento

Além da divisão por escopo, testes também se distinguem por **quem** os executa e **quando**, ao longo do ciclo de vida:

* **Testes em desenvolvimento**: escritos e executados pelos próprios desenvolvedores, continuamente, enquanto o código é escrito — testes unitários, de componente e de sistema.
* **Testes de release**: executados por um time (ou processo automatizado) separado, validando uma versão candidata a ser lançada antes de ir para produção.
* **Testes de usuário**: envolvem usuários reais ou representantes do cliente, tipicamente perto do fim do ciclo:
    * **Teste alfa**: usuários selecionados testam o sistema no ambiente de desenvolvimento, com a equipe observando.
    * **Teste beta**: uma versão é liberada a um grupo maior de usuários reais, em seu próprio ambiente, coletando feedback antes do lançamento geral.
    * **Teste de aceitação**: o cliente (ou seu representante) valida formalmente que o sistema atende aos requisitos acordados — muitas vezes usando os próprios critérios de aceitação escritos junto com os requisitos (Capítulo 3).

## 🔴🟢🔵 Test-Driven Development (TDD)

TDD é uma prática (não apenas uma técnica de teste) em que o teste é escrito **antes** do código de produção, seguindo um ciclo curto e repetitivo:

1. **🔴 Red**: escreva um teste para uma funcionalidade que ainda não existe. Ele deve falhar (o código ainda não existe para satisfazê-lo).
2. **🟢 Green**: escreva o código mínimo necessário para o teste passar — sem se preocupar ainda com elegância.
3. **🔵 Refactor**: com o teste passando como rede de segurança, melhore o código (Capítulo 5) sem mudar seu comportamento externo.

A promessa do TDD não é "ter mais testes" — é que escrever o teste primeiro força a pensar na interface e no comportamento esperado antes dos detalhes de implementação, e garante que todo código de produção tenha, desde o nascimento, um teste que o cubra.

---

## 🧑‍💻 Hands-on: testando uma API com pytest e FastAPI

Vamos aplicar os conceitos acima construindo e testando uma pequena **API de tarefas** com [FastAPI](https://fastapi.tiangolo.com/) e [pytest](https://docs.pytest.org/), a dupla mais usada hoje no ecossistema Python para esse tipo de projeto.

### Preparando o ambiente

```bash
mkdir api-tarefas && cd api-tarefas
python -m venv .venv
source .venv/bin/activate   # no Windows: .venv\Scripts\activate

pip install fastapi "uvicorn[standard]" pytest httpx pytest-cov
```

!!! note "Por que `httpx`?"
    O `TestClient` do FastAPI (usado nos testes abaixo) é construído sobre o `httpx`, uma biblioteca de cliente HTTP moderna — é ela que permite simular requisições à API sem precisar de um servidor rodando de verdade durante os testes.

### A aplicação: `app/main.py`

```python
from fastapi import FastAPI, HTTPException
from pydantic import BaseModel

app = FastAPI(title="API de Tarefas")


class TarefaEntrada(BaseModel):
    titulo: str
    concluida: bool = False


class Tarefa(TarefaEntrada):
    id: int


_tarefas: dict[int, Tarefa] = {}
_proximo_id = 1


@app.post("/tarefas", response_model=Tarefa, status_code=201)
def criar_tarefa(dados: TarefaEntrada) -> Tarefa:
    global _proximo_id
    if not dados.titulo.strip():
        raise HTTPException(status_code=422, detail="Título não pode ser vazio")
    tarefa = Tarefa(id=_proximo_id, **dados.model_dump())
    _tarefas[tarefa.id] = tarefa
    _proximo_id += 1
    return tarefa


@app.get("/tarefas", response_model=list[Tarefa])
def listar_tarefas() -> list[Tarefa]:
    return list(_tarefas.values())


@app.get("/tarefas/{tarefa_id}", response_model=Tarefa)
def obter_tarefa(tarefa_id: int) -> Tarefa:
    tarefa = _tarefas.get(tarefa_id)
    if tarefa is None:
        raise HTTPException(status_code=404, detail="Tarefa não encontrada")
    return tarefa


@app.patch("/tarefas/{tarefa_id}/concluir", response_model=Tarefa)
def concluir_tarefa(tarefa_id: int) -> Tarefa:
    tarefa = _tarefas.get(tarefa_id)
    if tarefa is None:
        raise HTTPException(status_code=404, detail="Tarefa não encontrada")
    tarefa.concluida = True
    return tarefa
```

Não deixe de criar um `app/__init__.py` vazio para que `app` seja reconhecido como um pacote Python.

### Os testes: `test_main.py`

```python
from fastapi.testclient import TestClient

from app.main import app

client = TestClient(app)


def test_criar_tarefa_retorna_201_e_id():
    resposta = client.post("/tarefas", json={"titulo": "Estudar pytest"})
    assert resposta.status_code == 201
    corpo = resposta.json()
    assert corpo["titulo"] == "Estudar pytest"
    assert corpo["concluida"] is False
    assert "id" in corpo


def test_criar_tarefa_com_titulo_vazio_retorna_422():
    resposta = client.post("/tarefas", json={"titulo": "   "})
    assert resposta.status_code == 422


def test_listar_tarefas_inclui_tarefa_criada():
    client.post("/tarefas", json={"titulo": "Ler capítulo 6"})
    resposta = client.get("/tarefas")
    assert resposta.status_code == 200
    titulos = [t["titulo"] for t in resposta.json()]
    assert "Ler capítulo 6" in titulos


def test_obter_tarefa_inexistente_retorna_404():
    resposta = client.get("/tarefas/9999")
    assert resposta.status_code == 404


def test_concluir_tarefa_muda_status():
    criada = client.post("/tarefas", json={"titulo": "Testar conclusão"}).json()
    resposta = client.patch(f"/tarefas/{criada['id']}/concluir")
    assert resposta.status_code == 200
    assert resposta.json()["concluida"] is True
```

Repare que cada teste segue implicitamente o padrão **Arrange-Act-Assert**: prepara os dados de entrada, executa a ação sob teste, e verifica o resultado — uma estrutura que vale a pena manter explícita mesmo quando o código não a nomeia.

### Executando e medindo cobertura

```bash
pytest -v --cov=app --cov-report=term-missing
```

Saída esperada (validada neste material):

```
test_main.py::test_criar_tarefa_retorna_201_e_id PASSED
test_main.py::test_criar_tarefa_com_titulo_vazio_retorna_422 PASSED
test_main.py::test_listar_tarefas_inclui_tarefa_criada PASSED
test_main.py::test_obter_tarefa_inexistente_retorna_404 PASSED
test_main.py::test_concluir_tarefa_muda_status PASSED

Name              Stmts   Miss  Cover   Missing
-----------------------------------------------
app/main.py          34      2    94%   41, 48
-----------------------------------------------
TOTAL                34      2    94%

5 passed in 1.12s
```

94% de cobertura é um bom resultado, mas repare que o relatório aponta exatamente **quais** linhas não foram exercitadas (`Missing: 41, 48`) — nesse caso, os caminhos de sucesso de `obter_tarefa` e `concluir_tarefa` quando a tarefa **existe**, que não estão diretamente cobertos por um teste isolado (embora sejam exercitados indiretamente por outros testes). Esse é o valor real de medir cobertura: não é uma meta a perseguir por si só, mas um mapa de quais partes do comportamento do sistema ainda não têm uma rede de segurança.

!!! danger "Cobertura alta não é sinônimo de bons testes"
    100% de cobertura significa apenas que toda linha foi **executada** por algum teste — não que o comportamento foi **verificado** corretamente. É possível ter cobertura total com testes que não fazem nenhuma asserção significativa. Use cobertura para encontrar código não testado, nunca como meta isolada de qualidade.

### Praticando TDD nesta API

Como exercício de TDD, tente adicionar um endpoint `DELETE /tarefas/{tarefa_id}` seguindo o ciclo Red-Green-Refactor:

1. **Red**: escreva `test_deletar_tarefa_remove_da_lista`, chamando um endpoint que ainda não existe.
2. **Green**: implemente o `DELETE /tarefas/{tarefa_id}` da forma mais simples possível para o teste passar.
3. **Refactor**: revise o código à luz das práticas de clean code do Capítulo 5.

---

## 🎯 Atividades

1. Rode a suíte de testes deste capítulo localmente e confirme os mesmos resultados de cobertura.
2. Complete o exercício de TDD acima (endpoint `DELETE`) e verifique se a cobertura de `app/main.py` sobe.
3. Escreva um teste que verifique explicitamente o caso de sucesso de `obter_tarefa` (uma tarefa que existe) — a linha que faltava na cobertura do exemplo.
4. Classifique um bug real que você já enfrentou (em qualquer projeto) em termos de defeito, erro e falha, conforme a distinção do início deste capítulo.

## 📚 Para saber mais

* SOMMERVILLE, Ian. *Engenharia de Software*. 10. ed. — Capítulo 8.
* COHN, Mike. *Succeeding with Agile*. Addison-Wesley, 2009 (origem da pirâmide de testes).
* BECK, Kent. *Test-Driven Development: By Example*. Addison-Wesley, 2002.
* Documentação oficial: [pytest](https://docs.pytest.org/), [FastAPI Testing](https://fastapi.tiangolo.com/tutorial/testing/)
