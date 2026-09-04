# Debug com watsonx Orchestrate

## Visão Geral

Quando um agente responde de um jeito inesperado, a pergunta que importa não é o que ele respondeu, e sim **por que** ele chegou até ali. 

O modo de debug do **watsonx Orchestrate** registra cada passo da execução, mostra o caminho percorrido no fluxo e expõe os dados brutos que trafegaram entre o usuário, o modelo, os colaboradores e as bases de conhecimento.

Neste laboratório o cenário é **multiagente**:

```
Usuário → Assistente de Compra de Veículos (supervisor)
              └─→ Agente de suporte ao revendedor (colaborador)
                      └─→ Catálogo de Carro com preços (knowledge base)
```

São três camadas de decisão, e o debug é o que permite enxergar todas elas.

**O que você vai fazer:**

- Reproduzir uma interação no Draft Preview e ler o raciocínio direto no chat;
- Abrir a janela de Debug a partir da própria resposta;
- Percorrer a linha do tempo passo a passo, incluindo os passos aninhados no colaborador;
- Ler variáveis de entrada e saída de cada nó e as propriedades de cada agente;
- Interpretar o log em JSON e alternar entre as visualizações do fluxo.

## Índice

- [Debug com watsonx Orchestrate](#debug-com-watsonx-orchestrate)
  - [Visão Geral](#visão-geral)
  - [Índice](#índice)
  - [Passo 1: Reproduza a conversa](#passo-1-reproduza-a-conversa)
  - [Passo 3: Abra a janela de Debug](#passo-3-abra-a-janela-de-debug)
  - [Passo 4: Os controles da barra superior](#passo-4-os-controles-da-barra-superior)
  - [Passo 5: O passo User input](#passo-5-o-passo-user-input)
  - [Passo 6: O passo Agent e a decisão de roteamento](#passo-6-o-passo-agent-e-a-decisão-de-roteamento)
  - [Passo 7: Dentro do colaborador](#passo-7-dentro-do-colaborador)
  - [Passo 8: A volta para o agente orquestrador](#passo-8-a-volta-para-o-agente-orquestrador)
  - [Passo 9: O passo Answer](#passo-9-o-passo-answer)
  - [Passo 10: Ajuste o espaço de trabalho](#passo-10-ajuste-o-espaço-de-trabalho)
  - [Resumo](#resumo)
  - [Sou desenvolvedor e quero me aprofundar no watsonx Orchestrate](#sou-desenvolvedor-e-quero-me-aprofundar-no-watsonx-orchestrate)
  - [Próximos Passos](#próximos-passos)

---

## Passo 1: Reproduza a conversa

Na tela `Build agents and tools`, clique no card **Assistente de Compra de Veículos**.

- O contador `Agents` marca `2` → o agente tem **dois colaboradores** registrados.

![test](../../Assets_for_BuildBooks/labs/lab05/lab05_01.png)

![test](../../Assets_for_BuildBooks/labs/lab05/lab05_03.png)

Leia a resposta

- **Veículo:** Porsche 911 Carrera GTS (ID VEH-004)
- **Valor:** R$ 776.974,10
- **Abaixo da resposta:** 4 ícones → positivo, negativo, copiar e **joaninha** (debug)

![test](../../Assets_for_BuildBooks/labs/lab05/lab05_04.png)

---

Antes de abrir o modo debug, boa parte do diagnóstico já está no próprio chat.

Clique em `Show Reasoning`


![test](../../Assets_for_BuildBooks/labs/lab05/lab05_05.png)

O link vira `Hide Reasoning`

Surgem `Step 1` e `Step 2` → cada um é uma chamada que o agente fez antes de responder

![test](../../Assets_for_BuildBooks/labs/lab05/lab05_06.png)

Expanda o `Step 1` (o roteamento)

| Campo | Conteúdo |
|---|---|
| `Tool` | `chat_with_collaborator_agente_de_suporte_ao_revendedor` |
| `Input` | A pergunta original do usuário, intacta |
| `Output` | `Transferring to - chat_with_collaborator_agente_de_suporte_ao_revendedor` |

o agente orquestrador não tentou responder. Leu a pergunta, reconheceu consulta de catálogo e delegou.

![test](../../Assets_for_BuildBooks/labs/lab05/lab05_07.png)

Expanda o `Step 2` (a busca na base)**

| Campo | Conteúdo |
|---|---|
| `Tool` | `Catálogo_de_Carro_com_preços` |
| `Input` | `{"query": "preço"}` |
| `Output` | 22 linhas, começando pelo `title` do documento `Catalog_with_prices_clean.pdf` |

A busca **não** usou a frase do usuário, e sim uma consulta reescrita pelo agente. Quando a resposta vem incompleta, esse campo `query` é o primeiro suspeito.

![test](../../Assets_for_BuildBooks/labs/lab05/lab05_08.png)

Clique no botão de expansão abaixo da resposta.

- **Arquivo de origem:** `Catalog_with_prices_clean.pdf`
- **Trecho recuperado:** registro do `VEH-004 Porsche 911 Carrera GTS`, com cor e valor
- **`View source`:** abre o documento completo

A prova de que a resposta veio da base, e não de conhecimento próprio do modelo.

![test](../../Assets_for_BuildBooks/labs/lab05/lab05_09.png)

---

## Passo 3: Abra a janela de Debug

O `Show Reasoning` mostra o resumo. Para ver dados brutos, tempos e configurações vigentes, use o debug.

Clique na joaninha (🐞)

Passe o mouse para confirmar o rótulo `Debug`.

![test](../../Assets_for_BuildBooks/labs/lab05/lab05_10.png)

**Entendendo a divisão da tela**

| Área | Conteúdo |
|---|---|
| Esquerda | `Agent flow` — mapa de todos os nós do agente |
| Direita | `Execution timeline`, `Variables` e `Node properties` |

**Estrutura visível no canvas:**

- `Assistente de Compra de Veículos` → modelo
- `Assistente de Compra de Veículos` → colaborador `Agente de Busca`
- `Assistente de Compra de Veículos` → colaborador `Agente de suporte ao revendedor` → base `Catálogo de Carro com preços`
- Nós de `Answer` em cada ramo

**Resumo da execução: 9 passos / 5ms**

| # | Passo | Tempo |
|---|---|---|
| 1 | `User input` | — |
| 2 | `Agent` · `Model invoked` · Agent reasoning | 0.75ms |
| 3 | `Collaborator: Agente de suporte ao revendedor` | **3.15ms** |
| 4 | `Agent` · `Model invoked` · Agent processing | 0.75ms |
| 5 | `Answer` | 0.00ms |

> **Onde investigar primeiro:** os 3.15ms concentrados no colaborador indicam onde está o trabalho pesado.

Abra o seletor `Legends`, no rodapé do canvas.

![test](../../Assets_for_BuildBooks/labs/lab05/lab05_11.png)

**Aprenda a ler o diagrama**

**Tipos de nó:**

| Nó | Papel |
|---|---|
| `User input` | Ponto de entrada da conversa |
| `Agent` | Orquestra as tarefas |
| `LLM` | Chamada ao modelo de linguagem |
| `Tool` | Função externa |
| `API` | Endpoint HTTP ou REST |
| `Knowledge base` | Busca por recuperação |
| `Workflow` | Processo de várias etapas |
| `Answer` | Nó de resposta final |

**Estados do nó:**

- `Not yet executed` — ainda não executado
- `Used in the execution` — participou desta execução
- `Current active node` — nó selecionado agora

**Tipos de conexão:**

- `Agent flow` — ligação estrutural do agente
- `Current taken path` — caminho realmente percorrido nesta execução
- `Not used in this execution` — ligações que ficaram de fora

**Etiquetas:**

- `COLLAB` — marca um agente colaborador
- Ícone de camadas — o nó tem nós filhos

![test](../../Assets_for_BuildBooks/labs/lab05/lab05_12.png)

---

## Passo 4: Os controles da barra superior

**1. Restart (`⌘ + R`)**

Reinicia a leitura do rastreamento e volta a seleção para o primeiro passo.

![test](../../Assets_for_BuildBooks/labs/lab05/lab05_13.png)

**2. `Highlight all nodes used in this run`**

Destaca no diagrama todos os nós que participaram da execução.

![test](../../Assets_for_BuildBooks/labs/lab05/lab05_14.png)

Com o realce ativo:

- **Realçados:** `Agente de suporte ao revendedor`, o modelo e a base `Catálogo de Carro com preços`
- **Apagado:** `Agente de Busca` — existe no agente, mas não foi acionado nesta conversa

![test](../../Assets_for_BuildBooks/labs/lab05/lab05_15.png)

**3 — Visão limpa (terceiro ícone)**

Remove do canvas tudo que não participou da execução. Sobram apenas:

- `User input`
- Supervisor
- Colaborador
- Nós de modelo
- Base de conhecimento
- Nós de `Answer`

> **Quando usar:** em agentes com muitos colaboradores e ferramentas, elimina o ruído e deixa o caminho real evidente.

![test](../../Assets_for_BuildBooks/labs/lab05/lab05_16.png)

**Setas de navegação (`⌘⇧→`)**

Percorrem os passos em ordem, sem precisar clicar na linha do tempo.

![test](../../Assets_for_BuildBooks/labs/lab05/lab05_17.png)

---

## Passo 5: O passo User input

Clique no primeiro passo da linha do tempo. O nó correspondente acende no canvas.

**5.1 — Aba `Input`**

| Campo | Valor |
|---|---|
| `Request` | `qual o carro mais caro que vcs tem?` |
| `Message` | `qual o carro mais caro que vcs tem?` |

> **Por que são iguais:** nesse ponto ainda não houve processamento. O que entrou é exatamente o que o usuário digitou.

![test](../../Assets_for_BuildBooks/labs/lab05/lab05_18.png)

**5.2 — Aba `Output`**

- `No output information available for this step`

> **Esperado:** o nó de entrada só recebe a mensagem e repassa adiante.

![test](../../Assets_for_BuildBooks/labs/lab05/lab05_19.png)

**5.3 — Aba `Node logs` (cabeçalho do span)**

| Campo | Valor | O que indica |
|---|---|---|
| `stepIndex` | `0` | Primeiro passo da execução |
| `spanId` | `__user_input__` | Identificador do nó |
| `operationName` | `user_input` | Tipo de operação |
| `parentSpanId` | `null` | Não tem passo pai |
| `parentAgentId` | `c649b05f-...` | ID do Assistente de Compra de Veículos |
| `depth` | `0` | Nível principal, sem aninhamento |
| `isCollaboratorNode` | `false` | Não é nó de colaborador |
| `collaboratorId` | `null` | Nenhum colaborador envolvido |

> 💡 O botão `Raw`, no canto direito, alterna entre a versão formatada e o JSON puro.

![test](../../Assets_for_BuildBooks/labs/lab05/lab05_20.png)

**5.4 — Clique em `Show more` (corpo do span)**

| Campo | Valor | O que indica |
|---|---|---|
| `duration` | `0` | Passo instantâneo |
| `hasModelCall` | `false` | Nenhuma chamada ao modelo saiu daqui |
| `input` | `{message: ...}` | Mensagem original |
| `output` | `null` | Não produziu saída |
| `status` | `success` | Executou sem erro |
| `graphNodeIds` | `[]` | Não gerou passos filhos |

![test](../../Assets_for_BuildBooks/labs/lab05/lab05_21.png)

---

## Passo 6: O passo Agent e a decisão de roteamento

Avance para o segundo passo: `Agent` · `Model invoked` · **Agent reasoning**. É aqui que o supervisor decide o que fazer com a pergunta.

**6.1 — Aba `Summary` (o campo mais revelador)**

| Campo | Conteúdo |
|---|---|
| `Input request` | A pergunta do usuário |
| `Output response` | O raciocínio do modelo: identifica a pergunta como consulta de catálogo sobre preço e decide rotear para o agente de suporte ao revendedor |

> **Leitura:** este texto é a resposta direta para *"por que ele chamou esse colaborador e não o outro"*.

**No canvas:** o caminho tracejado sai do `User input`, passa pelo supervisor e chega ao nó `groq/openai/gpt-oss`.

**Node properties ganha 4 abas:** `About` · `Collaborators` · `Guidelines` · `LLM Model`

Em `About`:

- `Name`: `Untitled_Agent_1_0690we`
- `Display name`: `Assistente de Compra de Veículos`
- `Description` e `Instructions` vigentes

![test](../../Assets_for_BuildBooks/labs/lab05/lab05_22.png)

**6.2 — Role o painel e leia as instruções na íntegra**

- Bloco `# REGRAS DE ROTEAMENTO`
- Primeira regra: `## 1. CONSULTAS SOBRE CATÁLOGO → Agente de suporte ao revendedor`
- Seguida da lista de situações que devem ser encaminhadas

> **A comparação que importa:** coloque lado a lado o raciocínio do `Summary` e essa regra. Quando o roteamento sai errado, é isso que revela se o problema está na **instrução mal escrita** ou na **interpretação do modelo**.

![test](../../Assets_for_BuildBooks/labs/lab05/lab05_23.png)

**6.3 — Aba `Collaborators`**

Lista os dois agentes registrados: `Agente de Busca` e `Agente de suporte ao revendedor`.

Para o selecionado, mostra:

- `Id`, `Tenant ID`, `Workspace ID`
- `Name` (nome interno), `Description`, `Instructions`

> **Para que serve:** conferir se o colaborador que deveria ter sido chamado está disponível e com que instruções ele opera.

![test](../../Assets_for_BuildBooks/labs/lab05/lab05_24.png)

**6.4 — Campos de auditoria e ambiente**

| Bloco | Campos |
|---|---|
| Auditoria | `Created by`, `Created on`, `Updated by`, `Updated at` |
| `Environments` | Ambiente `Draft`, com `Id`, `Name` e estado da integração |

> **Para que serve:** Checar se alguém alterou o agente entre a execução e a investigação.

![test](../../Assets_for_BuildBooks/labs/lab05/lab05_25.png)

**6.5 — `Agent mapping` e `Additional properties`**

| Bloco | Campos |
|---|---|
| `Agent mapping` | `Hidden`, `Display name` que o supervisor enxerga |
| `Additional properties` | `Context access enabled`, `Hide reasoning`, `Sync tool flow interrupt`, `Restrictions`, `Bundled` |

![test](../../Assets_for_BuildBooks/labs/lab05/lab05_26.png)

**6.6 — `Chat with docs` (configuração de recuperação)**

- `Enabled`
- Suporte a documento completo
- `Vector index`
- `Generation`
- `Query rewrite`
- `Confidence thresholds`
- `Citations`
- `HAP filtering`
- `Query source` — quem monta a consulta
- `Agent query description` — orienta a busca na base

> **Quando o agente busca a coisa errada na base, a explicação costuma estar aqui.**

![test](../../Assets_for_BuildBooks/labs/lab05/lab05_27.png)

**6.7 — Fim do painel**

Últimos interruptores: `Memory enabled` e `Is schedulable`.

> **Dica de diagnóstico:** quando um agente parece ignorar o contexto de mensagens anteriores, é aqui que se descobre que a memória simplesmente não estava ligada.

![test](../../Assets_for_BuildBooks/labs/lab05/lab05_28.png)

![test](../../Assets_for_BuildBooks/labs/lab05/lab05_29.png)

---

## Passo 7: Dentro do colaborador

**7.1 — Expanda o `Collaborator: Agente de suporte ao revendedor`**

Clique na seta à direita do nome. Quatro subpassos aparecem indentados:

| # | Subpasso | Tempo | O que faz |
|---|---|---|---|
| 1 | `Agent` · Agent reasoning | 0.72ms | O colaborador decide o que fazer |
| 2 | `Knowledge: Catálogo de Carro com preços` | 1.33ms | `Searching knowledge base for relevant information` |
| 3 | `Agent` · Agent processing | 0.77ms | Redige a resposta com o que foi recuperado |
| 4 | `Answer` | 0.01ms | Prepara a devolução para o supervisor |

**Node properties do colaborador:**

- `Display name`: `Agente de suporte ao revendedor`
- `Description`: responde perguntas e qualifica vendas para a concessionária
- `Instructions`: abrem com *"Você é um Agente Virtual de Vendas da ABC Automóveis"*

![test](../../Assets_for_BuildBooks/labs/lab05/lab05_30.png)

**7.2 — Role até o fim das propriedades**

| Campo | Valor |
|---|---|
| `Chat with doc` | `Disabled` |
| `Memory enabled` | `Disabled` |
| `Agent style` | `React_intrinsic` |
| `Node type` | **`Collaborator`** |
| `Agent type` | `native` |

![test](../../Assets_for_BuildBooks/labs/lab05/lab05_31.png)

**7.3 — Subpasso 1: `Agent reasoning` do colaborador**

- **No canvas:** acende o nó de modelo pendurado no **colaborador**, não mais o do supervisor
- **Instruções:** trecho final inclui a diretriz de responder somente em Português do Brasil
- **`Node type`:** volta a ser `Agent`

> **Diferença:** o passo anterior era o colaborador como *entidade*; este é o colaborador *em execução*.

![test](../../Assets_for_BuildBooks/labs/lab05/lab05_32.png)

**7.4 — Subpasso 2: `Knowledge: Catálogo de Carro com preços`**

| Campo | Valor |
|---|---|
| `Display name` | `Catálogo de Carro com preços` |
| `Node type` | `Knowledge` |
| `Created by` | `IBMid-691000KKI2` |
| `Updated at` | Data da última atualização da base |

> **Amarrando as pontas:** esses 1.33ms são o momento em que o `{"query": "preço"}` do Passo 2 foi executado contra o índice.

![test](../../Assets_for_BuildBooks/labs/lab05/lab05_33.png)

![test](../../Assets_for_BuildBooks/labs/lab05/lab05_34.png)

**7.5 — Subpasso 3: `Agent processing` (0.77ms)**

- Segunda passagem pelo modelo dentro do colaborador, agora **com o conteúdo recuperado em mãos**
- Instruções mostram o bloco `# FONTE DE CONHECIMENTO` — o trecho que orienta como usar o material recuperado

![test](../../Assets_for_BuildBooks/labs/lab05/lab05_35.png)

![test](../../Assets_for_BuildBooks/labs/lab05/lab05_36.png)

**7.6 — Subpasso 4: `Answer` do colaborador (0.01ms)**

- O nó acende no ramo do colaborador
- `Node type`: `Answer`

> **O que ele faz:** nada de reescrita. Só empacota o que o `Agent processing` produziu e devolve para quem chamou.

![test](../../Assets_for_BuildBooks/labs/lab05/lab05_37.png)

---

## Passo 8: A volta para o agente orquestrador

Volte ao nível principal e selecione o quarto passo: `Agent` · **Agent processing** (0.75ms).

- **No canvas:** o caminho tracejado sai do colaborador e volta para o orquestrador, que passa mais uma vez pelo modelo antes de encerrar
- **Node properties:** voltam a ser as do `Assistente de Compra de Veículos`

![test](../../Assets_for_BuildBooks/labs/lab05/lab05_38.png)

![test](../../Assets_for_BuildBooks/labs/lab05/lab05_39.png)

**8.1 — Aba `Input` — o achado mais importante do laboratório**

O campo `Request` traz:

- `thread_id` da conversa
- A mensagem que **o colaborador devolveu**: Porsche 911 Carrera GTS, o valor, e uma pergunta de follow-up sobre o tipo de carro de interesse

**Compare com o que o usuário viu no chat:**

| Camada | Fecho da mensagem |
|---|---|
| Colaborador | *"Você tem interesse em algum tipo de carro específico, como esportivo, utilitário ou familiar?"* |
| Supervisor (entregue ao usuário) | *"Caso queira saber mais detalhes sobre ele ou comparar com outros modelos, estou à disposição!"* |

> O que chega ao usuário **não é necessariamente** o que o colaborador escreveu. Quando o tom ou o conteúdo final destoam do esperado, é aqui que se descobre em qual camada a alteração aconteceu.

![test](../../Assets_for_BuildBooks/labs/lab05/lab05_40.png)

**8.2 — Contexto da chamada**

| Campo | Valor | O que indica |
|---|---|---|
| `Async flag` | `false` | Execução síncrona |
| `In async execution` | `0` | Nenhuma execução assíncrona em curso |
| `Is Collaborator` | `false` | Quem processa agora é o **agente principal** |
| `Use supervisor interrupt handoff` | `false` | Não houve transferência para supervisor externo |
| `Agent depth` | `1` | Nível do agente na hierarquia |
| `Code act is question` | `false` | — |

![test](../../Assets_for_BuildBooks/labs/lab05/lab05_41.png)

![test](../../Assets_for_BuildBooks/labs/lab05/lab05_42.png)

**8.3 — Node properties (segundo turno do supervisor)**

As mesmas abas do `Agent reasoning`. Em `About`:

| Campo | Valor |
|---|---|
| `Name` | `Untitled_Agent_1_0690we` |
| `Display name` | `Assistente de Compra de Veículos` |
| `Chat with doc` | `Disabled` |
| `Memory enabled` | `Disabled` |
| `Agent style` | `React_intrinsic` |
| `Node type` | `Agent` |

![test](../../Assets_for_BuildBooks/labs/lab05/lab05_43.png)

![test](../../Assets_for_BuildBooks/labs/lab05/lab05_44.png)

**8.4 — Aba `Collaborators`**

A mesma lista de dois agentes, com identificadores, nomes internos, descrições e instruções.

![test](../../Assets_for_BuildBooks/labs/lab05/lab05_45.png)

![test](../../Assets_for_BuildBooks/labs/lab05/lab05_46.png)

**8.5 — Bloco `Collaborators: 1 item`**

| Campo | Valor |
|---|---|
| `LLM` | `groq/openai/gpt-oss-120b` |
| `Style` | `react_intrinsic` |
| `Created by` / `Created on` | Campos de auditoria |

> Um agente colaborador pode rodar em um **modelo diferente** do supervisor. Isso explica diferenças de comportamento entre as camadas.

![test](../../Assets_for_BuildBooks/labs/lab05/lab05_47.png)

![test](../../Assets_for_BuildBooks/labs/lab05/lab05_48.png)

**8.6 — Demais blocos**

Na sequência aparecem: `Environments` (Draft) · `Agent mapping` · `Chat with docs` · `Additional properties` (`Restrictions`, `Bundled`, `Memory enabled`, `Is schedulable`).

![test](../../Assets_for_BuildBooks/labs/lab05/lab05_49.png)

![test](../../Assets_for_BuildBooks/labs/lab05/lab05_50.png)

![test](../../Assets_for_BuildBooks/labs/lab05/lab05_51.png)

![test](../../Assets_for_BuildBooks/labs/lab05/lab05_52.png)

**8.7 — Aba `Output`**

O campo `Response` traz a resposta **já reescrita pelo supervisor**, palavra por palavra igual à que apareceu no Draft Preview.

> **O ciclo se fecha:** entrou o texto do agente colaborador, saiu o texto do usuário.

![test](../../Assets_for_BuildBooks/labs/lab05/lab05_53.png)

![test](../../Assets_for_BuildBooks/labs/lab05/lab05_54.png)

**8.8 — Aba `LLM Model`**

| Campo | Valor |
|---|---|
| `Id` | `groq/openai/gpt-oss-120b` |
| `Label` | `GPT-OSS 120B — OpenAI (via Groq)` |
| `Type` | `Groq` |
| `Tags` | 3 itens |


> O watsonx Orchestrate não faz cobrança por token e sim por **MAUs** (Monthly Active Users).
> Saiba mais sobre MAUs [nesta documentação](https://www.ibm.com/docs/en/watsonx/watson-orchestrate/base?topic=entitlements-licenses-cloud) ou com o time comercial responsável pela sua conta.

![test](../../Assets_for_BuildBooks/labs/lab05/lab05_55.png)

---

## Passo 9: O passo Answer

Selecione o último passo da linha do tempo: `Answer` (0.00ms). O nó acende no fim do caminho tracejado.

**9.1 — Aba `Summary`**

- `Input request` traz **exatamente** o mesmo texto que o usuário recebeu

> **Ponto central:** o nó de resposta não reescreve nada. Ele entrega o que o passo anterior produziu.

![test](../../Assets_for_BuildBooks/labs/lab05/lab05_56.png)

![test](../../Assets_for_BuildBooks/labs/lab05/lab05_57.png)

**9.2 — Aba `Input`**

- `Request` completo
- Os mesmos indicadores de execução: `Async flag`, `In async execution`, `Is Collaborator`, `Use supervisor interrupt handoff`, `Agent depth`, `Code act is question`

![test](../../Assets_for_BuildBooks/labs/lab05/lab05_58.png)

![test](../../Assets_for_BuildBooks/labs/lab05/lab05_59.png)

![test](../../Assets_for_BuildBooks/labs/lab05/lab05_60.png)

**9.3 — Aba `Output`**

- `Response` repete a resposta final, a mesma vista no Draft Preview

![test](../../Assets_for_BuildBooks/labs/lab05/lab05_61.png)

![test](../../Assets_for_BuildBooks/labs/lab05/lab05_62.png)

**9.4 — Aba `Node logs`**

| Campo | Valor |
|---|---|
| `stepIndex` | `7` |
| `displayIndex` | `8` |
| `spanId` | `5067dd8b7a8d4bcf` |
| `operationName` | `answer` |
| `parentSpanId` | `null` |

> **Compare com o `stepIndex 0` do `User input`:** a numeração percorre **todos** os passos, incluindo os aninhados dentro do colaborador.

![test](../../Assets_for_BuildBooks/labs/lab05/lab05_63.png)

**9.5 — `Show more`**

Em execuções que passam por base de conhecimento, o span carrega também os **vetores de embedding** usados na recuperação — longas listas de números em ponto flutuante.

- Raramente úteis na leitura manual
- Explicam por que o span fica tão extenso
- Use o botão `Raw` para copiar o JSON puro e analisar fora da ferramenta

---

## Passo 10: Ajuste o espaço de trabalho

Com spans longos, a tela padrão fica apertada. Dois controles resolvem isso.

**10.1 — Divisor horizontal (as reticências)**

Fica entre a linha do tempo e o bloco `Variables`. Arraste para cima.

![test](../../Assets_for_BuildBooks/labs/lab05/lab05_64.png)

Resultado:

- A linha do tempo encolhe e ganha barra de rolagem própria
- O `Variables` passa a exibir muito mais conteúdo de uma vez

![test](../../Assets_for_BuildBooks/labs/lab05/lab05_65.png)

**10.2 — Alça vertical (o círculo no meio da divisão)**

Fica entre o `Agent flow` e o rastreamento.

![test](../../Assets_for_BuildBooks/labs/lab05/lab05_66.png)

Resultado:

- Amplia o painel direito e reduz o canvas
- **Use quando** a leitura dos logs for mais importante que o diagrama

> Para encerrar, o `X` no canto superior direito fecha a debug e devolve você à aba `Build`.

![test](../../Assets_for_BuildBooks/labs/lab05/lab05_67.png)

---

## Resumo

Parabéns!

Você concluiu o laboratório de debug de agentes no watsonx Orchestrate.

**O que você fez:**

- Reproduziu uma resposta no Draft Preview em modo de debug
- Leu o raciocínio direto no chat pelo `Show Reasoning`
- Abriu o rastreamento pelo ícone de inseto
- Percorreu os 9 passos, incluindo os aninhados dentro do colaborador
- Leu variáveis de entrada e saída de cada nó
- Comparou o texto do colaborador com o texto final entregue ao usuário
- Interpretou os spans em JSON e alternou entre as visualizações do fluxo

**O método de investigação em arquitetura multiagente:**

| # | Verificação | Onde olhar |
|---|---|---|
| 1 | O roteamento foi para o colaborador esperado? | Canvas com o realce ativado |
| 2 | Por que o supervisor escolheu esse caminho? | `Output response` do `Agent reasoning` × regras de roteamento |
| 3 | A busca na base trouxe o conteúdo certo? | Campo `query` e a citação do documento |
| 4 | A resposta foi alterada na volta? | `Input` do último passo do supervisor × `Output` final |
| 5 | O modelo e as configurações eram os esperados? | Abas `LLM Model` e `Node properties` de cada camada |

Na maioria dos casos o problema aparece em um desses cinco pontos.

---

## Sou desenvolvedor e quero me aprofundar no watsonx Orchestrate

Todas as operações realizadas também estão disponíveis em uma experiência utilizando o ADK, o Agent Development Kit. [Clique aqui](https://developer.watson-orchestrate.ibm.com/) para saber mais sobre como criar agentes, tools, bases de conhecimento e muito mais.

## Próximos Passos

➜ [Clique aqui para navegar para o próximo lab, Monitoramento em Tempo Real e Control Plane do watsonx Orchestrate](./Step_by_Step_Lab6.md)
