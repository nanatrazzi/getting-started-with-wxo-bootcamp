# ADK 3 · Multiagente, regras determinísticas e publicação repetível

> Divida o agente em especialistas que delegam entre si, controle comportamento com guidelines, configure a tela de entrada do chat e transforme tudo em um script que recria a solução do zero.

---

## Visão Geral

No ADK 2 você construiu um agente com duas tools: uma que verifica elegibilidade de upgrade e outra que consulta detalhes dos planos. Funciona bem com duas tools. Com dez, começa a hesitar.

Neste lab você refatora a solução em dois agentes, do jeito que ela cresceria em um projeto real: um **especialista em planos**, que carrega as tools e domina o assunto, e um **agente de atendimento**, que conversa com o cliente e delega quando a pergunta envolve análise de conta.

Depois disso, você acrescenta as duas coisas que separam um protótipo de algo que vai para produção: **guidelines**, para regras de negócio que não podem depender da boa vontade do modelo, e um **script de publicação**, para que a solução inteira possa ser recriada do zero em qualquer tenant.

---

## Por que dividir em vários agentes

Um agente escolhe a tool comparando descrições. Quanto mais tools disputam a atenção dele, mais contexto ele precisa processar por turno e maior a chance de escolher errado. Dividir por domínio resolve isso: cada agente enxerga poucas tools e um agente de nível acima decide para quem mandar a pergunta.

| Sinal de que é hora de dividir | O que fazer |
|---|---|
| O agente passou de seis ou sete tools | Agrupe por assunto e crie um especialista para cada grupo |
| Tools de assuntos diferentes estão sendo confundidas | Separe em agentes distintos, com descrições que se excluem |
| As `instructions` viraram um documento longo com regras de vários domínios | Cada especialista carrega as regras do próprio domínio |
| Times diferentes cuidam de partes diferentes da solução | Um agente por time, com repositórios e ciclos de release próprios |

> **Divisão também tem custo:** Cada delegação é uma rodada a mais de raciocínio, e isso aparece em latência. Dividir dois agentes com uma tool cada não faz sentido. Divida quando a confusão entre tools ou o tamanho das instruções começar a atrapalhar — não antes.

---

## Criando o agente especialista

O especialista recebe as duas tools que você escreveu no ADK 2 e as regras do domínio de planos. Crie `agents/especialista_planos.yaml`:

```yaml
spec_version: v1
kind: native
name: especialista_planos
description: |
  Especialista em planos do Nexus Pro. Acione este agente quando
  a pergunta envolver elegibilidade para upgrade, comparação de planos,
  funcionalidades incluídas, preço ou limites de uso.
  Não trata problemas técnicos, falhas no produto nem solicitações
  de cancelamento.
instructions: |
  Você analisa planos e uso de conta do Nexus Pro.

  - Para saber se o cliente é elegível para upgrade, use verificar_elegibilidade_upgrade.
    Nunca tome essa decisão de cabeça.
  - Para detalhar funcionalidades e preços, use consultar_detalhes_plano.
  - Se faltar o plano atual, o percentual de uso ou o tempo no plano,
    peça exatamente o que falta.
  - Responda em no máximo três frases, com os dados que vieram das tools.
llm: groq/openai/gpt-oss-120b
style: react_core
collaborators: []
tools:
  - verificar_elegibilidade_upgrade
  - consultar_detalhes_plano
```

> **A description é o roteador:** É por esse texto que o agente principal decide delegar. Repare que ele diz o que o especialista faz **e o que ele não faz**. A segunda parte evita delegação errada tanto quanto a primeira garante a delegação certa — e é justamente a parte que quase todo mundo esquece de escrever.

---

## Transformando o agente principal em roteador

O `suporte_cliente` perde as tools e ganha um colaborador. Ele passa a cuidar da conversa com o cliente e do que fazer quando a pergunta sai do escopo técnico.

### Guidelines: regras que não podem falhar

Instruções em texto livre orientam o modelo, mas não garantem nada. Quando existe uma regra de negócio que precisa valer sempre, ela vira uma **guideline**, no formato *quando a condição X acontecer, faça Y*.

| Campo | Para que serve |
|---|---|
| `condition` | A situação que dispara a regra. Obrigatório. |
| `action` | O que o agente deve fazer quando a condição for atendida. |
| `tool` | Tool a ser acionada, quando a regra exigir uma. Opcional. |

A ordem importa: as guidelines são avaliadas na sequência em que aparecem na lista.

> **Guidelines custam:** Cada guideline acrescenta uma chamada de modelo por turno da conversa, o que significa mais latência e mais consumo de tokens. Use guidelines para o que precisa ser determinístico. Para tom de voz, formato e preferências gerais, as `instructions` resolvem sem custo adicional.

### Mensagem de boas-vindas e atalhos

Dois blocos opcionais controlam a primeira tela do chat. O `welcome_content` define a saudação e a linha de apoio. O `starter_prompts` define até três atalhos clicáveis, que enviam uma mensagem pronta como se o usuário tivesse digitado.

Vale mais do que parece: os atalhos ensinam o usuário a fazer perguntas que o agente sabe responder bem, e reduzem a chance de a primeira interação ser frustrante.

### O arquivo completo

Substitua o conteúdo de `agents/suporte_cliente.yaml` por este:

```yaml
spec_version: v1
kind: native
name: suporte_cliente
description: >
  Agente de suporte ao cliente do Nexus Pro. Primeiro ponto de contato
  para dúvidas sobre planos, conta e uso do produto.
instructions: |
  Você é o assistente de suporte ao cliente do Nexus Pro e é o primeiro
  contato do cliente.

  Regras de conduta:
  - Responda sempre em português do Brasil, em tom cordial e direto.
  - Nunca invente informações sobre planos, preços ou funcionalidades.
  - Toda pergunta sobre elegibilidade para upgrade, comparação de planos
    ou limites de uso deve ser encaminhada ao especialista_planos.
  - Não resolva problemas técnicos complexos. Se o cliente descrever
    um erro ou falha, oriente a abrir um chamado com a equipe técnica.
  - Ao juntar informação de mais de uma fonte, responda em um parágrafo só.
llm: groq/openai/gpt-oss-120b
style: react_core
memory_enabled: true
hide_reasoning: false

collaborators:
  - especialista_planos

tools: []

guidelines:
  - condition: "O cliente descreve um erro, falha, bug ou comportamento inesperado no produto."
    action: "Não tente diagnosticar nem resolver. Oriente o cliente a abrir um chamado técnico e forneça o caminho para isso."
  - condition: "O cliente pergunta o preço exato de um plano ou solicita desconto."
    action: "Forneça o preço tabelado consultando o especialista_planos. Não ofereça desconto nem negocie valores."
  - condition: "O cliente demonstra insatisfação com o produto ou com um atendimento anterior."
    action: "Reconheça o problema em uma frase, sem justificar, e ofereça registrar o caso com o time de sucesso do cliente."

welcome_content:
  welcome_message: "Suporte Nexus Pro."
  description: "Posso ajudar com dúvidas sobre planos, funcionalidades e sua conta."

starter_prompts:
  prompts:
    - id: "upgrade"
      title: "Devo mudar de plano?"
      subtitle: "Informe seu plano atual e uso mensal"
      prompt: "Estou no plano Basic, uso 90% do limite todo mês há 4 meses. Devo fazer upgrade?"
    - id: "comparar_planos"
      title: "O que tem em cada plano?"
      subtitle: "Funcionalidades e preços"
      prompt: "Qual a diferença entre o plano Pro e o Enterprise?"
    - id: "problema_tecnico"
      title: "Encontrei um problema"
      subtitle: "Erros e falhas precisam de chamado técnico"
      prompt: "O produto está retornando um erro ao tentar exportar meus dados."
```

Repare no terceiro atalho: ele leva o cliente direto para o caminho em que a primeira guideline dispara. É uma forma de testar a regra e, ao mesmo tempo, dar ao usuário um caminho seguro para um assunto delicado.

---

## Ordem de importação e script de deploy

Dependência precisa existir antes de quem depende dela. A ordem é sempre a mesma: **tools, depois colaboradores, depois o agente principal**.

1. **Tools**, porque o especialista as referencia pelo nome.
2. **Agentes colaboradores**, porque o principal os referencia pelo nome.
3. **Agente principal**, que amarra tudo.

Como essa sequência vai se repetir toda vez, escreva-a uma vez só. Crie `deploy.sh` na raiz do projeto:

```bash
#!/usr/bin/env bash
set -e

echo "Ambiente ativo:"
orchestrate env list

echo "--- tools ---"
orchestrate tools import -k python -f tools/verificar_elegibilidade_upgrade.py
orchestrate tools import -k python -f tools/consultar_detalhes_plano.py

echo "--- agentes colaboradores ---"
orchestrate agents import -f agents/especialista_planos.yaml

echo "--- agente principal ---"
orchestrate agents import -f agents/suporte_cliente.yaml

echo "Publicacao concluida."
```

Dê permissão de execução e rode:

```bash
chmod +x deploy.sh
./deploy.sh
```

> **Dica:** Esse script é a menor versão possível de um pipeline. Ele já resolve o problema real: qualquer pessoa do time, em qualquer máquina, recria a solução inteira com um comando. A partir daqui, colocá-lo em uma esteira de CI/CD é só uma questão de onde ele roda.

---

## Testando a delegação

Comece pela pergunta que deve sair do agente principal e chegar ao especialista:

```bash
orchestrate chat ask --agent-name suporte_cliente \
  "Estou no plano Basic, uso 90% todo mês há 4 meses. Devo fazer upgrade e o que ganha no Pro?" \
  --include-reasoning
```

No raciocínio você deve ver a delegação para o `especialista_planos` e, dentro dela, as chamadas das duas tools. Se o agente principal tentar responder sozinho, ajuste a `description` do especialista: ela está vaga ou não deixa claro que aquele assunto é dele.

Agora teste cada guideline:

```bash
orchestrate chat ask --agent-name suporte_cliente \
  "O produto está retornando um erro ao tentar exportar meus dados."
```

Resposta esperada: nenhuma tentativa de diagnóstico, orientação para abrir chamado técnico.

```bash
orchestrate chat ask --agent-name suporte_cliente \
  "Vocês têm desconto para o plano Pro?"
```

Resposta esperada: preço tabelado sem negociação, sem promessa de desconto.

Por fim, abra o agente no **Agent Builder** e confira a tela inicial do chat: a mensagem de boas-vindas e os três atalhos que você configurou devem estar lá. Clique no terceiro e veja a guideline de problema técnico disparar na interface.

> **Quando a guideline não dispara:** Guidelines são avaliadas por um modelo, não por regex. Se uma não dispara, quase sempre a `condition` está abstrata demais. Descreva a situação com as palavras que o usuário realmente usaria — "erro, falha, bug" funciona melhor do que "problema no sistema".

---

## Exportando o que você construiu

O caminho inverso também existe. A ADK exporta um agente com todas as suas dependências, o que é útil para levar uma solução de um tenant para outro ou para trazer para o código algo que alguém montou pela interface.

```bash
orchestrate agents export -n suporte_cliente -k native -o build/suporte_cliente.zip
```

O ZIP traz o agente, os colaboradores e as tools. Se você quiser apenas o YAML do agente, sem as dependências:

```bash
orchestrate agents export -n suporte_cliente -k native \
  -o build/suporte_cliente.yaml --agent-only
```

> **Dica:** Exportar é uma boa forma de aprender a sintaxe: monte algo pela interface, exporte e leia o YAML gerado. É o mesmo formato que você escreve à mão.

---

## O que vem depois

O agente que você publicou por linha de comando é indistinguível de qualquer outro no tenant — aparece no Agent Builder, pode ser avaliado, monitorado e protegido com guardrails da mesma forma que qualquer agente criado pela interface.

A ADK muda como o agente é construído, não como ele é operado. A partir daqui, os próximos assuntos naturais para evoluir a solução são:

| Assunto | Para que serve |
|---|---|
| **Guardrails e PII** | Aplicar controles de segurança e filtros de dados sensíveis ao agente publicado pela ADK |
| **Avaliação** | Criar casos de teste para as perguntas que o agente deve responder e medir se as tools certas são chamadas na ordem certa |
| **Agentic Control Plane** | Acompanhar adoção, qualidade e consumo de tokens, e observar o custo das guidelines em FinOps |

---

## Resumo

Você dividiu a solução em dois agentes, escreveu regras determinísticas com guidelines, configurou a experiência de entrada do chat e transformou a publicação em um script repetível.

### O que ficou no seu projeto

```
wxo-adk-lab/
├── deploy.sh
├── agents/
│   ├── suporte_cliente.yaml
│   └── especialista_planos.yaml
└── tools/
    ├── verificar_elegibilidade_upgrade.py
    └── consultar_detalhes_plano.py
```

Cinco arquivos de texto que descrevem uma solução multiagente inteira. Eles cabem em um repositório Git, passam por revisão de código e recriam o mesmo resultado em qualquer tenant.

### Antes de encerrar, confirme

- `orchestrate agents list` mostra os dois agentes
- A pergunta sobre planos é delegada ao especialista, e isso aparece no raciocínio
- As três guidelines disparam nas situações previstas
- A tela inicial do chat mostra a saudação e os três atalhos
- O `deploy.sh` roda do início ao fim sem erro
- O ZIP exportado existe em `build/`

---

## Próximos Passos

Esta trilha cobriu o núcleo da ADK com o mínimo de dependências possível. A partir daqui, os próximos assuntos naturais envolvem sistemas externos:

| Assunto | Para que serve |
|---|---|
| **Connections** | Guardar credenciais na plataforma e injetá-las na tool em execução, para que ela chame APIs reais sem segredo no repositório |
| **Tools OpenAPI** | Transformar uma especificação de API existente em tools, sem escrever Python |
| **Agentic workflows** | Processos determinísticos com ramificação, paralelismo e aprovação humana, expostos ao agente como tool |
| **Knowledge bases** | Responder com base em documentos indexados, com citação da fonte |
| **Toolkits MCP** | Conectar servidores Model Context Protocol locais ou remotos |
| **Developer Edition** | Rodar a plataforma inteira localmente para iterar sem tocar em um tenant compartilhado |

> **Documentação:** A referência oficial fica em [developer.watson-orchestrate.ibm.com](https://developer.watson-orchestrate.ibm.com/), versionada por release — consulte a página **What's new**, já que a ADK é atualizada com frequência.
