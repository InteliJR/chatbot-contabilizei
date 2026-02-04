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
- [ ] Wireframes ou protótipos recebidos e validados
- [ ] Modelo de dados alinhado entre frontend e backend
- [x] User Stories priorizadas e estimadas
- [x] Capacidade técnica e de tempo confirmada
- [ ] Entendimento dos custos de manutenção

---

## 📤 Checklist de Saída

- [ ] Funcionalidades desenvolvidas conforme requisitos
- [ ] Deploy realizado (ou instruções de deploy definidas)
- [ ] Documentação técnica entregue (API, banco, estrutura de dados, etc.)
- [ ] Entrega validada com PO

---

## 🛠 Tecnologias Utilizadas

**Backend:**
<!-- Exemplo: Node.js + Express -->


**Banco de Dados:**
<!-- Exemplo: PostgreSQL -->

**Hospedagem:**
<!-- Exemplo: Vercel (frontend), Railway (backend), Supabase (DB) -->

**Outros Serviços:**
<!-- Exemplo: Firebase Auth, SendGrid, AWS S3 -->

---

## 💸 Custos de Manutenção

<!-- Detalhar os custos mensais previstos para manter a aplicação em funcionamento -->
<div align="center">

| Serviço                     | Valor Mensal Estimado | Observações                        |
|----------------------------|------------------------|------------------------------------|
| Hospedagem do Frontend     | R$ 10,00               | Plano gratuito da Vercel é suficiente |
| API / Backend              | R$ 25,00               | Uso do Railway com plano básico   |
| Banco de Dados             | R$ 20,00               | Supabase com 1GB de dados          |
| Domínio                    | R$ 40,00               | Registro anual dividido mensalmente |
| Outros                     | R$ 15,00               | Envio de e-mails via SendGrid     |

</div>


**Total:** R$ 110,00 / mês

---

## 🧱 Infraestrutura de Dados

### 🔗 Modelo Lógico do Banco de Dados

<!-- Inserir imagem ou link para o modelo lógico (diagrama) -->

**Link para o modelo:** 
<!-- Exemplo: https://dbdiagram.io/xyz -->


#### 🛠 Exemplo de Interface (Contrato de Dados)

**Arquivo:** `src/interfaces/IReserva.ts`

```ts
// Interface que define o formato dos dados que o backend deve retornar para o frontend

export interface IReserva {
  id: number;
  active: boolean;
  sala?: string; // opcional
  inicio: Date; // ISO string
  fim: Date; // ISO string
  usuario: {
    nome: string;
    email: string;
  }
}
```
