# Conectando a ADK(Agente Development Kit) ao seu tenant e criando o primeiro agente em código

---

## Visão Geral

No **watsonx Orchestrate**, você pode criar agentes diretamente pela interface utilizando formulários de uma forma simples e intuitiva. No entanto, essa não é a única forma de desenvolver agentes, tools e fluxos.

O **Agent Development Kit (ADK)** amplia e as possibilidades de desenvolvimento. Com ele, agentes, tools e outros recursos passam a ser definidos como código e arquivos de configuração, podendo ser armazenados em repositórios Git, versionados, revisados por meio de pull requests e promovidos entre ambientes através de pipelines de CI/CD.

**Mas o principal benefício do ADK não é apenas transformar agentes em código.**

O ADK também permite acessar capacidades que não estão disponíveis na interface gráfica do produto. Isso inclui maior flexibilidade para estruturar projetos, reutilizar componentes, criar integrações customizadas, automatizar processos de deployment e trabalhar com recursos avançados de desenvolvimento e governança.

O ponto não é digitar menos, é conseguir repetir, controlar e escalar. Com uma abordagem baseada em código, é possível recriar exatamente o mesmo agente em qualquer tenant, comparar versões com ferramentas de diff, automatizar implantações e estabelecer práticas de desenvolvimento mais alinhadas à engenharia de software moderna.

Neste guia você instalará o ADK, conectará a CLI ao tenant do **watsonx Orchestrate** e publicará recursos diretamente pelo terminal, entendendo como aproveitar as vantagens do modelo _as code_ além das capacidades disponíveis na interface visual.

---

## O que é a ADK

É um pacote Python chamado `ibm-watsonx-orchestrate`. Depois de instalado, você tem um comando chamado `orchestrate` que fala diretamente com a API do tenant.

### O que você declara em código

| Tipo | O que é | Como você declara |
|---|---|---|
| **Agente nativo** | Recebe a pergunta, decide o que chamar e monta a resposta. | Arquivo YAML com `kind: native` |
| **Tool** | Uma ação concreta: calcular, consultar, validar algo. | Função Python com o decorador `@tool` |
| **Collaborator** | Outro agente que este pode acionar como se fosse uma tool. | Lista `collaborators` no YAML |
| **Guideline** | Regra determinística no formato "quando X, faça Y". | Bloco `guidelines` no YAML |

Esses quatro tipos cobrem toda a trilha: agente no ADK 1, tools no ADK 2, composição multiagente no ADK 3.

### O que este laboratório não usa

A ADK também suporta bancos de dados, APIs externas, servidores MCP, OAuth e a Developer Edition (versão local da plataforma). **Nada disso entra aqui.**

> **Escopo desta trilha:** Você vai usar **só o watsonx Orchestrate e Python**. Sem Docker, sem banco de dados, sem API externa, sem credenciais de terceiros. As tools terão os dados dentro do próprio arquivo, e a CLI aponta para o tenant que você já tem.

---

## Preparando o ambiente

### Conferindo a versão do Python

A ADK precisa de **Python 3.11 ou mais recente**.

Abra o seu terminal o terminal no Visual Studio Code ou editor de sua prefêrencia e execute o seguinte comando:

```bash
python --version
```

Se estiver numa versão anterior, instale antes de continuar. No macOS: `brew install python@3.11`. No Windows: baixe o instalador oficial e marque "Add Python to PATH". No Linux: use o gerenciador de pacotes da distro, `pyenv` ou `uv`.

### Criando a pasta do projeto

Crie a pasta e um ambiente virtual, isso evita que a ADK misture com outros projetos Python da sua máquina:

```bash
mkdir -p wxo-adk-lab/{agents,tools}
cd wxo-adk-lab
python -m venv .venv
source .venv/bin/activate
```

No Windows: `.\.venv\Scripts\activate`.

> **Nota:** Cada vez que abrir um terminal novo, entre na pasta e ative o venv de novo. Sem isso, o comando `orchestrate` não vai existir no PATH.

### Instalando a ADK

```bash
pip install --upgrade ibm-watsonx-orchestrate
```

Quando terminar, confirme:

```bash
orchestrate --version
```

Anote o número — versão é sempre a primeira pergunta quando algo dá errado. O `--help` funciona em qualquer nível do comando:

```bash
orchestrate --help
orchestrate agents --help
orchestrate agents import --help
```

---

## Conectando ao seu tenant

A ADK precisa saber para onde enviar o que você escreve. Essa configuração se chama **ambiente**, e você pode ter vários cadastrados ao mesmo tempo.

### Obtendo a URL e a API key


Faça login na IBM Cloud

Na página inicial, acesse o menu lateral esquerdo como indicado na imagem a seguir

[ADK - Lab 1](../../Assets_for_BuildBooks/labs/adk1/lab-adk-1-01.png)

Na lista de recursos, em `Product` digite `watsonx Orchestrate`

Clique no recurso disponível

>[!TIP]
> Caso tenha mais de um recurso, busque o seu instrutor para obter informações sobre qual deve utilizar.
> Se a conta for sua, utilize o de sua preferência

[ADK - Lab 1](../../Assets_for_BuildBooks/labs/adk1/lab-adk-1-02.png)

Na página de gerenciamento de sua instância:

1. Copie o valor da API Key do produto e guarde em um local seguro.

> [!TIP]
> Essa API Key pertence ao produto e faz uso somente dele, caso queira uma API Key com mais acessos e permissões, utilize uma API Key da conta, você pode obter mais informações acessando este link: https://www.ibm.com/docs/pt-br/masv-and-l/cd?topic=cli-creating-your-cloud-api-key
>

2. Clique em `Lauch watsonx Orchestrate`

[ADK - Lab 1](../../Assets_for_BuildBooks/labs/adk1/lab-adk-1-03.png)

Na interface do watsonx Orchestrate, clique no ícone do seu usuário (canto superior direito) 

[ADK - Lab 1](../../Assets_for_BuildBooks/labs/adk1/lab-adk-1-04.png)

Clique em **Settings**.

[ADK - Lab 1](../../Assets_for_BuildBooks/labs/adk1/lab-adk-1-04-2.png)

Vá para a aba **API details**.

[ADK - Lab 1](../../Assets_for_BuildBooks/labs/adk1/lab-adk-1-05.png)

Copie a **service instance URL**, algo no formato `https://api.<região>.watson-orchestrate.ibm.com/instances/<id>`.

[ADK - Lab 1](../../Assets_for_BuildBooks/labs/adk1/lab-adk-1-06.png)

### Registrando e ativando o ambiente

Registre a instância com o nome que quiser. O `--activate` já deixa esse ambiente como o ativo:

```bash
orchestrate env add -n workshop -u <service-instance-url> --type ibm_iam --activate
```

O `--type` depende de onde roda a instância:

| Onde roda | Valor de `--type` |
|---|---|
| IBM Cloud | `ibm_iam` |
| AWS | `mcsp` |
| On-premises | `cpd` |

Na maioria dos casos a ADK detecta o tipo pela URL e você pode omitir a flag. Na primeira ativação, a CLI vai pedir a API key.

Confira se funcionou:

```bash
orchestrate env list
```

O ambiente ativo aparece marcado. Para trocar: `orchestrate env activate <nome>`.

> **Atenção:** Todo comando age sobre o **ambiente ativo**. Antes de qualquer import, rode `orchestrate env list` e confira. São dois segundos que evitam publicar no tenant errado.

---

## Seu primeiro agente em YAML

O cenário da trilha é o suporte ao cliente de um produto fictício chamado **Nexus Pro**: um agente que responde dúvidas sobre planos, funcionalidades e conta. Nos próximos labs ele vai ganhar tools e um agente especialista. Por enquanto, ele só conversa — e está certo assim. A ideia aqui é fechar o ciclo **escrever → importar → testar** antes de adicionar qualquer coisa.

> **Tenant compartilhado:** Se mais de uma pessoa está usando o mesmo tenant, acrescente suas iniciais ao `name` — `suporte_cliente_ana`, por exemplo — e ajuste os comandos nos próximos labs. Dois participantes com o mesmo nome sobrescrevem o trabalho um do outro.

### Anatomia do arquivo

Crie o arquivo `agents/suporte_cliente.yaml` com o conteúdo abaixo:

```yaml
spec_version: v1
kind: native
name: suporte_cliente
description: >
  Agente de suporte ao cliente do Nexus Pro. Atende dúvidas sobre
  planos disponíveis, funcionalidades do produto e situação da conta.
instructions: |
  Você é o assistente de suporte ao cliente do Nexus Pro.

  Regras de conduta:
  - Responda sempre em português do Brasil, em tom cordial e direto.
  - Nunca invente informações sobre planos, preços ou funcionalidades.
    Se não tiver o dado, diga exatamente o que precisa saber para responder.
  - Não resolva problemas técnicos complexos. Se o cliente descrever
    um erro ou falha, oriente a abrir um chamado com a equipe técnica.
  - Feche respostas longas com uma frase curta de próximo passo.
llm: groq/openai/gpt-oss-120b
style: react_core
memory_enabled: false
hide_reasoning: false
collaborators: []
tools: []
```

O que cada campo faz:

| Campo | Para que serve |
|---|---|
| `spec_version` | Versão do formato. Use `v1`. |
| `kind` | `native` para agentes do watsonx Orchestrate. |
| `name` | Identificador único — letras, números e underscore. O import usa esse campo para decidir se cria ou atualiza. |
| `description` | Lida por **outros agentes** quando este é usado como colaborador. Não afeta respostas diretas ao usuário. |
| `instructions` | O comportamento do agente: tom, limites, quando acionar tools. |
| `llm` | Provedor e modelo no formato `provedor/modelo`. |
| `style` | Use `react_core`. Os valores `default`, `react` e `planner` estão obsoletos. |
| `memory_enabled` | Mantém histórico entre turnos da sessão. Padrão: desligado. |
| `hide_reasoning` | Esconde o raciocínio do usuário final. Deixe visível enquanto estiver desenvolvendo. |

> **Description e instructions não são a mesma coisa:** Misturar os dois é a causa mais comum de agente que nunca é acionado como colaborador. A **description** é o que os outros agentes leem para saber o que este resolve. As **instructions** são o manual interno: como ele se comporta. No ADK 3 essa diferença vai ficar clara na prática.

### Importando o agente

Da raiz do projeto:

```bash
orchestrate agents import -f agents/suporte_cliente.yaml
```

Se der certo, a CLI confirma. Liste para checar:

```bash
orchestrate agents list
```

Com `-v` você vê o JSON completo — útil quando quer saber exatamente o que foi salvo na plataforma:

```bash
orchestrate agents list -v
```

### Testando pela CLI

Sem precisar abrir o navegador:

```bash
orchestrate chat ask --agent-name suporte_cliente "Quem é você e o que consegue fazer?"
```

Agora teste se as instruções estão valendo. Mande algo que ele não tem como saber:

```bash
orchestrate chat ask --agent-name suporte_cliente "Qual é o melhor plano para mim?"
```

O comportamento esperado é ele perguntar o que o cliente precisa fazer com o produto, não inventar uma recomendação. Se inventar, a instrução de "não inventar informações" está fraca — reescreva antes de continuar.

Para ver o raciocínio por trás da resposta:

```bash
orchestrate chat ask --agent-name suporte_cliente "Qual é o melhor plano para mim?" --include-reasoning
```

> **Dica:** O `--include-reasoning` vai ser muito útil no ADK 2, quando as tools entrarem. É por ele que você descobre se o agente escolheu a tool certa e com quais argumentos.

### Vendo o agente na interface

Abra o watsonx Orchestrate no navegador e vá ao **Agent Builder**. O `suporte_cliente` aparece lá, igual a qualquer agente criado pela interface, com o mesmo chat de teste e os mesmos campos editáveis.

> **Cuidado com edições paralelas:** Se você editar pela interface e reimportar o YAML depois, o arquivo vence — o import sempre sobrescreve. Escolha uma fonte da verdade e fique nela. Em projeto de código, essa fonte é o arquivo.

---

## Atualizando e removendo

Não existe comando de atualização separado. O import é idempotente por nome: rodar de novo com o mesmo `name` atualiza o que já existe.

Teste isso agora. Mude alguma linha das `instructions` — adicione, por exemplo, que o agente deve sempre mencionar o nome do produto ao se apresentar — e importe de novo:

```bash
orchestrate agents import -f agents/suporte_cliente.yaml
```

Converse e confirme que o comportamento mudou. Esse ciclo rápido de editar e reimportar é o ritmo normal de trabalho com a ADK.

Se quiser uma confirmação antes de sobrescrever algo que já existe:

```bash
orchestrate agents import -f agents/suporte_cliente.yaml --safe
```

Para remover:

```bash
orchestrate agents remove -n suporte_cliente -k native
```

> **Nota:** Não remova o agente agora — ele é a base dos próximos dois labs. Esse comando é para quando você quiser limpar o tenant no final da trilha.

---

## Resumo

ADK instalada, CLI conectada ao tenant, agente publicado sem abrir a interface uma vez. Os comandos do dia a dia:

| Comando | O que faz |
|---|---|
| `orchestrate env list` | Lista os ambientes e mostra qual está ativo |
| `orchestrate env activate <nome>` | Troca o ambiente ativo |
| `orchestrate agents import -f <arquivo>` | Cria ou atualiza um agente |
| `orchestrate agents list -v` | Lista agentes com detalhe completo |
| `orchestrate agents remove -n <nome> -k native` | Remove um agente |
| `orchestrate chat ask --agent-name <nome> "..."` | Conversa com o agente pelo terminal |
| `orchestrate --debug <comando>` | Saída detalhada para investigar erros |

### Antes de seguir, confirme

- `orchestrate --version` retorna um número de versão
- `orchestrate env list` mostra o seu ambiente como ativo
- `orchestrate agents list` inclui o `suporte_cliente`
- O agente responde no `chat ask` e pede as informações que faltam em vez de inventar
- O agente aparece no Agent Builder

---

## Próximos Passos

No **ADK 2**, o agente ganha tools em Python puro. Você vai ver como o modelo enxerga a assinatura de uma função e por que a docstring acaba sendo a parte mais crítica do código.

> **Documentação:** A referência oficial da ADK está em [developer.watson-orchestrate.ibm.com](https://developer.watson-orchestrate.ibm.com/), versionada por release.
