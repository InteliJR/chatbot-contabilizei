# Guia de Verificação — API de Conhecimento RAG + Sistema de Testes

Este documento consolida o guia operacional da API (endpoints, curl, Swagger, Postman, GCP) e o
manual técnico do sistema de testes automatizados em um único ponto de referência.

---

## Estatísticas do Sistema de Testes

| Métrica | Valor Atual | Target |
|---------|-------------|--------|
| Total de Testes | **96** | ≥ 95 |
| Taxa de Sucesso | **100%** (96/96) | 100% |
| Cobertura de Código | **88.53%** | ≥ 70% |
| Tempo de Execução | ~10s | < 30s |
| Testes Unitários | ~76 | — |
| Testes de Integração | ~20 | — |

**Tecnologias de Teste:**
- Framework: pytest 7.4.3+
- Cobertura: coverage.py
- Async: pytest-asyncio
- Threshold mínimo: 70% (configurado em `pytest.ini`)

---

## 1. Testes Automatizados

### 1.1 Execução Básica

```bash
cd arquitetura_RAG

# Todos os testes
./venv/bin/pytest tests/ -v

# Por tipo
./venv/bin/pytest -m unit -v
./venv/bin/pytest -m integration -v
./venv/bin/pytest -m critical -v

# End-to-End
./venv/bin/pytest tests/integration/test_e2e_knowledge_lifecycle.py -v

# Sem cobertura (mais rápido)
./venv/bin/pytest tests/ -v --no-cov
```

### 1.2 Execução Avançada

```bash
# Arquivo específico
./venv/bin/pytest tests/unit/test_chunking.py -v

# Teste específico
./venv/bin/pytest tests/unit/test_chunking.py::TestSafeRecursiveSplit::test_table_not_broken -v

# Modo silencioso
./venv/bin/pytest tests/ -q

# Parar no primeiro erro
./venv/bin/pytest tests/ -x

# Reexecutar apenas testes que falharam
./venv/bin/pytest --lf

# Com cobertura e relatório HTML
./venv/bin/pytest tests/ --cov --cov-report=html
xdg-open htmlcov/index.html  # Linux

# Relatório de cobertura no terminal
./venv/bin/pytest tests/ --cov --cov-report=term-missing
```

### 1.3 Estrutura de Diretórios

```
arquitetura_RAG/
├── tests/                                    # Diretório principal de testes
│   ├── conftest.py                           # Fixtures compartilhadas
│   ├── __init__.py
│   ├── unit/                                 # Testes unitários
│   │   ├── __init__.py
│   │   ├── test_chunking.py                  # Chunking de documentos
│   │   ├── test_retrieval_logic.py           # Lógica de busca RAG
│   │   ├── test_embeddings.py                # Geração de embeddings
│   │   ├── test_llm_metadata.py              # Inferência de metadados via LLM
│   │   ├── test_url_fetcher.py               # Fetch e parse de URLs
│   │   ├── test_vertex_service.py            # Upsert/search/delete no Vertex AI
│   │   └── test_firestore_service.py         # CRUD no Firestore
│   └── integration/                          # Testes de integração
│       ├── __init__.py
│       ├── test_api_contract.py              # Contrato dos endpoints
│       └── test_e2e_knowledge_lifecycle.py   # Ciclo completo: ingest→chat→delete
├── pytest.ini                                # Configuração do pytest
├── requirements-dev.txt                      # Dependências de teste
├── htmlcov/                                  # Relatórios de cobertura HTML (gerado)
└── .coverage                                 # Banco de dados de cobertura (gerado)
```

#### pytest.ini

```ini
[pytest]
testpaths = tests
python_files = test_*.py
python_classes = Test*
python_functions = test_*

addopts =
    --verbose
    --strict-markers
    --color=yes
    --tb=short
    --cov=app_rag
    --cov-fail-under=70

markers =
    unit: Testes unitários
    integration: Testes de integração
    critical: Testes de funcionalidades críticas
```

### 1.4 Módulos de Teste Detalhados

#### test_chunking.py

**Arquivo testado:** `app_rag/services/chunking.py`

**Funções testadas:** `smart_parent_split()`, `smart_recursive_split()`

**Justificativa:** Tabelas tributárias não podem ser quebradas no meio — isso levaria a informações incorretas no RAG.

| Teste | Descrição | Marcador |
|-------|-----------|---------|
| `test_parent_split_respects_limit` | Nenhum parent excede 6000 chars | unit |
| `test_parent_split_prefers_paragraph_breaks` | Quebras preferem parágrafos, não meio de frases | unit |
| `test_chunks_respect_size_limit` | Chunks não excedem 800 chars | unit |
| `test_chunks_have_overlap` | Overlap de 200 chars preserva contexto | unit |
| `test_table_not_broken` | Tabelas Markdown permanecem íntegras | unit, **critical** |
| `test_empty_text_handling` | Strings vazias não geram exceções | unit |
| `test_very_short_text_not_chunked` | Textos < chunk_size não são divididos | unit |

```bash
pytest tests/unit/test_chunking.py -v
pytest tests/unit/test_chunking.py::TestSafeRecursiveSplit::test_table_not_broken -v
```

---

#### test_retrieval_logic.py

**Arquivo testado:** `app_rag/main.py`

**Função testada:** `retrieve_context(query_vector, k=5, filtro_tipo=None)`

**Justificativa:** Núcleo do RAG. Implementa busca vetorial, boost para blogs, threshold e deduplicação.

| Teste | Descrição | Marcador |
|-------|-----------|---------|
| `test_blog_boost_applied_correctly` | Blogs recebem boost de 15% (0.70 → 0.805) | unit, **critical** |
| `test_threshold_minimum_score` | Documentos com score < 0.50 são descartados | unit, **critical** |
| `test_parent_deduplication` | Múltiplos chunks do mesmo parent: mantém só o de maior score | unit, **critical** |
| `test_filter_by_type_blog` | Filtro por tipo "blog" funciona | unit |
| `test_filter_by_type_conversa` | Filtro por tipo "conversa" funciona | unit |
| `test_empty_results_from_vertex` | Resultados vazios sem exceções | unit |
| `test_top_k_limit_respected` | Parâmetro k limita número de documentos | unit |

```bash
pytest tests/unit/test_retrieval_logic.py -v
```

---

#### test_embeddings.py

**Arquivo testado:** `app_rag/services/embedding.py`

**Funções testadas:** `get_embedding()`, `get_embeddings_batch()`

| Teste | Descrição | Marcador |
|-------|-----------|---------|
| `test_newlines_replaced_by_space` | `\n` substituídos antes de envio à API | unit |
| `test_returns_1536_dimensions` | Embedding retorna vetor de 1536 dimensões | unit |
| `test_empty_text_handling` | Texto vazio é enviado ao cliente e retorna embedding normalmente (contrato determinístico) | unit |
| `test_batch_returns_correct_count` | Batch retorna 1 embedding por texto | unit |
| `test_batch_replaces_newlines` | Batch substitui `\n` em todos os textos | unit |

```bash
pytest tests/unit/test_embeddings.py -v
```

---

#### test_llm_metadata.py

**Arquivo testado:** `app_rag/services/llm_metadata.py`

**Funções testadas:** `infer_blog_metadata()`, `infer_qa_metadata()`

**Justificativa:** Valida que a inferência de metadados via LLM (gpt-4o-mini) retorna `BlogMetadata` e `QAMetadata` com os campos corretos, mockando o cliente OpenAI.

```bash
pytest tests/unit/test_llm_metadata.py -v
```

---

#### test_url_fetcher.py

**Arquivo testado:** `app_rag/services/url_fetcher.py`

**Função testada:** `fetch_url_content(url)` → retorna `UrlContent(text, title)`

**Justificativa:** Valida fetch de URL, conversão HTML→Markdown, proteção anti-SSRF (`_validate_url` bloqueia hosts internos e protocolos não-HTTP) e tratamento de URLs inválidas ou indisponíveis.

```bash
pytest tests/unit/test_url_fetcher.py -v
```

---

#### test_vertex_service.py

**Arquivo testado:** `app_rag/services/vertex_service.py`

**Funções testadas:** `upsert_datapoints()`, `remove_datapoints()`, `search_neighbors()`

**Justificativa:** Valida operações no Vertex AI Vector Search mockando o endpoint, verificando formato de datapoints e tratamento de erros.

```bash
pytest tests/unit/test_vertex_service.py -v
```

---

#### test_firestore_service.py

**Arquivo testado:** `app_rag/services/firestore_service.py`

**Funções testadas:** `create_document()`, `get_document()`, `delete_document()`, `find_documents_by_base_article_id()`, `list_documents()`

**Justificativa:** Valida CRUD no Firestore (`rag-db-v1`, collection `knowledge_base`) mockando o cliente. Inclui testes de mapeamento dos filtros de tipo:
- `tipo=blog` → `WHERE type == "blog_section"`
- `tipo=conversa` → `WHERE type != "blog_section"` (inclui todos os não-blog)

```bash
pytest tests/unit/test_firestore_service.py -v
```

---

#### test_api_contract.py

**Arquivo testado:** `app_rag/main.py`

**Endpoints testados:** `/knowledge/ingest/blog`, `/knowledge/ingest/qa`, `/knowledge/delete`, `GET /knowledge`, `/chat`

**Justificativa:** Valida o contrato completo da API — campos de request/response, status codes e comportamento dos endpoints.

| Teste | Descrição | Marcador |
|-------|-----------|---------|
| `test_chat_endpoint_valid_request` | Request válido retorna `fontes` | integration, **critical** |
| `test_response_structure` | Response tem campo `fontes`, sem campo `resposta` | integration |
| `test_fontes_structure` | Cada fonte tem id, score, conteudo, titulo, tipo | integration |
| `test_filtro_tipo_blog_applied` | Filtro por tipo "blog" é aplicado | integration |
| `test_no_results_returns_empty_fontes` | Sem resultados retorna `{"fontes": []}` | integration |

```bash
pytest tests/integration/test_api_contract.py -v
```

---

#### test_e2e_knowledge_lifecycle.py

**Arquivo testado:** `app_rag/main.py` (fluxo completo)

**Justificativa:** Testa o ciclo completo (ingestão → busca → exclusão → verificação de remoção), mockando apenas as APIs externas.

| Classe | Teste | Descrição | Marcador |
|--------|-------|-----------|---------|
| `TestKnowledgeLifecycleE2E` | `test_qa_ingest_then_chat_then_delete` | Ciclo completo Q&A | integration, **critical** |
| `TestKnowledgeLifecycleE2E` | `test_artigo_ingest_then_chat_then_delete` | Ciclo completo Artigo | integration, **critical** |
| `TestPerformanceMetrics` | `test_qa_ingest_under_5_seconds` | Q&A ingestão < 5s | integration |
| `TestPerformanceMetrics` | `test_artigo_ingest_under_30_seconds` | Artigo ingestão < 30s | integration |
| `TestChatRegression` | `test_chat_with_blog_result` | Chat com resultado blog | integration |
| `TestChatRegression` | `test_chat_with_conversa_result` | Chat com resultado conversa | integration |
| `TestChatRegression` | `test_chat_no_results_graceful` | Chat sem resultados | integration |

```bash
pytest tests/integration/test_e2e_knowledge_lifecycle.py -v
```

### 1.5 Fixtures e Mocking

Fixtures definidas em `tests/conftest.py` e injetadas automaticamente pelo pytest.

#### Fixtures de Mocks

| Fixture | O que simula |
|---------|-------------|
| `mock_firestore_client` | Cliente Firestore (sem acesso real ao banco) |
| `mock_vertex_endpoint` | Vertex AI Endpoint (upsert, search, remove) |
| `mock_openai_client` | Cliente OpenAI (embeddings, chat completions) |

```python
def test_exemplo(mock_openai_client):
    mock_openai_client.embeddings.create.return_value.data[0].embedding = [0.1] * 1536
    # ... teste ...
```

#### Fixtures de Dados

| Fixture | Retorna |
|---------|---------|
| `sample_query_vector` | Lista de 1536 floats |
| `sample_blog_document` | Dict com estrutura de documento de blog |
| `sample_conversa_document` | Dict com estrutura de conversa |
| `sample_vertex_neighbors` | Lista de neighbors mockados |
| `sample_source_docs` | Lista de objetos `SourceDoc` |

#### Padrão de Mocking com `@patch`

```python
@patch('main.search_neighbors')
@patch('main.get_embedding')
@pytest.mark.asyncio
async def test_exemplo(self, mock_embed, mock_search):
    mock_embed.return_value = [0.1] * 1536
    mock_search.return_value = [Mock(id="doc1_chk0", distance=0.85)]
    response = await chat(ChatRequest(pergunta="Teste?"))
    assert len(response.fontes) == 1
```

#### Padrão AAA

```python
def test_exemplo():
    # ARRANGE
    entrada = "valor de teste"
    esperado = "resultado esperado"

    # ACT
    resultado = funcao(entrada)

    # ASSERT
    assert resultado == esperado
```

### 1.6 Cobertura de Código

```bash
# Relatório no terminal
./venv/bin/pytest tests/ --cov --cov-report=term-missing

# Relatório HTML interativo
./venv/bin/pytest tests/ --cov --cov-report=html
xdg-open htmlcov/index.html
```

**Status atual:** 88.53% (acima do threshold de 70%).

**Interpretação do relatório:**
- **Stmts:** Total de linhas de código
- **Miss:** Linhas não testadas
- **Cover:** Porcentagem de cobertura
- **Missing:** Números das linhas não testadas

### 1.7 Resolução de Problemas

| Problema | Causa | Solução |
|----------|-------|---------|
| `pytest: command not found` | venv não ativado | `source venv/bin/activate` ou usar `./venv/bin/pytest` |
| `ModuleNotFoundError: No module named 'pytest'` | pytest não instalado no venv | `./venv/bin/pip install -r requirements-dev.txt` |
| `FileNotFoundError: pytest.ini` | Executando pytest no diretório errado | Executar a partir de `arquitetura_RAG/` |
| `ImportError: cannot import name 'X'` | Caminho incorreto para módulo | Verificar `sys.path.insert` no topo dos arquivos de teste |
| `Coverage.py warning: No data was collected` | Configuração incorreta | Verificar `pytest.ini` e rodar `pytest --collect-only` |
| `FAILED coverage: 70% not reached` | Cobertura abaixo do threshold | Adicionar mais testes ou usar `--no-cov-fail` temporariamente |
| `PermissionError: htmlcov/index.html` | Arquivo aberto em outro programa | `rm -rf htmlcov/ && pytest tests/ --cov --cov-report=html` |

### 1.8 Boas Práticas

**Antes de commitar:**
```bash
./venv/bin/pytest tests/ -v          # todos os testes
./venv/bin/pytest tests/ --cov       # verificar cobertura
```

**Nomenclatura:**
- Arquivos: `test_*.py`
- Classes: `Test*`
- Funções: `test_*`
- Nomes descritivos do comportamento testado (ex: `test_blog_boost_applied_correctly`)

**Marcadores — sempre categorizar:**
```python
@pytest.mark.unit
@pytest.mark.critical
def test_funcionalidade_critica():
    """
    Descrição do que está sendo testado.
    Cenário: entrada → saída esperada.
    """
    ...
```

**Mocking — nunca acessar APIs externas nos testes:**
- Usar `@patch('main.funcao')` para serviços (OpenAI, Firestore, Vertex AI)
- Fixtures em `conftest.py` para dados reutilizáveis

---

## 2. Iniciar a API Localmente

```bash
cd arquitetura_RAG/app_rag
../venv/bin/uvicorn main:app --reload --port 8000
```

- Swagger UI: **http://localhost:8000/docs**
- OpenAPI JSON: **http://localhost:8000/openapi.json**

---

## 3. Mapa de Endpoints

| # | Metodo | Rota | Descricao |
|---|--------|------|-----------|
| 1 | POST | `/knowledge/ingest/blog` | Ingestao de artigo (URL, arquivo ou texto). LLM infere metadados. |
| 2 | POST | `/knowledge/ingest/qa` | Ingestao de Q&A (arquivo ou texto). LLM infere metadados. |
| 3 | POST | `/knowledge/delete` | Exclusao unificada (1 ou mais IDs separados por virgula) |
| 4 | GET | `/knowledge` | Listagem com paginacao |
| 5 | POST | `/chat` | Busca RAG — retorna apenas fontes (sem resposta LLM) |

### Fluxo de ingestao
Ambos os endpoints de ingestao chamam o LLM automaticamente para inferir os metadados (titulo, categoria, tags). Para blogs, o LLM tambem reestrutura o conteudo em markdown limpo.

---

## 4. Testes via Terminal (curl)

### 4.1 Ingerir Blog via URL
```bash
curl -s -X POST http://localhost:8000/knowledge/ingest/blog \
  -F "url=https://contabilizei.com.br/contabilidade/o-que-e-contabilidade" \
  | python3 -m json.tool
```

**Resposta esperada:**
```json
{
    "status": "success",
    "message": "Artigo ingerido com sucesso",
    "documento": {
        "id": "api_ingest_20260217XXXXXX_Titulo_Inferido_pela_LLM",
        "tipo": "artigo",
        "titulo": "Titulo extraido pela LLM da pagina",
        "categoria": "Contabilidade",
        "tags": ["contabilidade", "empresa", "CNPJ"],
        "chunks_criados": 5,
        "vetores_criados": 5,
        "parents_criados": 3,
        "resumo": "Resumo gerado pela LLM em 2-3 linhas."
    }
}
```

### 4.2 Ingerir Blog via Arquivo
```bash
cat > /tmp/artigo.md << 'EOF'
MEI: tudo o que voce precisa saber

O Microempreendedor Individual e a forma mais simples de formalizar
um negocio no Brasil. Oferece CNPJ, emissao de nota fiscal e beneficios.

Quem pode ser MEI?
Profissionais que faturam ate R$81 mil por ano e atuam em atividades permitidas.

Quanto custa ser MEI?
O DAS (boleto mensal) custa em torno de R$71 a R$75 por mes em 2026.
EOF

curl -s -X POST http://localhost:8000/knowledge/ingest/blog \
  -F "arquivo=@/tmp/artigo.md" \
  | python3 -m json.tool
```

### 4.3 Ingerir Blog via Texto Raw
```bash
curl -s -X POST http://localhost:8000/knowledge/ingest/blog \
  -F "conteudo=O Simples Nacional e um regime tributario simplificado para micro e pequenas empresas no Brasil. Ele unifica o pagamento de tributos em uma unica guia, o DAS. As empresas podem optar pelo Simples Nacional se tiverem faturamento ate R\$4,8 milhoes por ano." \
  | python3 -m json.tool
```

### 4.4 Ingerir Q&A via Texto Raw
```bash
curl -s -X POST http://localhost:8000/knowledge/ingest/qa \
  -F "conteudo=Pergunta: O que e o DAS do MEI? Resposta: O DAS (Documento de Arrecadacao do Simples Nacional) e o boleto mensal que todo MEI deve pagar para manter seu CNPJ ativo e ter acesso a beneficios previdenciarios. O valor em 2026 e de R\$71 para comercio e R\$75 para servicos." \
  | python3 -m json.tool
```

**Resposta esperada:**
```json
{
    "status": "success",
    "message": "Conhecimento Q&A ingerido com sucesso",
    "documento": {
        "id": "api_ingest_20260217XXXXXX_Titulo_Inferido_pela_LLM",
        "tipo": "qa",
        "titulo": "O que e o DAS do MEI e quanto custa",
        "categoria": "Tributacao",
        "tags": ["MEI", "DAS", "boleto"],
        "chunks_criados": 1,
        "vetores_criados": 1,
        "parents_criados": null,
        "resumo": null
    }
}
```

> **ANOTE O ID RETORNADO** — voce vai precisar dele para exclusao.

### 4.5 Ingerir Q&A via Upload de Arquivo
```bash
echo "Pergunta: Qual o prazo do DAS? Resposta: O prazo e ate o dia 20 de cada mes. Se cair no fim de semana ou feriado, o prazo antecipa para o ultimo dia util anterior." > /tmp/qa.txt

curl -s -X POST http://localhost:8000/knowledge/ingest/qa \
  -F "arquivo=@/tmp/qa.txt" \
  | python3 -m json.tool
```

### 4.6 Verificar no /chat
```bash
# Busca que deve retornar as fontes do conteudo recem-ingerido
curl -s -X POST http://localhost:8000/chat \
  -H "Content-Type: application/json" \
  -d '{"pergunta": "O que e o DAS do MEI?"}' | python3 -m json.tool

# Filtrar apenas por blog
curl -s -X POST http://localhost:8000/chat \
  -H "Content-Type: application/json" \
  -d '{"pergunta": "O que e o Simples Nacional?", "filtro_tipo": "blog"}' | python3 -m json.tool
```

**Resposta esperada:**
```json
{
    "fontes": [
        {
            "id": "api_ingest_20260219XXXXXX_titulo_secao",
            "score": 0.92,
            "conteudo": "Texto do chunk recuperado...",
            "titulo": "Titulo do artigo",
            "url": "https://contabilizei.com.br/...",
            "tipo": "blog"
        }
    ]
}
```

> O campo `resposta` foi removido — o agente consumidor e responsavel por gerar a resposta final com base nas fontes.

### 4.7 Listar Conhecimento
```bash
# Listar todos
curl -s http://localhost:8000/knowledge | python3 -m json.tool

# Filtrar por tipo
curl -s "http://localhost:8000/knowledge?tipo=conversa" | python3 -m json.tool   # Q&A e conversas
curl -s "http://localhost:8000/knowledge?tipo=blog" | python3 -m json.tool       # Artigos (alias: blog_section)

# Com paginacao
curl -s "http://localhost:8000/knowledge?limit=10&offset=0" | python3 -m json.tool
```

> **Valores do filtro `tipo`:**
> - `tipo=blog` ou `tipo=blog_section` — retorna seções de artigos (`type == "blog_section"`)
> - `tipo=conversa` — retorna Q&A e conversas (`type != "blog_section"`)
> - sem filtro — retorna todos os documentos

### 4.8 Validações de Segurança
```bash
# URL para host interno bloqueado (anti-SSRF) → 422
curl -s -X POST http://localhost:8000/knowledge/ingest/blog \
  -F "url=http://169.254.169.254/metadata" | python3 -m json.tool
# {"detail": "URL aponta para recurso interno bloqueado."}

# Protocolo inválido → 422
curl -s -X POST http://localhost:8000/knowledge/ingest/blog \
  -F "url=file:///etc/passwd" | python3 -m json.tool
# {"detail": "Protocolo não permitido: 'file'. Use http ou https."}

# Arquivo muito grande (> 100KB) → 422
# Conteúdo muito curto (< 10 chars) → 422
# Extensão não suportada (.csv, .pdf) → 422
```

### 4.9 Excluir 1 Documento
```bash
curl -s -X POST http://localhost:8000/knowledge/delete \
  -H "Content-Type: application/json" \
  -d '{"ids": "SEU_DOCUMENT_ID"}' | python3 -m json.tool
```

### 4.10 Excluir Multiplos Documentos
```bash
curl -s -X POST http://localhost:8000/knowledge/delete \
  -H "Content-Type: application/json" \
  -d '{"ids": "ID_1, ID_2, ID_3"}' | python3 -m json.tool
```

**Resposta esperada (todos removidos):**
```json
{
    "status": "success",
    "message": "3 de 3 documentos removidos",
    "total_solicitados": 3,
    "total_removidos": 3,
    "resultados": [
        {"document_id": "ID_1", "tipo": "conversa", "status": "success"},
        {"document_id": "ID_2", "tipo": "blog_section", "status": "success"},
        {"document_id": "ID_3", "tipo": "conversa", "status": "success"}
    ]
}
```

---

## 5. Testes via Swagger UI

Acesse **http://localhost:8000/docs**

### 5.1 Ingerir Blog
1. Clique em **POST /knowledge/ingest/blog** > "Try it out"
2. Escolha o modo:
   - **URL**: preencha apenas `url` com a URL do artigo
   - **Arquivo**: no campo `arquivo`, clique em "Choose File" e selecione `.txt` ou `.md`
   - **Texto**: preencha apenas `conteudo` com o texto raw
3. Clique "Execute"
4. Verifique: status 201, `"status": "success"`, titulo/categoria/tags inferidos

### 5.2 Ingerir Q&A
1. Clique em **POST /knowledge/ingest/qa** > "Try it out"
2. Escolha o modo:
   - **Arquivo**: selecione arquivo `.txt` ou `.md`
   - **Texto**: preencha `conteudo` com o texto do Q&A
3. Clique "Execute"
4. Verifique: status 201, `"chunks_criados": 1`, titulo/categoria/tags inferidos

### 5.3 Excluir Documentos
1. Clique em **POST /knowledge/delete** > "Try it out"
2. Cole no body:
```json
{"ids": "SEU_ID_AQUI"}
```
   Ou multiplos:
```json
{"ids": "ID_1, ID_2, ID_3"}
```
3. Clique "Execute"
4. Verifique: `total_removidos` == numero de IDs validos

### 5.4 Chat
1. Clique em **POST /chat** > "Try it out"
2. Cole no body:
```json
{
  "pergunta": "O que e o DAS do MEI?"
}
```
3. Clique "Execute"
4. Verifique: response contem campo `fontes` com documentos relevantes (sem campo `resposta`)

---

## 6. Testes via Postman

### 6.1 Configuracao Inicial
1. Abra o Postman e crie uma nova Collection: **"Contabilizei RAG API"**
2. Crie uma variavel de ambiente `baseUrl` com valor `http://localhost:8000`

### 6.2 Ingerir Blog via URL

- **Metodo**: POST
- **URL**: `{{baseUrl}}/knowledge/ingest/blog`
- **Body**: selecione `form-data`

| Key | Value | Type |
|-----|-------|------|
| url | https://contabilizei.com.br/contabilidade/o-que-e-contabilidade | Text |

Clique em **Send**.

### 6.3 Ingerir Blog via Arquivo

- **Metodo**: POST
- **URL**: `{{baseUrl}}/knowledge/ingest/blog`
- **Body**: selecione `form-data`

| Key | Value | Type |
|-----|-------|------|
| arquivo | *selecione o arquivo .txt ou .md* | File |

> No campo `Type` da linha `arquivo`, mude de `Text` para `File`, depois clique em "Select Files".

### 6.4 Ingerir Blog via Texto Raw

- **Metodo**: POST
- **URL**: `{{baseUrl}}/knowledge/ingest/blog`
- **Body**: selecione `form-data`

| Key | Value | Type |
|-----|-------|------|
| conteudo | O Simples Nacional e um regime tributario para PMEs... | Text |

### 6.5 Ingerir Q&A via Texto Raw

- **Metodo**: POST
- **URL**: `{{baseUrl}}/knowledge/ingest/qa`
- **Body**: selecione `form-data`

| Key | Value | Type |
|-----|-------|------|
| conteudo | Pergunta: O que e o DAS? Resposta: O DAS e o boleto mensal do MEI. | Text |

### 6.6 Ingerir Q&A via Arquivo

- **Metodo**: POST
- **URL**: `{{baseUrl}}/knowledge/ingest/qa`
- **Body**: selecione `form-data`

| Key | Value | Type |
|-----|-------|------|
| arquivo | *selecione o arquivo .txt ou .md* | File |

### 6.7 Excluir Documentos

- **Metodo**: POST
- **URL**: `{{baseUrl}}/knowledge/delete`
- **Body**: selecione `raw` + `JSON`

```json
{
  "ids": "api_ingest_20260217XXXXXX_titulo"
}
```

Para multiplos:
```json
{
  "ids": "ID_1, ID_2, ID_3"
}
```

### 6.8 Listar Conhecimento

- **Metodo**: GET
- **URL**: `{{baseUrl}}/knowledge`
- **Params** (opcional):

| Key | Value |
|-----|-------|
| tipo | conversa |
| limit | 10 |
| offset | 0 |

### 6.9 Chat

- **Metodo**: POST
- **URL**: `{{baseUrl}}/chat`
- **Body**: selecione `raw` + `JSON`

```json
{
  "pergunta": "O que e o DAS do MEI?"
}
```

### 6.10 Dicas no Postman

- **Salvar IDs**: no painel "Tests" de cada requisicao de ingestao, cole o script para salvar o ID automaticamente:
```javascript
const json = pm.response.json();
pm.environment.set("last_doc_id", json.documento.id);
```
- Depois use `{{last_doc_id}}` nos campos de exclusao.

---

## 7. Verificacao Direta no Google Cloud

### 7.1 Firestore

**Via Console Web:**
1. Acesse https://console.cloud.google.com/firestore
2. Projeto: `ctbz-ia-assessoria-poc` | Banco: `rag-db-v1`
3. Collection: `knowledge_base`
4. Verifique campos: `type`, `source`, `base_article_id` (artigos), `metadata.categoria`, `metadata.tags`

**Via Python:**
```python
from google.cloud import firestore
db = firestore.Client(project="ctbz-ia-assessoria-poc", database="rag-db-v1")

# Listar ingestoes via API
docs = db.collection("knowledge_base") \
    .where("source", "==", "api_ingest_v1") \
    .limit(20).stream()

for doc in docs:
    d = doc.to_dict()
    print(f"{doc.id} | {d.get('type')} | {d.get('title','')[:60]}")
    print(f"  categoria: {d.get('metadata', {}).get('categoria')}")
    print(f"  tags: {d.get('metadata', {}).get('tags')}")
```

### 7.2 Vertex AI Vector Search

**Via gcloud:**
```bash
gcloud ai indexes describe 2352965878556917760 \
  --region=us-central1 \
  --project=ctbz-ia-assessoria-poc
```

---

## 8. Checklist de Verificacao Final

### Testes Automatizados
- [ ] `./venv/bin/pytest tests/ -v` - Todos os testes passam
- [ ] Cobertura >= 70%

### Blog — URL
- [ ] `POST /knowledge/ingest/blog` com `url` retorna 201
- [ ] LLM infere titulo, categoria, tags e resumo automaticamente
- [ ] `parents_criados >= 1`, `chunks_criados >= 1`
- [ ] `/chat` retorna o conteudo quando pergunta relacionada

### Blog — Arquivo / Texto
- [ ] `POST /knowledge/ingest/blog` com `arquivo` (.md) retorna 201
- [ ] `POST /knowledge/ingest/blog` com `conteudo` retorna 201
- [ ] LLM reestrutura o conteudo em markdown
- [ ] Extensao invalida (.csv) retorna 422

### Q&A — Arquivo / Texto
- [ ] `POST /knowledge/ingest/qa` com `conteudo` retorna 201
- [ ] `POST /knowledge/ingest/qa` com `arquivo` (.txt) retorna 201
- [ ] LLM infere titulo, categoria e tags
- [ ] `chunks_criados == 1`, `vetores_criados == 1`
- [ ] `/chat` retorna o conteudo quando pergunta relacionada

### Exclusao
- [ ] `POST /knowledge/delete` com 1 ID remove corretamente
- [ ] `POST /knowledge/delete` com N IDs (comma-separated) remove todos
- [ ] ID inexistente retorna `"status": "partial"` com mensagem de erro
- [ ] Exclusao de artigo remove todos os parents + vetores
- [ ] `/chat` NAO retorna conteudo excluido

### Listagem e Chat
- [ ] `GET /knowledge` retorna lista paginada
- [ ] `GET /knowledge?tipo=conversa` filtra corretamente
- [ ] `/chat` retorna `{"fontes": [...]}` com documentos relevantes
- [ ] `/chat` sem resultados retorna `{"fontes": []}`

---

## 9. Logs de Observabilidade

```
# Blog via URL
INFO: Ingestao blog URL: url='https://...'
INFO: Inferindo metadados de blog via LLM (gpt-4o-mini), content_len=5234
INFO: Metadados blog inferidos: titulo='...' categoria='...' tags=[...]
INFO: Ingestao blog concluida: id=... titulo='...' chunks=5 duration_ms=3210

# Q&A via texto
INFO: Ingestao Q&A texto
INFO: Inferindo metadados de Q&A via LLM (gpt-4o-mini), content_len=342
INFO: Metadados Q&A inferidos: titulo='...' categoria='...' tags=[...]
INFO: Ingestao Q&A concluida: id=... titulo='...' duration_ms=1450

# Exclusao
INFO: Exclusao: 2 documento(s) solicitado(s)
INFO: Exclusao concluida: 2/2 removidos duration_ms=890
```

**Metricas de performance esperadas:**
- Ingestao Q&A: < 5.000ms (inclui chamada LLM)
- Ingestao blog (10 secoes): < 30.000ms (inclui chamada LLM)
- Ingestao via URL: < 60.000ms (inclui download + LLM)
- Exclusao unitaria: < 10.000ms
- Exclusao em lote (10 IDs): < 30.000ms
- Chat (busca RAG): < 2.000ms (sem chamada LLM)

---

## 10. Integracao com o Agente do Pedro (Cloud Run)

### URL Base da API (producao)

```
https://rag-api-55837972640.us-central1.run.app
```

> Nao use a URL do Swagger (`/docs`) — use a URL base acima.

### Autenticacao (service-to-service GCP)

O servico esta privado. O agente do Pedro (`agente-contabil`) ja tem permissao `roles/run.invoker`.
A autenticacao e feita via token OIDC da service account do Cloud Run — sem configuracao adicional.

```python
import google.auth.transport.requests
from google.oauth2 import id_token
import requests

RAG_URL = "https://rag-api-55837972640.us-central1.run.app"

def buscar_fontes(pergunta: str, filtro_tipo: str = None) -> list[dict]:
    auth_req = google.auth.transport.requests.Request()
    token = id_token.fetch_id_token(auth_req, RAG_URL)

    payload = {"pergunta": pergunta}
    if filtro_tipo:
        payload["filtro_tipo"] = filtro_tipo  # "blog" ou "conversa"

    response = requests.post(
        f"{RAG_URL}/chat",
        json=payload,
        headers={"Authorization": f"Bearer {token}"},
        timeout=10,
    )
    response.raise_for_status()
    return response.json()["fontes"]
```

### Campos de cada fonte retornada

| Campo | Tipo | Descricao |
|-------|------|-----------|
| `id` | str | ID do documento no Firestore |
| `score` | float | Relevancia (0.50 a ~1.15) |
| `conteudo` | str | Texto do chunk recuperado |
| `titulo` | str | Titulo do artigo ou arquivo |
| `url` | str ou null | URL original (blogs) |
| `tipo` | str | `"blog"` ou `"conversa"` |
