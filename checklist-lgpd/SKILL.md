---
name: checklist-lgpd
description: Revisa um app ou site feito com IA (vibe coding) contra 24 itens de LGPD, Código de Defesa do Consumidor, Decreto do e-commerce, Lei de Acessibilidade e direitos autorais, e entrega um relatório com o que está ok, o que falta e onde mexer no código. Use quando pedirem "checklist LGPD", "meu app está na lei?", "revisa privacidade", "posso ser processado por esse app?", antes de lançar um app que coleta dados ou vende pela internet, ou ao rodar /checklist-lgpd.
---

# Checklist LGPD do app feito com IA

Você vai revisar o app como um auditor que lê código, não como advogado. O objetivo é achar o que falta e mostrar onde mexer. **Você nunca declara que o app "está em conformidade com a LGPD".** Você declara o que encontrou e o que não conseguiu verificar.

Material educativo, conferido no texto oficial (Planalto) em 02/11/2026. Não é parecer jurídico. O relatório termina sempre recomendando revisão por um advogado.

## Como trabalhar

1. **Mapeie o app antes de julgar.** Liste as telas e rotas que tocam em dado pessoal ou dinheiro: cadastro e login, formulários, banner de cookies, scripts de analytics e anúncio, e-mails de marketing, checkout e assinatura, área da conta, páginas institucionais (privacidade, termos, segurança, home) e o `package.json` ou equivalente.
2. **Percorra os 24 itens abaixo, na ordem.** Para cada um, procure a evidência no código (arquivo e linha) ou no texto das páginas.
3. **Classifique cada item** com um destes estados:
   - ✅ **Ok**: achou a evidência.
   - ❌ **Falta**: o app precisa disso e não tem.
   - ⚠️ **Parcial**: existe, mas incompleto (diga o que falta).
   - ➖ **Não se aplica**: explique por quê (ex.: o app não vende nada).
   - ❓ **Não deu para verificar**: depende de algo fora do código (contrato, processo interno, servidor). Diga o que a pessoa precisa conferir.
4. **Não altere código na revisão.** Entregue o relatório primeiro. Só corrija o que a pessoa escolher, um item por vez, e mostre o diff.
5. **Não invente.** Se a evidência não está no repositório, o estado é ❓, nunca ✅.

## Os 24 itens

A coluna **Força** diz quanto o item pesa: **Lei** = texto legal expresso; **Leitura** = aplicação da lei ao caso, sem a palavra exata na norma; **Orientação** = guia oficial sem força de lei. Use essa força no relatório: um ❌ em item de **Lei** vem antes de um ❌ em **Orientação**.

### Cadastro e consentimento

| # | O que conferir | Onde procurar | Base | Força |
|---|---|---|---|---|
| 1 | **Política de privacidade** com finalidade, forma e duração do uso, quem é o controlador, contato e com quem os dados são compartilhados | rota `/privacidade`, rodapé, cadastro | LGPD art. 9º | Lei |
| 2 | **Termos de uso** publicados e linkados | rota `/termos`, rodapé, cadastro | Decreto 7.962/2013, art. 4º | Lei |
| 3 | **Caixa de aceite desmarcada**: o aceite é um ato do usuário, nunca vem marcado | formulário de cadastro (`checked`, `defaultChecked`, valor inicial `true`) | LGPD art. 5º, XII | Leitura |
| 4 | **Prova do aceite guardada**: quem aceitou, quando e qual versão do texto | tabela ou log de consentimento no banco | LGPD art. 8º, § 2º | Lei |
| 5 | **Aceite de marketing separado** do aceite dos termos, e opcional | cadastro, checkout | LGPD art. 8º, § 4º | Lei |
| 6 | **Só os dados necessários**: cada campo tem finalidade (ex.: CPF para marcar horário não tem) | schemas de formulário, colunas do banco | LGPD art. 6º, III | Lei |
| 7 | **Dado de criança só com consentimento específico de um dos pais ou responsável**, coletado antes | cadastro, verificação de idade | LGPD art. 14, § 1º | Lei |

### Cookies e e-mail

| # | O que conferir | Onde procurar | Base | Força |
|---|---|---|---|---|
| 8 | **Banner de cookies com botão de recusar** do mesmo peso que o de aceitar; cookies não essenciais (anúncio, analytics) começam desligados; nada de "aceito ao continuar navegando" | componente de banner, scripts carregados antes do consentimento | Guia de cookies da ANPD (2022) | Orientação |
| 9 | **Sair da lista de e-mails** de forma gratuita e fácil, com link em todo e-mail de marketing | templates de e-mail, preferências da conta | LGPD art. 8º, § 5º | Lei |

### Página de venda

| # | O que conferir | Onde procurar | Base | Força |
|---|---|---|---|---|
| 10 | **Sem cronômetro ou escassez falsos** (contagem que reinicia ao recarregar, "só 2 vagas" fixo) | componentes de oferta, timers no front | CDC art. 37, § 1º | Lei |
| 11 | **Sem avaliação inventada** | seeds, arrays fixos de depoimentos | CDC art. 37, § 1º | Lei |
| 12 | **Promessa só com prova guardada** ("dobra seu faturamento") | textos da home e anúncios | CDC art. 36, parágrafo único | Lei |
| 13 | **Selo ou certificado de segurança só com comprovante** | página de segurança, rodapé, badges | CDC arts. 36 e 37 | Leitura |
| 14 | **Logo de cliente só com autorização** por escrito | seção "clientes" ou "usado por" | Lei 9.279/1996 (marcas) | Leitura |
| 15 | **Sem truque de tela** que explore o consumidor (botão de recusar escondido, culpa no "não", caixa escondida) | modais, pop-ups de saída e de cancelamento | CDC arts. 37 e 39, IV | Leitura |
| 16 | **Avisar o que foi feito por IA** quando a omissão puder enganar (foto de produto, depoimento, rosto) | imagens e textos gerados | CDC art. 37, § 1º | Leitura |

### Pagamento e assinatura

| # | O que conferir | Onde procurar | Base | Força |
|---|---|---|---|---|
| 17 | **Sete dias para desistir** de compra on-line, com devolução do valor e estorno comunicado ao cartão; política de reembolso publicada | checkout, página de reembolso, webhook do pagamento | CDC art. 49; Decreto 7.962/2013, art. 5º | Lei |
| 18 | **Cancelar pelo mesmo caminho da compra**, sem obrigar ligação ou chat | área da conta, fluxo de assinatura | Decreto 7.962/2013, art. 5º, § 1º | Lei e Leitura |

### Conta do usuário

| # | O que conferir | Onde procurar | Base | Força |
|---|---|---|---|---|
| 19 | **Ver, baixar e pedir para apagar os próprios dados** | área da conta, rotas de exportação e exclusão | LGPD art. 18 | Lei |
| 20 | **Prazo para apagar cada dado** quando a finalidade acabar, salvo obrigação legal | jobs de limpeza, política de retenção | LGPD arts. 15 e 16 | Lei |

### Acessibilidade

| # | O que conferir | Onde procurar | Base | Força |
|---|---|---|---|---|
| 21 | **Contraste, texto alternativo e teclado**: texto legível (referência WCAG 4.5:1), `alt` nas imagens com conteúdo, tudo operável por Tab | CSS e tema, `<img>`, foco visível, `tabindex` | Lei 13.146/2015, art. 63 | Leitura |

### Bastidores

| # | O que conferir | Onde procurar | Base | Força |
|---|---|---|---|---|
| 22 | **Licença de cada imagem, fonte e código**, incluindo as obrigações das dependências open source (manter aviso de copyright, termos copyleft) | `package.json`, `LICENSE`, pasta de fontes e imagens | Lei 9.610/1998; Lei 9.609/1998 | Leitura |
| 23 | **Canal de denúncia e remoção** se o app hospeda conteúdo de usuários | página de contato, fluxo de denúncia | Marco Civil arts. 19 e 21 | Leitura |
| 24 | **Identificação e contatos obrigatórios**: CNPJ ou CPF e endereço físico e eletrônico em destaque; nome e contato do encarregado de dados; plano para vazamento (comunicar a ANPD e os titulares afetados quando houver risco ou dano relevante, em até 3 dias úteis) | rodapé, política de privacidade, runbook de incidente | Decreto 7.962/2013, art. 2º; LGPD arts. 41 e 48; Resolução CD/ANPD 15/2024 | Lei |

## O que está em jogo

Use isto no resumo do relatório, sem exagero:

- A ANPD pode aplicar multa de até 2% do faturamento no Brasil, limitada a R$ 50 milhões por infração (LGPD art. 52, II).
- Quem causar dano ao violar a LGPD tem de reparar (art. 42).
- Exemplo real: em agosto de 2026 a ANPD multou a controladora do TikTok em R$ 153,7 milhões por falhas com dados de crianças e adolescentes (decisão publicada no DOU de 25/08/2026). A primeira multa a empresa privada, em 2023, foi de R$ 14,4 mil a uma microempresa: tamanho não protege ninguém.

## Formato do relatório

Salve como `relatorio-lgpd.md` na raiz do projeto e mostre o resumo na conversa.

```markdown
# Relatório LGPD · <nome do app> · <data>

**Resumo:** X ok · Y faltam · Z parciais · W não verificados · K não se aplicam.
Prioridade: <os 3 ❌ de força "Lei" com maior risco>.

| # | Item | Estado | Evidência | O que fazer |
|---|---|---|---|---|
| 1 | Política de privacidade | ❌ | não há rota /privacidade | Criar página com finalidade, prazo, controlador, contato e compartilhamento; linkar no rodapé e no cadastro |
| 3 | Caixa de aceite desmarcada | ⚠️ | `src/app/cadastro/page.tsx:42` usa `defaultChecked` | Remover `defaultChecked` e bloquear o envio sem marcar |
| ... |

## Para conferir fora do código
- <itens ❓ e o que a pessoa precisa verificar>

> Revisão técnica feita por agente de código, com base na lei conferida em 02/11/2026. Não é parecer jurídico: leve este relatório a um advogado antes de lançar.
```

## Pendências conhecidas da própria lista

Diga isto se a pessoa perguntar sobre os itens abaixo, em vez de afirmar com certeza:

- Item 16: o PL 2338/2023 (marco da IA) ainda tramitava na última conferência; hoje não há lei geral de rotulagem.
- Item 18: o Decreto 11.034/2022 (SAC) vale para serviços regulados, não como regra geral de cancelamento.
- Item 23: a tese do STF de 26/06/2025 sobre o art. 19 do Marco Civil muda a responsabilidade por conteúdo de terceiros; confirmar o alcance com um advogado.
- Item 24: agentes de pequeno porte têm regras próprias para o encarregado e prazos dobrados (Resolução CD/ANPD 2/2022); conferir se o app se enquadra.
