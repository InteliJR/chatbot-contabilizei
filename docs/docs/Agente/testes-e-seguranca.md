# 🔒 Segurança e Testes

## 📋 Segurança Implementada

### **Camadas de Proteção**

1. **API Key Authentication**
   - Header obrigatório: `X-API-Key`
   - Middleware valida em todos endpoints (exceto `/health`, `/docs`)
   - Retorna 401 sem key, 403 com key inválida

2. **CORS (Cross-Origin Resource Sharing)**
   - Origens permitidas configuráveis via `.env`
   - Protege contra requests de domínios não autorizados
   - Configurado para backends específicos

3. **Rate Limiting**
   - Limite: **50 requisições/minuto por IP**
   - Protege contra DDoS e abuso
   - Retorna 429 quando limite excedido

4. **Validações de Input**
   - Mensagem não pode estar vazia
   - `conversation_id` obrigatório
   - Tamanho máximo de mensagem: 10.000 caracteres

---

## ⚙️ Configuração

### **Variáveis de Ambiente (.env)**

```env
# API Keys (para backends autorizados)
API_KEY_BACKEND=sua-chave-producao-aqui
API_KEY_DEV=dev-key-local-11111

# CORS (origens permitidas)
ALLOWED_ORIGIN_1=http://localhost:3000
ALLOWED_ORIGIN_2=https://seu-backend-producao.com

# Rate Limiting
RATE_LIMIT=50/minute
```

### **API Keys Padrão**

| Key Name | Value | Uso |
|----------|-------|-----|
| `dev` | `dev-key-local-11111` | Desenvolvimento/Testes |
| `backend` | `dev-backend-key-12345` | Produção (backend) |

---

## 🔐 Como Usar

### **Request com Autenticação**

```bash
curl -X POST https://sua-api.run.app/chat \
  -H "X-API-Key: dev-key-local-11111" \
  -H "Content-Type: application/json" \
  -d '{
    "message": "Preciso de ajuda",
    "conversation_id": "conv_123",
    "user_id": "user_456"
  }'
```

### **Respostas de Segurança**

```bash
# Sem API Key
401 Unauthorized
{"error":"unauthorized","message":"API Key is required. Include X-API-Key header."}

# API Key Inválida
403 Forbidden
{"error":"forbidden","message":"Invalid API Key"}

# Rate Limit Excedido
429 Too Many Requests
{"error":"Rate limit exceeded: 50 per 1 minute"}
```

---

## 🧪 Testes

### **1. Health Check (sempre público)**

```bash
curl https://agente-contabil-00001-b2d-55837972640.us-central1.run.app/health

# Resposta esperada:
{
  "status": "healthy",
  "message": "API funcionando corretamente"
}
```

### **2. Teste de Segurança (sem API Key)**

```bash
curl -X POST https://agente-contabil-00001-b2d-55837972640.us-central1.run.app/chat \
  -H "Content-Type: application/json" \
  -d '{"message":"teste","conversation_id":"123"}'

# Deve retornar: 401 Unauthorized
```

### **3. Teste de Segurança (API Key inválida)**

```bash
curl -X POST https://agente-contabil-00001-b2d-55837972640.us-central1.run.app/chat \
  -H "X-API-Key: chave-invalida-123" \
  -H "Content-Type: application/json" \
  -d '{"message":"teste","conversation_id":"123"}'

# Deve retornar: 403 Forbidden
```

### **4. Teste de Conversação (com API Key válida)**

```bash
curl -X POST https://agente-contabil-00001-b2d-55837972640.us-central1.run.app/chat \
  -H "X-API-Key: dev-key-local-11111" \
  -H "Content-Type: application/json" \
  -d '{
    "message": "Preciso de ajuda com MEI",
    "conversation_id": "test_conversation_001"
  }'

# Deve retornar: 200 OK com SSE streaming
# event: start
# event: delta
# event: done
```

### **5. Teste de Memória Conversacional**

```bash
# Mensagem 1: Apresentação
curl -H "X-API-Key: dev-key-local-11111" \
  -H "Content-Type: application/json" \
  -d '{"message":"Meu nome é João e tenho uma padaria","conversation_id":"test_memory"}' \
  https://agente-contabil-00001-b2d-55837972640.us-central1.run.app/chat

# Mensagem 2: Teste de memória (deve lembrar o nome)
curl -H "X-API-Key: dev-key-local-11111" \
  -H "Content-Type: application/json" \
  -d '{"message":"Qual é meu nome?","conversation_id":"test_memory"}' \
  https://agente-contabil-00001-b2d-55837972640.us-central1.run.app/chat

# Deve responder: "Seu nome é João"
```

### **6. Teste de Captura de Lead**

```bash
curl -H "X-API-Key: dev-key-local-11111" \
  -H "Content-Type: application/json" \
  -d '{
    "message": "Meu nome é Maria Silva, email maria@empresa.com, telefone (11) 99999-8888, tenho uma loja de roupas",
    "conversation_id": "test_lead_capture",
    "user_id": "user_maria"
  }' \
  https://agente-contabil-00001-b2d-55837972640.us-central1.run.app/chat

# Lead será salvo em background no Firestore
```

### **7. Teste de Validação de Input**

```bash
# Mensagem vazia (deve dar erro)
curl -H "X-API-Key: dev-key-local-11111" \
  -H "Content-Type: application/json" \
  -d '{"message":"","conversation_id":"test"}' \
  https://agente-contabil-00001-b2d-55837972640.us-central1.run.app/chat

# Deve retornar: 400 Bad Request
# {"detail":"Mensagem não pode estar vazia"}

# Sem conversation_id (deve dar erro)
curl -H "X-API-Key: dev-key-local-11111" \
  -H "Content-Type: application/json" \
  -d '{"message":"teste"}' \
  https://agente-contabil-00001-b2d-55837972640.us-central1.run.app/chat

# Deve retornar: 400 Bad Request
# {"detail":[{"type":"missing","loc":["body","conversation_id"],...}]}
```

### **8. Teste de Rate Limiting**

```bash
# Fazer múltiplas requests rápidas
for i in {1..60}; do
  curl -s -o /dev/null -w "Request $i: %{http_code}\n" \
    -H "X-API-Key: dev-key-local-11111" \
    https://agente-contabil-00001-b2d-55837972640.us-central1.run.app/conversations/test_$i
  sleep 0.5
done

# Após ~50 requests/min: deve retornar 429 Too Many Requests
```

---

## 📊 Resultados Esperados

| Teste | Status Esperado | Resposta |
|-------|----------------|----------|
| Health check | 200 OK | `{"status":"healthy"}` |
| Chat sem API Key | 401 | `{"error":"unauthorized"}` |
| Chat API Key inválida | 403 | `{"error":"forbidden"}` |
| Chat API Key válida | 200 | SSE streaming |
| Mensagem vazia | 400 | `{"detail":"Mensagem não pode estar vazia"}` |
| Sem conversation_id | 400 | Field required error |
| >50 req/min | 429 | Rate limit exceeded |

---

## 🚀 Status do Sistema

**URL Produção:** https://agente-contabil-00001-b2d-55837972640.us-central1.run.app  
**Deployment:** agente-contabil-00001-b2d-00003-7zh  
**Status:** ✅ Operacional  
**Uptime:** Min-instances=1 (sem cold start)  

### **Funcionalidades Ativas**
- ✅ API Key Authentication
- ✅ CORS configurado
- ✅ Rate Limiting (50/min)
- ✅ SSE Streaming
- ✅ Memória conversacional
- ✅ Captura de leads async
- ✅ Validações de input
- ✅ Logging estruturado

---

## 📝 Próximos Passos

Quando criar o backend intermediário:

1. **Configurar API Key de produção** no `.env`
2. **Adicionar origem do backend** no CORS
3. **Backend incluir header** `X-API-Key` em todas requests
4. **Testar integração completa** backend → API
5. **Monitorar logs** para requests não autorizadas
6. **Ajustar rate limit** conforme necessidade

---

## 🔍 Monitoramento

### **Logs Importantes**

```bash
# Ver logs no Cloud Run
gcloud run logs read agente-contabil-00001-b2d --region=us-central1 --limit=50

# Logs de segurança
"Request without API key: /chat from 203.0.113.1"
"Invalid API key: abc12345... from 203.0.113.1"
"Authenticated request with key: dev to /chat"
```

### **Alertas Recomendados**

- 🚨 Múltiplas tentativas com API Key inválida (possível ataque)
- 🚨 Rate limit atingido frequentemente (ajustar limite)
- 📊 Uso por API Key (monitorar billing)
