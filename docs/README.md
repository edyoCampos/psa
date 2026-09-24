# docs/

Prints exigidos pelo desafio (ver seção "O que você deve entregar" do PDF):

- `workflow-montado.jpg` — canvas do n8n com os nós conectados (Receber Mensagem WhatsApp → Normalizar Usuario e Texto → Classificar Intencao Ajuda → Montar Resposta Prioritaria/Padrao → Unificar Resposta → Retornar Usuario e Resposta).
- `teste-funcionando-com-ajuda.jpg` — teste real (painel do n8n) com `{"from": "5551999990000", "mensagem": "Olá, preciso de AJUDA com meu pedido"}` → `{"usuario": "5551999990000", "resposta": "Olá! Vou te ajudar agora mesmo."}`.
- `teste-funcionando-sem-ajuda.jpg` — teste real com `{"from": "5551999990000", "mensagem": "Qual o horário de atendimento?"}` → `{"usuario": "5551999990000", "resposta": "Mensagem recebida. Em breve retornaremos."}`.

Ambos os testes confirmam HTTP 200 e o corpo exato esperado pelo desafio.
