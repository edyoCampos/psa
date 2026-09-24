# PSA — Triagem de Mensagens no n8n

Teste técnico da **Profissionais SA**: workflow n8n que simula a triagem inicial de mensagens recebidas por WhatsApp — recebe a mensagem, extrai os dados, classifica a intenção por palavra-chave e devolve uma resposta automática, tudo na mesma requisição HTTP.

## Cenário

A PSA recebe dezenas de mensagens por dia via WhatsApp (leads, clientes, parceiros). Antes de qualquer automação inteligente, é preciso saber receber uma mensagem, extrair dados, classificar a intenção e devolver uma resposta. Este workflow simula essa triagem básica.

## Arquitetura do workflow

```
Receber Mensagem WhatsApp (Webhook)
        │
        ▼
Normalizar Usuario e Texto (Code)
        │
        ▼
Classificar Intencao Ajuda (IF: texto contém "ajuda"?)
        │
   ┌────┴────┐
   ▼true      ▼false
Montar        Montar
Resposta      Resposta
Prioritaria   Padrao
   │              │
   └──────┬───────┘
          ▼
   Unificar Resposta (Merge)
          │
          ▼
Retornar Usuario e Resposta (Respond to Webhook)
```

| Nó | Tipo | O que faz |
|---|---|---|
| **Receber Mensagem WhatsApp** | Webhook | Ponto de entrada `POST /webhook/triagem-mensagens`. Mantém a conexão aberta até o `Respond to Webhook` rodar (`responseMode: responseNode`). |
| **Normalizar Usuario e Texto** | Code (bônus) | Extrai `usuario` (= `from`) e `texto` (= `mensagem` em minúsculas) do payload recebido. Ver código abaixo. |
| **Classificar Intencao Ajuda** | IF | Verifica se `texto` contém a substring `"ajuda"`. |
| **Montar Resposta Prioritaria** | Set | Ramo verdadeiro: define `resposta = "Olá! Vou te ajudar agora mesmo."` |
| **Montar Resposta Padrao** | Set | Ramo falso: define `resposta = "Mensagem recebida. Em breve retornaremos."` |
| **Unificar Resposta** | Merge | Une os dois ramos (mutuamente exclusivos) num único caminho de saída, por posição. |
| **Retornar Usuario e Resposta** | Respond to Webhook | Devolve `{ "usuario": ..., "resposta": ... }` como JSON, HTTP 200. |

Três sticky notes no canvas (Entrada / Regra / Saída) documentam contrato de entrada, regra de classificação e contrato de saída diretamente no fluxo.

## Nó Code (bônus)

Substitui o nó Set de normalização da Etapa 1 por um nó Code em JavaScript:

```javascript
// Extrai remetente e normaliza mensagem para classificação case-insensitive
const raw = $input.item.json;

const usuario = raw.from ?? '';
const texto = String(raw.mensagem ?? '').toLowerCase();

return {
  json: {
    usuario,
    texto,
  },
};
```

- `usuario` sempre presente na resposta (string vazia se `from` não vier).
- `texto` sempre convertido para string antes do `.toLowerCase()`, para não quebrar se `mensagem` vier como um tipo não-string.

## Como importar e testar

### 1. Suba o n8n localmente

**Opção Docker:**
```bash
docker run -it --rm --name n8n-psa \
  -p 5678:5678 \
  -v n8n_psa_data:/home/node/.n8n \
  docker.n8n.io/n8nio/n8n
```

**Opção npx (sem Docker):**
```bash
npx n8n
```

Acesse `http://localhost:5678`.

### 2. Importe o workflow

No painel do n8n: menu **⋮** → **Import from File** → selecione `workflow/triagem-mensagens-whatsapp.json`.

### 3. Ative o "Listen for test event"

Abra o nó **Receber Mensagem WhatsApp**, clique em **Listen for test event** (ou ative o workflow para usar a URL de produção).

### 4. Dispare o webhook

**Via curl** (URL de teste, com o editor escutando):
```bash
curl -X POST http://localhost:5678/webhook-test/triagem-mensagens \
  -H "Content-Type: application/json" \
  -d '{
    "from": "5551999990000",
    "mensagem": "Olá, preciso de AJUDA com meu pedido"
  }'
```

**Via curl** (URL de produção, com o workflow ativado):
```bash
curl -X POST http://localhost:5678/webhook/triagem-mensagens \
  -H "Content-Type: application/json" \
  -d '{
    "from": "5551999990000",
    "mensagem": "Qual o horário de atendimento?"
  }'
```

**Via Postman/Insomnia:** método `POST`, URL `http://localhost:5678/webhook-test/triagem-mensagens` (ou `/webhook/...` com o workflow ativo), body raw JSON com `from` e `mensagem`.

### Respostas esperadas

| `mensagem` contém "ajuda"? | Resposta |
|---|---|
| Sim (ex.: `"preciso de AJUDA"`) | `{"usuario": "5551999990000", "resposta": "Olá! Vou te ajudar agora mesmo."}` |
| Não (ex.: `"Qual o horário de atendimento?"`) | `{"usuario": "5551999990000", "resposta": "Mensagem recebida. Em breve retornaremos."}` |

## Limitações conhecidas

- Classificação por substring: mensagens com palavras derivadas de "ajuda" (ex.: "ajudar", "ajudante") também caem no ramo de ajuda — é o comportamento pedido no enunciado (Etapa 2: "verifica se o campo texto contém a palavra 'ajuda'").
- Sem autenticação no webhook — aceitável para este teste local, não recomendado para produção sem validação de origem.
- Sem persistência, histórico ou encaminhamento a atendente humano — fora do escopo do desafio.
- Payload sem `from`/`mensagem` não é rejeitado; é tratado como mensagem geral.

## Estrutura do repositório

```
workflow/
  triagem-mensagens-whatsapp.json   # workflow exportado do n8n
docs/
  workflow-montado.png              # print do canvas (adicionar)
  teste-funcionando.png             # print do teste do webhook (adicionar)
README.md
.gitignore
```
