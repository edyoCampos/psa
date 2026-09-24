# docs/

Prints exigidos pelo desafio (ver seção "O que você deve entregar" do PDF).

O PDF pede duas coisas no print de teste: *"o resultado do webhook sendo chamado **e** a resposta retornada"*. Por isso são 3 arquivos, não 1:

- `workflow-montado.jpg` — canvas do n8n com os nós conectados (Receber Mensagem WhatsApp → Normalizar Usuario e Texto → Classificar Intencao Ajuda → Montar Resposta Prioritaria/Padrao → Unificar Resposta → Retornar Usuario e Resposta).
- `teste-funcionando-fluxo-completo.jpg` — **prova de que o webhook foi chamado**: toast "Workflow executed successfully", ✓ verde em cada nó do caminho percorrido (ramo `true`, "com ajuda"), contadores "1 item" fluindo ponta a ponta.
- `teste-funcionando-com-ajuda.jpg` — **a resposta retornada**, em detalhe: painel do nó final com `{"from": "5551999990000", "mensagem": "Olá, preciso de AJUDA com meu pedido"}` → `{"usuario": "5551999990000", "resposta": "Olá! Vou te ajudar agora mesmo."}`.
- `teste-funcionando-sem-ajuda.jpg` — o mesmo, para o caminho `false`: `{"from": "5551999990000", "mensagem": "Qual o horário de atendimento?"}` → `{"usuario": "5551999990000", "resposta": "Mensagem recebida. Em breve retornaremos."}`.

Todos os testes confirmam HTTP 200 e o corpo exato esperado pelo desafio.
