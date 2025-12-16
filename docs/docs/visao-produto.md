# 📄 Visão de Produto

## 🗓 Informações Gerais

- **Nome do Projeto:** Agente de IA Especializado em Contabilidade

- **Cliente:** Contabilizei Tecnologia LTDA

- **Responsável da Visão de Produto (PO):** Thiago Gomes

- **Duração Total (contratual):** 6 semanas

- **Período na Etapa de Desenvolvimento (estimado):** 5 semanas (Refatoração, Integração, Homologação e Documentação)

---

## ✅ Checklist de Entrada (para iniciar o projeto)

- [x] Reunião de Kickoff com o cliente realizada
- [x] Objetivo do projeto compreendido
- [x] Tecnologias necessárias mapeadas (Google Cloud, ADK, Gemini, HubSpot)
- [x] Estimativa de esforço feita (522 horas)
- [x] Capacidade do time verificada
- [x] Escopo inicial aprovado pelo cliente (Contrato assinado em 18/11/2025)

---

## 📤 Checklist de Saída (para encaminhar o projeto às próximas áreas)

- [X] Documento de Visão preenchido e validado
- [X] Matriz "é/não é/faz/não faz" definida
- [X] Epics e User Stories redigidas
- [X] Arquitetura técnica documentada
- [X] Datas de entrada/saída em cada fase definidas (Anexo I - Cronograma)
- [X] Contrato e escopo revisados e claros
- [X] Alinhamento técnico com área de Desenvolvimento realizado

---

## 📘 Resumo do Projeto

**Descrição:**
Desenvolvimento de um agente de Inteligência Artificial especializado em contabilidade que atua como consultor, capaz de escalar o atendimento a leads e clientes da Contabilizei, fornecendo informações contábeis precisas e confiáveis de forma natural e contextualizada.

**Objetivos:**
- Reduzir esforço humano em atendimentos repetitivos
- Melhorar experiência do usuário mantendo qualidade e consistência
- Evoluir PoC existente para padrão de produção
- Garantir conformidade com LGPD e políticas de compliance
- Integrar diretamente com HubSpot (CRM da Contabilizei)
- Converter visitantes interessados em contabilidade em leads qualificados

**Público-Alvo:**
- Visitantes interessados em aprender sobre contabilidade (potenciais leads)
- Leads em processo de conversão
- Clientes atuais da Contabilizei com dúvidas contábeis
- Equipe de atendimento (receptores de transbordo)

## 👤 Personas

**Visitante/Curioso:**
- Pessoa interessada em aprender sobre contabilidade em geral
- Pode ser empreendedor em fase de pesquisa, estudante ou profissional buscando conhecimento
- Busca entender conceitos básicos, tirar dúvidas introdutórias
- Representa oportunidade de conversão em lead qualificado pela Contabilizei
- Espera respostas educativas e acessíveis

**Lead/Prospect:**
- Pessoa interessada em serviços contábeis da Contabilizei
- Já demonstrou intenção de contratar ou avaliar soluções contábeis
- Busca esclarecer dúvidas básicas e intermediárias sobre contabilidade aplicada ao seu negócio
- Precisa de respostas rápidas, objetivas e confiáveis antes de contratar

**Cliente Ativo:**
- Já contratou serviços da Contabilizei
- Possui dúvidas sobre sua situação contábil específica
- Necessita de orientação contextualizada sobre regimes tributários, obrigações, alíquotas
- Espera que o agente tenha contexto sobre seu histórico

**Atendente Humano:**
- Recebe casos de transbordo quando o agente atinge limite de confiança
- Precisa de contexto completo da conversa para dar continuidade ao atendimento
- Analisa KPIs de qualidade do agente
- Pode identificar oportunidades de conversão em leads durante transbordo

---

## 🧩 Matriz "É / Não É / Faz / Não Faz"

<div align="center">

| Categoria  | Descrição |
|-----------|-----------|
| **É**     | Um agente conversacional de IA especializado em contabilidade brasileira, integrado ao HubSpot, com arquitetura em Google Cloud e capacidade de raciocínio contextual usando OpenAI Chat Kit |
| **Não É** | Um sistema de gestão contábil completo, não substitui contadores humanos, não é um chatbot genérico baseado apenas em templates |
| **Faz**   | Responde dúvidas contábeis educativas e consultivas, investiga contexto do usuário fazendo perguntas, relaciona informações (alíquotas, regimes, obrigações), mantém memória conversacional, identifica e qualifica potenciais leads, realiza transbordo inteligente para humanos, registra logs auditáveis |
| **Não Faz** | Processar folhas de pagamento, gerar documentos fiscais oficiais, tomar decisões contábeis críticas sem validação humana, operar sem guardrails de segurança |

</div>

---

## 🧠 Matriz de Certezas, Suposições e Dúvidas

<div align="center">

| Tipo        | Descrição                                                                |
|-------------|--------------------------------------------------------------------------|
| **Certeza**   | O agente deve ser plugável e integrado ao HubSpot; Contabilidade brasileira tem alta interdependência de variáveis (alíquotas, regime, localização, faturamento); Conformidade LGPD é obrigatória; Cliente testará exaustivamente casos de borda; Tecnologia base é OpenAI Chat Kit |
| **Suposição** | Maioria das dúvidas será de nível básico a intermediário; Usuários esperam respostas curtas e objetivas; Transbordo bem implementado aumenta taxa de conversão; Visitantes educados sobre contabilidade têm maior chance de conversão em leads |
| **Dúvida**    | Quais KPIs exatos a Contabilizei considera prioritários? Qual volume de conversas simultâneas esperado? Há base de conhecimento estruturada disponível para RAG? Como identificar momento ideal para tentar converter visitante em lead? |

</div>

---

## 🧱 Epics e User Stories

### 🔹 Epics

- **Epic 1:** Core do Agente (OpenAI Chat Kit + Memória Conversacional)
- **Epic 2:** Pipeline RAG e Base de Conhecimento Contábil
- **Epic 3:** Integração com HubSpot
- **Epic 4:** Segurança, Guardrails e Compliance LGPD
- **Epic 5:** Lógica de Transbordo Inteligente
- **Epic 6:** Qualificação e Conversão de Leads
- **Epic 7:** Observabilidade e KPIs
- **Epic 8:** Infraestrutura Cloud (GCP)
- **Epic 9:** Documentação Técnica

### 🔸 User Stories

#### US1
- **Usuário:** Como um visitante curioso
- **Objetivo:** Quero aprender conceitos básicos de contabilidade
- **Justificativa:** Para entender melhor o mundo contábil e avaliar se preciso de ajuda profissional

#### US2
- **Usuário:** Como um lead
- **Objetivo:** Quero tirar dúvidas sobre regimes tributários
- **Justificativa:** Para entender qual melhor se adequa ao meu negócio antes de contratar

#### US3
- **Usuário:** Como um cliente
- **Objetivo:** Quero que o agente entenda meu contexto (atividade, localização, faturamento)
- **Justificativa:** Para receber orientações contábeis personalizadas e corretas

#### US4
- **Usuário:** Como um visitante
- **Objetivo:** Quero ser identificado como potencial lead quando demonstrar interesse comercial
- **Justificativa:** Para receber informações sobre serviços da Contabilizei no momento certo

#### US5
- **Usuário:** Como um lead
- **Objetivo:** Quero ser direcionado a um atendente humano quando minha dúvida for complexa
- **Justificativa:** Para garantir que receberei suporte adequado sem frustração

#### US6
- **Usuário:** Como atendente humano
- **Objetivo:** Quero receber o histórico completo da conversa no transbordo
- **Justificativa:** Para dar continuidade ao atendimento sem pedir informações repetidas

#### US7
- **Usuário:** Como gestor da Contabilizei
- **Objetivo:** Quero visualizar KPIs de performance do agente (taxa de transbordo, CAC, média de mensagens, taxa de conversão)
- **Justificativa:** Para avaliar ROI e identificar oportunidades de melhoria

#### US8
- **Usuário:** Como responsável de compliance
- **Objetivo:** Quero garantir que dados pessoais sejam tratados conforme LGPD
- **Justificativa:** Para evitar riscos legais e proteger privacidade dos usuários

#### US9
- **Usuário:** Como desenvolvedor da Contabilizei
- **Objetivo:** Quero documentação técnica completa da solução
- **Justificativa:** Para ter autonomia na manutenção e evolução do sistema

#### US10
- **Usuário:** Como o agente de IA
- **Objetivo:** Preciso prevenir alucinações e prompt injections
- **Justificativa:** Para manter confiabilidade e segurança das respostas

---

## ⚙️ Requisitos Funcionais

### Core do Agente
- **RF01** - O agente deve ser construído utilizando OpenAI Chat Kit
- **RF02** - O agente deve manter memória conversacional persistente via Firestore
- **RF03** - O agente deve fazer perguntas investigativas quando faltar contexto
- **RF04** - O agente deve relacionar informações interdependentes (alíquotas, regime, localização, faturamento, atividade)
- **RF05** - Respostas devem ser curtas, objetivas e seguras

### Pipeline RAG e Base de Conhecimento
- **RF06** - Implementar pipeline RAG avançado com otimização de busca e ranqueamento
- **RF07** - Realizar fine-tuning adaptado ao jargão contábil brasileiro
- **RF08** - Armazenar documentos contábeis no Cloud Storage

### Integração HubSpot
- **RF09** - Integrar diretamente com HubSpot API
- **RF10** - Sincronizar histórico de conversas com CRM
- **RF11** - Atualizar status de leads/clientes no HubSpot
- **RF12** - Registrar visitantes como potenciais leads no HubSpot quando demonstrarem interesse comercial

### Lógica de Transbordo
- **RF13** - Definir gatilhos de transbordo baseados em:
  - Limite de confiança do agente
  - Complexidade da dúvida
  - Número de tentativas de resolução
  - Solicitação explícita do usuário
- **RF14** - Transferir contexto completo da conversa no transbordo
- **RF15** - Notificar atendente humano via HubSpot ou webhook

### Qualificação e Conversão de Leads
- **RF16** - Identificar sinais de interesse comercial em visitantes (perguntas sobre serviços, preços, como contratar)
- **RF17** - Coletar informações qualificadoras de forma natural durante conversa (tipo de negócio, porte, localização)
- **RF18** - Sugerir soluções da Contabilizei quando contexto for apropriado
- **RF19** - Registrar nível de qualificação do lead no HubSpot

### Segurança e Compliance
- **RF20** - Implementar guardrails de segurança para:
  - Restringir tópicos sensíveis
  - Prevenir prompt injection
  - Controlar estilo de resposta
- **RF21** - Aplicar mecanismos de anonimização conforme LGPD
- **RF22** - Gerenciar credenciais via Secret Manager
- **RF23** - Controlar acesso com Google Cloud IAM (roles específicos)
- **RF24** - Não aceitar envio de chaves API/senhas por e-mail (usar cofre seguro)

### Observabilidade e KPIs
- **RF25** - Registrar logs centralizados via Cloud Logging
- **RF26** - Monitorar métricas em tempo real (latência, erros, uso de recursos)
- **RF27** - Configurar alertas automáticos (Cloud Monitoring + Pub/Sub)
- **RF28** - Rastrear KPIs:
  - Taxa de transbordo
  - CAC de leads que chegam ao transbordo
  - Média de mensagens por conversa
  - Média de mensagens até transbordo
  - Taxa de conversão visitante → lead
  - Identificação de anomalias

### Infraestrutura Cloud (GCP)
- **RF29** - Orquestração via Google Cloud Run
- **RF30** - Funções event-driven via Cloud Functions (webhooks, gatilhos)
- **RF31** - Banco de dados Firestore para histórico e logs
- **RF32** - Ambiente isolado para desenvolvimento (sem acesso a dados produtivos)
- **RF33** - Acesso auditado via Beyond Trust (sessões gravadas)
- **RF34** - Workspace GitLab isolado e dedicado

### Documentação
- **RF35** - Diagrama de Arquitetura e Integrações atualizado
- **RF36** - Manual de Deploy com lista de variáveis de ambiente
- **RF37** - Documentação das APIs (Swagger)
- **RF38** - Inventário de recursos em nuvem utilizados
- **RF39** - Testes unitários nos principais módulos

### Qualidade de Código
- **RF40** - Código modular e manutenível
- **RF41** - Comentários claros e nomenclatura padronizada
- **RF42** - Variáveis de ambiente para diferentes ambientes de deploy
- **RF43** - Controle de versão no Git

---

## 📱 Responsividade

**O projeto será responsivo?**
- [ ] Sim (não aplicável — backend/agente conversacional)
- [x] Não

**Observação:** O projeto é um agente de IA conversacional (backend), sem interface visual própria. A responsividade será responsabilidade da interface do HubSpot onde o agente será integrado.

---

## 📌 Observações Finais

### Riscos Conhecidos
- **Alucinação do modelo:** Mitigado por RAG otimizado, guardrails e lógica de transbordo
- **Prompt injection:** Mitigado por guardrails de segurança e validações
- **Casos de borda:** Cliente testará exaustivamente — testes unitários e validações rigorosas são essenciais
- **Conversão prematura:** Risco de tentar converter visitantes antes do momento certo — requer lógica sofisticada de qualificação

### Restrições Técnicas
- Acesso restrito a ambiente isolado no GCP (sem dados produtivos)
- Todas as sessões auditadas via Beyond Trust
- Credenciais apenas via Secret Manager (nunca por e-mail)
- Conformidade estrita com LGPD
- Uso de OpenAI Chat Kit como tecnologia base

### Dependências Externas
- OpenAI Chat Kit
- API do HubSpot
- Google Cloud Platform (Cloud Run, Functions, Firestore, Storage, Secret Manager)
- Beyond Trust (gestão de acessos)
- GitLab (repositório isolado)

### Critérios de Sucesso
- Redução mensurável de atendimentos humanos repetitivos
- Taxa de transbordo controlada e justificada
- Ausência de alucinações em casos testados
- Respostas precisas considerando interdependências contábeis
- Taxa de conversão visitante → lead mensurável e crescente
- Conformidade LGPD validada
- KPIs de observabilidade funcionando
- Documentação técnica completa entregue

### Suporte Pós-Entrega
- 3 meses de suporte técnico após entrega (Cláusula 59 do contrato)

---