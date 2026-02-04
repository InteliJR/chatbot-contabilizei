# Documentação Técnica: Pipeline de Ingestão de Conhecimento (ETL de Conversas)

## 1. Visão Geral
Este documento detalha a arquitetura do subsistema de ETL (Extract, Transform, Load) responsável por processar o histórico de atendimento não estruturado (conversas humanas) e transformá-lo em uma Base de Conhecimento (Knowledge Base) estruturada e semanticamente rica para o sistema RAG.

O objetivo é converter "logs de chat sujos" em artefatos de conhecimento atômicos, higienizados e indexáveis.

## 2. Arquitetura Lógica

A solução opera em regime *Batch* (processamento em lote), desenhada com foco em resiliência a falhas de rede e consistência de dados.

![Fluxograma da Arquitetura](../assets/arquitetura_ETL_conversas.png)
*Figura 1: Fluxo de Dados do Pipeline de Conversas*

### Componentes Chave:
1.  **Ingestão Resiliente:** Leitura segura de CSV sem cabeçalho e verificação de integridade mínima.
2.  **Núcleo Cognitivo (GPT-4o):** Extração semântica utilizando *Dual-Layer Strategy* (Macro/Micro) e validação estrita de PII (Dados Pessoais).
3.  **Persistência Estruturada:** Geração de arquivos Markdown com metadados (Frontmatter) organizados por diretórios.

---

## 3. Estratégia de Engenharia de Dados

Para garantir robustez em ambiente de produção, foram implementados os seguintes padrões de design (Design Patterns):

### 3.1. Idempotência e "Smart Resume"
O sistema verifica a existência do diretório de saída antes de processar cada registro.
* **Comportamento:** Se a pasta `knowledge_base_rag/linha_042` já existe, o script entende que o registro foi processado com sucesso e **pula** para o próximo.
* **Benefício:** Permite reiniciar o processamento após falhas (ex: queda de internet) sem duplicar custos de API ou reprocessar dados já consolidados.

### 3.2. Resiliência de API (Exponential Backoff)
Implementação de um *decorator* de retentativa para chamadas à API da OpenAI.
* **Lógica:** Em caso de `RateLimitError` (429) ou instabilidade (5xx), o sistema aguarda progressivamente (15s, 30s, 45s) antes de falhar definitivamente.
* **Fallback:** Erros irrecuperáveis são logados, mas não interrompem o lote inteiro.

### 3.3. Higienização de Links (Strict URL Policy)
Para evitar "alucinações" de links quebrados comuns em LLMs:
* Links incompletos (com "...") são descartados na origem.
* Apenas URLs válidas são persistidas no campo de metadados.

---

## 4. Estratégia Cognitiva (System Prompt V4)

Utilizamos uma abordagem de **Mineração de Conhecimento em Dupla Camada** para maximizar a recuperação (Retrieval) futura:

| Tipo | Descrição | Objetivo no RAG |
| :--- | :--- | :--- |
| **MACRO (Panorama)** | Pergunta abrangente que resume o tópico inteiro da conversa. | Capturar intenções genéricas ("Como funciona o DAS?"). |
| **MICRO (Atômica)** | Perguntas específicas sobre detalhes citados (taxas, prazos, regras). | Capturar dúvidas precisas ("Qual a multa do DAS?"). |

**Regras de Compliance aplicadas via Prompt:**
* **Anonimização:** Remoção total de nomes, CPFs e valores financeiros do cliente.
* **Auditoria de Preços:** Substituição de valores comerciais fixos por termos dinâmicos ("consulte tabela").

---

## 5. Especificação de Saída (Output Schema)

Os dados são persistidos em arquivos `.md` (Markdown) contendo um cabeçalho YAML (Frontmatter) para facilitar a indexação no Vertex AI Search.

### Estrutura de Diretórios
```text
/knowledge_base_rag
    ├── /linha_001/
    │     ├── 00_MACRO_Titulo_do_Topico.md
    │     ├── 01_MICRO_Detalhe_Especifico_1.md
    │     └── 02_MICRO_Detalhe_Especifico_2.md
    ├── /linha_002/
    │     └── ...
```


### Modelo do Arquivo Markdown
```json
source_row: 10
qa_id: 1
category: "Tributação"
tags: ["Simples Nacional", "DAS", "Multas"]
links: ["[https://contabilizei.com.br/blog/das](https://contabilizei.com.br/blog/das)"]
type: "MICRO"
-----

# Título da Pergunta (H1)

Texto da resposta técnica, higienizada e revisada pela IA...

**Links Úteis:**
- https://contabilizei.com.br/blog/das

```

## 6. Como Executar

Pré-requisitos

- Python 3.10+

- Bibliotecas: pandas, openai, pydantic

- Variável de ambiente OPENAI_API_KEY configurada.

### Comando

```py
python etl_production.py
```


### Monitoramento
O script emite logs estruturados no console ([INFO], [PROCESS], [SUCCESS]) para acompanhamento em tempo real.
