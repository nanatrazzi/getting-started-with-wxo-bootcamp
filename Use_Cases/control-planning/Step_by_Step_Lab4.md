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



![test](../../Assets_for_BuildBooks/labs/lab04/lab04_13.png)

Já o teste `qual o número da ibm?` percorreu um caminho bem mais longo, trinta e oito passos no total, com vinte e duas chamadas ao modelo de linguagem e seis chamadas de ferramenta, levando cerca de sete segundos e meio para responder. A diferença mostra como perguntas parecidas podem levar o agente por raciocínios de tamanhos muito distintos.

![test](../../Assets_for_BuildBooks/labs/lab04/lab04_14.png)

Os outros dois testes, `qual o número da Savana Moia da IBM?` e `Qual o número da IBM?`, seguem o padrão mais simples, dois passos e uma única chamada ao modelo, com tempos de resposta próximos de cinco segundos.

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

Abaixo está um detalhamento das métricas que você viu ao longo deste laboratório e o que cada uma significa.

Nas métricas de roteamento e precisão, o Orchestrate agent routing F1 mede, através da média harmônica entre precisão e recall, quão corretamente o agente mestre roteia as consultas para os agentes especializados. O Keyword match verifica se a resposta contém as palavras-chave esperadas, o Semantic match avalia se a resposta é semanticamente parecida com a saída esperada, e o Text match verifica se a resposta corresponde exatamente à saída de texto esperada, ou, quando o Response summary está habilitado, se ela bate com o resumo gerado.

Nas métricas de execução, o Total steps é o número total de ações realizadas ao longo dos testes, o LLM steps é quantas vezes o modelo de linguagem foi invocado para gerar respostas, e o Average agent response time mede, em segundos, o tempo médio para gerar cada resposta.

Nas métricas de uso de ferramentas, o Total tool calls conta quantas vezes agentes ou ferramentas externas foram acionados durante os testes, o Expected tool calls indica quantas chamadas de ferramenta eram esperadas, o Correct tool calls quantas foram feitas corretamente, o Missed tool calls quantas chamadas esperadas não ocorreram, e o Tool calls with incorrect parameters quantas chamadas foram feitas com parâmetros errados. O Tool call recall mostra a proporção de chamadas necessárias que de fato aconteceram, medindo se todas as ferramentas necessárias estão sendo usadas, o Tool call precision mostra a proporção de chamadas relevantes em relação ao total de chamadas feitas, medindo se as ferramentas estão sendo chamadas de forma apropriada, e o Tool match success indica se as ferramentas corretas foram chamadas.

Nas métricas de sucesso, o Journey success indica se o cenário de teste completo alcançou o resultado pretendido, e o Journey completion indica se uma interação de múltiplas etapas foi concluída sem erros.

Você pode clicar em `Download`, no painel de resultados, para baixar os dados de qualquer execução e analisá-los posteriormente.

## Resumo

Parabéns!  🎉  Você concluiu o laboratório de avaliação de agentes no watsonx Orchestrate.

Ao longo do laboratório, você deu feedback direto sobre uma resposta bloqueada pelo PII Filter, transformou conversas reais em casos de teste reutilizáveis usando Save as test com Response summary, executou uma avaliação completa com Evaluate all, revisou as onze métricas agregadas de uma execução e os detalhes individuais de cada teste, reexecutou um caso isolado com Re-run test e comparou o histórico de execuções na aba Evaluations.

Com isso, você agora sabe transformar interações do dia a dia em uma bateria de testes automatizada, interpretar métricas de roteamento, execução, uso de ferramentas e sucesso de jornada, e usar esses recursos para acompanhar a qualidade de um agente sempre que ele for alterado.

### Sou desenvolvedor e quero me aprofundar no watsonx Orchestrate

Todas as operações realizadas também estão disponíveis em uma experiência utilizando o ADK, o Agent Development Kit. [Clique aqui](https://developer.watson-orchestrate.ibm.com/) para saber mais sobre como criar agentes, tools, bases de conhecimento e muito mais.

## Próximos Passos

➜ [Clique aqui para navegar para o próximo lab, Monitorando Agentes em Tempo Real com watsonx Orchestrate](./Step_by_Step_Lab5.md)
