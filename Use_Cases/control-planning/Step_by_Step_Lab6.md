# Control Plane Lab do watsonx Orchestrate

## Visão Geral

Este laboratório apresenta o Agentic Control Plane do watsonx Orchestrate, um conjunto de painéis e ferramentas que reúne, em um só lugar, a visão de adoção, custos, qualidade, confiabilidade e segurança dos seus agentes.

Ao longo das atividades, você vai navegar pelas diferentes abas do dashboard, aprender a depurar uma conversa específica usando o Debug, e criar um Controle de Content Guardrails para bloquear conteúdo impróprio nas interações de um agente.

Conhecer essas ferramentas é essencial para operar agentes de IA com confiança, permitindo identificar rapidamente pontos de atenção, investigar comportamentos inesperados e aplicar salvaguardas antes que um agente seja exposto a usuários reais.

## Índice

- [Control Plane Lab do watsonx Orchestrate](#control-plane-lab-do-watsonx-orchestrate)
  - [Visão Geral](#visão-geral)
  - [Índice](#índice)
  - [Explorando o Dashboard do Control Plane](#explorando-o-dashboard-do-control-plane)
    - [Overview: visão geral do ambiente](#overview-visão-geral-do-ambiente)
    - [Adoption: engajamento e uso dos agentes](#adoption-engajamento-e-uso-dos-agentes)
    - [FinOps: consumo de tokens](#finops-consumo-de-tokens)
    - [Quality: qualidade das respostas](#quality-qualidade-das-respostas)
    - [Reliability: confiabilidade e desempenho](#reliability-confiabilidade-e-desempenho)
    - [Security and Risk: controles de segurança](#security-and-risk-controles-de-segurança)
  - [Depurando uma Conversa com o Debug](#depurando-uma-conversa-com-o-debug)
    - [Avaliando o agente na aba Evaluate](#avaliando-o-agente-na-aba-evaluate)
    - [Analisando conversas na aba Analyze](#analisando-conversas-na-aba-analyze)
    - [Abrindo o Debug de uma conversa](#abrindo-o-debug-de-uma-conversa)
  - [Criando um Controle de Content Guardrails](#criando-um-controle-de-content-guardrails)
    - [Selecionando o tipo de controle](#selecionando-o-tipo-de-controle)
    - [Configurando o controle](#configurando-o-controle)
    - [Atribuindo o controle a um agente](#atribuindo-o-controle-a-um-agente)
    - [Revisando e criando o controle](#revisando-e-criando-o-controle)
  - [Testando o Controle Criado](#testando-o-controle-criado)
  - [Resumo](#resumo)
    - [Sou desenvolvedor e quero me aprofundar no watsonx Orchestrate](#sou-desenvolvedor-e-quero-me-aprofundar-no-watsonx-orchestrate)
  - [Próximos Passos](#próximos-passos)

## Explorando o Dashboard do Control Plane

Ao acessar o ambiente da sua instância do watsonx Orchestrate, você chega diretamente ao dashboard do Agentic Control Plane, com uma saudação personalizada e um resumo de quantos agentes estão em produção (Live) e quantos usuários interagiram com eles nos últimos 30 dias.

![Overview dashboard](../../Assets_for_BuildBooks/labs/lab06/lab06_01.png)

Clique no ícone circular de IA, no canto inferior esquerdo, para abrir o assistente do painel de controle. Ele oferece sugestões prontas, como identificar pontos de atenção de performance, lacunas de cobertura de testes e investigar feedback negativo, além de um campo para perguntas livres em linguagem natural.

![Assistente do dashboard](../../Assets_for_BuildBooks/labs/lab06/lab06_02.png)

### Overview: visão geral do ambiente

A aba **Overview**, selecionada por padrão, reúne seis cartões de resumo: Messages, Feedback, Deployment status, Evaluation status, Agents e Controls, cada um com a taxa de sucesso ou a distribuição relevante da última semana. À direita, o painel **Needs attention** já aponta o que precisa da sua atenção, agrupado por categoria — Evaluation, Adoption, Credentials, Execution e Quality — sem que você precise procurar manualmente.

![Métricas da aba Overview](../../Assets_for_BuildBooks/labs/lab06/lab06_03.png)

Role a página para baixo para ver as seções **Usage trends** e **Operational trends**, com gráficos de usuários ativos, agentes ativos, mensagens, uso de tokens, taxa de falha, conversas com erro e latência P50 ao longo dos últimos sete dias.

![Usage e Operational trends](../../Assets_for_BuildBooks/labs/lab06/lab06_04.png)

### Adoption: engajamento e uso dos agentes

Volte ao topo da página e clique na aba **Adoption**.

![Navegando para Adoption](../../Assets_for_BuildBooks/labs/lab06/lab06_05.png)

A seção **Engagement depth** mostra a relação entre usuários, conversas e chamadas de modelo, como usuários por agente, conversas por usuário, mensagens por conversa e chamadas de LLM por conversa. Logo abaixo, a tabela **Agent analytics** detalha, por agente, o número de conversas, usuários únicos, duração média, tokens consumidos, chamadas de LLM, taxa de erro e latência P95.

![Engagement depth e Agent analytics](../../Assets_for_BuildBooks/labs/lab06/lab06_06.png)

A tabela também traz uma legenda de cores para interpretar rapidamente a taxa de erro e a latência de cada agente, além de paginação para instâncias com muitos agentes.

![Legenda da tabela de agentes](../../Assets_for_BuildBooks/labs/lab06/lab06_07.png)

Role a página para ver o gráfico **Adoption trends**, que compara conversas, usuários ativos e agentes ativos ao longo do tempo, e o painel **Model usage distribution**, que mostra quantos modelos estão em uso e quantos agentes utilizam cada um deles.

![Adoption trends e Model usage distribution](../../Assets_for_BuildBooks/labs/lab06/lab06_08.png)

### FinOps: consumo de tokens

Volte ao topo e clique na aba **FinOps**.

![Navegando para FinOps](../../Assets_for_BuildBooks/labs/lab06/lab06_09.png)

O **Token summary** resume o total de tokens consumidos na semana, com a divisão entre tokens de entrada e saída e o número de chamadas de LLM. Logo abaixo, **Token usage** permite alternar a visão **By agent** ou **By model** e traz um gráfico de rosca com a distribuição percentual de tokens entre os agentes, complementado por uma tabela detalhada.

![Token summary e Token usage](../../Assets_for_BuildBooks/labs/lab06/lab06_10.png)

Role a página para ver **Token trends**, com pílulas de alternância para Total, Input e Output tokens ao longo dos últimos sete dias.

![Token trends](../../Assets_for_BuildBooks/labs/lab06/lab06_11.png)

### Quality: qualidade das respostas

Volte ao topo e clique na aba **Quality**.

![Navegando para Quality](../../Assets_for_BuildBooks/labs/lab06/lab06_12.png)

A seção **Insights** mostra quantos agentes já possuem avaliações configuradas, o total de feedback de usuários (positivo e negativo) e as métricas de Helpfulness score e Hallucination score. A tabela **Agent feedback** detalha, por agente, o total de mensagens, mensagens com falha, feedback positivo e negativo, e a proporção de feedback positivo.

![Insights e Agent feedback](../../Assets_for_BuildBooks/labs/lab06/lab06_13.png)

Role a página para ver três painéis lado a lado — **Top agents by positive feedback**, **Top agents by negative feedback** e **Tool call success** — seguidos pelo gráfico **Feedback trends**, que permite alternar entre mensagens totais, bem-sucedidas, com falha, feedback positivo e negativo.

![Rankings de feedback e Tool call success](../../Assets_for_BuildBooks/labs/lab06/lab06_14.png)

### Reliability: confiabilidade e desempenho

Volte ao topo e clique na aba **Reliability**.

![Navegando para Reliability](../../Assets_for_BuildBooks/labs/lab06/lab06_15.png)

A seção **Utilization** mostra, na janela de sete dias, quantos modelos estão ativos, a média de mensagens por conversa, mensagens por agente ativo e quantos agentes estão sob carga com falhas de trace. A tabela **Agent latency** detalha, por agente, mensagens, mensagens com falha, taxa de erro e as latências P50, P95 e P99.

![Utilization e Agent latency](../../Assets_for_BuildBooks/labs/lab06/lab06_16.png)

Role a página para ver **Deployment readiness** (total de chamadas de ferramenta, conversas com falha e latência P95), **Runtime inventory** (contagem de agentes, toolkits, tools e bases de conhecimento) e os gráficos **Latency trends** e **Failed messages over time**.

![Deployment readiness, Runtime inventory e Latency trends](../../Assets_for_BuildBooks/labs/lab06/lab06_17.png)

O painel **Runtime inventory** dá um retrato rápido de tudo o que está publicado na sua instância: agentes, toolkits, tools e bases de conhecimento.

![Runtime inventory em destaque](../../Assets_for_BuildBooks/labs/lab06/lab06_18.png)

Já o gráfico **Latency trends** permite alternar entre os percentis P50, P95 e P99 para identificar picos de latência ao longo da semana.

![Latency trends em destaque](../../Assets_for_BuildBooks/labs/lab06/lab06_19.png)

### Security and Risk: controles de segurança

Volte ao topo e clique na aba **Security and Risk**.

![Navegando para Security and Risk](../../Assets_for_BuildBooks/labs/lab06/lab06_20.png)

Essa aba resume o painel **Controls summary**, com o total de controles configurados na instância e sua divisão entre Agent controls, Tool controls e Model controls, além da lista **Recent controls**. Como você ainda não criou nenhum controle, todos os contadores aparecem zerados — isso muda na última parte deste laboratório, quando você criar seu primeiro controle.

![Security and Risk sem controles](../../Assets_for_BuildBooks/labs/lab06/lab06_21.png)

> [!NOTE]
> Ao voltar para a aba Overview, um tour guiado pode aparecer sugerindo navegar pelas demais abas usando a seta `>` ao final da barra de abas. Sinta-se à vontade para explorar as abas adicionais por conta própria.

![Tour guiado no dashboard](../../Assets_for_BuildBooks/labs/lab06/lab06_22.png)

## Depurando uma Conversa com o Debug

Agora que você já conhece o dashboard, vamos investigar uma conversa específica de um agente. Clique no ícone de menu hambúrguer, no canto superior esquerdo, para abrir a navegação lateral.

![Abrindo o menu lateral](../../Assets_for_BuildBooks/labs/lab06/lab06_23.png)

No menu, selecione **Build** para acessar a área de construção de agentes e ferramentas.

![Menu lateral com Build em destaque](../../Assets_for_BuildBooks/labs/lab06/lab06_24.png)

Na página **Build agents and tools**, clique no agente **Car Sales Assistant**.

![Página Build agents and tools](../../Assets_for_BuildBooks/labs/lab06/lab06_25.png)

Na aba **Behavior** do agente, observe as instruções configuradas. Neste ambiente de laboratório, o agente foi propositalmente configurado com uma instrução inadequada sobre o modelo "Kia Nero", que usaremos mais adiante para testar o controle de conteúdo. Clique em **Evaluate**, no menu superior, para seguir para a próxima etapa.

![Aba Behavior do Car Sales Assistant](../../Assets_for_BuildBooks/labs/lab06/lab06_26.png)

### Avaliando o agente na aba Evaluate

Na sub-aba **Evaluations**, como nenhuma avaliação foi executada ainda, a lista aparece vazia, com a opção de iniciar uma nova avaliação.

![Aba Evaluate vazia](../../Assets_for_BuildBooks/labs/lab06/lab06_27.png)

Clique na sub-aba **Tests**. Como nenhum teste foi salvo ainda, o watsonx Orchestrate sugere ter uma conversa com o agente e clicar em `Save as test` para criar o primeiro caso de teste.

![Sub-aba Tests vazia](../../Assets_for_BuildBooks/labs/lab06/lab06_28.png)

### Analisando conversas na aba Analyze

Abra novamente o menu lateral e selecione **Analyze**.

![Menu lateral com Analyze em destaque](../../Assets_for_BuildBooks/labs/lab06/lab06_29.png)

A página **Analytics** mostra o total de conversas, usuários únicos e a duração média das conversas no período selecionado, o gráfico **Agent trend**, comparando o volume de conversas entre os agentes, e a tabela **User Feedback**, com a contagem de feedback positivo e negativo por agente.

![Página Analytics](../../Assets_for_BuildBooks/labs/lab06/lab06_30.png)

Esses dados dão uma visão consolidada de como os agentes estão sendo usados e recebidos pelos usuários.

![Visão consolidada da página Analytics](../../Assets_for_BuildBooks/labs/lab06/lab06_31.png)

Clique no nome **Car Sales Assistant**, na tabela de feedback, para abrir os detalhes desse agente.

![Selecionando o Car Sales Assistant na tabela](../../Assets_for_BuildBooks/labs/lab06/lab06_32.png)

Na aba **Overview** do agente, você encontra métricas mais específicas: contagem de tokens de entrada e saída, duração média de conversação, latência média por mensagem, o gráfico **Usage trend**, o donut de **User feedback** e o painel **Evaluation**, que inclui a métrica de toxicidade (ainda em Preview).

![Overview do agente Car Sales Assistant](../../Assets_for_BuildBooks/labs/lab06/lab06_33.png)

Clique na aba **Conversations** para ver a lista de conversas individuais. Selecione uma conversa para acompanhar a troca de mensagens completa entre o usuário e o agente, junto com o painel **Details**, que traz o identificador da conversa, o identificador do usuário, o horário de início e o total de feedbacks recebidos.

![Aba Conversations com uma conversa aberta](../../Assets_for_BuildBooks/labs/lab06/lab06_34.png)

### Abrindo o Debug de uma conversa

Abaixo de uma das respostas do agente, clique no ícone de engrenagem para abrir a ferramenta de depuração daquela etapa específica da conversa.

![Ícone de debug em uma resposta](../../Assets_for_BuildBooks/labs/lab06/lab06_35.png)

A janela **Debug** se abre em tela cheia, dividida em duas áreas principais. À esquerda, o **Agent flow** exibe um diagrama com os nós percorridos durante a conversa — agentes, modelos de LLM, bases de conhecimento e respostas — conectados na ordem em que foram acionados. No topo, o **Trace ID** identifica de forma única aquela execução, com navegação entre execuções anteriores e seguintes pelos botões `Previous` e `Next`. Logo abaixo do diagrama, a **Execution timeline** lista, passo a passo, cada etapa da execução — entrada do usuário, raciocínio do agente, chamadas a colaboradores e a resposta final — com o tempo gasto em cada uma. Na parte inferior, o painel **Variables** alterna entre as abas `Summary`, `Input`, `Output` e `Node logs`, enquanto **Node properties** detalha as propriedades do nó selecionado.

![Visão geral anotada da janela Debug](../../Assets_for_BuildBooks/labs/lab06/lab06_36.png)

Use os ícones de layout na barra de ferramentas, acima do diagrama, para alternar entre diferentes formas de visualizar o fluxo do agente.

![Alternando o layout do Agent flow](../../Assets_for_BuildBooks/labs/lab06/lab06_37.png)

Um segundo ícone de layout apresenta o fluxo de forma mais linear e simplificada, facilitando o acompanhamento sequencial das etapas.

![Layout linear do Agent flow](../../Assets_for_BuildBooks/labs/lab06/lab06_38.png)

![Layout linear do Agent flow confirmado](../../Assets_for_BuildBooks/labs/lab06/lab06_39.png)

Clique na etapa **User input**, na Execution timeline, e depois na aba `Input` do painel Variables para ver os dados brutos enviados pelo usuário naquele passo — os campos Request e Message trazem o texto exato da pergunta.

![Aba Input com os dados do usuário](../../Assets_for_BuildBooks/labs/lab06/lab06_40.png)

Clique na etapa **Agent** da Execution timeline e depois na aba `About`, dentro de **Node properties**, para ver os detalhes técnicos do agente responsável por aquele passo, como o nome interno, o nome de exibição e a descrição configurada.

![Aba About do Node properties](../../Assets_for_BuildBooks/labs/lab06/lab06_41.png)

Clique na aba `LLM Model` para ver qual modelo de linguagem foi usado naquele passo, incluindo seu identificador, rótulo, descrição e o provedor responsável por hospedá-lo.

![Aba LLM Model do Node properties](../../Assets_for_BuildBooks/labs/lab06/lab06_42.png)

## Criando um Controle de Content Guardrails

Com a janela de Debug ainda aberta, abra o menu lateral e, em **Manage**, selecione **Controls** para acessar a área de criação de controles.

![Navegando para Manage > Controls](../../Assets_for_BuildBooks/labs/lab06/lab06_43.png)

Como nenhum controle foi criado ainda, a página **Controls** aparece vazia, com uma mensagem de boas-vindas explicando que os controles ajudam a impor regras sobre o comportamento de agentes, modelos e ferramentas MCP. Clique em `Create Control`.

![Página Controls vazia](../../Assets_for_BuildBooks/labs/lab06/lab06_44.png)

### Selecionando o tipo de controle

A janela **Create Control** se abre com um assistente de quatro etapas: `Select Control`, `Configure Control`, `Assign Assets` e `Review`. Na primeira etapa, os controles disponíveis são organizados por tipo de ativo — **Agents** (Content Guardrails, Output Length Guard, Regex Pattern, Secrets Detector e PII Filter) e **Tools** (Content Guardrails, Output Length Guard, Rate Limiter, SQLSanitizer e Secrets Detector).

![Etapa Select Control](../../Assets_for_BuildBooks/labs/lab06/lab06_45.png)

Selecione **Content Guardrails**, na seção Agents. Essa opção aplica um serviço externo de detecção de conteúdo para identificar conteúdo sexual, violência, discurso de ódio, conteúdo prejudicial, tentativas de jailbreak e viés social. Clique em `Next`.

![Content Guardrails selecionado](../../Assets_for_BuildBooks/labs/lab06/lab06_46.png)

### Configurando o controle

Na etapa **Configure Control**, dê um nome ao controle no campo `Control instance name`, como no exemplo abaixo. Em `Enforcement type`, marque `Input`, para que o controle analise as mensagens enviadas pelos usuários antes de chegarem ao agente.

```
Controle_de_palavras_de_baixo_calão
```

![Nome e tipo de enforcement do controle](../../Assets_for_BuildBooks/labs/lab06/lab06_47.png)

Role para baixo até a seção `Toggle detection for each content type`. Por padrão, todos os tipos de conteúdo — Sexual Content, Violence, HAP (Hate, Abuse and Profanity), Harm, Jailbreak e Social Bias — vêm desativados (`Off`).

![Tipos de conteúdo desativados](../../Assets_for_BuildBooks/labs/lab06/lab06_48.png)

Ative todos os toggles, deixando-os em `On`, e revise o campo `Block message`, que já vem preenchido com uma mensagem padrão explicando ao usuário por que o conteúdo foi bloqueado. Você pode personalizar esse texto como preferir, ou usar o texto sugerido abaixo. Clique em `Next`.

```
Esse conteúdo não é apropriado para esta conversa. Peço que mantenhamos uma comunicação respeitosa e construtiva. Estou aqui para ajudar da melhor forma possível e fornecer suporte adequado às suas necessidades.
```

![Todos os tipos de conteúdo ativados e Block message preenchido](../../Assets_for_BuildBooks/labs/lab06/lab06_49.png)

### Atribuindo o controle a um agente

Na etapa **Assign Assets**, clique em `Add Agent` para escolher a quais agentes esse controle será aplicado.

![Etapa Assign Assets](../../Assets_for_BuildBooks/labs/lab06/lab06_50.png)

Na janela **Add Agent**, marque a caixa de seleção ao lado de um agente criado por você — no exemplo, `Assistente de Compra de Veiculos` — e clique em `Select`.

![Selecionando o agente na janela Add Agent](../../Assets_for_BuildBooks/labs/lab06/lab06_51.png)

O agente selecionado aparece na tabela, junto com sua descrição. Clique em `Next`.

![Agente atribuído ao controle](../../Assets_for_BuildBooks/labs/lab06/lab06_52.png)

### Revisando e criando o controle

A etapa **Review** resume toda a configuração: o tipo de controle, o nome da instância, o tipo de ativo, o hook configurado (`Input`) e, logo abaixo, os detalhes de configuração com cada tipo de conteúdo habilitado. Revise as informações e clique em `Create control`.

![Etapa Review](../../Assets_for_BuildBooks/labs/lab06/lab06_53.png)

Uma notificação confirma que o controle foi criado com sucesso. Na página **Asset Controls**, o total de controles e o número de agentes com controles passam a mostrar `1`, e o novo controle aparece listado, com a informação de que está aplicado a `1 agent`.

![Controle criado com sucesso](../../Assets_for_BuildBooks/labs/lab06/lab06_54.png)

## Testando o Controle Criado

Abra o menu lateral novamente e selecione **Build** para voltar à área de agentes.

![Menu lateral com Build em destaque](../../Assets_for_BuildBooks/labs/lab06/lab06_55.png)

Na página **Build agents and tools**, clique no agente que você acabou de vincular ao controle.

![Selecionando o agente com o controle aplicado](../../Assets_for_BuildBooks/labs/lab06/lab06_56.png)

No painel **Preview**, à direita, envie uma mensagem com palavras de baixo calão para testar o controle. O agente não chega a processar o conteúdo ofensivo: a resposta é bloqueada antes mesmo de chegar ao modelo, e o agente retorna a mensagem de bloqueio configurada na etapa de configuração, pedindo que a comunicação seja mantida respeitosa e construtiva.

![Controle bloqueando conteúdo impróprio](../../Assets_for_BuildBooks/labs/lab06/lab06_57.png)

## Resumo

Parabéns! 🎉 Você concluiu o Control Plane Lab do watsonx Orchestrate.

Ao longo deste laboratório, você navegou pelas seis abas do dashboard do Agentic Control Plane — Overview, Adoption, FinOps, Quality, Reliability e Security and Risk —, entendendo como cada uma resume um aspecto diferente da operação dos seus agentes: uso geral, engajamento, custo de tokens, qualidade das respostas, confiabilidade e segurança. Em seguida, você usou a ferramenta Debug para investigar, passo a passo, o fluxo de execução de uma conversa real, inspecionando o agente, o modelo de linguagem utilizado e os dados de entrada e saída de cada etapa. Por fim, você criou um Controle de Content Guardrails, configurou os tipos de conteúdo a serem bloqueados, atribuiu o controle a um agente e testou seu funcionamento na prática.

Com isso, você agora sabe onde encontrar as principais métricas de adoção, custo, qualidade, confiabilidade e segurança dos seus agentes, como depurar uma conversa até o nível de cada chamada de modelo, e como aplicar controles de segurança para proteger seus agentes contra conteúdo impróprio antes que ele chegue aos usuários.

### Sou desenvolvedor e quero me aprofundar no watsonx Orchestrate

Todas as operações realizadas também estão disponíveis em uma experiência utilizando o ADK, o Agent Development Kit. [Clique aqui](https://developer.watson-orchestrate.ibm.com/) para saber mais sobre como criar agentes, tools, bases de conhecimento e muito mais.

## Próximos Passos

Este é o último laboratório desta série. Abaixo está uma coletânea de links oficiais, documentação, tutoriais e novidades da IBM watsonx Orchestrate e do Agent Development Kit (ADK) para você continuar se aprofundando.

> Última atualização da coletânea: 24/08/2026.

| Recurso | Link |
|---|---|
| Portal oficial da ADK (Welcome) | https://developer.watson-orchestrate.ibm.com/ |
| O que é a ADK | https://developer.watson-orchestrate.ibm.com/getting_started/what_is |
| Instalação / primeiros passos | https://developer.watson-orchestrate.ibm.com/getting_started/installing |
| Comandos CLI úteis | https://developer.watson-orchestrate.ibm.com/getting_started/cli |
| Exemplos | https://developer.watson-orchestrate.ibm.com/getting_started/examples |
| Índice completo da doc (`llms.txt`) | https://developer.watson-orchestrate.ibm.com/llms.txt |

---

<b>Repositórios e pacotes</b>

| Recurso | Link |
|---|---|
| GitHub — `IBM/ibm-watsonx-orchestrate-adk` | https://github.com/IBM/ibm-watsonx-orchestrate-adk |
| PyPI — `ibm-watsonx-orchestrate` | https://pypi.org/project/ibm-watsonx-orchestrate/ |

---

<b>Agentes</b>

| Tópico | Link |
|---|---|
| Visão geral de agentes | https://developer.watson-orchestrate.ibm.com/agents/overview |
| Criando agentes nativos | https://developer.watson-orchestrate.ibm.com/agents/build_agent |
| Escolhendo o estilo do agente (ReAct, Flows, etc.) | https://developer.watson-orchestrate.ibm.com/agents/agent_styles |
| Conectar agentes externos (LangGraph, A2A) | https://developer.watson-orchestrate.ibm.com/agents/connect_agent |
| Importar e implantar agentes | https://developer.watson-orchestrate.ibm.com/agents/import_agent |
| Descrições e instruções de agentes | https://developer.watson-orchestrate.ibm.com/agents/descriptions |
| Gerenciar agentes | https://developer.watson-orchestrate.ibm.com/agents/manage_agent |
| Skills specifications (Public Preview) | https://developer.watson-orchestrate.ibm.com/agents/skills |

---

<b>Ferramentas (Tools) e Toolkits</b>

| Tópico | Link |
|---|---|
| Visão geral de tools | https://developer.watson-orchestrate.ibm.com/tools/overview |
| Tools em Python | https://developer.watson-orchestrate.ibm.com/tools/create_tool |
| Tools baseadas em OpenAPI | https://developer.watson-orchestrate.ibm.com/tools/create_openapi_tool |
| Gerenciamento de dependências Python | https://developer.watson-orchestrate.ibm.com/tools/python_dependency_management |
| Estrutura de resposta e anotações de tools | https://developer.watson-orchestrate.ibm.com/tools/tool_response_structure |
| Logs de tools Python | https://developer.watson-orchestrate.ibm.com/tools/log_behavior |
| Toolkits — visão geral | https://developer.watson-orchestrate.ibm.com/tools/toolkits/overview |
| Toolkits MCP remotos | https://developer.watson-orchestrate.ibm.com/tools/toolkits/remote_mcp_toolkits |
| Toolkits MCP locais | https://developer.watson-orchestrate.ibm.com/tools/toolkits/local_mcp_toolkits |
| Toolkits Python | https://developer.watson-orchestrate.ibm.com/tools/toolkits/python_toolkits |

---

<b>Agentic Workflows (Flows)</b>

| Tópico | Link |
|---|---|
| Entendendo agentic workflows | https://developer.watson-orchestrate.ibm.com/tools/flows/overview |
| Construindo um flow | https://developer.watson-orchestrate.ibm.com/tools/flows/building_flow |
| Mapeamento de entrada/saída | https://developer.watson-orchestrate.ibm.com/tools/flows/data_map |
| Nó de decisões (Public Preview) | https://developer.watson-orchestrate.ibm.com/tools/flows/decisions_node |
| Tratamento de erros | https://developer.watson-orchestrate.ibm.com/tools/flows/error_handling |
| Atividades multiusuário | https://developer.watson-orchestrate.ibm.com/tools/flows/multi_user |
| Mascaramento de dados sensíveis | https://developer.watson-orchestrate.ibm.com/tools/flows/masking_sensitive_data |
| Scheduler (Public Preview) | https://developer.watson-orchestrate.ibm.com/tools/flows/scheduler |
| Rodar workflows via MCP (Public Preview) | https://developer.watson-orchestrate.ibm.com/tools/flows/mcp_workflows |
| Testando um flow | https://developer.watson-orchestrate.ibm.com/tools/flows/testing_flow |

---

<b>MCP Server (Model Context Protocol)</b>

| Tópico | Link |
|---|---|
| Instalando o ADK MCP Server | https://developer.watson-orchestrate.ibm.com/mcp_server/wxOmcp_installation |
| Configurando o ADK MCP Server | https://developer.watson-orchestrate.ibm.com/mcp_server/wxOmcp_configuration |
| Conectando clientes MCP | https://developer.watson-orchestrate.ibm.com/mcp_server/wxOmcp_integration |
| Documentation MCP Server | https://developer.watson-orchestrate.ibm.com/mcp_server/wxOmcp_docs_server |

---

<b>Developer Edition (ambiente local)</b>

| Tópico | Link |
|---|---|
| O que é a Developer Edition | https://developer.watson-orchestrate.ibm.com/developer_edition/wxOde_overview |
| Instalando a Developer Edition | https://developer.watson-orchestrate.ibm.com/developer_edition/wxOde_setup |
| Gerenciando o servidor local | https://developer.watson-orchestrate.ibm.com/developer_edition/manage_local_server |
| Gerenciando a UI local | https://developer.watson-orchestrate.ibm.com/developer_edition/manage_ui |
| Ambiente air-gapped | https://developer.watson-orchestrate.ibm.com/environment/air_gap_environment |
| Boletins de segurança | https://developer.watson-orchestrate.ibm.com/developer_edition/security_bulletins |

<b>Modelos (LLMs)</b>

| Tópico | Link |
|---|---|
| Escolhendo o LLM | https://developer.watson-orchestrate.ibm.com/llm/getting_started_llm |
| Gerenciando modelos virtuais | https://developer.watson-orchestrate.ibm.com/llm/managing_llm |
| Políticas de modelo (Public Preview) | https://developer.watson-orchestrate.ibm.com/llm/model_policies |
| Observabilidade com Langfuse | https://developer.watson-orchestrate.ibm.com/llm/observability |

---

<b>SDK (Python)</b>

| Tópico | Link |
|---|---|
| Introdução ao SDK | https://developer.watson-orchestrate.ibm.com/sdk/sdk_intro |
| Client | https://developer.watson-orchestrate.ibm.com/sdk/client |
| Context | https://developer.watson-orchestrate.ibm.com/sdk/context |
| Memory | https://developer.watson-orchestrate.ibm.com/sdk/memory |
| Chat models | https://developer.watson-orchestrate.ibm.com/sdk/chat_wxo |
| Embeddings | https://developer.watson-orchestrate.ibm.com/sdk/wxo_embeddings |

---

<b>Avaliação (Evaluation) e Observabilidade</b>

| Tópico | Link |
|---|---|
| Visão geral de avaliação | https://developer.watson-orchestrate.ibm.com/evaluate/overview |
| Avaliando agentes e tools | https://developer.watson-orchestrate.ibm.com/evaluate/evaluate |
| Criando dataset de avaliação | https://developer.watson-orchestrate.ibm.com/evaluate/create_data |
| Avaliação rápida | https://developer.watson-orchestrate.ibm.com/evaluate/quick_eval |
| Rubric Evaluations | https://developer.watson-orchestrate.ibm.com/evaluate/rubric |
| Teste de vulnerabilidade de LLM | https://developer.watson-orchestrate.ibm.com/evaluate/llm_vulnerability |
| Traces — visão geral | https://developer.watson-orchestrate.ibm.com/traces/overview |
| Traces via CLI | https://developer.watson-orchestrate.ibm.com/traces/traces_with_cli |
| Traces via Python | https://developer.watson-orchestrate.ibm.com/traces/traces_with_python |

---

<b>Knowledge Bases, Conexões e Canais</b>

| Tópico | Link |
|---|---|
| Knowledge bases — visão geral | https://developer.watson-orchestrate.ibm.com/knowledge_base/overview |
| Criando knowledge bases | https://developer.watson-orchestrate.ibm.com/knowledge_base/build_kb |
| Por que usar conexões | https://developer.watson-orchestrate.ibm.com/connections/overview |
| Criando conexões | https://developer.watson-orchestrate.ibm.com/connections/build_connections |
| Canais — visão geral (Teams, Slack, WhatsApp, SMS...) | https://developer.watson-orchestrate.ibm.com/channels/overview |
| Adicionando voz a um agente | https://developer.watson-orchestrate.ibm.com/voice/adding_voice_to_agent |
| Configuração de voz | https://developer.watson-orchestrate.ibm.com/voice/managing_voice |
| Plug-ins | https://developer.watson-orchestrate.ibm.com/plugins/plugins |
| Workspaces | https://developer.watson-orchestrate.ibm.com/workspaces/overview |

---

<b>Tutoriais e Guias</b>

| Tutorial | Link |
|---|---|
| Seu primeiro agente (Hello World) | https://developer.watson-orchestrate.ibm.com/tutorials/tutorial_1_hello_world |
| Agente Empower | https://developer.watson-orchestrate.ibm.com/tutorials/tutorial_2_arrows_internal_employees |
| Suporte multilíngue | https://developer.watson-orchestrate.ibm.com/tutorials/tutorial_3_multi_language_support |
| Agente Healthcare Provider | https://developer.watson-orchestrate.ibm.com/tutorials/tutorial_4_healthcare_provider |
| Usando watsonx Orchestrate + BeeAI Framework | https://developer.watson-orchestrate.ibm.com/tutorials/tutorial_beeai_framework |
| Usando watsonx Orchestrate + Langflow | https://developer.watson-orchestrate.ibm.com/tutorials/tutorial_langflow |
| CI/CD — Parte 1 | https://developer.watson-orchestrate.ibm.com/tutorials/ci_cd/deployment-cicd-approach-1 |
| CI/CD — Parte 2 | https://developer.watson-orchestrate.ibm.com/tutorials/ci_cd/deployment-cicd-approach-2 |
| CI/CD — Parte 3 | https://developer.watson-orchestrate.ibm.com/tutorials/ci_cd/deployment-cicd-approach-3 |
| CI/CD — Parte 4 | https://developer.watson-orchestrate.ibm.com/tutorials/ci_cd/deployment-cicd-approach-4 |
| Guia de Performance (geral) | https://developer.watson-orchestrate.ibm.com/tutorials/performance/performance-guide-v2 |
| Tutoriais avançados | https://developer.watson-orchestrate.ibm.com/tutorials/advanced_tutorials |
| Getting Started (IBM Developer) | https://developer.ibm.com/tutorials/getting-started-with-watsonx-orchestrate/ |

---

<b>Documentação oficial do produto</b>

| Recurso | Link |
|---|---|
| Página de desenvolvedores do watsonx Orchestrate | https://www.ibm.com/products/watsonx-orchestrate/developers |
| Construindo agentes com a ADK | https://www.ibm.com/docs/en/watsonx/watson-orchestrate/base?topic=agents-building-using-adk |
| Release notes (SaaS) | https://www.ibm.com/docs/en/watsonx/watson-orchestrate/base?topic=release-notes |
| Release notes (On-premises) | https://www.ibm.com/docs/en/watsonx/watson-orchestrate/base?topic=notes-release-premises |

---

<b>Novidades / "Coisas novas" (2026)</b>

| Item | Link |
|---|---|
| Troubleshooting | https://developer.watson-orchestrate.ibm.com/release/troubleshooting |
| **Agentic Control Plane** (anúncio, jun/2026) | https://www.ibm.com/new/announcements/introducing-the-agentic-control-plane |
| What's new — comunidade (mar/2026) | https://community.ibm.com/community/user/blogs/daiane-camila-bizari2/2026/04/02/whats-new-in-ibm-watsonx-orchestrate |
| "Orchestrate More, Worry Less" (mar/2026) | https://community.ibm.com/community/user/blogs/alan-francis-cheeramvelil/2026/04/12/orchestrate-more-worry-less-whats-new-in-ibm-watso |
| Newsletter técnica (jun/2026) | https://community.ibm.com/community/user/blogs/gustavo-villegas/2026/05/28/watsonx-orchestrate-news-a-touchpoint-june2026 |

> Dica: consulte sempre a página **What's new** para ver a versão mais recente, já que a ADK é atualizada com frequência.
