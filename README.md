# 📘 Agente de IA Especializado em Contabilidade

Solução de inteligência artificial que atua como consultor contábil, escalando o atendimento da Contabilizei através de respostas precisas e contextualizadas, com integração ao HubSpot e capacidade de qualificação de leads.

---

## 📄 Documentação

A documentação completa do projeto pode ser acessada através do link abaixo:

**[Documentação do Projeto](https://intelijr.github.io/chatbot-contabilizei/)**

> A documentação é mantida utilizando o [Docusaurus](https://docusaurus.io/). Para informações sobre como configurar e manter a documentação, consulte o [guia de configuração](./docs/README.md).

---

## 🚀 Tecnologias Utilizadas

- **OpenAI Chat Kit** - Base do agente conversacional
- **Google Cloud Platform**
  - Cloud Run - Orquestração principal
  - Cloud Functions - Event-driven functions
  - Firestore - Banco de dados (histórico e logs)
  - Cloud Storage - Armazenamento de documentos
  - Secret Manager - Gestão de credenciais
  - Cloud Logging & Monitoring - Observabilidade
- **HubSpot API** - Integração com CRM
- **Beyond Trust** - Gestão de acessos auditada
- **GitLab** - Controle de versão
- **Python** - Linguagem principal

---

## 🛠️ Como Rodar o Projeto

```bash
# Clone o repositório
git clone https://github.com/inteli-junior/chatbot-contabilizei.git

# Acesse o diretório do projeto
cd chatbot-contabilizei

# Instale as dependências
pip install -r requirements.txt

# Configure as variáveis de ambiente
cp .env.example .env
# Edite o arquivo .env com suas credenciais

# Execute o projeto localmente
python main.py
```

> **Nota:** Para deploy em produção no Google Cloud Run, consulte o [Manual de Deploy](./docs/docs/desenvolvimento/deploy.md) na documentação.

---

## 🗂️ Estrutura de Diretórios

```bash
.
├── .github/                       # Configurações de CI/CD e templates de PR
│
├── src/                           # Código fonte principal
│   ├── agent/                     # Core do agente (OpenAI Chat Kit)
│   ├── integrations/              # Integrações (HubSpot, Firestore)
│   ├── rag/                       # Pipeline RAG e conhecimento contábil
│   ├── security/                  # Guardrails e validações
│   └── utils/                     # Utilitários e helpers
│
├── tests/                         # Testes unitários
│
├── docs/                          # Documentação Docusaurus
│   ├── docs/
│   │   ├── visao-produto.md       # Documento de Visão de Produto
│   │   ├── arquitetura.md         # Arquitetura e diagramas
│   │   ├── desenvolvimento.md     # Guias de desenvolvimento
│   │   └── api.md                 # Documentação de APIs (Swagger)
│
├── .env.example                   # Exemplo de variáveis de ambiente
├── requirements.txt               # Dependências Python
├── .gitignore                     # Arquivos ignorados pelo Git
└── README.md                      # Este documento
```

---

## 👥 Time do Projeto

<div align="center">
<table>
  <tr>
    <td align="center">
      <a href="https://www.linkedin.com/in/sophia-emanuele-de-senne-silva">
        <img src="https://media.licdn.com/dms/image/v2/D4D03AQEvhjT-DvsRmQ/profile-displayphoto-shrink_400_400/profile-displayphoto-shrink_400_400/0/1726735016729?e=1767225600&v=beta&t=umCf7NvjCsWQUjTSYWxemZ2NqmOOadMKIOCxF8GB1vs" width="150px;" alt="Foto de Sophia" style="border-radius:50%"/>
        <br />
        <b>Sophia</b>
        <br />
        <i>Scrum Master</i>
      </a>
      <br />
      <a href="https://github.com/SophiSenne">
        <img src="https://img.shields.io/badge/GitHub-%23121011.svg?logo=github&logoColor=white)" alt="GitHub" height="25"/>
      </a>
      <a href="https://www.linkedin.com/in/sophia-emanuele-de-senne-silva/">
        <img src="https://custom-icon-badges.demolab.com/badge/LinkedIn-0A66C2?logo=linkedin-white&logoColor=fff" alt="LinkedIn" height="25"/>
      </a>
    </td>
    <td align="center">
      <a href="https://www.linkedin.com/in/pedro-pinheiro-rodrigues-b129b62b7/">
        <img src="https://media.licdn.com/dms/image/v2/D4D03AQGVuyMIFhBTBQ/profile-displayphoto-scale_400_400/B4DZoykInWG8Ag-/0/1761784925700?e=1767225600&v=beta&t=sOuSGBSZAoUc6yg5WsAD-tJiJu5bQFmU0o_MoVglSO0" width="150px;" alt="Foto de Pedro" style="border-radius:50%"/>
        <br />
        <b>Pedro</b>
        <br />
        <i>AI Architect</i>
      </a>
      <br />
      <a href="https://github.com/pedropinrodrigues">
        <img src="https://img.shields.io/badge/GitHub-%23121011.svg?logo=github&logoColor=white)" alt="GitHub" height="25"/>
      </a>
      <a href="https://www.linkedin.com/in/pedro-pinheiro-rodrigues-b129b62b7/">
        <img src="https://custom-icon-badges.demolab.com/badge/LinkedIn-0A66C2?logo=linkedin-white&logoColor=fff" alt="LinkedIn" height="25"/>
      </a>
    </td>
    <td align="center">
      <a href="https://www.linkedin.com/in/odanielaugusto/">
        <img src="https://media.licdn.com/dms/image/v2/D4D03AQF-9oAltLLPMA/profile-displayphoto-scale_400_400/B4DZn4nMP8KQAg-/0/1760812648197?e=1767225600&v=beta&t=fKImdz5TRmXjFHCoE8AGCTbktGYbGaYG-lpvqP5MKfY" width="150px;" alt="Foto de Zé" style="border-radius:50%"/>
        <br />
        <b>Zé</b>
        <br />
        <i>RAG Engineer</i>
      </a>
      <br />
      <a href="https://github.com/ze">
        <img src="https://img.shields.io/badge/GitHub-%23121011.svg?logo=github&logoColor=white)" alt="GitHub" height="25"/>
      </a>
      <a href="https://www.linkedin.com/in/odanielaugusto/">
        <img src="https://custom-icon-badges.demolab.com/badge/LinkedIn-0A66C2?logo=linkedin-white&logoColor=fff" alt="LinkedIn" height="25"/>
      </a>
    </td>
  </tr>
</table>

</div>
