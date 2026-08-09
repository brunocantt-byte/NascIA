# NascIA — Assistente de Boletos via IA

Automação que extrai dados estruturados de boletos em PDF (data de vencimento, fornecedor, valor, CNPJ, código de barras, PIX Copia e Cola) usando IA, armazena tudo organizado no Google Sheets, e permite consultar essas informações por chat via Telegram.

## Visão geral

O projeto resolve um problema comum de controle financeiro: boletos chegam em PDF, espalhados, sem organização centralizada. O NascIA automatiza a extração desses dados e disponibiliza um assistente conversacional para consultas rápidas, tipo "quanto devo para o fornecedor X" ou "o que vence essa semana".

## Arquitetura

```
┌─────────────────┐
│  Pasta de PDFs   │
│  (boletos)       │
└────────┬─────────┘
         │
         ▼
┌─────────────────────┐
│  n8n (orquestração)  │
│  - Leitura de PDFs   │
│  - Extração via IA   │
│  - Validação de dados│
└────────┬─────────────┘
         │
         ▼
┌─────────────────────┐        ┌──────────────────────┐
│  Google Sheets        │◄──────│  Telegram Bot          │
│  (armazenamento)      │───────►  (interface de chat)   │
└─────────────────────┘        └──────────────────────┘
```

## Fluxo 1 — Extração de boletos

1. **Trigger manual** inicia o processamento
2. **Leitura da pasta** — varre todos os PDFs de uma pasta local
3. **Loop** — processa um PDF por vez
4. **Extração de texto** do PDF
5. **IA (Google Gemini)** extrai os campos estruturados em JSON:
   - Data de vencimento
   - Nome do fornecedor (cedente/beneficiário)
   - Valor
   - Empresa pagadora
   - CNPJ/CPF do pagador
   - Código de barras / linha digitável
   - PIX Copia e Cola (quando disponível)
6. **Validação automática** — checagem de dígito verificador de CNPJ/CPF (algoritmo oficial), consistência de datas, tamanho do código de barras, e sinalização de campos ausentes ou suspeitos
7. **Gravação no Google Sheets** — cada boleto processado vira uma linha, com uma coluna de alertas para revisão manual quando necessário

## Fluxo 2 — Consulta via chat

1. **Telegram Trigger** recebe a pergunta do usuário
2. **Google Sheets** busca todos os dados registrados
3. **Agregação** dos dados em um único contexto
4. **IA (Google Gemini)** responde com base exclusivamente nos dados reais, cobrindo perguntas sobre fornecedor específico, totais, vencimentos próximos, período, código de barras e PIX
5. **Telegram** envia a resposta de volta ao usuário

## Tecnologias

- **n8n** — orquestração do fluxo (self-hosted)
- **Google Gemini** (via API) — extração e interpretação de linguagem natural
- **Google Sheets API** — armazenamento estruturado
- **Telegram Bot API** — interface conversacional
- **ngrok** — túnel HTTPS para desenvolvimento local
- **JavaScript** (nós Code do n8n) — parsing, formatação e validação de dados

## Decisões técnicas e cuidados

- **Validação matemática de CNPJ/CPF**: em vez de confiar cegamente na extração da IA, o fluxo recalcula o dígito verificador para detectar erros de leitura.
- **Sem confiança cega em dados críticos**: código de barras e PIX Copia e Cola são sinalizados para conferência manual antes de qualquer pagamento — a IA nunca decide um pagamento sozinha.
- **Prevenção de alucinação de datas**: regras explícitas no prompt impedem a IA de usar a data atual como vencimento quando o dado não está claro no documento.
- **Agregação antes da consulta**: os dados são agrupados em um único contexto antes de chegar ao modelo de linguagem, evitando respostas fragmentadas ou inconsistentes.
- **Deduplicação de eventos do Telegram**: proteção contra reprocessamento de mensagens em caso de reenvio pela API do Telegram.

## Como rodar localmente

### Pré-requisitos
- n8n instalado (`npm install n8n -g`)
- Conta Google Cloud com Sheets API e Drive API ativadas
- Bot do Telegram criado via [@BotFather](https://t.me/BotFather)
- [ngrok](https://ngrok.com) para expor um endpoint HTTPS (necessário para o Telegram Trigger)

### Passos
1. Clone este repositório
2. Importe o arquivo `boletos_workflow.json` no n8n (Menu → Import from File)
3. Configure as credenciais:
   - Google Sheets/Drive (OAuth2)
   - Google Gemini (API Key)
   - Telegram (Bot Token)
4. Ajuste o caminho da pasta de PDFs e o ID da planilha do Google Sheets nos respectivos nós
5. Inicie o ngrok: `ngrok http 5678`
6. Inicie o n8n com a URL do webhook configurada:
   ```
   set N8N_RESTRICT_FILE_ACCESS_TO=<caminho da pasta de boletos>
   set WEBHOOK_URL=<url do ngrok>
   n8n start
   ```
7. Publique o workflow no editor do n8n

## Estrutura da planilha

| Coluna | Descrição |
|---|---|
| data_vencimento | Data de vencimento do boleto |
| nome_fornecedor | Cedente/beneficiário |
| valor | Valor numérico (permite soma direta) |
| empresa_pagadora | Sacado/pagador |
| cnpj_pagador | CNPJ ou CPF formatado |
| codigo_barras | Linha digitável |
| pix_copia_cola | Código PIX, quando disponível |
| alertas | Sinalizações automáticas de inconsistência |

## Status do projeto

Projeto em desenvolvimento contínuo como parte de portfólio de estudos em automação com IA (Alura).

## Autor

Bruno Cantanhede — [@brunocantt-byte](https://github.com/brunocantt-byte)
