# Skills do @andreias.ai

Skills abertas para agentes de código (Claude Code e compatíveis), feitas a partir dos conteúdos do [@andreias.ai](https://www.instagram.com/andreias.ai/).

| Skill | O que faz |
|---|---|
| [`criar-api-rest`](criar-api-rest/) | O agente escreve a API começando pelo contrato OpenAPI: validação em toda entrada, 401/403/422, 429 com `Retry-After`, webhook com assinatura conferida, segredo só no servidor e testes que provam o contrato. Inclui o PDF com as 7 checagens. |
| [`checklist-lgpd`](checklist-lgpd/) | O agente revisa o seu app feito com IA contra 24 itens de LGPD, CDC, e-commerce, acessibilidade e direitos autorais, aponta arquivo e linha do que falta e entrega um `relatorio-lgpd.md`. Rode com `/checklist-lgpd`. Material educativo, não é parecer jurídico. |

## Como usar

Copie a pasta da skill para `.claude/skills/` do seu projeto (ou `~/.claude/skills/` para todos os projetos):

```bash
git clone https://github.com/Schneider-Gr/andreias-skills
cp -r andreias-skills/criar-api-rest .claude/skills/
```

Depois é só pedir ao agente para criar um endpoint: a skill entra sozinha.

Adapte ao seu projeto: linguagem, framework e comandos de teste.
