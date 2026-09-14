# ADK 2 · Escrevendo tools em Python e ligando ao seu agente

> Transforme funções Python em capacidades do agente com o decorador `@tool`, entenda como o modelo enxerga a sua assinatura e descubra por que a docstring decide se a tool vai ser chamada.

---

## Visão Geral

No ADK 1 você criou um agente que sabe conversar, mas não faz nada além disso. Uma **tool** é o que muda isso: uma função Python que o agente aciona quando decide que precisa dela.

O mecanismo é mais simples do que parece. Você escreve uma função comum, coloca um decorador em cima e importa. A ADK lê a assinatura e a documentação da função e monta sozinha o schema que o modelo vai enxergar. **Você nunca escreve JSON Schema à mão.**

A consequência disso é o ponto central deste lab: como o modelo só conhece a sua tool pelo que a assinatura e a docstring dizem, a qualidade desses dois elementos determina se ele vai chamar a tool certa, na hora certa, com os argumentos certos.

> **Escopo:** As duas tools deste lab são **Python puro**, com os dados dentro do próprio arquivo. Nenhuma delas acessa banco de dados, API externa, disco ou rede. É de propósito: o comportamento que interessa aqui é o do agente, não o da integração.

---

## Como o Orchestrate enxerga uma tool

Uma tool no watsonx Orchestrate é uma função Python decorada com `@tool`, importado do pacote da ADK:

```python
from ibm_watsonx_orchestrate.agent_builder.tools import tool
```

Quando você importa o arquivo, a ADK extrai quatro coisas:

| O que o modelo vê | De onde a ADK tira |
|---|---|
| **Nome da tool** | O nome da função. Pode ser sobrescrito com `@tool(name="...")`. |
| **Descrição** | A docstring da função. É o texto que o modelo lê para decidir se aquela tool serve para a pergunta. |
| **Schema de entrada** | As anotações de tipo dos parâmetros. A descrição de cada um sai da seção `Args:` da docstring. |
| **Schema de saída** | A anotação de tipo do retorno. |

Ou seja: a função é o comportamento, e a documentação é a interface. Um código impecável com docstring genérica produz uma tool que o agente ignora.

> **Formato da docstring:** Use o estilo **Google**, com as seções `Args:` e `Returns:`. É o formato que a ADK sabe interpretar para gerar as descrições de cada parâmetro.

---

## Primeira tool: cálculo puro

O Nexus Pro tem três planos — Basic, Pro e Enterprise — com limites de uso mensais. Verificar se um cliente pode fazer upgrade é uma conta simples, mas que o modelo erra com frequência quando tenta fazer de cabeça — exatamente o tipo de coisa que merece virar tool.

### Escrevendo a função

Crie o arquivo `tools/verificar_elegibilidade_upgrade.py`:

```python
from ibm_watsonx_orchestrate.agent_builder.tools import tool

PLANOS = ["basic", "pro", "enterprise"]
LIMITE_USO_UPGRADE = 80  # percentual mínimo de uso para sugerir upgrade


@tool
def verificar_elegibilidade_upgrade(
    plano_atual: str,
    uso_mensal_percentual: int,
    meses_no_plano: int,
) -> dict:
    """Verifica se um cliente é elegível para upgrade de plano no Nexus Pro.

    Um cliente é elegível quando está usando mais de 80% do limite do plano
    atual há pelo menos 2 meses consecutivos, ou quando já está no plano Basic
    há mais de 6 meses. Use esta tool sempre que o cliente perguntar se deve
    mudar de plano ou se o plano atual está adequado para o uso dele.

    Args:
        plano_atual (str): Plano atual do cliente. Valores aceitos: basic, pro, enterprise.
        uso_mensal_percentual (int): Percentual de uso do limite mensal, de 0 a 100.
        meses_no_plano (int): Quantidade de meses completos desde a contratação do plano atual.

    Returns:
        dict: Chaves elegivel, motivo, proximo_plano e acao_recomendada.
    """
    plano_atual = plano_atual.lower()
    if plano_atual not in PLANOS:
        return {
            "elegivel": False,
            "motivo": f"Plano '{plano_atual}' não reconhecido. Use: basic, pro ou enterprise.",
            "proximo_plano": None,
            "acao_recomendada": "Confirme o plano atual do cliente antes de continuar.",
        }

    if plano_atual == "enterprise":
        return {
            "elegivel": False,
            "motivo": "O cliente já está no plano mais completo.",
            "proximo_plano": None,
            "acao_recomendada": "Verifique se o cliente precisa de funcionalidades adicionais via consultoria.",
        }

    idx = PLANOS.index(plano_atual)
    proximo_plano = PLANOS[idx + 1]

    elegivel_por_uso = uso_mensal_percentual >= LIMITE_USO_UPGRADE
    elegivel_por_tempo = plano_atual == "basic" and meses_no_plano >= 6

    if elegivel_por_uso and elegivel_por_tempo:
        motivo = f"Uso em {uso_mensal_percentual}% do limite e {meses_no_plano} meses no plano Basic."
    elif elegivel_por_uso:
        motivo = f"Usando {uso_mensal_percentual}% do limite mensal há {meses_no_plano} mês(es)."
    elif elegivel_por_tempo:
        motivo = f"{meses_no_plano} meses no plano Basic — bom momento para avaliar o Pro."
    else:
        motivo = f"Uso em {uso_mensal_percentual}% e {meses_no_plano} mês(es) no plano. Nenhum critério atingido."

    return {
        "elegivel": elegivel_por_uso or elegivel_por_tempo,
        "motivo": motivo,
        "proximo_plano": proximo_plano if (elegivel_por_uso or elegivel_por_tempo) else None,
        "acao_recomendada": f"Apresentar o plano {proximo_plano.capitalize()} ao cliente." if (elegivel_por_uso or elegivel_por_tempo) else "Manter plano atual e reavaliar no próximo mês.",
    }
```

Repare que não há nada de especial no corpo da função: é Python comum, sem importar biblioteca externa, sem ler arquivo, sem acessar rede. A única linha da ADK é o import do decorador.

### Importando e inspecionando o schema

```bash
orchestrate tools import -k python -f tools/verificar_elegibilidade_upgrade.py
```

A flag `-k python` indica o tipo da tool e o `-f` aponta o arquivo. Toda função decorada com `@tool` naquele arquivo vira uma tool no seu tenant.

Agora veja o que a plataforma entendeu:

```bash
orchestrate tools list -v
```

Procure pela sua tool na saída e leia o `input_schema`. Você deve encontrar os três parâmetros, cada um com seu tipo e a descrição que você escreveu na seção `Args:`. **Esse é literalmente o texto que o modelo vai ler.** Se ali estiver vago, o agente vai chutar.

> **Uma tool por arquivo:** Nada impede colocar várias funções decoradas no mesmo arquivo, mas manter uma por arquivo deixa você atualizar, versionar e remover cada tool de forma independente. É a organização recomendada.

### Ligando a tool ao agente

Importar a tool não basta: o agente só enxerga o que estiver na lista `tools` do YAML dele. Abra `agents/suporte_cliente.yaml`, acrescente a tool e complemente as instruções para orientar o uso:

```yaml
instructions: |
  Você é o assistente de suporte ao cliente do Nexus Pro.

  Regras de conduta:
  - Responda sempre em português do Brasil, em tom cordial e direto.
  - Nunca invente informações sobre planos, preços ou funcionalidades.
  - Não resolva problemas técnicos complexos. Se o cliente descrever
    um erro ou falha, oriente a abrir um chamado com a equipe técnica.
  - Para saber se o cliente é elegível para upgrade, use sempre a tool
    verificar_elegibilidade_upgrade. Nunca tome essa decisão de cabeça.
  - A tool precisa de três dados: plano atual, percentual de uso mensal
    e meses no plano. Se faltar algum, pergunte antes de chamar a tool.

tools:
  - verificar_elegibilidade_upgrade
```

Reimporte o agente:

```bash
orchestrate agents import -f agents/suporte_cliente.yaml
```

### Testando os três caminhos

Teste primeiro o caminho em que todos os dados estão na pergunta:

```bash
orchestrate chat ask --agent-name suporte_cliente \
  "Estou no plano Basic, uso 90% do limite todo mês e já faz 4 meses. Devo fazer upgrade?" \
  --include-reasoning
```

No raciocínio você deve ver a chamada da tool com os três argumentos preenchidos corretamente. A resposta deve indicar elegibilidade e sugerir o plano Pro.

Agora teste o caminho em que falta informação:

```bash
orchestrate chat ask --agent-name suporte_cliente \
  "Acho que preciso de um plano melhor."
```

O comportamento correto é o agente perguntar o plano atual, o uso e há quanto tempo está no plano, em vez de chamar a tool com valores inventados. Se chutar dados, sua instrução precisa ficar mais explícita.

E o caminho em que a resposta é não:

```bash
orchestrate chat ask --agent-name suporte_cliente \
  "Estou no plano Pro, uso 40% do limite e faz 1 mês. Devo mudar de plano?"
```

Aqui o agente deve dizer que não há necessidade de upgrade no momento, com os números que vieram da tool, não do modelo.

> **Dica:** Sempre que o agente errar, leia o raciocínio antes de mexer no código. Na maioria das vezes o problema não está na função: está na descrição de um parâmetro ou em uma instrução ausente no YAML.

---

## A docstring é o contrato

Vale fazer um experimento rápido para ver o efeito na prática. Troque a docstring da sua tool por uma versão pobre:

```python
    """Verifica upgrade.

    Args:
        plano_atual (str): plano
        uso_mensal_percentual (int): uso
        meses_no_plano (int): meses

    Returns:
        dict: resultado
    """
```

Reimporte a tool e o agente e refaça as mesmas três perguntas. O que costuma acontecer: o agente passa a hesitar entre responder sozinho e chamar a tool, e às vezes confunde os parâmetros porque nada na descrição diz o que cada um significa.

Depois volte para a versão original. A diferença entre as duas é a diferença entre uma tool que funciona e uma que dá suporte na segunda-feira.

| Escreva na docstring | Por quê |
|---|---|
| O que a tool faz **e quando usá-la** | O modelo escolhe entre tools comparando descrições. "Use esta tool quando..." é a frase mais útil que você pode escrever. |
| Unidade e formato de cada parâmetro | Evita que o agente passe metros onde você espera quilômetros, ou texto onde você espera número. |
| Um exemplo de valor válido | Reduz drasticamente o erro de formato em campos com padrão específico. |
| O que a tool **não** faz | Impede que ela seja chamada para perguntas fora do escopo. |

---

## Segunda tool: saída estruturada com Pydantic

Retornar `dict` funciona, mas o modelo precisa deduzir o que vem dentro. Quando o retorno tem forma definida, vale declará-la com **Pydantic**: além de gerar um schema de saída preciso, você ganha validação automática.

O Pydantic já faz parte do runtime das tools, então não é preciso declarar dependência nenhuma.

### Modelos de entrada e saída

Crie `tools/consultar_detalhes_plano.py`. Os planos estão embutidos no próprio arquivo — é a nossa "base de dados", sem banco de dados:

```python
from typing import List, Optional

from pydantic import BaseModel, Field
from ibm_watsonx_orchestrate.agent_builder.tools import tool

CATALOGO = [
    {
        "id": "basic",
        "nome": "Basic",
        "preco_mensal": 29.90,
        "limite_usuarios": 3,
        "limite_requisicoes": 1000,
        "funcionalidades": ["Dashboard básico", "Suporte por e-mail", "Exportação CSV"],
    },
    {
        "id": "pro",
        "nome": "Pro",
        "preco_mensal": 79.90,
        "limite_usuarios": 15,
        "limite_requisicoes": 10000,
        "funcionalidades": ["Dashboard avançado", "Suporte prioritário", "API access", "Integrações"],
    },
    {
        "id": "enterprise",
        "nome": "Enterprise",
        "preco_mensal": 199.90,
        "limite_usuarios": 999,
        "limite_requisicoes": 999999,
        "funcionalidades": ["Tudo do Pro", "SLA garantido", "Gerente de conta", "SSO", "Auditoria completa"],
    },
]


class DetalhesFuncionalidade(BaseModel):
    nome: str = Field(description="Nome do plano.")
    preco_mensal: float = Field(description="Preço mensal em reais.")
    limite_usuarios: int = Field(description="Número máximo de usuários incluídos.")
    limite_requisicoes: int = Field(description="Limite de requisições mensais. 999999 significa ilimitado.")
    funcionalidades: List[str] = Field(description="Lista de funcionalidades incluídas no plano.")


class RespostaConsulta(BaseModel):
    plano: Optional[DetalhesFuncionalidade] = Field(
        None, description="Detalhes do plano consultado, se encontrado."
    )
    observacao: str = Field(description="Explicação curta do resultado da consulta.")


@tool
def consultar_detalhes_plano(nome_plano: str) -> RespostaConsulta:
    """Retorna os detalhes de um plano do Nexus Pro: preço, limites e funcionalidades incluídas.

    Use esta tool quando o cliente perguntar o que está incluído em um plano,
    qual o preço, quantos usuários cabem ou quais funcionalidades estão disponíveis.
    Não use para decidir se o cliente deve fazer upgrade — para isso use
    verificar_elegibilidade_upgrade.

    Args:
        nome_plano (str): Nome do plano a consultar. Valores aceitos: basic, pro, enterprise.

    Returns:
        RespostaConsulta: Detalhes do plano e uma observação sobre a busca.
    """
    plano = next((p for p in CATALOGO if p["id"] == nome_plano.lower()), None)

    if plano is None:
        return RespostaConsulta(
            plano=None,
            observacao=f"Plano '{nome_plano}' não encontrado. Opções disponíveis: basic, pro, enterprise.",
        )

    return RespostaConsulta(
        plano=DetalhesFuncionalidade(**plano),
        observacao=f"Detalhes do plano {plano['nome']} recuperados com sucesso.",
    )
```

### Importando e testando

```bash
orchestrate tools import -k python -f tools/consultar_detalhes_plano.py
```

Rode `orchestrate tools list -v` outra vez e compare o `output_schema` das duas tools. A primeira devolve um `dict` genérico; a segunda descreve cada campo, com tipo e descrição. Quanto mais claro o retorno, menos o modelo precisa interpretar.

Acrescente a nova tool ao agente:

```yaml
tools:
  - verificar_elegibilidade_upgrade
  - consultar_detalhes_plano
```

```bash
orchestrate agents import -f agents/suporte_cliente.yaml
```

Agora faça uma pergunta que exige as duas tools na mesma resposta:

```bash
orchestrate chat ask --agent-name suporte_cliente \
  "Estou no plano Basic, uso 90% todo mês há 4 meses. Devo fazer upgrade e o que ganha no Pro?" \
  --include-reasoning
```

No raciocínio você deve ver as duas chamadas em sequência e uma resposta única que combina os dois resultados. Esse encadeamento é o agente fazendo o trabalho que antes seria de três telas abertas.

---

## Limites do ambiente de execução

Tools rodam em um ambiente isolado e gerenciado pela plataforma. Vale conhecer as restrições antes de esbarrar nelas:

| Restrição | Consequência prática |
|---|---|
| **Sistema de arquivos somente leitura** | A tool não pode gravar arquivos. Se precisar guardar algo, isso tem que sair da tool. |
| **Ambiente isolado por tool** | Cada tool tem seu próprio ambiente virtual. Uma não enxerga o que a outra instalou. |
| **Sem estado entre chamadas** | Não confie em variável global para guardar informação de uma chamada para a próxima. |
| **Dependências travadas** | Se a tool precisar de biblioteca externa, ela vai em um `requirements.txt` com versão exata, validado contra a lista de pacotes permitidos do tenant. |

> **Nota:** Essas restrições são o motivo de as tools deste lab terem os dados embutidos no código. Em um projeto real, a mesma tool consultaria o sistema de origem por API — e aí entram as **connections**, que guardam a credencial fora do repositório.

---

## Armadilhas comuns

| Sintoma | Causa provável | Correção |
|---|---|---|
| O agente nunca chama a tool | A tool não está na lista `tools` do YAML, ou a docstring é genérica demais | Confirme com `tools list` e reescreva a primeira linha da docstring dizendo **quando** usar |
| A tool é chamada com argumento trocado | Descrições de parâmetro parecidas demais entre si | Diferencie explicitamente: "plano atual do cliente" e "percentual de uso do limite mensal" |
| O agente responde por conta própria em vez de usar a tool | Falta instrução no YAML mandando usar | Acrescente uma regra explícita nas `instructions` |
| Erro de import por nome inválido | Nome de função ou de arquivo com hífen, acento ou espaço | Use apenas letras, números e underscore |
| A alteração no código não surtiu efeito | A tool foi reimportada, mas o agente não | Reimporte também o agente sempre que mudar a lista de tools |
| Mensagem de incompatibilidade no import | Versões diferentes da ADK entre membros do time | Padronizem a versão e reimportem |

---

## Resumo

Você escreveu duas tools em Python puro, entendeu como a ADK converte assinatura e docstring em schema e viu o agente encadear duas chamadas para responder uma pergunta.

| Comando | O que faz |
|---|---|
| `orchestrate tools import -k python -f <arquivo>` | Publica as funções decoradas do arquivo como tools |
| `orchestrate tools list` | Lista as tools do ambiente ativo |
| `orchestrate tools list -v` | Mostra os schemas de entrada e saída gerados |
| `orchestrate tools remove -n <nome>` | Remove uma tool |

### Antes de seguir, confirme

- `orchestrate tools list` mostra `verificar_elegibilidade_upgrade` e `consultar_detalhes_plano`
- O `input_schema` da primeira tool traz as descrições que você escreveu
- O agente chama a tool certa com os argumentos certos no caminho completo
- Quando falta um dado, ele pergunta em vez de inventar
- Uma pergunta composta dispara as duas tools e vira uma resposta só

---

## Próximos Passos

No **ADK 3** você divide o agente em dois. Vai criar um especialista que carrega as tools, transformar o agente atual em um roteador que delega, acrescentar regras determinísticas com guidelines e configurar a tela de entrada do chat — tudo pelo mesmo YAML.
