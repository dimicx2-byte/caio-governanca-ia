# Guia de Integrações — CAIO AI Governance Platform

Este documento descreve como o protótipo se comporta hoje e **como ligar cada integração de verdade** ao migrar para produção (com backend).

---

## 1. Gerador de PDF (jsPDF)

**Como funciona na v1**
- A geração de PDF do AI Bridge Canvas usa a biblioteca **jsPDF** carregada via CDN:
  ```html
  <script src="https://cdnjs.cloudflare.com/ajax/libs/jspdf/2.5.1/jspdf.umd.min.js"></script>
  ```
- O botão **"Gerar PDF"** (na barra de ações do Canvas) monta o documento com os post‑its coloridos por grupo e **baixa o arquivo `.pdf` diretamente** (sem diálogo de impressão).
- Botão **"Vincular ao Kanban"** é uma ação **separada**: cria/atualiza o card *"AI Bridge Canvas preenchido"* no Kanban do projeto e anexa um snapshot do canvas.

**Requisitos / observações**
- Precisa de **internet** para baixar a lib (CDN). Sem conexão, o app exibe um aviso.
- Para uso 100% offline, baixe `jspdf.umd.min.js` e referencie localmente:
  ```html
  <script src="./vendor/jspdf.umd.min.js"></script>
  ```
- As páginas dos demais módulos têm **"Exportar PDF"** via impressão do navegador (Salvar como PDF), com um CSS de impressão dedicado.

---

## 2. Pagamentos — PagSeguro / PagBank

**Como funciona na v1 (simulado)**
- No painel **Admin → Configurações de Pagamento & Checkout** é possível definir **preço mensal**, **parcelamento** e **conectar o PagSeguro** (e‑mail + token), tudo armazenado em `localStorage` (`aiGov_billing_config`).
- O **checkout** (cartão/PIX/boleto), **cupons** e a ativação do Premium são **simulados** — nenhuma cobrança real é feita.

**Como ligar de verdade (produção)**

> Importante: **token/credenciais nunca devem ficar no front‑end.** A cobrança real exige um servidor.

1. **Conta e credenciais**
   - Crie a conta no PagBank/PagSeguro e gere o **token de integração** (e, para a API nova, `client_id`/`secret` OAuth) no painel do PagSeguro.
   - Guarde como **variáveis de ambiente** no backend (ex.: `PAGSEGURO_TOKEN`, `PAGSEGURO_EMAIL`).

2. **Backend (ex.: Node/Express)** — endpoints sugeridos:
   - `POST /api/checkout` → recebe plano, método (cartão/PIX/boleto), cupom e dados do tenant; chama a **API de Orders/Checkout do PagSeguro** para criar a cobrança e retorna a URL/QR de pagamento.
   - `POST /api/webhooks/pagseguro` → recebe a **notificação** do PagSeguro; ao status `PAID`, ativa o plano Premium do tenant no banco (equivalente ao `completeUpgrade()` de hoje).
   - `GET /api/billing` → devolve preço/parcelamento configurados pelo admin.

3. **Webhook**
   - No painel do PagSeguro, cadastre a URL pública do webhook (ex.: `https://SEU_APP/api/webhooks/pagseguro`).
   - Valide a autenticidade da notificação antes de ativar o plano.

4. **Front‑end**
   - Troque a função de processamento simulada (`finalizeCheckout` / `completeUpgrade`) por uma chamada ao seu `POST /api/checkout` e redirecione/abra o pagamento retornado.
   - Toda a UI (preço, parcelas, cupom, métodos, planos) **já está pronta** para isso.

**Mapeamento UI → backend**
| Elemento na UI | Onde plugar |
|---|---|
| Preço/parcelamento (Admin) | `GET/PUT /api/billing` |
| Cupom aplicado | validar em `POST /api/checkout` |
| Botão "Pagar e ativar Premium" | `POST /api/checkout` |
| Ativação do Premium | `POST /api/webhooks/pagseguro` (status PAID) |

---

## 3. Login com Google (OAuth real)

**Como funciona na v1 (simulado)**
- "Continuar com Google" abre um **seletor de contas local** e cria/loga um tenant a partir do e‑mail. Nenhuma autenticação real ocorre.

**Como ligar de verdade**
1. Crie um **OAuth Client ID** no Google Cloud Console (tipo *Web*), com as **Authorized JavaScript origins** e **redirect URIs** do seu domínio.
2. Use o **Google Identity Services** no front para obter o **ID token**.
3. No backend, **verifique o token** (assinatura/aud/iss) e crie/recupere o usuário; emita sua própria sessão/JWT.
4. Substitua `loginGoogleIdentity()` para chamar o seu endpoint de login Google.

---

## 4. Migração de armazenamento (localStorage → banco)

Hoje os dados vivem em `localStorage` por navegador. Para produção:
- Modele as entidades: **contas/usuários**, **empresas (tenants)**, **projetos**, **canvas**, **tarefas (Kanban)**, **cupons**, **billing**, **treinamentos**.
- Exponha uma API REST/GraphQL e troque os helpers (`getWorkspace/saveWorkspace`, `getAccounts/saveAccounts`, etc.) por chamadas ao backend.
- Armazene **anexos** (tarefas) e **PDFs** em storage de objetos (S3/Blob), guardando apenas a URL.

---

> Fonte do método: CAIO — Chief Artificial Intelligence Officer: Manual de Governança em IA (Dimitri de Melo).
