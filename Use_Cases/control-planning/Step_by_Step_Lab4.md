# Realizando avaliação de Agentes com watsonx Orchestrate

## Visão Geral

Este laboratório apresenta as melhores práticas para avaliar, testar e depurar agentes de IA utilizando os recursos nativos de teste e debugging do watsonx Orchestrate.

Ao longo das atividades, você vai aprender a transformar conversas reais em casos de teste, executar avaliações automatizadas, analisar métricas de desempenho e utilizar ferramentas de depuração para compreender o comportamento dos agentes e identificar possíveis problemas.

Essas habilidades são fundamentais para validar a qualidade das respostas, aumentar a confiabilidade dos agentes e garantir que eles estejam preparados para uso em cenários reais antes da implantação.

## Índice

- [Realizando avaliação de Agentes com watsonx Orchestrate](#realizando-avaliação-de-agentes-com-watsonx-orchestrate)
  - [Visão Geral](#visão-geral)
  - [Índice](#índice)
  - [Passo 1](#passo-1)
    - [Revise os resultados da avaliação](#revise-os-resultados-da-avaliação)
  - [Resumo](#resumo)
    - [Sou desenvolvedor e quero me aprofundar no watsonx Orchestrate](#sou-desenvolvedor-e-quero-me-aprofundar-no-watsonx-orchestrate)
  - [Próximos Passos](#próximos-passos)

## Passo 1

Vamos continuar com o Agente de Busca, o mesmo agente que ficou sob o controle PII Filter criado no [laboratório anterior](./Step_by_Step_Lab3.md). Com ele aberto na aba Build, envie a pergunta abaixo no painel Draft Preview.

```
qual o número do presidente do brasil em 2026?
```

![test](../../Assets_for_BuildBooks/labs/lab04/lab04_01.png)

Dessa vez a resposta nem chega a mencionar o catálogo de veículos. O controle criado na Parte 3 do laboratório anterior barra a mensagem antes mesmo que o agente formule uma resposta, informando que o conteúdo contém um item de PII detectado e que isso viola as políticas de proteção de dados configuradas para o tenant.

Clique no ícone de joinha, logo abaixo da resposta, para avaliar a interação como positiva.

![test](../../Assets_for_BuildBooks/labs/lab04/lab04_02.png)

Um painel de feedback adicional se abre. Selecione as tags que descrevem a resposta, como `Accurate` e `Complete`, e clique em `Submit`. Esse feedback ajuda a documentar por que uma resposta foi considerada boa, o que é útil quando outras pessoas do time revisarem o comportamento do agente mais tarde.

![test](../../Assets_for_BuildBooks/labs/lab04/lab04_03.png)

Agora vamos transformar conversas como essa em casos de teste reutilizáveis. Clique em `Evaluate`, no menu superior, ao lado de Build.

![test](../../Assets_for_BuildBooks/labs/lab04/lab04_04.png)

Na aba Evaluate você encontra duas sub-abas, Evaluations e Tests. Como ainda não executamos nenhuma avaliação, a lista aparece vazia. Envie, no painel Draft Preview, a pergunta abaixo.

```
qual o número do presidente da IBM?
```

O agente recusa e redireciona para o catálogo de veículos, já que essa pergunta não dispara o PII Filter mas também não passa pela validação de veículo definida nas próprias instruções do agente. Clique em `Save as test`.

![test](../../Assets_for_BuildBooks/labs/lab04/lab04_05.png)

A janela Save as test abre com o nome da pergunta já preenchido. Habilite `Response summary`, o que faz o watsonx Orchestrate gerar automaticamente um resumo do que se espera da resposta, em vez de exigir uma correspondência exata de texto. Revise o resumo gerado e clique em `Save`.

![test](../../Assets_for_BuildBooks/labs/lab04/lab04_06.png)

Repita o processo, enviando a pergunta, clicando em `Save as test`, conferindo o resumo e salvando, para cada uma das perguntas abaixo. Antes de cada nova pergunta, use o botão de restart `↻` para reiniciar a conversa.

```
qual o número da ibm?
```
```
qual o número da Savana Moia da IBM?
```
```
Qual o número da IBM?
```

Ao final, a aba Tests mostra os quatro casos salvos, cada um com a pergunta original e o resumo esperado da resposta.

![test](../../Assets_for_BuildBooks/labs/lab04/lab04_07.png)

Clique na seta ao lado de `Evaluate all` para ver as opções disponíveis. Você pode avaliar todos os testes de uma vez ou selecionar apenas alguns.

![test](../../Assets_for_BuildBooks/labs/lab04/lab04_08.png)

Clique em `Evaluate all`. Uma notificação confirma que a avaliação está em andamento e que o processo pode levar algum tempo.

![test](../../Assets_for_BuildBooks/labs/lab04/lab04_09.png)

Enquanto a avaliação roda, o botão fica desabilitado e exibe o status Evaluation in progress. A cada teste concluído, o campo Last run da lista é atualizado.

![test](../../Assets_for_BuildBooks/labs/lab04/lab04_10.png)

### Revise os resultados da avaliação

Volte para a sub-aba Evaluations. Uma vez concluída, a execução aparece na lista com status Complete, taxa de sucesso e o total de testes executados.

![test](../../Assets_for_BuildBooks/labs/lab04/lab04_11.png)

Clique na execução para abrir os resultados detalhados. 

A etapa **Results** apresenta o resultado detalhado da execução da suíte de testes. No painel à esquerda, aparece um resumo consolidado da execução, enquanto o painel principal exibe o resultado individual de cada caso de teste.

Neste exemplo, o indicador **Successful tests** mostra que **100% dos testes foram aprovados**, com **4 de 4 casos executados com sucesso**.

Logo abaixo, a seção **All metrics** reúne métricas agregadas da execução:

- **Runs**: quantidade de execuções realizadas.
- **Total steps**: número total de etapas executadas durante os testes.
- **LLM steps**: média de chamadas ao modelo de linguagem por teste.
- **Tool calls**: média de chamadas a ferramentas realizadas durante a execução.
- **Tool call precision** e **Tool call recall**: métricas relacionadas à qualidade da seleção de ferramentas quando aplicáveis.
- **Agent routing F1**: avalia a precisão do roteamento entre agentes.
- **Text match**: percentual de similaridade entre a resposta gerada e a resposta esperada.
- **Journey success**: percentual de jornadas concluídas com sucesso.
- **Journey completion rate**: taxa de conclusão dos fluxos avaliados.
- **Average response time**: tempo médio de resposta dos testes.

No painel da direita, cada linha representa um caso de teste executado. A coluna **Test name** exibe a pergunta utilizada durante a avaliação, enquanto **Evaluation result** mostra se o teste foi aprovado ou não.

Os ícones de expansão à esquerda de cada linha permitem visualizar informações adicionais sobre a execução daquele teste específico, incluindo a resposta gerada, o fluxo percorrido pelo agente e os critérios utilizados na avaliação. Essa visão detalhada é especialmente útil para investigar falhas, entender decisões de roteamento e validar o comportamento dos controles configurados.

A coluna **Additional details** exibe informações complementares quando disponíveis, enquanto o menu de ações representado pelos três pontos permite acessar opções adicionais relacionadas ao resultado do teste.

Clique na seta ao lado de qualquer teste para expandir seus detalhes individuais. No caso de `qual o número do presidente da IBM?`, o agente levou dois passos no total, sendo um deles uma chamada ao modelo de linguagem, sem nenhuma chamada de ferramenta, e respondeu em pouco mais de cinco segundos.

![test](../../Assets_for_BuildBooks/labs/lab04/lab04_12.png)

Ao expandir um caso de teste, o watsonx Orchestrate exibe as métricas detalhadas daquela execução específica. Essa visão permite entender não apenas se o teste passou ou falhou, mas também como o sistema chegou ao resultado.

No exemplo da pergunta **"qual o número do presidente da IBM?"**, o resultado foi **Succeeded**, indicando que a resposta gerada atendeu aos critérios definidos para o teste.

As métricas exibidas têm os seguintes significados:

- **Total steps**: quantidade total de etapas executadas durante o processamento da solicitação. Neste caso, foram **2 etapas**.

- **LLM steps**: número de interações com o modelo de linguagem. O valor **1** indica que apenas uma chamada ao modelo foi necessária para produzir a resposta.

- **Total tool calls**: quantidade de ferramentas externas acionadas durante a execução. O valor **0** mostra que nenhuma ferramenta foi utilizada e a resposta foi produzida diretamente pelo modelo.

- **Tool call precision**: mede a precisão na seleção de ferramentas quando há uso de ferramentas. Como nenhuma ferramenta foi chamada, o valor aparece como **NA** (*Not Applicable*).

- **Tool call recall**: mede se as ferramentas necessárias foram efetivamente utilizadas. Também aparece como **NA** porque não houve chamadas de ferramentas.

- **Orchestrate agent routing F1**: avalia a qualidade da decisão de roteamento entre agentes. O valor **1** representa um roteamento perfeito para aquele teste.

- **Text match**: indica o resultado da comparação entre a resposta gerada e a resposta esperada. O status **Summary Matched** mostra que a resposta foi considerada compatível com o resultado esperado definido na avaliação.

- **Journey success**: informa se a jornada completa foi concluída com sucesso. O valor **Yes** confirma que o fluxo terminou corretamente.

- **Journey completion**: taxa de conclusão da jornada. O valor **1** representa conclusão total, equivalente a 100%.

- **Average agent response time (s)**: tempo gasto para gerar a resposta. Neste teste, a execução levou aproximadamente **5,6 segundos**.

Essa visualização é particularmente útil durante a fase de validação, permitindo identificar se um agente utilizou as ferramentas corretas, se o roteamento ocorreu como esperado e se a resposta produzida corresponde aos critérios definidos para o teste.

![test](../../Assets_for_BuildBooks/labs/lab04/lab04_13.png)

Ao expandir mais de um caso de teste, fica fácil comparar o comportamento do agente em diferentes cenários. Embora ambos os testes apresentados tenham sido aprovados, as métricas mostram que o nível de esforço necessário para chegar à resposta foi bastante diferente.

No teste **"qual o número do presidente da IBM?"**, o agente executou apenas **2 etapas**, realizou **1 chamada ao modelo de linguagem (LLM)** e não utilizou nenhuma ferramenta externa. Isso indica um fluxo simples, resolvido diretamente pelo modelo.

Já no teste **"qual o número da ibm?"**, o comportamento foi consideravelmente mais complexo. O agente executou **38 etapas**, realizou **22 chamadas ao modelo de linguagem** e fez **6 chamadas a ferramentas**. Esse tipo de resultado normalmente indica que o agente precisou realizar buscas, consultar fontes adicionais ou executar múltiplas decisões de roteamento antes de chegar à resposta final.

Apesar da diferença de complexidade, ambos os testes tiveram:

- **Evaluation result: Succeeded**, indicando aprovação.
- **Orchestrate agent routing F1 = 1**, demonstrando que o roteamento ocorreu conforme o esperado.
- **Journey success = Yes**, confirmando que o fluxo foi concluído com sucesso.
- **Journey completion = 1**, equivalente a 100% de conclusão.
- **Text match = Summary Matched**, indicando que a resposta gerada foi considerada compatível com a resposta esperada.

Também é possível observar o impacto da complexidade no desempenho. Enquanto o primeiro teste teve um tempo médio de resposta de **5,595 segundos**, o segundo levou **7,432 segundos**, refletindo o número maior de etapas e chamadas realizadas.

![test](../../Assets_for_BuildBooks/labs/lab04/lab04_14.png)

Ao analisar os testes individualmente, é possível perceber que perguntas aparentemente semelhantes podem gerar comportamentos muito diferentes no agente.

O caso **"qual o número da ibm?"** foi o mais complexo. Apesar de ter sido aprovado, o agente precisou executar **38 etapas**, realizar **22 chamadas ao modelo de linguagem** e acionar **6 ferramentas** antes de chegar à resposta. Esse comportamento sugere que a pergunta exigiu múltiplas verificações, pesquisas ou decisões de roteamento ao longo do fluxo.

Em contraste, o teste **"qual o número da Savana Moia da IBM?"** foi resolvido de forma muito mais direta. O agente executou apenas **2 etapas**, realizou **1 chamada ao modelo de linguagem** e não utilizou nenhuma ferramenta externa. Isso indica um fluxo simples, com baixa complexidade operacional.

Apesar da diferença significativa de processamento, ambos os testes apresentaram os mesmos indicadores de qualidade:

- **Succeeded**: o teste foi aprovado.
- **Orchestrate agent routing F1 = 1**: o roteamento ocorreu corretamente.
- **Text match = Summary Matched**: a resposta gerada correspondeu ao resultado esperado.
- **Journey success = Yes**: a jornada foi concluída com sucesso.
- **Journey completion = 1**: a execução atingiu 100% de conclusão.

A diferença também aparece no tempo de resposta. O teste mais simples levou aproximadamente **5,2 segundos**, enquanto o teste que envolveu múltiplas etapas e ferramentas levou cerca de **7,4 segundos**.

Esse tipo de análise é útil para identificar oportunidades de otimização. Quando um teste apresenta um número muito elevado de etapas, chamadas ao modelo ou uso excessivo de ferramentas, pode ser um indicativo de que o agente está seguindo um caminho mais complexo do que o necessário para resolver a solicitação.

![test](../../Assets_for_BuildBooks/labs/lab04/lab04_15.png)

Cada teste também tem um menu de opções, acessível pelo ícone de três pontos, com a ação `Re-run test`. Use-a quando quiser reexecutar um único caso, por exemplo depois de ajustar as instruções do agente, sem precisar rodar a bateria inteira novamente.

![test](../../Assets_for_BuildBooks/labs/lab04/lab04_16.png)

Ao clicar em Re-run test, a mesma notificação de avaliação em andamento aparece, desta vez para um teste só.

![test](../../Assets_for_BuildBooks/labs/lab04/lab04_17.png)

Na aba Tests, o campo Last run do teste reexecutado é atualizado com o novo horário, confirmando que ele rodou de forma isolada.

![test](../../Assets_for_BuildBooks/labs/lab04/lab04_18.png)

![test](../../Assets_for_BuildBooks/labs/lab04/lab04_19.png)

Volte para a sub-aba Evaluations.

![test](../../Assets_for_BuildBooks/labs/lab04/lab04_20.png)

Agora a lista mostra duas execuções, a avaliação completa com os quatro testes e a reexecução isolada logo acima, com seu próprio horário e taxa de sucesso.

![test](../../Assets_for_BuildBooks/labs/lab04/lab04_21.png)

Abra a execução mais recente para conferir que ela contém apenas o teste reexecutado, com cem por cento de sucesso.

![test](../../Assets_for_BuildBooks/labs/lab04/lab04_22.png)

Expanda o teste para ver seus detalhes e, quando terminar de revisar, feche a janela de resultados.

![test](../../Assets_for_BuildBooks/labs/lab04/lab04_23.png)

Você pode clicar em `Download`, no painel de resultados, para baixar os dados de qualquer execução e analisá-los posteriormente.

## Resumo

Parabéns!  🎉  Você concluiu o laboratório de avaliação de agentes no watsonx Orchestrate.

Ao longo do laboratório, você deu feedback direto sobre uma resposta bloqueada pelo PII Filter, transformou conversas reais em casos de teste reutilizáveis usando Save as test com Response summary, executou uma avaliação completa com Evaluate all, revisou as onze métricas agregadas de uma execução e os detalhes individuais de cada teste, reexecutou um caso isolado com Re-run test e comparou o histórico de execuções na aba Evaluations.

Com isso, você agora sabe transformar interações do dia a dia em uma bateria de testes automatizada, interpretar métricas de roteamento, execução, uso de ferramentas e sucesso de jornada, e usar esses recursos para acompanhar a qualidade de um agente sempre que ele for alterado.

### Sou desenvolvedor e quero me aprofundar no watsonx Orchestrate

Todas as operações realizadas também estão disponíveis em uma experiência utilizando o ADK, o Agent Development Kit. [Clique aqui](https://developer.watson-orchestrate.ibm.com/) para saber mais sobre como criar agentes, tools, bases de conhecimento e muito mais.

## Próximos Passos

➜ [Clique aqui para navegar para o próximo lab, Monitorando Agentes em Tempo Real com watsonx Orchestrate](./Step_by_Step_Lab5.md)
