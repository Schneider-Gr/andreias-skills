# Skills do @andreias.ai

Skills abertas para agentes de código (Claude Code e compatíveis), feitas a partir dos conteúdos do [@andreias.ai](https://www.instagram.com/andreias.ai/).

| Skill | O que faz |
|---|---|
| [`criar-api-rest`](criar-api-rest/) | O agente escreve a API começando pelo contrato OpenAPI: validação em toda entrada, 401/403/422, 429 com `Retry-After`, webhook com assinatura conferida, segredo só no servidor e testes que provam o contrato. Inclui o PDF com as 7 checagens. |

## Como usar

Copie a pasta da skill para `.claude/skills/` do seu projeto (ou `~/.claude/skills/` para todos os projetos):

```bash
git clone https://github.com/Schneider-Gr/andreias-skills
cp -r andreias-skills/criar-api-rest .claude/skills/
```

Depois é só pedir ao agente para criar um endpoint: a skill entra sozinha.

Adapte ao seu projeto: linguagem, framework e comandos de teste.
