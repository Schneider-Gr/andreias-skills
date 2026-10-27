---
name: criar-api-rest
description: Cria ou altera uma API REST no padrão da casa, com o contrato OpenAPI escrito antes do código, rotas por recurso, validação de schema em toda entrada, autenticação e autorização por objeto, erros padronizados (401, 403, 422, 429) e testes que provam o contrato. Use quando o pedido for criar um endpoint, uma rota, um recurso REST, uma API nova ou uma integração exposta por HTTP, mesmo que a palavra "API" não apareça.
---

# Criar API REST

Você vai escrever uma API que outro sistema vai chamar. A API é a porta mais exposta do sistema, então a ordem importa: **contrato, depois código, depois teste**. Não pule etapas, nem quando o pedido parecer pequeno.

## Etapa 1 · Contrato antes do código

1. Liste os recursos (substantivos no plural: `/pedidos`, `/clientes`) e o que se faz com cada um.
2. Escreva ou atualize o `openapi.yaml` **antes** de qualquer rota. Use a versão do OpenAPI que o seu ferramental suporta (3.1 ou 3.2).
3. Para cada operação, defina: método, caminho, schema do corpo, parâmetros, respostas de sucesso **e** de erro, e o esquema de segurança.
4. Mostre o contrato a quem pediu e espere aprovação. Só depois implemente.

Mapa de verbos (RFC 9110):

| Ação | Verbo | Caminho | Sucesso |
|---|---|---|---|
| Listar | GET | `/pedidos` | 200 |
| Ler um | GET | `/pedidos/{id}` | 200 |
| Criar | POST | `/pedidos` | 201 + cabeçalho `Location` |
| Substituir | PUT | `/pedidos/{id}` | 200 ou 204 |
| Apagar | DELETE | `/pedidos/{id}` | 204 |

Regras: verbo no método, nunca no caminho (`POST /pedidos`, não `/criarPedido`). GET nunca altera estado. Listagem sempre paginada, com limite máximo no servidor.

Esqueleto mínimo:

```yaml
openapi: 3.2.1
info: { title: API de pedidos, version: 1.0.0 }
security: [ { chave: [] } ]
paths:
  /pedidos/{id}:
    get:
      operationId: lerPedido
      parameters:
        - { name: id, in: path, required: true, schema: { type: string, format: uuid } }
      responses:
        "200": { description: Pedido, content: { application/json: { schema: { $ref: "#/components/schemas/Pedido" } } } }
        "401": { $ref: "#/components/responses/Problema" }
        "403": { $ref: "#/components/responses/Problema" }
        "404": { $ref: "#/components/responses/Problema" }
        "429": { $ref: "#/components/responses/Problema" }
components:
  securitySchemes:
    chave: { type: apiKey, in: header, name: X-Api-Key }
  responses:
    Problema:
      description: Erro no formato RFC 9457
      content: { application/problem+json: { schema: { $ref: "#/components/schemas/Problema" } } }
  schemas:
    Problema:
      type: object
      properties: { type: { type: string }, title: { type: string }, status: { type: integer }, detail: { type: string } }
    Pedido:
      type: object
      additionalProperties: false
      required: [id, status]
      properties: { id: { type: string, format: uuid }, status: { type: string, enum: [aberto, pago, entregue] } }
```

## Etapa 2 · Implementação

### Validação em toda entrada
- Valide corpo, query, parâmetros de caminho e cabeçalhos contra o schema do contrato, na borda, antes de qualquer lógica.
- Rejeite campos que não estão no schema (`additionalProperties: false`). Isso evita que o cliente grave campos que não deveria, como `role` ou `preco`.
- Limite tamanho do corpo, tamanho de strings e tamanho de listas.
- JSON malformado = 400. JSON válido que viola o schema = 422.

### Autenticação e autorização
- Credencial só no servidor e em variável de ambiente. Nunca no código, no front-end, no app móvel nem em log.
- Sem credencial ou credencial inválida = **401**. Credencial válida sem permissão = **403**.
- **Autorização por objeto em toda rota com `{id}`**: confira que o recurso pertence a quem está pedindo antes de devolver ou alterar. Esse é o risco número 1 da OWASP API Security Top 10 (API1:2023).
- Escopo mínimo: cada integração recebe a própria chave, só com as permissões de que precisa.

### Erros padronizados
- Todo erro sai em `application/problem+json` (RFC 9457) com `type`, `title`, `status` e `detail`.
- Nunca devolva stack trace, SQL, caminho de arquivo ou mensagem do banco.
- Em 404 versus 403 de objeto alheio, prefira 404 para não confirmar que o recurso existe.

### Limite de uso
- Aplique rate limit por chave ou por usuário.
- Ao estourar, responda **429** com o cabeçalho `Retry-After` (RFC 6585, seção 4).

### Reenvio seguro
- Em POST que cria algo com efeito colateral (cobrança, pedido, envio), aceite um cabeçalho `Idempotency-Key` e devolva a mesma resposta para a mesma chave. É um rascunho da IETF (draft-ietf-httpapi-idempotency-key-header), não RFC; documente no contrato.

### Webhooks que a sua API recebe
- Confira a assinatura (HMAC do corpo bruto com o segredo do provedor, comparação em tempo constante) antes de ler qualquer campo; sem assinatura válida, responda 401 e não processe.
- Recuse aviso antigo pelo timestamp assinado (janela curta, ex.: 5 minutos) para barrar reenvio gravado.
- Guarde o id de cada evento processado: aviso repetido é descartado com 200, sem efeito colateral de novo.
- Responda 2xx rápido e processe em fila; o provedor reenvia quando não recebe resposta a tempo.

### Chamadas que a sua API faz para fora
- Toda chamada de saída tem timeout explícito.
- Ao receber 429, espere o `Retry-After` antes de tentar de novo; nos demais erros temporários, use backoff com limite de tentativas.
- Trate a resposta de terceiros como entrada não confiável: valide também (OWASP API10:2023).

### Log
- Registre método, rota, status, duração e um id de correlação.
- Nunca registre credencial, token, senha, cartão, documento pessoal nem o corpo inteiro da requisição.

## Etapa 3 · Testes que provam o contrato

Escreva, no mínimo, para cada operação:
1. Caminho feliz com o status de sucesso do contrato.
2. Sem credencial → 401.
3. Credencial de outro usuário pedindo o objeto → 403 ou 404.
4. Corpo com campo extra ou tipo errado → 422.
5. Resposta validada contra o schema do `openapi.yaml`.

E um teste de limite: passar do rate limit devolve 429 com `Retry-After`. Se houver webhook: sem assinatura → 401, e o mesmo evento enviado duas vezes só tem efeito uma vez.

Rode os testes e mostre a saída. Não diga que terminou sem ela.

## Etapa 4 · Antes de entregar

Confira e relate item por item:
- [ ] `openapi.yaml` atualizado e igual ao que o código faz.
- [ ] Nenhuma credencial no código, no cliente ou no log (procure por `key`, `token`, `secret`); segredos só em variável de ambiente no servidor.
- [ ] Webhook: assinatura conferida antes de processar e evento repetido descartado.
- [ ] Toda entrada validada; campos extras rejeitados.
- [ ] Autorização por objeto em toda rota com `{id}`.
- [ ] Erros em `application/problem+json`, sem detalhe interno.
- [ ] 429 com `Retry-After`.
- [ ] Timeout em toda chamada de saída.
- [ ] Testes rodados, com a saída mostrada.

O diff passa por revisão humana antes do merge. Esta skill não faz merge nem deploy.

## Fontes
- RFC 9110, HTTP Semantics (2022): métodos e códigos 200, 201, 204, 401, 403, 404, 422.
- RFC 6585, seção 4 (2012): 429 Too Many Requests e `Retry-After`.
- RFC 9457 (2023): Problem Details for HTTP APIs, `application/problem+json`.
- OWASP API Security Top 10 (2023): API1 Broken Object Level Authorization, API4 Unrestricted Resource Consumption, API10 Unsafe Consumption of APIs.
- OpenAPI Specification: spec.openapis.org.
- draft-ietf-httpapi-idempotency-key-header (rascunho, versão 08 de 22/10/2026).

Feito por @andreias.ai. Adapte ao seu projeto: ajuste a linguagem, o framework e os comandos de teste.
