---
sidebar_position: 3
---

# ⚙️ Tecnologias

## 🗓 Informações Gerais

- **Nome do Projeto:** Agente de IA Especializado em Contabilidade

- **Tech Lead:**
<!-- Nome da pessoa responsável pela coordenação e entrega da parte técnica do projeto -->

- **Data de Entrada na Área:**
<!-- Exemplo: 10/04/2025 -->

- **Data Estimada de Conclusão da Área:**
<!-- Exemplo: 08/06/2025 -->

- **Link para Documento de Visão de Produto:**
- [Visão de Produto](./visao-produto.md)

## Checklist de Entrada e Saída da Área de Tecnologia

### ✅ Checklist de Entrada

- [x] Documento de Visão de Produto validado

### 📤 Checklist de Saída

- [x] Stack definida e aprovada
- [x] Diagrama de arquitetura completo
- [ ] Plano de implantação claro
- [ ] Documento validado com o time de Desenvolvimento

## Stack Tecnológica

A solução é composta por dois repositórios principais: **agent-structure** (agente-contabil) e **rag-agent-architecture** (RAG API).

### Agente Contábil (agent-structure)

| Categoria | Tecnologia | Versão / Observação |
|-----------|------------|---------------------|
| Linguagem | Python | 3.11 |
| Framework Web | FastAPI | ≥ 0.128.0 |
| Servidor ASGI | Uvicorn | ≥ 0.40.0 |
| LLM / Agent | LangChain + LangGraph | langchain 1.2.4, langgraph ≥ 0.2.0 |
| LLM Provider | OpenAI | gpt-4o-mini (ChatOpenAI) |
| Banco de dados | Firebase / Firestore | firebase-admin ≥ 6.6.0, google-cloud-firestore ≥ 2.18.0 |
| Autenticação RAG | Google Auth | google-auth ≥ 2.35.0 (OIDC para Cloud Run) |
| Rate Limiting | SlowAPI | ≥ 0.1.9 |
| Variáveis de ambiente | python-dotenv | 1.0.0 |

### RAG API (rag-agent-architecture)

| Categoria | Tecnologia | Uso |
|-----------|------------|-----|
| Framework Web | FastAPI | API REST de ingestão e busca |
| Embeddings | OpenAI text-embedding-3-small | Vetores de 1.536 dimensões |
| Busca vetorial | Vertex AI Vector Search | STREAM_UPDATE, filtros por categoria/tag |
| Armazenamento | Firestore (rag-db-v1) | Conteúdo textual e metadados (collection `knowledge_base`) |
| Inferência de metadados | GPT-4o-mini | Título, categoria, tags (structured output) |
| Web scraping | BeautifulSoup4, httpx | Conversão HTML → Markdown para ingestão via URL |
| Cloud | Google Cloud Platform | Cloud Run, Cloud Storage, Vertex AI |

### Infraestrutura GCP

| Serviço | Projeto | Região |
|---------|---------|--------|
| Cloud Run (agente-contabil) | ctbz-ia-assessoria-poc | us-central1 |
| Cloud Run (RAG API) | ctbz-ia-assessoria-poc | us-central1 |
| Firestore (rag-db-v1) | ctbz-ia-assessoria-poc | — |
| Vertex AI Vector Search | ctbz-ia-assessoria-poc | us-central1 |
| Cloud Storage | ctbz-ia-assessoria-poc | Staging de Markdown normalizados |

## Arquitetura da Solução

### Visão Geral da Arquitetura

A solução foi arquitetada sobre o **Google Cloud Platform (GCP)** com foco em três pilares: **precisão tributária**, **segurança de dados (LGPD)** e **eficiência de custo (FinOps)**.

A arquitetura compreende três grandes componentes:

1. **Pipeline de Ingestão em Lote (Batch ETL)** — carga inicial de fontes históricas (780+ artigos, conversas, site institucional)
2. **Pipeline de Ingestão via API (On-Demand)** — adição contínua de novos conhecimentos em tempo real
3. **Fluxo de Execução Cognitiva (Runtime)** — orquestração do agente com RAG, ferramentas e histórico de conversa

A arquitetura técnica está detalhada em [Arquitetura Geral Detalhada](./Desenvolvimento/arquitetura_geral_detalhada.md).

### Componentes Principais

#### Serviços e Microsserviços

- **agente-contabil** (agent-structure): API FastAPI que expõe `/chat` com SSE, orquestra o agente LangGraph, ferramentas (RAG, leads, transbordo) e integra com Firestore para histórico e leads
- **RAG API** (rag-agent-architecture): API REST de ingestão (`/knowledge/ingest/blog`, `/knowledge/ingest/qa`) e busca (`/chat` — apenas retrieval, sem geração LLM)
- **Pipeline ETL (Batch)**: Cloud Run Jobs para processamento em lote de blogs, conversas e site

#### Ferramentas do Agente

- **buscar_conhecimento_contabil** (RAG tool): Consulta a RAG API e retorna respostas fundamentadas na base de conhecimento
- **registrar_lead**: Persiste potenciais leads no Firestore durante a conversa
- **transbordo**: Marca transferência para atendimento humano (implementação futura)

#### Integrações Externas

- **RAG API**: O agente chama `https://rag-api-55837972640.us-central1.run.app/chat` com autenticação OIDC
- **OpenAI**: ChatOpenAI (gpt-4o-mini) para orquestração do agente e inferência de metadados
- **Firestore**: Histórico de conversas, cache de RAG, leads
- **Vertex AI**: Busca vetorial e armazenamento de embeddings

## Estrutura de Implantação

### Ambiente de Desenvolvimento

- **Como subir localmente (agente-contabil):**
  ```bash
  cd agent-structure
  cp .env.example src/.env   # configurar OPENAI_API_KEY, etc.
  docker build -t agente-contabil:test .
  docker run -d -p 8080:8080 --env-file src/.env agente-contabil:test
  ```
- **Docker:** Imagem oficial Python 3.11-slim, porta 8080
- **Variáveis de ambiente principais:**
  - `OPENAI_API_KEY` — obrigatória para o LLM
  - `API_KEY_DEV` / `API_KEY_PROD_SITE` — validação de API Key no header `X-API-Key`
  - `ALLOWED_ORIGIN_1`, `ALLOWED_ORIGIN_2` — CORS
  - `RATE_LIMIT` — ex: 50/minute
  - `RAG_CACHE_ENABLED`, `RAG_CACHE_TTL_DAYS` — cache de respostas RAG no Firestore

### Ambiente de Produção

- **URL agente-contabil:** https://agente-contabil-55837972640.us-central1.run.app
- **URL RAG API:** https://rag-api-55837972640.us-central1.run.app
- **Estratégia de deploy:** Implantação via `gcloud run deploy` (Cloud Run padrão)
- **Infraestrutura:** Google Cloud Platform (Cloud Run, Firestore, Vertex AI, Cloud Storage)
- **Observabilidade:** Cloud Logging, logs via `gcloud run services logs tail`

### Diagrama de Implantação (opcional)
> Diagrama com servidores, buckets, serviços gerenciados, DNS, CDNs, etc.

## Considerações de Segurança

- **Políticas de CORS:** Configuradas em `ALLOWED_ORIGINS`; produção restringe origens autorizadas
- **Proteção de dados sensíveis:** PII Masking (DLP) planejado no runtime; dados sensíveis substituídos por tokens antes de envio a LLMs (LGPD)
- **Gestão de segredos:** Variáveis de ambiente no Cloud Run; API keys em `.env` local e `gcloud run services update --set-env-vars`
- **Autenticação e autorização:**
  - Agente: header `X-API-Key` obrigatório (exceto `/health`, `/docs`, `/redoc`)
  - RAG API: token OIDC via service account (IAM `roles/run.invoker` para agente-contabil)
  - Rate limiting (SlowAPI) para mitigar abuso e DDoS
