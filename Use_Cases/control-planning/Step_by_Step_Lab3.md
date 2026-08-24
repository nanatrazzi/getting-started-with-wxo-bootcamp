# Controles no watsonx Orchestrate

## Índice
- [Controles no watsonx Orchestrate](#controles-no-watsonx-orchestrate)
  - [Índice](#índice)
  - [Visão Geral](#visão-geral)
    - [O que é PII (Personal Identifiable Information)?](#o-que-é-pii-personal-identifiable-information)
  - [Pré-requisitos](#pré-requisitos)
  - [Descrição do Caso de Uso](#descrição-do-caso-de-uso)
    - [Parte 1: Acessar o watsonx Orchestrate e configurar o agente](#parte-1-acessar-o-watsonx-orchestrate-e-configurar-o-agente)
    - [Parte 2: Testando sem Asset Controls](#parte-2-testando-sem-asset-controls)
    - [Parte 3: Criando Controls para Filtragem de PII](#parte-3-criando-controls-para-filtragem-de-pii)
    - [Parte 4: Testando com Asset Controls](#parte-4-testando-com-asset-controls)
  - [Resultados e importância dos controles construídos com o Orchestrate](#resultados-e-importância-dos-controles-construídos-com-o-orchestrate)
  - [Resumo](#resumo)
  - [Próximos Passos](#próximos-passos)

## Visão Geral

Neste laboratório você vai aprender a proteger agentes de IA contra vazamento de PII, sigla em inglês para Personally Identifiable Information, ou informações de identificação pessoal, usando controls no watsonx Orchestrate.

Os controls são políticas de governança aplicadas sobre os ativos do ambiente, sejam agentes, modelos ou tools. Diferentemente de instruções escritas no prompt, eles operam no caminho da mensagem, fora do modelo, e valem independentemente do que o agente decide responder.

Ao longo do laboratório vamos identificar dados sensíveis sendo expostos por um agente sem proteção e, em seguida, configurar um controle que impede essa exposição.

### O que é PII (Personal Identifiable Information)?

PII é qualquer dado que possa ser usado para identificar um indivíduo. Alguns exemplos são nome completo, número de telefone, CPF, endereço e dados de cartão de crédito.

> [!Note]
> Se o dado pode ser usado para identificar uma pessoa, ele é considerado PII.

## Pré-requisitos

Para realizar este laboratório, é necessário ter concluído previamente os seguintes laboratórios:

Laboratório 1, [Envenenamento de Dados](./Step_by_Step_Lab1.md), e Laboratório 2, [Agente externo](./Step_by_Step_Lab2.md). Esses dois laboratórios servem como base para as atividades que serão executadas a seguir, já que vamos reaproveitar os agentes criados neles.

## Descrição do Caso de Uso

Neste laboratório, assim como no [Laboratório 1](./Step_by_Step_Lab1.md), vamos trabalhar com guardrails. A diferença está no mecanismo. No Laboratório 1 usamos guidelines, que são instruções de comportamento que orientam o agente sobre como agir. Aqui vamos usar controles do watsonx Orchestrate, uma camada de proteção que atua sobre o que entra e o que sai do agente, independentemente do que esse agente decide fazer.

Guidelines dependem do agente seguir a orientação. Controles são aplicados de forma determinística, mesmo quando o agente é convencido a ignorar suas instruções.

Temos o seguinte cenário: Um usuário faz uma pergunta simples e, por trás dela, o agente aciona uma ferramenta externa de busca para responder. Sem controles configurados, tudo que essa ferramenta retorna chega ao usuário sem nenhuma verificação sobre a sensibilidade do que está sendo divulgado. A missão deste laboratório é primeiro comprovar essa exposição e depois implementar um controle que a impeça.

### Parte 1: Acessar o watsonx Orchestrate e configurar o agente

Vamos reaproveitar o agente orquestrador criado no [Laboratório 2](./Step_by_Step_Lab2.md), o **Assistente de Compra de Veículos**. Ele é o agente que recebe a pergunta do usuário e decide, com base nas próprias instruções, se deve responder diretamente ou encaminhar a consulta para um agente especializado, como o Agente de Busca.

Abra o agente na aba Build e confira, na sessão Behavior, o campo Instructions. Nele está o comando que faz o roteamento funcionar:

```
Utilize o agente **agente de buscas** para localizar as informações solicitadas pelo usuário.

Em seguida apresente o resultado de forma clara, objetiva e completa, preservando os detalhes relevantes encontrados.
```
Essa é a instrução que vamos colocar à prova na próxima parte. Ela diz ao Assistente de Compra de Veículos para sempre delegar a busca de informações ao agente especializado, sem qualquer ressalva sobre o tipo de dado que pode voltar dessa busca.

![Controles - watsonx Orchestrate](../../Assets_for_BuildBooks/labs/lab03/lab03_01.png)

### Parte 2: Testando sem Asset Controls

Agora vamos testar o agente sem nenhum controle configurado, para entender como ele se comporta antes de aplicarmos qualquer proteção.

No painel Draft Preview, envie a pergunta abaixo.

```
Qual o número da IBM?
```

![Controles - watsonx Orchestrate](../../Assets_for_BuildBooks/labs/lab03/lab03_02.png)

Essa é uma informação que qualquer pessoa consegue encontrar em segundos com uma busca simples no Google, que retorna o número público de atendimento da IBM no Brasil.

![Controles - watsonx Orchestrate](../../Assets_for_BuildBooks/labs/lab03/lab03_03.png)

O Assistente de Compra de Veículos responde com uma lista de números de telefone da IBM em diferentes países, incluindo Estados Unidos, Brasil, Portugal e Espanha.

![Controles - watsonx Orchestrate](../../Assets_for_BuildBooks/labs/lab03/lab03_04.png)

Clique em `Show Reasoning`, ao lado da resposta, para entender de onde esses dados vieram. O primeiro passo do raciocínio mostra que o agente acionou a ferramenta `chat_with_collaborator_Agente_de_Buscas`, repassando a pergunta original, e o resultado foi transferido para o agente especializado `Agente_de_Buscas_48414t`, que é quem de fato pesquisa na internet.

![Controles - watsonx Orchestrate](../../Assets_for_BuildBooks/labs/lab03/lab03_05.png)

O ponto de atenção aqui não é o dado em si, já que um telefone institucional público é uma informação de baixo risco. O ponto de atenção é o mecanismo. A resposta não veio de uma base de conhecimento controlada por você, mas de uma busca livre feita por um agente externo, chamado como ferramenta. Sem controles configurados, tudo que essa fonte externa retorna chega ao usuário sem nenhuma verificação intermediária, seja um telefone público ou um dado bem mais sensível.

### Parte 3: Criando Controls para Filtragem de PII

Vamos aplicar um controle que impede o agente de expor números de telefone ao usuário, não importa a origem da informação.

Clique no menu hambúrguer, no canto superior esquerdo da tela.

![Controles - watsonx Orchestrate](../../Assets_for_BuildBooks/labs/lab03/lab03_06.png)

Clique em `Manage` para expandir a sessão.

Em seguida, clique em `Controls`.

![Controles - watsonx Orchestrate](../../Assets_for_BuildBooks/labs/lab03/lab03_07.png)

Nesse momento o painel de controles está vazio, sem nenhum controle criado. Clique em `Create Control`.

![Controles - watsonx Orchestrate](../../Assets_for_BuildBooks/labs/lab03/lab03_08.png)

A tela Select Control lista os tipos de controle disponíveis, organizados por onde eles atuam.

Em Agents, aplicados diretamente ao agente, estão o Content Guardrails, que detecta e bloqueia conteúdo sexual, violento, discurso de ódio, conteúdo nocivo, tentativas de jailbreak e viés social, o Output Length Guard, que limita o tamanho mínimo e máximo da saída em caracteres ou tokens, o Regex Pattern, que detecta conteúdo que corresponda a uma expressão regular fornecida pelo usuário e redige ou bloqueia, o Secrets Detector, que identifica credenciais e segredos em entradas e saídas, e o PII Filter, que detecta e mascara PII em argumentos, entradas e saídas do agente.

![Controles - watsonx Orchestrate](../../Assets_for_BuildBooks/labs/lab03/lab03_09.png)

Em Tools, aplicados às ferramentas utilizadas por agentes e workflows, estão o Content Guardrails e o Output Length Guard, com a mesma função da versão de agente mas atuando sobre o que a ferramenta processa, o Rate Limiter, que impõe limites de chamadas por tool ou tenant, e o SQLSanitizer, que detecta SQL de risco e pode remover comentários ou bloquear a execução.

![Controles - watsonx Orchestrate](../../Assets_for_BuildBooks/labs/lab03/lab03_10.png)

Em Models, aplicados ao modelo de inteligência artificial generativa utilizado pelos agentes, estão o Fallback, que define política de fallback e tratamento de códigos de status, o Load Balance, que distribui chamadas entre múltiplos modelos com pesos configuráveis, e o Retry, que repete a chamada ao mesmo modelo até um número configurável de tentativas em códigos HTTP específicos.

![Controles - watsonx Orchestrate](../../Assets_for_BuildBooks/labs/lab03/lab03_11.png)

Para este laboratório, selecione `PII Filter` e clique em `Next`.

![Controles - watsonx Orchestrate](../../Assets_for_BuildBooks/labs/lab03/lab03_12.png)

Na etapa Configure Control, comece pelo nome e pela descrição da instância. Use:

```
Números de telefone
```

```
Com esse controle, o agente bloqueia solicitações de pedidos de número de telefones.
```

![Controles - watsonx Orchestrate](../../Assets_for_BuildBooks/labs/lab03/lab03_13.png)

Em Enforcement type, marque tanto `Input` quanto `Output`. Input analisa o que o usuário envia ao agente, impedindo que dados sensíveis entrem no fluxo. Output analisa o que o agente devolve, impedindo que dados sensíveis cheguem até o usuário. Com os dois marcados, o filtro cobre a conversa inteira, na entrada e na saída.

![Controles - watsonx Orchestrate](../../Assets_for_BuildBooks/labs/lab03/lab03_14.png)

Em Detection Type você define o escopo do filtro, ou seja, quais categorias de dado o controle vai procurar em cada mensagem. Abra o menu suspenso e role a lista para ver todos os tipos de PII disponíveis, que somam onze ao todo.

![Controles - watsonx Orchestrate](../../Assets_for_BuildBooks/labs/lab03/lab03_15.png)

| Tipo | O que detecta |
|---|---|
| Detect Bank Account | Números de conta bancária |
| Detect BSN | Número de identificação nacional holandês |
| Detect Credit Card | Números de cartão de crédito |
| Detect Date Of Birth | Datas de nascimento |
| Detect Driver License | Números de carteira de motorista |
| Detect Email | Endereços de e-mail |
| Detect IP Address | Endereços IP |
| Detect Medical Record | Identificadores de prontuário médico |
| Detect Passport | Números de passaporte |
| Detect Phone | Números de telefone |
| Detect SSN | Número de Seguro Social dos Estados Unidos |

![Controles - watsonx Orchestrate](../../Assets_for_BuildBooks/labs/lab03/lab03_16.png)

![Controles - watsonx Orchestrate](../../Assets_for_BuildBooks/labs/lab03/lab03_17.png)

Para este laboratório, marque `Select all`. O campo passa a exibir o indicador 11, confirmando que todos os tipos de PII estão ativos, mesmo que o foco do teste seja o número de telefone.

![Controles - watsonx Orchestrate](../../Assets_for_BuildBooks/labs/lab03/lab03_18.png)

Em Default mask strategy você define o que acontece quando um dado sensível é encontrado. Redact substitui o valor por um marcador fixo, ocultando o conteúdo por completo. Partial mantém parte do valor visível, como os últimos dígitos de um cartão, e mascara o restante. Hash troca o valor por um hash, de forma que o mesmo dado sempre gera o mesmo resultado, mas sem possibilidade de reversão. Tokenize substitui o valor por um token que preserva a referência ao dado original, útil quando o fluxo precisa continuar identificando o registro. Remove apaga o valor do texto sem deixar marcador algum.

![Controles - watsonx Orchestrate](../../Assets_for_BuildBooks/labs/lab03/lab03_19.png)

Selecione `Remove`.

![Controles - watsonx Orchestrate](../../Assets_for_BuildBooks/labs/lab03/lab03_20.png)

Em Enforcement Mode você define qual atitude o controle toma quando encontra algo. Enquanto o Detection Type define o que procurar e o Default mask strategy define como transformar o dado encontrado, o Enforcement Mode decide o que fazer com essa detecção. As opções são combináveis. Block On Detection interrompe o fluxo, bloqueando a mensagem em vez de entregar o conteúdo mascarado, e é a postura mais restritiva, prevalecendo sobre a estratégia de mascaramento quando ativa. Include Detection Details informa na resposta o que foi detectado, indicando os tipos de PII encontrados. Log Detections registra as detecções para auditoria e monitoramento, sem alterar o que o usuário vê.

![Controles - watsonx Orchestrate](../../Assets_for_BuildBooks/labs/lab03/lab03_21.png)

Para este laboratório, marque as três opções, Block On Detection, Include Detection Details e Log Detections. O campo passa a exibir o indicador 3.

Em seguida, mantenha os valores padrão dos três campos numéricos que aparecem na sequência. Max text bytes, com 10485760 bytes, cerca de dez megabytes, define o tamanho máximo de texto que o controle vai inspecionar em uma única mensagem, existindo para evitar que uma mensagem muito grande sobrecarregue o processo de verificação. Max nested depth, com 32, define até que nível de aninhamento o controle vai descer procurando a informação sensível dentro de uma estrutura de dados, e trinta e dois níveis já é bem mais do que respostas reais costumam ter. Max collection items, com 4096, define quantos itens de uma lista ou array o controle vai examinar, mantendo previsível o custo da inspeção em respostas volumosas.

![Controles - watsonx Orchestrate](../../Assets_for_BuildBooks/labs/lab03/lab03_22.png)

Os dois últimos campos desta etapa são Custom patterns e Allowlist patterns. Custom patterns aceita expressões regulares para cobrir formatos que as onze categorias nativas não contemplam, como documentos brasileiros, por exemplo o CPF, ou identificadores internos da organização. Allowlist patterns funciona ao contrário, criando exceções para valores que têm formato de PII mas não são sensíveis, como o telefone institucional já divulgado publicamente no site da empresa. Neste laboratório, deixe os dois campos vazios, já que o objetivo é justamente observar o controle bloqueando o número de telefone no teste seguinte.

Clique em `Next`.

![Controles - watsonx Orchestrate](../../Assets_for_BuildBooks/labs/lab03/lab03_23.png)

A etapa Assign Assets define a quais agentes o controle será aplicado. Como escolhemos PII Filter na seção Agents, a tela pede que você indique quais agentes ficarão sob esse controle. Clique em `Add Agent`.

![Controles - watsonx Orchestrate](../../Assets_for_BuildBooks/labs/lab03/lab03_24.png)

A janela Add Agent lista todos os agentes disponíveis no ambiente. Marque os agentes ligados à busca externa de informações, que são exatamente os que colocam o agente em contato com dados fora do seu controle direto. Neste laboratório, selecione `Agente_consultor_1924dk`, `Agente_de_Buscas_1944Wq`, `Agente_de_Buscas_48414t`, `Agente de Busca` e `Agente de suporte ao revendedor`. O painel Selected agents, à direita, confirma o total de cinco agentes escolhidos. Clique em `Select`.

![Controles - watsonx Orchestrate](../../Assets_for_BuildBooks/labs/lab03/lab03_25.png)

Note que o Assistente de Compra de Veículos, o agente orquestrador testado na Parte 2, não entra nessa lista. O controle é aplicado diretamente sobre os agentes especializados que realizam buscas e retornam dados externos, que é onde a informação sensível de fato entra no fluxo.

A etapa Review reúne tudo o que foi configurado, organizado em blocos, cada um com um link Edit que leva de volta à etapa correspondente caso algo precise de ajuste. Confira o bloco Control details, com o tipo de controle, nome, descrição, tipo de ativo e os hooks de Input e Output.

![Controles - watsonx Orchestrate](../../Assets_for_BuildBooks/labs/lab03/lab03_26.png)

Role a página para ver o restante da configuração, incluindo os valores de Max text bytes, Max nested depth e Max collection items, além da tabela Applies to, que lista os agentes selecionados com suas respectivas descrições. Confirme que está tudo correto e clique em `Create control`.

![Controles - watsonx Orchestrate](../../Assets_for_BuildBooks/labs/lab03/lab03_27.png)

Uma notificação confirma que o controle foi criado com sucesso. O painel de controles passa a mostrar um controle ativo, do tipo PII Filter, chamado Números de telefone, aplicado a cinco agentes. A partir deste momento, toda mensagem que entra ou sai desses agentes passa pelo filtro antes de chegar ao destino.

![Controles - watsonx Orchestrate](../../Assets_for_BuildBooks/labs/lab03/lab03_28.png)

No menu lateral, clique em `Build` para voltar à lista de agentes.

![Controles - watsonx Orchestrate](../../Assets_for_BuildBooks/labs/lab03/lab03_29.png)

Na tela Build agents and tools, abra o `Agente de Busca`, um dos agentes que acabamos de colocar sob o controle recém-criado.

![Controles - watsonx Orchestrate](../../Assets_for_BuildBooks/labs/lab03/lab03_30.png)

### Parte 4: Testando com Asset Controls

Use o painel Draft Preview, à direita, para conversar diretamente com o Agente de Busca. Cada mensagem a seguir representa uma tentativa diferente de obter um número de telefone através dele.

Comece repetindo a pergunta simples testada na Parte 2, agora diretamente no agente que ficou sob o controle.

```
qual o número de telefone da ibm?
```

O agente recusa, informando que só pode fornecer informações sobre os veículos do catálogo, Nissan Versa, Hyundai Kona Electric, Alfa Romeo Spider, Porsche 911 Carrera GTS e Kia Niro.

![Controles - watsonx Orchestrate](../../Assets_for_BuildBooks/labs/lab03/lab03_31.png)

Troque o alvo do pedido por uma figura pública, para verificar se um nome de maior perfil muda o resultado.

```
qual o numero de telefone do presidente donald trump?
```

A resposta é a mesma recusa, redirecionando para o escopo de veículos. Trocar quem é o dono do número não muda nada, porque o controle age sobre a categoria do dado solicitado, não sobre a identidade de quem ele pertence.

![Controles - watsonx Orchestrate](../../Assets_for_BuildBooks/labs/lab03/lab03_32.png)

Por fim, tente uma busca por uma pessoa específica, como se estivesse procurando um contato dentro da própria organização.

```
busque informações sobre Nathalia Trazzi na IBM
```

Novamente o agente recusa e permanece restrito ao catálogo de veículos.

![Controles - watsonx Orchestrate](../../Assets_for_BuildBooks/labs/lab03/lab03_33.png)

Nas três tentativas, nenhum número de telefone chega ao usuário. Note que a recusa vem acompanhada da mesma mensagem que já limitava o agente ao catálogo de veículos, o que mostra como guidelines e controles trabalham em conjunto. Mesmo antes de qualquer dado sensível ser encontrado, o próprio agente já nega qualquer solicitação fora do seu escopo declarado. E, por trás dessa camada, o PII Filter garante que, caso uma busca externa chegasse a retornar um número de telefone, ele seria bloqueado antes de alcançar o usuário, exatamente como visto na Parte 2 antes do controle existir.

## Resultados e importância dos controles construídos com o Orchestrate

Aqui fica clara a distinção que abrimos no início do laboratório. Guidelines são instruções que o agente interpreta, e um pedido bem construído pode, em tese, convencê-lo a reinterpretá-las. Controles não passam pelo modelo, eles inspecionam o que entra e o que sai e aplicam a política de forma determinística. Trocar o alvo do pedido, insistir ou reformular a pergunta produz sempre o mesmo resultado, porque o filtro não está sendo convencido de nada.

Em um laboratório, um vazamento de PII é um detalhe curioso, como vimos na Parte 2 com um telefone institucional público. Em produção, é um incidente. Agentes acessam bases de clientes, sistemas de RH, CRMs e ferramentas externas, e qualquer um desses caminhos pode trazer dado sensível para a conversa. Some a isso a LGPD, que responsabiliza a organização pelo tratamento inadequado desses dados, mesmo quando a exposição não foi intencional. O controle protege nos dois sentidos, contra o usuário que insere dados sensíveis e contra a ferramenta que os retorna, e continua valendo mesmo que o modelo generativo seja trocado ou as instruções do agente sejam reescritas.

O Orchestrate ainda oferece o painel de Controls, com visibilidade do que está ativo em cada categoria de ativo, e a opção Log Detections, que registra o que foi barrado ao longo do tempo em vez de deixar você descobrir isso apenas quando alguém reclamar.

Nada disso dispensa o teste. A bateria de prompts que você rodou deve ser repetida sempre que algo mudar, seja um novo agente, uma nova ferramenta, um novo modelo ou um ajuste nas categorias de detecção. O Draft Preview em modo de debug existe justamente para isso, e o Save as test guarda os casos para reexecução. Testar o caminho feliz mostra que o agente funciona, testar o caminho adversarial mostra que ele resiste. Uma proteção que nunca foi testada é, na prática, apenas uma suposição.

## Resumo

Parabéns por concluir mais um laboratório! 🎉 

Você comprovou como um agente sem proteção pode expor dados de uma fonte externa sem nenhuma verificação, implementou um filtro de PII usando Asset Controls e o aplicou aos agentes responsáveis pela busca externa de informações, ganhando experiência prática com proteção de dados em nível corporativo.

Ao concluir este laboratório, você é capaz de identificar o que são dados PII e por que precisam de proteção, diferenciar guidelines de controles e saber quando usar cada um, criar e configurar um PII Filter no watsonx Orchestrate e atribuir um controle a um ou mais agentes.

## Próximos Passos

➜ [Clique aqui para navegar para o próximo lab, Monitorando Agentes em Tempo Real com watsonx Orchestrate](./Step_by_Step_Lab4.md)
