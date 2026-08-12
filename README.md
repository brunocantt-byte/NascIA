![NascIA](./assets/banner.png)

# NascIA — Assistente de Boletos via IA

Automação para extração, organização e consulta de dados de boletos e documentos de cobrança brasileiros usando IA, n8n, Google Sheets e Telegram.

O projeto foi desenvolvido para automatizar uma rotina comum de contas a pagar: receber documentos de cobrança, extrair informações estruturadas, validar campos críticos, armazenar os dados e permitir consultas em linguagem natural.

## Visão geral

O NascIA transforma documentos de cobrança em dados estruturados e consultáveis. A automação extrai informações como data de vencimento, fornecedor, valor, empresa pagadora, CNPJ/CPF, código de barras e PIX Copia e Cola, sinalizando campos ausentes ou suspeitos para revisão.

A consulta dos dados pode ser feita diretamente pelo Telegram, permitindo perguntas como:

- Quais boletos estão cadastrados?
- Quais vencem nos próximos dias?
- Qual é o maior boleto?
- Quais contas estão associadas a determinado fornecedor?

> **Demonstração:** os dados utilizados nas capturas e testes publicados no portfólio são fictícios ou anonimizados e não representam dados financeiros reais.

## Arquitetura

```text
                    ┌──────────────────────────┐
                    │     Documentos / PDFs     │
                    │  Boletos e cobranças      │
                    └────────────┬─────────────┘
                                 │
                                 ▼
                    ┌──────────────────────────┐
                    │          n8n              │
                    │      Hospedado na         │
                    │         Railway           │
                    │                          │
                    │  • Leitura de PDFs       │
                    │  • Extração via IA       │
                    │  • Validação de dados    │
                    │  • Orquestração          │
                    └────────────┬─────────────┘
                                 │
                    ┌────────────┴────────────┐
                    ▼                         ▼
          ┌──────────────────┐      ┌──────────────────┐
          │  Google Sheets   │      │   Telegram Bot   │
          │ Armazenamento    │◄────►│ Interface de     │
          │ estruturado      │      │ consulta         │
          └──────────────────┘      └──────────────────┘

                 Infraestrutura em produção
                        Railway + PostgreSQL
                        + volume persistente
