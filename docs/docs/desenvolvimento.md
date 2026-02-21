---
sidebar_position: 4
---

# 💻 Desenvolvimento

<!-- Este documento deve ser preenchido pela equipe de Desenvolvimento ao iniciar um projeto. -->

## 🗓 Informações Gerais

- **Nome do Projeto:** Agente de IA Especializado em Contabilidade
<!-- Exemplo: Sistema de Gestão de Reservas para Biblioteca -->

- **Scrum Master Responsável:** Sophia Senne
<!-- Nome do Scrum Master que acompanhará o projeto -->

- **Equipe de Desenvolvimento:** Daniel Gonçalves e Pedro Rodrigues
<!-- Lista com nome das pessoas desenvolvedoras envolvidas -->

- **Data de Entrada na Área:**
<!-- Exemplo: 10/04/2025 -->

- **Data Estimada de Conclusão da Área:**
<!-- Exemplo: 08/06/2025 -->

---

## ✅ Checklist de Entrada

- [x] Documento de Visão de Produto revisado e compreendido
- [x] Tecnologias e requisitos funcionais claramente definidos
- [x] User Stories priorizadas e estimadas
- [x] Capacidade técnica e de tempo confirmada
- [ ] Entendimento dos custos de manutenção

---

## 📤 Checklist de Saída

- [x] Funcionalidades desenvolvidas conforme requisitos
- [x] Deploy realizado (ou instruções de deploy definidas)
- [x] Documentação técnica entregue (API, banco, estrutura de dados, etc.)
- [ ] Entrega validada com PO

---

## 🛠 Tecnologias Utilizadas

* **Google Cloud Run**: Serviços principais (agente-contabil e RAG API). Executam contêineres Docker de forma serverless, escalando automaticamente conforme a demanda.

* **Cloud Run Admin API / gcloud run**: Gerencia o ciclo de vida dos serviços: deploy, atualizações, configuração de recursos (memória, CPU, timeout) e deleção.

* **Cloud Firestore**: Dois usos distintos:
  - **Banco padrão (agente-contabil):** collections `conversations`, `leads`, `rag_cache` — histórico de conversas, potenciais leads e cache de respostas RAG.
  - **rag-db-v1 (RAG API):** collection `knowledge_base` — conteúdo textual e metadados indexados para busca vetorial.

* **Vertex AI Vector Search**: Busca por similaridade vetorial (embeddings `text-embedding-3-small`), com filtros por categoria e tags.

* **OpenAI API**: ChatOpenAI (gpt-4o-mini) para orquestração do agente; text-embedding-3-small para embeddings. Inferência de metadados na RAG API via gpt-4o-mini.

* **Google Auth (OIDC)**: Token de identidade para autenticação entre agente-contabil e RAG API.

* **Cloud Storage**: Staging de arquivos Markdown normalizados no pipeline Batch ETL, para auditoria e reprocessamento.

* **Cloud Build API**: Automatiza o build das imagens Docker.


---

## 💸 Custos de Manutenção

<!-- Detalhar os custos mensais previstos para manter a aplicação em funcionamento -->
<div align="center">

| Serviço                     | Valor Mensal Estimado | Observações                                                                 |
|----------------------------|------------------------|-----------------------------------------------------------------------------|
| Cloud Run (agente-contabil)| Variável               | Cobrança por uso (requests, vCPU/seg, memória). Tier gratuito até ~2M requests |
| Cloud Run (RAG API)        | Variável               | Idem. Custos maiores em ingestão contínua                                   |
| Firestore                  | Variável               | Leitura/escrita/armazenamento. Tier gratuito generoso                       |
| Vertex AI Vector Search    | Variável               | Indexação e queries. Depende do tamanho do índice e volume de buscas        |
| OpenAI API                 | Variável               | gpt-4o-mini (chat) + text-embedding-3-small. Proporcional a tokens           |
| Cloud Storage              | Baixo                  | Staging de Markdown no ETL. Custos de storage geralmente baixos             |

</div>

**Observação:** O projeto utiliza **Google Cloud Platform** e **OpenAI**. Os custos são majoritariamente variáveis e dependem de tráfego, volume de conversas e ingestão de conhecimento.

---

## 🧱 Infraestrutura de Dados

### 🔗 Modelo Lógico do Banco de Dados

O projeto utiliza **Firestore** em dois contextos:

| Banco            | Projeto GCP              | Uso                                                                 |
|------------------|--------------------------|---------------------------------------------------------------------|
| **Default**      | ctbz-ia-assessoria-poc   | Agente: conversas, leads, cache RAG                                 |
| **rag-db-v1**    | ctbz-ia-assessoria-poc   | RAG: base de conhecimento (documentos, metadados)                   |

**Collections (banco default):**

- **conversations** — Histórico de conversas (mensagens user/assistant)
- **leads** — Potenciais leads qualificados durante o atendimento
- **rag_cache** — Cache de respostas RAG por hash da pergunta (TTL configurável)

**Collection (rag-db-v1):**

- **knowledge_base** — Documentos de conhecimento (`blog_section`, `conversa`), metadados e conteúdo textual. Vetores ficam no Vertex AI Vector Search.

---

#### Contratos de Dados (Exemplos)

**Chat Request (agente-contabil):**

```python
# POST /chat
class ChatRequest(BaseModel):
    message: str
    conversation_id: str
    user_id: Optional[str] = None
```

**Documento de conversa (Firestore `conversations`):**

```python
{
    "conversation_id": str,
    "messages": [
        {"role": "user" | "assistant", "content": str, "timestamp": datetime}
    ],
    "user_id": Optional[str],
    "created_at": datetime,
    "updated_at": datetime
}
```

**Documento de lead (Firestore `leads`):**

```python
{
    "conversation_id": str,
    "assunto": str,
    "tipo_pessoa": Optional[str],  # PF, MEI, ME, EPP, etc.
    "segmento": Optional[str],
    "nome": Optional[str],
    "email": Optional[str],
    "telefone": Optional[str],
    "user_id": Optional[str],
    "status": str,  # novo, contatado, qualificado, etc.
    "created_at": datetime,
    "updated_at": Optional[datetime]
}
```

**Chat Request / Response RAG API:**

```python
# POST /chat (RAG API - apenas retrieval)
class ChatRequest(BaseModel):
    pergunta: str
    user_id: Optional[str] = "anonimo"
    filtro_tipo: Optional[str] = None  # blog, conversa
    plataforma: Optional[str] = "web"

class SourceDoc(BaseModel):
    id: str
    score: float
    conteudo: str
    titulo: str
    url: Optional[str]
    tipo: str  # blog | conversa

class ChatResponse(BaseModel):
    fontes: List[SourceDoc]
```

**Arquitetura de dados detalhada:** [Arquitetura Geral Detalhada](./Desenvolvimento/arquitetura_geral_detalhada.md) e documentação do pipeline RAG em [Arquitetura ETL Blog](./Desenvolvimento/arquitetura_ELT_blog.md) / [Arquitetura ETL Conversas](./Desenvolvimento/arquitetura_ELT_conversa.md).

---

## 📚 Documentação Relacionada

- [Deploy](./Desenvolvimento/deploy.md) — instruções de deploy do agente e RAG
- [Testes e Segurança](./Desenvolvimento/testes-e-seguranca.md) — estratégia de testes e requisitos de segurança
- [Tecnologias](./tecnologias.md) — stack e arquitetura técnica
