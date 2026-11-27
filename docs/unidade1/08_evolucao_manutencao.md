# Evolução, Manutenção e Débito Técnico

## ♻️ O software não termina no deploy

Um erro comum de percepção — inclusive fora da equipe de engenharia — é tratar o lançamento de um software como o "fim" do projeto. Na prática, para qualquer sistema usado por um tempo relevante, a fase de **operação e manutenção** costuma consumir mais tempo, esforço e orçamento do que o desenvolvimento inicial. Um sistema em uso real está sujeito a mudanças constantes: o negócio muda, a legislação muda, novos concorrentes aparecem, a infraestrutura sobre a qual ele roda evolui — e o software precisa acompanhar, ou se torna um passivo em vez de um ativo.

## 📨 Solicitações de mudança

Toda mudança em um sistema em produção começa, formal ou informalmente, como uma **solicitação de mudança**: um bug reportado por um usuário, uma nova funcionalidade pedida pelo time de produto, uma atualização de segurança exigida por uma dependência vulnerável. Um processo saudável de gestão de mudanças garante que cada solicitação seja:

* **Registrada e rastreável** — normalmente em uma ferramenta de gestão de trabalho (Jira, Linear, GitHub Issues), nunca apenas em uma conversa informal que se perde.
* **Avaliada quanto a impacto e risco** — uma mudança em um componente crítico, com muitas dependências, merece mais cautela do que uma mudança isolada.
* **Priorizada junto com o restante do trabalho do time** — manutenção compete por tempo com novas funcionalidades, e ignorar essa competição sistematicamente é uma das causas mais comuns de acúmulo de débito técnico.

## 🔧 Tipos de manutenção

A classificação clássica de tipos de manutenção de software, ainda amplamente usada, distingue quatro categorias pela **motivação** da mudança:

| Tipo | Motivação | Exemplo |
|---|---|---|
| **Corretiva** | Corrigir defeitos descobertos em produção | Corrigir um cálculo de frete incorreto |
| **Adaptativa** | Adaptar o sistema a mudanças no ambiente | Atualizar integração após mudança na API de um parceiro |
| **Evolutiva (perfectiva)** | Atender a novos requisitos do negócio | Adicionar um novo método de pagamento |
| **Preventiva** | Melhorar a manutenibilidade futura, sem mudar comportamento externo | Refatorar um módulo antigo, atualizar uma dependência desatualizada |

Historicamente, a manutenção corretiva costumava ser tratada como prioridade única — "primeiro apagamos incêndio, depois pensamos em melhorar". A prática moderna, especialmente em times que seguem princípios de DevOps (Capítulo 2), tende a dar peso deliberado também à manutenção preventiva: tratá-la como investimento contínuo, e não como um projeto especial que só acontece quando o sistema já está insustentável.

## 💳 Débito técnico

O termo **débito técnico**, cunhado por Ward Cunningham, é uma metáfora poderosa: assim como uma dívida financeira, tomar um atalho técnico (pular testes, copiar e colar código em vez de abstrair, adiar uma refatoração necessária) pode ser uma decisão racional de curto prazo — mas, como qualquer dívida, acumula "juros": cada mudança futura naquele código fica mais lenta e mais arriscada até que a dívida seja paga (refatorada) ou o custo dos juros se torne insustentável.

Débito técnico não é, por si só, um mal absoluto — times de alto desempenho tomam débito técnico deliberadamente o tempo todo, como uma escolha de compromisso consciente (ex: lançar rápido uma funcionalidade experimental, aceitando que uma versão mais robusta virá depois se a experiência validar a ideia de negócio). O problema não é a dívida em si; é a dívida **não intencional e não rastreada** — código que acumula complexidade silenciosamente, sem que ninguém decida conscientemente aceitar esse custo.

!!! tip "O quadrante do débito técnico"
    Martin Fowler cruza duas perguntas: a dívida foi assumida de propósito (**deliberada** ou **inadvertida**)? E houve cuidado na decisão (**prudente** ou **imprudente**)? O resultado são quatro situações:

    | | Prudente | Imprudente |
    |---|---|---|
    | **Deliberada** | "Sabemos que não é o ideal, mas o prazo justifica, e vamos revisitar." | "Não há tempo para pensar em projeto." |
    | **Inadvertida** | "Só agora entendemos como deveríamos ter feito." | "O que é separação em camadas?" |

    A dívida deliberada e prudente é uma decisão de engenharia legítima, desde que o "revisitar" aconteça. A inadvertida e prudente é parte natural do aprendizado, mesmo em boas equipes. As duas imprudentes são as que mais acumulam juros: uma por descuido assumido, outra por desconhecimento.

## 🔨 Refatoração

**Refatoração** (*refactoring*), no sentido formalizado por Martin Fowler, é o processo de melhorar a estrutura interna do código **sem alterar seu comportamento externo observável** — uma distinção importante em relação a simplesmente "reescrever" um componente, que costuma envolver risco muito maior.

Refatoração segura depende diretamente da suíte de testes automatizados (Capítulo 6): sem testes que confirmem que o comportamento não mudou, refatoração vira uma aposta, não uma prática disciplinada. Por isso, débito técnico e cobertura de testes estão intimamente ligados — um dos investimentos mais eficazes contra o acúmulo de débito técnico é justamente garantir que exista uma rede de segurança de testes antes de mexer em código antigo e frágil.

Sinais comuns de que um trecho de código pede refatoração (os chamados *code smells*, também catalogados por Fowler):

* **Código duplicado** em múltiplos lugares;
* **Métodos ou classes longos demais**, fazendo muitas coisas ao mesmo tempo;
* **Nomes que não revelam intenção** (Capítulo 5);
* **Acoplamento excessivo** entre módulos que deveriam ser independentes;
* **Comentários que compensam código confuso**, em vez de explicar decisões não óbvias.

## 🩹 Dependências e vulnerabilidades

Um sistema que não muda fica mais vulnerável com o tempo, porque falhas continuam sendo descobertas nas bibliotecas que ele usa. Atualizar dependências é manutenção preventiva, e funciona melhor em passos pequenos e frequentes do que num salto anual.

O [Dependabot](https://docs.github.com/en/code-security/dependabot/working-with-dependabot/dependabot-options-reference), do GitHub, abre *pull requests* com as versões novas, e o pipeline do [Capítulo 7](07_integracao_entrega_continua.md) diz se algo quebrou. Para a API de tarefas, o arquivo `.github/dependabot.yml` seria:

```yaml
version: 2
updates:
  - package-ecosystem: "pip"
    directory: "/"
    schedule:
      interval: "weekly"
  - package-ecosystem: "github-actions"
    directory: "/"
    schedule:
      interval: "weekly"
  - package-ecosystem: "docker"
    directory: "/"
    schedule:
      interval: "weekly"
```

Esse arquivo configura atualizações de versão. Os alertas e as correções de segurança do Dependabot são ativados nas configurações do repositório. Imagens citadas dentro de comandos `run`, como as do Gitleaks e do Trivy no exemplo, não são atualizadas por ele: revise-as à mão.

### Quando uma vulnerabilidade aparece

1. **Confirmar**: o alerta se aplica à versão que você usa?
2. **Avaliar**: o trecho vulnerável é alcançado? Está exposto? Que dados estão em risco?
3. **Corrigir ou mitigar**: atualizar, desativar a funcionalidade ou restringir o acesso.
4. **Verificar e publicar**: testes, pipeline e implantação.
5. **Aprender**: registrar o ocorrido e o que evitaria a repetição.

Decida antes quem recebe o aviso e em quanto tempo responde. Em projetos abertos, um arquivo `SECURITY.md` diz como relatar uma falha sem expô-la numa *issue* pública. O [SSDF, do NIST](https://csrc.nist.gov/pubs/sp/800/218/final), reúne práticas de desenvolvimento seguro que incluem identificar e responder a vulnerabilidades.

## 🤖 Débito técnico em código gerado por IA

Gerar código ficou rápido; entendê-lo e mantê-lo, não necessariamente. Quando um trecho entra no sistema sem que ninguém do time consiga explicá-lo, nasce uma dívida que não aparece em nenhum quadro. Alguns sinais: soluções diferentes para o mesmo problema em pontos diferentes, abstrações que ninguém pediu e testes que apenas repetem a implementação.

O relatório DORA de 2025 descreve a IA como um [amplificador do que a organização já tem](https://cloud.google.com/blog/products/ai-machine-learning/announcing-the-2025-dora-report): associa seu uso a mais vazão de entregas, mas também a menos estabilidade, e aponta testes automatizados, controle de versão maduro e retorno rápido como proteção. Na prática: mudanças pequenas, revisão por quem vai manter, e refatoração como parte do trabalho, não como promessa.

## 🔭 Observabilidade: entendendo um sistema vivo

Manter e evoluir um sistema em produção com segurança depende de conseguir **observar** seu comportamento real — não apenas assumir que ele funciona como planejado. Três pilares, hoje tratados como parte essencial da manutenção de qualquer sistema relevante:

* **Logs estruturados**: registros de eventos relevantes, em formato pesquisável (tipicamente JSON), em vez de texto livre difícil de filtrar em escala.
* **Métricas**: séries temporais numéricas (taxa de erro, latência, uso de recursos) coletadas continuamente, tipicamente visualizadas em painéis (Grafana, Datadog) e usadas para alertar a equipe automaticamente quando algo sai do esperado.
* **Tracing distribuído**: rastreamento do caminho completo de uma requisição através de múltiplos serviços (especialmente relevante em arquiteturas de microsserviços — Capítulo 4), essencial para diagnosticar problemas de desempenho ou falhas que atravessam fronteiras de serviço.

Sem observabilidade adequada, manutenção corretiva vira um exercício de tentativa e erro: descobrir o que está errado em produção depende de conseguir enxergar, com dados reais, o que o sistema está de fato fazendo.

---

## 🎯 Atividades

1. Classifique três mudanças recentes em um projeto seu (ou de algum software que você usa e acompanha) segundo os quatro tipos de manutenção (corretiva, adaptativa, evolutiva, preventiva).
2. Descreva um exemplo de débito técnico "deliberado e prudente" que você tomaria conscientemente em um projeto real, explicando a troca de curto prazo por longo prazo envolvida.
3. Escolha um trecho de código com pelo menos um *code smell* (duplicação, método longo, nome ruim) e refatore-o, garantindo que os testes existentes (ou novos testes que você escreva antes) continuem passando depois da mudança.
4. Configure o Dependabot em um repositório seu e analise o primeiro *pull request* que ele abrir: o que mudou na dependência e como você sabe que é seguro aceitar?

## 📚 Para saber mais

* SOMMERVILLE, Ian. *Engenharia de Software*. 10. ed. — Capítulo 9.
* NIST SP 800-218. *Secure Software Development Framework (SSDF)*, versão 1.1, 2022 — [csrc.nist.gov](https://csrc.nist.gov/pubs/sp/800/218/final)
* GitHub, referência de opções do Dependabot — [docs.github.com](https://docs.github.com/en/code-security/dependabot/working-with-dependabot/dependabot-options-reference)
* Google Cloud. *Announcing the 2025 DORA Report* — [cloud.google.com](https://cloud.google.com/blog/products/ai-machine-learning/announcing-the-2025-dora-report)
* FOWLER, Martin. *Refactoring: Improving the Design of Existing Code*. 2. ed. Addison-Wesley, 2018.
* CUNNINGHAM, Ward. *The WyCash Portfolio Management System* (artigo original sobre débito técnico, 1992).
* Artigo *TechnicalDebtQuadrant* — [martinfowler.com/bliki/TechnicalDebtQuadrant.html](https://martinfowler.com/bliki/TechnicalDebtQuadrant.html)
