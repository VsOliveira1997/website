---
title: "Skills e MCPs: como adicionar novas capacidades em AIs"
date: 2026-04-16
categories: [AI, Tooling]
tags: [ai, skills, mcp, agents, automation]
---

Muita gente ainda resolve tudo no prompt. Funciona para tarefas curtas, mas fica muito confuso quando a IA precisa repetir um fluxo, seguir um padrão ou conversar com sistemas externos.

Para melhorar um agente, normalmente existem dois caminhos complementares:

- `skills`: ensinam a IA *como* executar uma tarefa
- `MCPs`: uma ponte entre a IA e o programa que estiver rodando, como no exemplo do MCP do Ghidra

Em resumo: skill organiza comportamento; MCP expõe capacidade.

## O que é uma skill

Skill é um pacote de instruções reutilizáveis. Em vez de depender de um prompt gigante toda vez, você define um guia claro para uma tarefa específica:

- quando usar aquela habilidade
- quais passos seguir
- quais cuidados tomar
- qual formato de saída devolver

Na prática, uma skill é boa quando reduz variabilidade. Se você sempre pede a mesma coisa para a IA, provavelmente isso deveria virar uma skill.

Exemplos:

- revisar código com foco em regressão
- fazer triagem inicial de bug bounty
- criar writeups com uma estrutura fixa
- analisar logs e devolver hipóteses priorizadas

Um exemplo simples de `SKILL.md`:

```md
---
name: api-review
description: Revisa endpoints com foco em auth, validação e exposição de dados.
---

Use esta skill quando o usuário pedir revisão de API, threat modeling rápido ou hunting de falhas lógicas.

Fluxo:
1. Mapear rotas e verbos.
2. Identificar autenticação e autorização por endpoint.
3. Procurar IDOR, BFLA, mass assignment e validação fraca.
4. Priorizar risco e impacto.
5. Responder com findings, evidências e próximos testes.

Regras:
- Não assumir controles sem evidência.
- Destacar o que é fato e o que é hipótese.
- Preferir saída curta e objetiva.
```

Isso já muda bastante o comportamento do agente, porque ele deixa de improvisar do zero e passa a seguir um playbook.

## O que é um MCP

MCP, ou `Model Context Protocol`, é uma forma padronizada de conectar modelos e agentes a ferramentas externas. Em vez de cada integração inventar sua própria gambiarra, o MCP define um formato para expor coisas como:

- tools
- resources
- prompts

Na prática, isso permite que a IA consulte dados, execute ações e interaja com sistemas reais com menos acoplamento.

Se você fez um MCP próprio, ele pode servir como ponte entre a IA e qualquer coisa que já exista no seu ambiente:

- uma API interna
- um painel administrativo
- banco de dados somente leitura
- repositórios
- documentação
- sistemas como CRM, ads, observabilidade ou automação

O ganho real é simples: a IA para de depender só do que está no contexto da conversa e passa a usar fontes vivas.

## Skill e MCP não competem

Esse é o ponto que muita gente confunde.

Uma skill não substitui um MCP, e um MCP não substitui uma skill.

- A `skill` diz quando usar uma abordagem e qual processo seguir.
- O `MCP` entrega as ferramentas que tornam esse processo executável.

Juntos, eles formam um agente bem mais consistente.

Exemplo mental:

- sem skill: a IA até tem acesso à ferramenta, mas usa mal
- sem MCP: a IA até sabe o processo, mas não consegue agir fora do chat
- com os dois: ela sabe o que fazer e tem onde fazer

## Como adicionar skills em uma IA

Cada produto organiza isso de um jeito diferente, mas a lógica costuma ser a mesma:

1. Criar um diretório ou arquivo de skill.
2. Escrever descrição, gatilhos de uso e fluxo de execução.
3. Limitar escopo, formato de saída e regras.
4. Testar com tarefas reais e refinar.

O erro mais comum é criar uma skill genérica demais. Skill boa é especializada o suficiente para ser acionada com clareza.

Uma estrutura mínima costuma ter:

- nome
- descrição
- quando usar
- passos
- restrições
- formato de resposta

Se você estiver usando um agente com suporte local a skills, normalmente vai acabar com algo nesse estilo:

```text
~/.codex/skills/
  api-review/
    SKILL.md
```

ou então dentro do próprio projeto:

```text
.ai/
  skills/
    api-review.md
```

O importante não é o nome da pasta. O importante é que a instrução seja acionável e específica. Também vale tomar cuidado com a quantidade de skills: dependendo da IA, muitas skills podem consumir tokens demais e atrapalhar a resposta.

## Como adicionar um MCP em uma IA

Aqui o padrão muda menos do que parece. Você precisa de três peças:

1. Um servidor MCP.
2. Um cliente que suporte MCP.
3. Uma configuração dizendo para a IA onde esse servidor está.

Dependendo da stack, o servidor pode ser um binário local, um script Node.js, um processo Python ou um endpoint remoto.

Um exemplo real é o [GhidraMCP](https://github.com/lauriewired/ghidramcp), que conecta a IA ao Ghidra para tarefas de reverse engineering. A configuração pode ficar assim:

```toml
[mcp_servers.ghidra]
command = "python"
args = [
  "/caminho/para/bridge_mcp_ghidra.py",
  "--ghidra-server",
  "http://127.0.0.1:8080/"
]
```

Depois disso, a IA passa a enxergar as ferramentas expostas por esse servidor.

No `Claude Code`, esse tipo de integração também pode ser compartilhado no projeto via `.claude.json`:

```json
{
  "mcpServers": {
    "ghidra": {
      "command": "python",
      "args": [
        "/caminho/para/bridge_mcp_ghidra.py",
        "--ghidra-server",
        "http://127.0.0.1:8080/"
      ]
    }
  }
}
```

No caso do `GhidraMCP`, isso fica mais fácil de visualizar porque ele expõe operações ligadas diretamente ao fluxo de reverse engineering, por exemplo:

- decompilar e analisar binários no Ghidra
- renomear métodos e dados automaticamente
- listar métodos, classes, imports e exports

Esse tipo de organização facilita entender o que a IA pode chamar, quais parâmetros cada operação espera e que tipo de resposta volta do servidor MCP.

## Fechando

No fim, a ideia é simples: `skills` ajudam a IA a seguir um processo, e `MCPs` ajudam a IA a acessar ferramentas e dados reais. Quando os dois trabalham juntos, o agente deixa de ser só um chat melhorado e passa a ser algo realmente útil no dia a dia.

## Referências

- [Anthropic: Intro to MCP](https://modelcontextprotocol.io/introduction)
- [Anthropic: Model Context Protocol overview](https://www.anthropic.com/news/model-context-protocol)
- [Subframe Docs: Agent skills](https://docs.subframe.com/guides/skills)
