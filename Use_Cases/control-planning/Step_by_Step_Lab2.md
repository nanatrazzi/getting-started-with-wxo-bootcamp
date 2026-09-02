# Adicionando Agentes Externos e Orquestradores com watsonx Orchestrate

## Índice

- [Adicionando Agentes Externos e Orquestradores com watsonx Orchestrate](#adicionando-agentes-externos-e-orquestradores-com-watsonx-orchestrate)
  - [Índice](#índice)
  - [Visão Geral](#visão-geral)
  - [Parte 1: Conectar Agente de Busca de Terceiros](#parte-1-conectar-agente-de-busca-de-terceiros)
  - [Parte 2: Criar o Agente Orquestrador](#parte-2-criar-o-agente-orquestrador)
    - [Testando o agente orquestrador](#testando-o-agente-orquestrador)
    - [Teste 2: quando o agente decide *não* buscar na web](#teste-2-quando-o-agente-decide-não-buscar-na-web)
  - [Resumo](#resumo)
    - [Sou desenvolvedor e quero me aprofundar no watsonx Orchestrate](#sou-desenvolvedor-e-quero-me-aprofundar-no-watsonx-orchestrate)
  - [Próximos Passos](#próximos-passos)

## Visão Geral

Neste laboratório, você vai explorar como conectar e orquestrar múltiplos agentes no watsonx Orchestrate para criar experiências mais completas e inteligentes.

Você vai aprender conceitos fundamentais de arquiteturas multiagentes: integração de um agente externo, descoberta de capacidades e tomada de decisão baseada em contexto, permitindo que cada agente contribua com sua especialidade para resolver as solicitações dos usuários.

> [!NOTE]
> **Pré-requisito:** este laboratório assume que você já concluiu o [laboratório 1](Step_by_Step_Lab1.md) e criou um agente com a base de conhecimento do catálogo de veículos. O agente criado naquele laboratório será reutilizado aqui como o agente responsável pelas consultas ao catálogo.

![watsonx Orchestrate](../../Assets_for_BuildBooks/lab-external-agents-orchestrator.PNG)

> ⚠️ **Atenção**
>
> Este laboratório utiliza um agente externo que não foi desenvolvido no watsonx Orchestrate. Para as atividades deste exercício, será utilizado um agente criado com o Langflow, integrado ao Orchestrate por meio do protocolo **A2A (Agent to Agent)**.
>
> Para estabelecer a comunicação entre os agentes, será necessário ter acesso à **URL do agente** e à respectiva **API Key**.
>
> Essas credenciais serão fornecidas pelo instrutor. Antes de prosseguir com o laboratório, certifique-se de obtê-las para garantir que todas as etapas possam ser executadas corretamente.

> Para saber mais sobre o protocolo de comunicação A2A, clique [aqui](https://www.ibm.com/br-pt/think/topics/agent2agent-protocol).

## Parte 1: Conectar Agente de Busca de Terceiros

Nesta parte, você vai conectar um agente externo que realiza buscas na web sobre carros, avaliações de proprietários e dados de mercado. Esse agente foi construído fora do watsonx Orchestrate, em Langflow, e vai se juntar à plataforma como um colaborador do seu agente de catálogo.

> Este agente usa o protocolo A2A para se comunicar com o watsonx Orchestrate. Para ter acesso a ele, utilize as credenciais e o link que o seu instrutor vai fornecer.

Nesse momento será necessário a criação de outro agente para isso, é necessário retornar a página de gerenciamento.

Utilize a seta de voltar conforme ilustrado na imagem abaixo:

![watsonx Orchestrate](../../Assets_for_BuildBooks/labs/lab02/lab01.png)

Clique no botão **Create agent +**, no canto superior direito da tela.

![Build agents and tools](../../Assets_for_BuildBooks/labs/lab02/lab02_01.png)

Uma janela se abre com as opções de criação. Escolha **Create from scratch**, a mesma opção usada no laboratório anterior para montar um agente do zero.

![Create an agent](../../Assets_for_BuildBooks/labs/lab02/lab02_02.png)

Você chega à tela de edição do agente, com o painel **Behavior** já selecionado. Preencha o perfil:

**1.** Em **Agent name**, informe `Agente de Busca`.

**2.** Em **Model**, mantenha `GPT-OSS 120B da OpenAI (via Groq)`.

**3.** Em **Description**, copie e cole o texto abaixo.

```
Este agente pesquisa no Google informações em tempo real, como avaliações de usuários, classificações e comparações de mercado, mas apenas para carros que estão em nosso catálogo. Não deve fornecer informações sobre veículos não vendidos pela nossa concessionária.
```

**4.** Note as quatro abas do topo, **Behavior**, **Knowledge**, **Tools** e **Agents**: é na aba **Agents** que você vai conectar o colaborador externo daqui a pouco.

![Perfil do Agente de Busca](../../Assets_for_BuildBooks/labs/lab02/lab02_03.png)

**5.** Navegue até a aba **Agents**

Clique em **Add Agents**.

![Aba Agents vazia](../../Assets_for_BuildBooks/labs/lab02/lab02_04.png)

**6.** Uma janela mostra três formas de adicionar um colaborador: **Catalog** (agentes pré construídos), **Local instance** (agentes já existentes na sua instância) e **Import** (registrar um agente externo). 

Escolha **Import**, já que o agente de busca foi construído em outra plataforma.

![Add Agents, escolha Import](../../Assets_for_BuildBooks/labs/lab02/lab02_05.png)

Essa é a tela de importação de outros agentes. As opções:

**External Agent** Permite registrar um agente hospedado por você ou por terceiros. O provedor mais comum é o external_chat, que integra qualquer agente capaz de expor um endpoint. Além dele, há suporte ao protocolo A2A e a integrações específicas com watsonx AI Agent Builder, watsonx Assistant e Salesforce Agentforce.

Agentes externos e agentes baseados em A2A podem ser desenvolvidos em qualquer framework, como BeeAI, LangGraph ou CrewAI, e hospedados na infraestrutura de sua escolha, por exemplo no Code Engine. Para integrações A2A, a versão suportada atualmente é a 0.3.0. Nesse modelo, o watsonx Orchestrate conhece apenas o endpoint e as credenciais de acesso; modelos, guardrails, observabilidade e logs permanecem sob responsabilidade da plataforma que hospeda o agente.

**LangGraph Agent** O agente é importado e executado nativamente no runtime do watsonx Orchestrate. Diferentemente dos agentes externos, que continuam hospedados em sua própria infraestrutura, o agente LangGraph passa a fazer parte da plataforma. Com isso, ele herda recursos como identidade, autenticação, monitoramento e políticas de governança do ambiente.

**watsonx Orchestrate Assistant** Adiciona um assistente criado diretamente no Assistant Builder do watsonx Orchestrate. Como o assistente permanece dentro do próprio tenant, não há necessidade de comunicação com serviços externos.

**External watsonx Assistant** Permite importar um assistente criado no IBM watsonx Assistant. Nesse caso, o assistente permanece em uma instância separada, com seu próprio perímetro de segurança, endpoint e credenciais de acesso.

**7.** Na tela **Import agent**, em **Choose agent type**, mantenha selecionada a opção **External agent**, que registra um agente construído com um provedor externo.

**8.** Clique em **Next**.

![Import agent, agent type](../../Assets_for_BuildBooks/labs/lab02/lab02_06.png)

Você chega à etapa **Register**, onde ficam os detalhes técnicos da conexão.

**9.** Em **External protocol**, mantenha `External agent via A2A standard`.

**10.** Em **A2A protocol version**, abra o menu e escolha `0.3.0`, a versão do protocolo usada neste laboratório.

![Register, protocolo A2A](../../Assets_for_BuildBooks/labs/lab02/lab02_07.png)

**11.** Em **External agent's URL**, cole o endereço público do agente externo que o seu instrutor compartilhou. É por essa URL que o watsonx Orchestrate busca o *Agent Card*, o documento que descreve as capacidades do agente no padrão A2A, e envia as requisições em tempo de execução.

**12.** Em **Display name**, na seção **Define new agent**, informe `Agente Langflow de Buscas` ou um nome de sua prefêrencia. Esse é o nome pelo qual o agente vai aparecer dentro do Orchestrate, independente de como foi construído externamente.

![URL do agente e display name](../../Assets_for_BuildBooks/labs/lab02/lab02_08.png)

Ainda na seção **Define new agent**, em **Description of agent capabilities**, cole o texto abaixo. Assim como a descrição do próprio agente, é este texto que o modelo de IA vai usar para decidir quando encaminhar uma solicitação a este colaborador:

```
Este agente se conecta ao serviço Tavily para realizar uma busca na web e retornar os principais resultados
```

**13.** Ainda na seção **Define new agent**, localize o campo **Description of agent capabilities** e cole o texto abaixo. 

```Este agente se conecta ao serviço Tavily para realizar buscas na web e retornar informações atualizadas de fontes externas. Pode ser utilizado para responder perguntas que exigem pesquisa na internet, obtenção de dados recentes e consulta a fontes públicas.```

> Assim como a descrição do agente, esse conteúdo será utilizado pelo modelo do watsonx Orchestrate para determinar quando uma solicitação deve ser encaminhada para esse colaborador. Uma descrição clara ajuda o Orchestrate a identificar corretamente os cenários em que esse agente deve ser utilizado.

Em **Advanced settings**, observe que existem opções relacionadas à forma como o agente se comunica com o Orchestrate:

- **Support streaming** permite que as respostas sejam enviadas gradualmente, à medida que são geradas, em vez de aguardar a resposta completa.
- **Support push notifications** habilita o envio de notificações assíncronas pelo agente para informar atualizações ou resultados posteriores.
- **Send conversation history** envia ao agente externo o histórico da conversa, fornecendo contexto adicional para a interação.

**Neste laboratório, nenhuma dessas funcionalidades será utilizada, pois o agente apenas receberá uma solicitação, realizará uma busca na web e retornará o resultado.**

**Portanto, mantenha todas as opções desabilitadas** e clique em **Next** para continuar.

![Descrição de capacidades e advanced settings](../../Assets_for_BuildBooks/labs/lab02/lab02_09.png)

Você chega à etapa **Connect**, que define como o Orchestrate vai se autenticar no agente externo.

> [!WARNING]
> A lista de **Connections** pode já exibir conexões A2A compatíveis criadas por outras pessoas no ambiente. Isso é normal em um ambiente compartilhado. Como você vai criar uma credencial dedicada para este agente, não é necessário reaproveitar nenhuma delas.

**14.** Clique em **Add A2A agent connection**.

![Connect, lista de conexões](../../Assets_for_BuildBooks/labs/lab02/lab02_10.png)

Na janela **Add connection**, faremos as seguintes configurações: **Define connection details**, **Configure draft environment** e **Configure live environment**.

**15.** Em **Connection ID (Required)**, copie e cole o seguinte nome: `apikey_external_agent`. 

**Esse campo aceita apenas letras, números, underscores e hífens, porque é o identificador técnico usado internamente e pela CLI ou ADK.**

**16.** Em **Display name**, repita `apikey_external_agent`, o nome amigável que vai aparecer nas telas do produto.

O campo **Connection description** é opcional e serve para adicionar uma descrição que ajude a identificar a finalidade daquela credencial. Para este laboratório, você pode deixá-lo em branco.

**17.** Clique em **Save and continue**.

![Define connection details](../../Assets_for_BuildBooks/labs/lab02/lab02_11.png)

**18.** O Orchestrate exibe um aviso: depois de criada, a conexão não pode ser renomeada nem excluída. Confira o ID e o nome informados e, estando tudo certo, clique em **Continue**.

![Aviso antes de criar a conexão](../../Assets_for_BuildBooks/labs/lab02/lab02_12.png)

Na etapa **Configure draft environment**, as credenciais usadas quando você testa e pré-visualiza o agente no chat de desenvolvimento, antes de publicá lo. 

Mais adiante você vai repetir a configuração para o ambiente **live**; manter os dois separados permite, por exemplo, apontar o draft para uma chave de teste e o live para a chave de produção.

**19.** Em **Authentication type**, abra a lista. Ela mostra os padrões suportados: **API Key** (uma chave fixa enviada em cada requisição), **Basic Auth** (usuário e senha em Base64), **Bearer Token** (um token no cabeçalho Authorization) e três variações de **OAuth2**, para fluxos com consentimento do usuário ou máquina a máquina. 

Selecione **API Key**, a opção usada neste laboratório.

![Authentication type](../../Assets_for_BuildBooks/labs/lab02/lab02_13.png)

Em **Server URL** cole o link do servidor onde seu agente está hospedado. 

> [!WARNING]
> Peça este link para os instrutores do laboratório.

Em **API Key Location**, mantenha `header`, o local esperado pelo agente externo. No campo ao lado não é necessária nenhuma ação.

![API Key Location e Server URL](../../Assets_for_BuildBooks/labs/lab02/lab02_14.png)

**20.** Role a tela até **Credential type**. 

Ele define quem fornece a credencial:

**Member credentials**, em que cada usuário informa a própria chave, indicado quando o acesso deve ser individualizado; 

**Team credentials**, em que uma única credencial, fornecida por você, é compartilhada por todos os usuários do ambiente.

![Credential type](../../Assets_for_BuildBooks/labs/lab02/lab02_15.png)


Selecione **Team credentials**, já que todos vão usar a mesma chave do serviço de busca.

![API Key preenchida](../../Assets_for_BuildBooks/labs/lab02/lab02_16.png)

**21.** No campo **API Key**, cole a chave usada para autenticar as requisições ao agente externo. O valor é mascarado assim que você segue para a próxima etapa. Clique em **Next**.

![Paste draft configuration](../../Assets_for_BuildBooks/labs/lab02/lab02_17.png)

**Configure live environment**

**22.** Para não repetir todo o preenchimento, clique em **Paste draft configuration**. Isso copia as definições do draft (tipo de autenticação, localização da chave e credenciais) para o ambiente live.

> [!WARNING]
> Embora essa configuração funcione para fins de demonstração neste laboratório, ela não é recomendada para ambientes de produção. Em cenários reais, normalmente são utilizadas credenciais, permissões e configurações específicas para cada ambiente, seguindo as políticas de segurança da organização. Para este exercício, não se preocupe com essa distinção e prossiga utilizando a mesma configuração.

![Paste draft configuration](../../Assets_for_BuildBooks/labs/lab02/lab02_18.png)

Revise se o **Authentication type** ficou como `API Key` e o **Credential type** como `Team credentials`; a tela tende a vir com `Member credentials` pré-selecionado, então vale conferir antes de seguir.

Confirme **Team credentials** e a API Key preenchida

![Team credentials confirmado no live](../../Assets_for_BuildBooks/labs/lab02/lab02_19.png)

**24.** Clique em **Finish**

![Salvando a conexão](../../Assets_for_BuildBooks/labs/lab02/lab02_20.png)

A notificação **New Connection added!** confirma que a conexão foi criada. Observe que a nova entrada **apikey_external_agent** aparece na lista, com as colunas **Draft** e **Live** indicando `API Key` e `Team credentials`, sinal de que os dois ambientes foram configurados corretamente. 

**25.** Selecione o *radio button* à esquerda dela para indicar que é essa credencial que o Orchestrate deve usar ao se comunicar com o agente externo.

**26.** Clique em **Done**

![Agente Langflow de Buscas listado como colaborador](../../Assets_for_BuildBooks/labs/lab02/lab02_22.png)

De volta à tela de edição do **Agente de Busca**, na aba **Agents**, o agente externo **Agente Langflow de Buscas** já aparece na lista de colaboradores, junto com a descrição que você definiu ao importá lo.

Com o agente externo importado, o **Agente de Busca** ganhou um especialista à disposição. O **watsonx Orchestrate** passa a atuar como orquestrador: interpreta o pedido do usuário, identifica que a tarefa é de busca na web e delega ao **Agente Langflow de Buscas**, de forma transparente, mesmo esse agente tendo sido construído em outra plataforma.

Vamos agora ensinar ao agente quando e como usar esse novo colaborador.

**27.** Clique na aba **Behavior**

![Instruções de validação de veículo](../../Assets_for_BuildBooks/labs/lab02/lab02_23.png)

**28.** No campo **Instructions**, adicione as instruções abaixo. Elas fazem o agente validar, antes de qualquer busca, se o veículo mencionado pelo usuário pertence ao catálogo, mesmo diante de nomes incompletos, apelidos ou pequenos erros de digitação:

```
# VALIDAÇÃO DE VEÍCULO

Antes de atender qualquer solicitação, identifique qual veículo do catálogo o usuário está tentando mencionar.

A identificação deve ser flexível e tolerar:

- Nomes incompletos
- Pequenas variações de escrita
- Erros de digitação
- Apelidos e abreviações comuns
- Omissão da marca ou da versão

Exemplos válidos:

- "Versa" → Nissan Versa
- "Nissan Versa" → Nissan Versa
- "Kona" → Hyundai Kona Electric
- "Hyundai Kona" → Hyundai Kona Electric
- "Spider" → Alfa Romeo Spider
- "Alfa Spider" → Alfa Romeo Spider
- "911" → Porsche 911 Carrera GTS
- "Porsche 911" → Porsche 911 Carrera GTS
- "Carrera GTS" → Porsche 911 Carrera GTS
- "Niro" → Kia Niro
- "Kia Nero" → Kia Niro
- "Kia Niro" → Kia Niro

Se houver alta confiança de que o veículo mencionado corresponde a um item do catálogo, chame o agente **Agente Langflow de Buscas** para ele tratar da solicitação. Todas as solicitações do usuário devem ir para o **Agente Langflow de Buscas**, a pergunta deve ser enviada para ele, e retornada.

Somente rejeite a solicitação quando não for possível associar o veículo informado a nenhum dos modelos do catálogo.

Se o veículo NÃO pertencer ao catálogo, responda exatamente:

"Desculpe, eu só posso fornecer informações sobre os seguintes veículos: Nissan Versa, Hyundai Kona Electric, Alfa Romeo Spider, Porsche 911 Carrera GTS e Kia Niro."
```

![Teste do agente com o Alfa Romeo Spider](../../Assets_for_BuildBooks/labs/lab02/lab02_24.png)


Teste o agente no **Draft Preview**. Envie a pergunta:

```
pesquisa sobre o Alfa Romeo Spider
```

O agente reconhece o veículo, delega a busca ao colaborador externo e devolve um resumo completo, com histórico do modelo, características técnicas e as diferentes gerações lançadas ao longo dos anos.

![Resposta detalhada do Agente Langflow de Buscas](../../Assets_for_BuildBooks/labs/lab02/lab02_25.png)

![Histórico e características do Alfa Romeo Spider](../../Assets_for_BuildBooks/labs/lab02/lab02_26.png)

![Conclusão da resposta sobre o Alfa Romeo Spider](../../Assets_for_BuildBooks/labs/lab02/lab02_27.png)

Envie agora uma segunda pergunta, comparando dois veículos bem diferentes entre si:

```
compare o nissan versa e o Alfa Romeo Spider
```

O agente monta uma tabela comparativa cobrindo segmento, ano de lançamento, motorização, transmissão e tração, e fecha com um resumo de para quem cada carro é indicado: o Versa para quem busca praticidade e economia, o Spider para quem valoriza estilo e prazer ao volante.

![Comparação entre Nissan Versa e Alfa Romeo Spider](../../Assets_for_BuildBooks/labs/lab02/lab02_29.png)

Pergunte em seguida se existem pessoas comparando os dois carros na internet, testando assim a capacidade do agente externo de ir além do que já foi respondido:

```
há pessoas comparando os dois na web?
```

![Pergunta de acompanhamento sobre comparações na web](../../Assets_for_BuildBooks/labs/lab02/lab02_30.png)

O agente confirma que esse tipo de comparação existe em fóruns e sites especializados, pondera que os dois veículos atendem públicos muito diferentes e ainda sugere comparações mais justas, como Versa com Honda Fit ou Toyota Yaris, e Spider com Mazda MX-5 Miata ou Porsche 718 Boxster. Essa resposta mostra o agente externo pesquisando na web em tempo real, e não apenas repetindo informações do catálogo interno.

![Resposta completa sobre comparações na web](../../Assets_for_BuildBooks/labs/lab02/lab02_31.png)

## Parte 2: Criar o Agente Orquestrador

Agora que o **Agente de Busca** já delega tarefas de pesquisa externa para o **Agente Langflow de Buscas**, vamos criar um terceiro agente: um orquestrador que recebe a pergunta do usuário e decide, sozinho, qual dos dois especialistas (o agente de catálogo do laboratório 1 ou o agente de busca externa que você acabou de configurar) deve responder, ou se os dois precisam ser consultados juntos.

Retorne à página de gerenciamento de agentes clicando no ícone de voltar `<` na interface do watsonx Orchestrate ao topo da página. 

Clique em **Create agent**.

![Lista de agentes com Create agent em destaque](../../Assets_for_BuildBooks/labs/lab02/lab02_32.png)

Clique em **Create from scratch**.

![Create an agent, Create from scratch](../../Assets_for_BuildBooks/labs/lab02/lab02_33.png)

Preencha o perfil do novo agente:

**1.** Em **Agent name**, informe `Assistente de Compra de Veículos`.

**2.** Em **Model**, mantenha `GPT-OSS 120B da OpenAI (via Groq)`.

**3.** Em **Description**, copie e cole:

```
Assistente inteligente de compra de carros que roteia consultas para agentes especializados. Fornece informações abrangentes tanto do nosso catálogo quanto de pesquisas externas de mercado.
```

**4.** Navegue até a aba **Agents**.

![Perfil do Assistente de Compra de Veículos](../../Assets_for_BuildBooks/labs/lab02/lab02_34.png)

**5.** Clique em **Add Agents**.

![Aba Agents vazia do orquestrador](../../Assets_for_BuildBooks/labs/lab02/lab02_35.png)

**6.** Na janela **Add Agents**, escolha **Local instance**, já que os dois agentes que você vai conectar agora, o de catálogo e o de busca, já existem nesta mesma instância do watsonx Orchestrate.

![Add Agents, Local instance](../../Assets_for_BuildBooks/labs/lab02/lab02_36.png)

**7.** Nessa tela é possível escolher os agentes disponíveis no catalogo na sua instância do **watsonx Orchestrate.**

![Seleção dos dois agentes especializados](../../Assets_for_BuildBooks/labs/lab02/lab02_37.png)

**7.** Marque os dois agentes criados até aqui.

Em seguida, clique em **Add to agent**

![Seleção dos dois agentes especializados](../../Assets_for_BuildBooks/labs/lab02/lab02_38.png)

A aba **Agents** agora lista os dois colaboradores, cada um com sua descrição.

Vamos ensinar ao agente orquestrador como decidir, a cada pergunta, para qual colaborador encaminhar a solicitação.

Navegue até a aba **Behaviour**


![Seleção dos dois agentes especializados](../../Assets_for_BuildBooks/labs/lab02/lab02_39.png)

**8.** Na aba **Behavior**, no campo **Instructions**, copie e cole a lógica de roteamento abaixo. Ela cobre três cenários: consultas que dependem só do catálogo, consultas que dependem só de pesquisa externa e consultas híbridas, que precisam dos dois agentes ao mesmo tempo:

```
Você é o Assistente de Compra de Veículos. Sua função é identificar a intenção do usuário e encaminhar a solicitação para o agente especializado correto.

# REGRAS DE ROTEAMENTO

## 1. CONSULTAS SOBRE CATÁLOGO → Agente de suporte ao revendedor

Encaminhe para o agente **Agente de suporte ao revendedor** quando o usuário pedir:

- Preço
- Especificações
- Disponibilidade
- Versões
- Comparação entre veículos do catálogo
- Informações da concessionária
- Informações de estoque

Exemplos:

- "Qual o preço do Versa?"
- "Me mostre os SUVs disponíveis"
- "Compare o Versa e o Kona"
- "Quais são as especificações do Porsche 911?"

## 2. PESQUISA EXTERNA → Agente de Busca

Encaminhe para o agente **Agente de Buscas** quando o usuário pedir:

- Avaliações de proprietários
- Reviews
- Opiniões de usuários
- Reclamações comuns
- Notícias
- Recalls
- Comparações de mercado
- Análises especializadas

Exemplos:

- "O que os donos falam do Porsche 911?"
- "Existem recalls do Kona?"
- "Quais são os principais problemas do Niro?"
- "Busque avaliações do Versa"

Observação:
- Esta regra se aplica tanto para veículos do catálogo quanto para veículos que não fazem parte do catálogo.

## 3. CONSULTAS HÍBRIDAS → AMBOS OS AGENTES

Quando a pergunta exigir informações internas do catálogo e pesquisa externa.

Exemplos:

- "O que os avaliadores dizem sobre o Porsche 911 e quais são suas especificações?"
- "Compare o Versa com seus concorrentes e mostre os dados técnicos."

Fluxo:

1. Consultar o agente **Agente de suporte ao revendedor**
2. Consultar o agente **Agente de Busca**
3. Consolidar os resultados em uma única resposta.

# REGRAS ADICIONAIS

- Sempre determine primeiro se o veículo pertence ao catálogo, consultando o agente **Agente de suporte ao revendedor.**
- Nunca rejeite uma consulta apenas porque o usuário informou um nome parcial.
- Sempre prefira interpretar o veículo pretendido quando houver alta confiança.
- Somente peça esclarecimentos quando houver ambiguidade real.
- Para pesquisas externas, utilize exclusivamente o agente **Agente de Busca**
```

### Testando o agente orquestrador

Com o **Assistente de Compra de Veículos** já conectado aos seus colaboradores, agora vamos verificar se eles realmente conversam entre si.

O objetivo desta parte é responder a três perguntas:

- O agente orquestrador consegue identificar sozinho **qual** especialista deve atender cada trecho de uma pergunta?
- Ele consegue **combinar** dados internos (catálogo da concessionária) com dados externos (pesquisa na web) em uma única resposta?
- E, quando a pergunta não exige busca externa, ele tem juízo de **não** acionar o agente de buscas à toa?


No painel `Draft Preview`, envie a consulta abaixo.

```
Busque sobre o Kia Niro com o Hyundai Kona Electric e faça uma comparação
```

Repare que essa pergunta é propositalmente ambígua quanto à origem da informação. Ela não diz "consulte o catálogo" nem "pesquise na internet". Quem precisa tomar essa decisão é o agente orquestrador, com base nas regras de roteamento definidas no campo `Instructions`.

![Seleção dos dois agentes especializados](../../Assets_for_BuildBooks/labs/lab02/lab02_40.png)

A resposta chega organizada em quatro blocos. O primeiro traz as **especificações técnicas** e já vem rotulado pelo próprio agente como *dados internos do catálogo*: tipo de motorização, potência, torque, bateria, autonomia EPA, câmbio, tração, capacidade de passageiros e recarga.

![Seleção dos dois agentes especializados](../../Assets_for_BuildBooks/labs/lab02/lab02_41.png)

Ainda no primeiro bloco aparecem os itens de segurança, o preço praticado no Brasil e a disponibilidade. Vale guardar esses dois valores, porque eles voltam a aparecer mais adiante:

| Modelo | Preço (catálogo) |
|---|---|
| Kia Niro | R$ 168.324,10 |
| Hyundai Kona Electric | R$ 139.834,10 |

O campo **Disponibilidade** aparece como *"Sem informação de estoque no catálogo"* para os dois modelos. Esse é um comportamento desejável: em vez de inventar um dado que não existe na base, o agente declara a ausência dele.

O segundo bloco muda de fonte. Sob o título **Avaliações de usuários e especialistas (pesquisa externa)**, o agente lista pontos positivos, pontos negativos e a opinião de especialistas para cada veículo — informação que não está no catálogo da concessionária e só poderia ter vindo de uma busca na web.
![Seleção dos dois agentes especializados](../../Assets_for_BuildBooks/labs/lab02/lab02_42.png)

O terceiro bloco é um **resumo rápido** em tabela e o quarto traz **recomendações** práticas, incluindo a sugestão de test-drive e a checagem de estoque e promoções na concessionária. A resposta termina com uma pergunta de continuidade, oferecendo agendar um test-drive.

![Seleção dos dois agentes especializados](../../Assets_for_BuildBooks/labs/lab02/lab02_43.png)

![Seleção dos dois agentes especializados](../../Assets_for_BuildBooks/labs/lab02/lab02_44.png)

Antes de seguir, role a conversa de volta para o topo e leia a resposta inteira. Ela mistura duas origens de dados em um texto só, e é exatamente isso que vamos rastrear a seguir.

![Seleção dos dois agentes especializados](../../Assets_for_BuildBooks/labs/lab02/lab02_45.png)

**Rastreando o caminho da resposta**

A resposta ficou boa, mas ainda é uma caixa-preta. Para entender quem foi acionado, clique em `Show Reasoning`, ao lado do nome do agente e do horário da mensagem.

![Seleção dos dois agentes especializados](../../Assets_for_BuildBooks/labs/lab02/lab02_46.png)

O painel se expande e mostra quatro passos numerados. Cada passo traz a ferramenta chamada, o **Input** que o agente enviou e o **Output** que recebeu de volta.

**Step 1 — o orquestrador chama o especialista interno**

O primeiro passo aciona a ferramenta `chat_with_collaborator_agente_de_suporte_ao_revendedor`. Repare no conteúdo do Input:

```json
{
  "message": "Quais são as especificações técnicas, preços e disponibilidade do Kia Niro e do Hyundai Kona Electric?"
}
```

Esse texto **não** foi escrito por você. O orquestrador leu a pergunta original, entendeu que parte dela dependia de dados da concessionária e reformulou uma sub-pergunta específica para o colaborador certo. O Output confirma a transferência: `Transferring to - chat_with_collaborator_agente_de_suporte_ao_revendedor`.


![Seleção dos dois agentes especializados](../../Assets_for_BuildBooks/labs/lab02/lab02_47.png)


**Step 2 — o especialista consulta a base de conhecimento**

Já dentro do Agente de Suporte ao Revendedor, o segundo passo chama a ferramenta `Catálogo_de_Carro_com_preços` com uma consulta enxuta:

```json
{
  "query": "Kia Niro"
}
```

O Output tem 22 linhas e vem do documento `Catalog_with_prices_clean.pdf`, com trechos do catálogo, preços e observações. É aqui que nasce o primeiro bloco da resposta.

![Seleção dos dois agentes especializados](../../Assets_for_BuildBooks/labs/lab02/lab02_48.png)

> **Nota:** ao expandir o Output com `Show more`, procure o trecho marcado como *QUALIDADE DE DADOS*. Ele avisa que o catálogo de origem escreve o modelo como **NERO**, enquanto o crossover elétrico da Kia se chama **Niro**, e que o texto foi mantido como impresso. Vale observar que o agente conseguiu casar a consulta "Kia Niro" com o registro "Kia Nero" do catálogo mesmo com a divergência de grafia.

**Step 3 — o orquestrador chama o especialista de busca**

Com os dados internos em mãos, o orquestrador parte para a segunda metade da pergunta e aciona `chat_with_collaborator_agente_de_busca`, novamente com uma sub-pergunta que ele mesmo formulou:

```json
{
  "message": "Busque avaliações de usuários e opiniões de especialistas sobre o Kia Niro e o Hyundai Kona Electric."
}
```
![Seleção dos dois agentes especializados](../../Assets_for_BuildBooks/labs/lab02/lab02_49.png)


**Step 4 — o segundo salto, até o agente externo**

Este é o passo mais interessante do laboratório. O Agente de Busca **não** responde sozinho: ele repassa a mesma mensagem para `chat_with_collaborator_Agente_Langflow_de_Buscas`, e o Output mostra a transferência para a instância `chat_with_collaborator_Agente_Langflow_de_Buscas_5015mM`, que é o agente externo importado via protocolo A2A no início deste laboratório.

![Seleção dos dois agentes especializados](../../Assets_for_BuildBooks/labs/lab02/lab02_50.png)

Colocando os quatro passos lado a lado, o desenho da execução fica assim:

```
Usuário
  └─ Assistente de Compra de Veículos (orquestrador)
       ├─ Agente de Suporte ao Revendedor          ← dados internos
       │    └─ Catálogo_de_Carro_com_preços        (base de conhecimento / PDF)
       └─ Agente de Busca                          ← dados externos
            └─ Agente Langflow de Buscas           (agente externo via A2A)
```

---

### Teste 2: quando o agente decide *não* buscar na web

Se o orquestrador sempre chamasse todos os colaboradores, o roteamento não teria valor nenhum. Vamos verificar se ele sabe se conter.

Clique no ícone `Reset chat`, no canto superior direito do painel `Draft Preview`, para começar uma conversa limpa. Isso é importante: sem o reset, o agente pode reaproveitar o contexto da comparação anterior e o teste perde a validade.

![Seleção dos dois agentes especializados](../../Assets_for_BuildBooks/labs/lab02/lab02_51.png)

Com o chat zerado, envie a consulta abaixo.

```
me apresente as opções disponíveis que vocês tem no catálogo e os preços
```
![Seleção dos dois agentes especializados](../../Assets_for_BuildBooks/labs/lab02/lab02_52.png)

Desta vez a resposta é uma tabela única com os cinco veículos em estoque, trazendo ID, marca, modelo, ano, cor e preço:

| ID | Marca | Modelo | Ano | Cor | Preço (R$) |
|---|---|---|---|---|---|
| VEH-001 | Nissan | Versa | 2024 | Azul | 95.778,20 |
| VEH-002 | Hyundai | Kona Electric | 2022 | Preto | 139.834,10 |
| VEH-003 | Alfa Romeo | Spider | 1991 | Vermelho | 103.082,00 |
| VEH-004 | Porsche | 911 Carrera GTS | – | Cinza | 776.974,10 |
| VEH-005 | Kia | Nero | 2026 | Amarelo | 168.324,10 |

Note que o registro VEH-005 aparece como **Kia Nero**, exatamente como está impresso no catálogo, confirmando a observação de qualidade de dados do Step 2 do teste anterior. E o preço de R$ 139.834,10 do Kona Electric é o mesmo que apareceu na comparação, o que prova que aquele bloco realmente veio da base interna.

Clique em `Show Reasoning` para conferir o caminho.

![Seleção dos dois agentes especializados](../../Assets_for_BuildBooks/labs/lab02/lab02_53.png)

Agora são apenas **dois** passos, e não quatro. O Step 1 encaminha para o Agente de Suporte ao Revendedor com a mensagem reformulada:

```json
{
  "message": "Por favor, apresente as opções disponíveis no catálogo e os preços."
}
```

![Seleção dos dois agentes especializados](../../Assets_for_BuildBooks/labs/lab02/lab02_54.png)

E o Step 2 consulta a base de conhecimento com a query `catalog`, retornando novamente 22 linhas do `Catalog_with_prices_clean.pdf`.

![Seleção dos dois agentes especializados](../../Assets_for_BuildBooks/labs/lab02/lab02_55.png)

O Agente de Busca e o agente externo do Langflow **não foram acionados**  e é esse o resultado esperado. A pergunta era sobre estoque próprio, um dado que existe internamente, então não havia razão para uma busca na web. Roteamento seletivo é o que separa um sistema multiagente de um sistema que apenas encarece e atrasa cada resposta.

> **Nota:** cada colaborador acionado é uma chamada de modelo a mais, com latência e consumo próprios. Comparar os quatro passos do primeiro teste com os dois passos do segundo é a forma mais direta de mostrar, na prática, o custo de um roteamento mal calibrado.

**Verificando a fonte da resposta**

Falta um último detalhe. Logo abaixo do texto da resposta há um pequeno botão de seta. Clique nele.

![Seleção dos dois agentes especializados](../../Assets_for_BuildBooks/labs/lab02/lab02_56.png)

O painel se abre e exibe o card do documento `Catalog_with_prices_clean.pdf`, com o trecho exato que fundamentou a resposta e um link `View source`. O contador `1/5` no rodapé indica que cinco trechos da base foram recuperados para montar aquela mensagem, e as setas permitem navegar entre eles.

![Seleção dos dois agentes especializados](../../Assets_for_BuildBooks/labs/lab02/lab02_57.png)

Enquanto o `Show Reasoning` responde *"por onde a resposta passou"*, esse painel responde *"em que documento ela se apoia"*. São duas perguntas diferentes e igualmente importantes quando você precisa auditar um agente:

- A resposta veio de uma fonte controlada por você ou de uma busca externa livre?
- O trecho recuperado sustenta mesmo a afirmação que o agente fez?

-----

## Resumo

 Parabéns! 🎉 
 Você concluiu o Laboratório 2. Além de reforçar tudo o que já sabia sobre criação de agentes, você deu um passo importante: aprendeu a conectar um agente construído fora do watsonx Orchestrate por meio do protocolo A2A, e a montar um agente orquestrador capaz de decidir, sozinho, para qual especialista encaminhar cada pergunta, seja o catálogo interno, a busca externa, ou os dois ao mesmo tempo.

Esse é o tipo de arquitetura multiagente que sustenta soluções reais de produção. Muito bem!

### Sou desenvolvedor e quero me aprofundar no watsonx Orchestrate

Todas as operações realizadas também estão disponíveis em uma experiência utilizando o ADK (Agent Development Kit). [Clique aqui](https://developer.watson-orchestrate.ibm.com/) para saber mais sobre como criar agentes, tools, bases de conhecimento e muito mais.

## Próximos Passos

<b>➜</b> [Clique aqui para acessar o próximo laboratório, Realizando avaliação de Agentes com watsonx Orchestrate](./Step_by_Step_Lab3.md)
