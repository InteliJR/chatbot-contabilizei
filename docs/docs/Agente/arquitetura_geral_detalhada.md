# Arquitetura Técnica do Sistema RAG: Agente de Inteligência Contabilizei

Esta seção descreve o funcionamento lógico e a infraestrutura do motor de **Retrieval-Augmented Generation (RAG)** projetado para o Agente de IA da Contabilizei .

![Visão Geral da Arquitetura](../assets/fluxograma_RAG_v2.png)
*Figura 1: Pipeline de Ingestão de Dados (ETL) – Demonstração do fluxo desde a coleta das fontes (Blog, Chat, Site) até a indexação híbrida no Vertex AI e Firestore.*

A solução foi arquitetada sobre o **Google Cloud Platform (GCP)** com foco em três pilares: **precisão tributária**, **segurança de dados (LGPD)** e **eficiência de custo (FinOps)** .

A arquitetura é dividida em dois grandes fluxos:
1.  **Pipeline de Ingestão de Dados (ETL)**
2.  **Fluxo de Execução Cognitiva (Runtime)** .

---

## 1. Contexto dos Dados e Estratégia de Ingestão (ETL)

O sistema consome três fontes de dados distintas, exigindo estratégias de processamento (*Chunking*) diferenciadas para maximizar a recuperação da informação .

### As Fontes de Dados
1.  **Blog Contabilizei (780+ Artigos):** Conteúdo denso e legislativo .
2.  **Histórico de Atendimento (Conversas Reais):** Dados não estruturados ("sujos") que representam a voz do cliente .
3.  **Site Institucional:** Dados estruturados sobre planos e serviços .

### O Pipeline de Processamento (Batch)
O processo de ingestão não ocorre em tempo real, mas via **Cloud Run Jobs** (processamento em lote), garantindo economia de recursos.

#### 1. Higienização e Normalização (Scripts Python)
Antes de entrar na nuvem, scripts dedicados convertem todas as fontes para o formato **Markdown** .

> **Justificativa:** O Markdown preserva a estrutura semântica (títulos, negritos) e, crucialmente, mantém a legibilidade de tabelas tributárias, que seriam perdidas em texto puro .

*Nota:* No caso das conversas de chat, utilizamos uma LLM intermediária para extrair pares limpos de Pergunta & Resposta (Q&A), descartando ruídos da conversa ("oi", "bom dia") .

#### 2. Staging Area (Cloud Storage)
Os arquivos Markdown normalizados são depositados em um bucket do **Google Cloud Storage** . Isso desacopla a coleta do processamento, criando um "Data Lake" de documentos brutos para auditoria futura .

#### 3. Estratégias de Fragmentação (Chunking Inteligente)
O "Ingestion Worker" (Cloud Run Job) lê os arquivos e aplica estratégias distintas baseadas no tipo de conteúdo :

* **Parent-Child Chunking (Para Blogs/Leis):** O documento é quebrado em fragmentos pequenos ("Filhos") para a busca vetorial, mas mantém o vínculo com o bloco de texto maior ("Pai").
    > **Justificativa:** Permite encontrar um trecho específico (ex: uma alíquota), mas entrega para a IA o parágrafo inteiro (contexto), evitando que exceções legais sejam cortadas .
* **Atomic Chunking (Para Q&A):** O par "Pergunta + Resposta" é tratado como uma unidade indivisível .

#### 4. Indexação Híbrida (Vertex AI + Firestore)
Adotamos uma arquitetura de separação de responsabilidades para performance extrema :

* **Vertex AI Vector Search:** Armazena apenas os vetores (Embeddings gerados pelo modelo `text-embedding-3-small` da OpenAI). É otimizado para busca matemática de similaridade em milissegundos .
* **Google Firestore:** Armazena o conteúdo textual (Payload) e metadados.

Quando o Vertex AI encontra o "ID" do vetor mais próximo, o sistema busca o texto completo no Firestore .

---

## 2. Fluxo de Execução Cognitiva (Runtime)

Quando um usuário envia uma mensagem, o sistema segue um pipeline rigoroso de segurança e orquestração antes de gerar qualquer resposta.

![Fluxo de Execução e Segurança](../assets/fluxograma_usuário_v2.png)
*Figura 2: Fluxo de Execução (Runtime) – Detalhamento das camadas de segurança (Edge), classificação de intenção e orquestração da resposta via RAG.*

### Camada 1: Segurança e Proteção (Edge)
Antes de invocar modelos caros, a requisição passa por filtros de proteção :

1.  **Rate Limiting (Redis):** Bloqueia abusos e ataques DDoS baseados em volume de requisições, protegendo o orçamento.
2.  **PII Masking (DLP):** Um middleware intercepta dados sensíveis (CPF, Salário, E-mail) e os substitui por tokens seguros (ex: `<CPF_ID>`) .
    > **Justificativa:** Garante que dados pessoais de clientes nunca sejam enviados para a API da OpenAI, assegurando conformidade total com a LGPD .
3.  **Input Guardrails:** Verifica tentativas de "Jailbreak" ou "Prompt Injection" para manipular o comportamento do bot .

### Camada 2: Orquestração e Raciocínio (Cloud Run)
O "Cérebro" do sistema é uma aplicação Python (FastAPI) no **Cloud Run** :

1.  **Router de Intenção:** Classifica a pergunta.
    * Se for uma intenção de compra, desvia para um fluxo de SDR (Sales Development) para coleta de leads .
    * Se for técnica, aciona o motor RAG .
2.  **Recuperação (Retrieval):**
    * Converte a pergunta do usuário em vetor (OpenAI).
    * Consulta o **Vertex AI** para obter os IDs dos documentos mais relevantes .
    * Recupera os textos originais ("Pais" ou Respostas de FAQ) no **Firestore** .
3.  **Re-ranking (Cross-Encoder):**
    * Uma etapa de refinamento onde um modelo especializado reordena os documentos recuperados, garantindo que o topo da lista seja semanticamente preciso para a pergunta fiscal específica .

### Camada 3: Geração e Auditoria
1.  **Geração (LLM):** O prompt é montado com o contexto recuperado e enviado ao **GPT-4o** . O modelo atua estritamente com base nos dados fornecidos ("Grounding"), minimizando alucinações .
2.  **Output Guardrails:** A resposta gerada é verificada contra termos proibidos ou consultoria jurídica indevida .
3.  **PII Unmasking:** Se a resposta for segura, os tokens `<CPF_ID>` são revertidos para os dados originais do usuário .
4.  **Traceability:** Todo o processo (Prompt, Documentos Usados, Latência, Tokens) é logado no **Cloud Logging** para auditoria e melhoria contínua .

Esta arquitetura garante que a Contabilizei disponha de um agente que não apenas responde, mas que opera dentro de limites estritos de segurança, conformidade e precisão técnica .