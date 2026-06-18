# CAIO — AI Governance Platform

Plataforma de **Governança de Inteligência Artificial** baseada no método **CAIO (Chief Artificial Intelligence Officer)**, de Dimitri de Melo. Conecta Tecnologia, Negócios e Governança através do **AI Bridge Canvas**, gestão ágil de projetos de IA (Kanban), diagnóstico de maturidade, compliance e trilhas de risco.

> Versão **1.0** — aplicação web autocontida (single‑page) pronta para publicação como site estático.

---

## Sumário
- [Visão geral](#visão-geral)
- [Funcionalidades (v1)](#funcionalidades-v1)
- [Primeiro acesso](#primeiro-acesso)
- [Rodar localmente](#rodar-localmente)
- [Publicar no GitHub](#publicar-no-github)
- [Deploy no Render (Static Site)](#deploy-no-render-static-site)
- [Deploy no GitHub Pages (alternativa)](#deploy-no-github-pages-alternativa)
- [Arquitetura e armazenamento](#arquitetura-e-armazenamento)
- [Integrações](#integrações)
- [Limitações e próximos passos](#limitações-e-próximos-passos)
- [Créditos e licença](#créditos-e-licença)

---

## Visão geral

O CAIO é uma **single‑page application** escrita em HTML, CSS e JavaScript puro (sem framework, sem build). Todo o estado é mantido no navegador via `localStorage`, o que torna o protótipo ideal para demonstração, validação de produto e como blueprint para a versão com backend.

- **Multi‑empresa (multi‑tenant) simulado:** cada conta vê apenas seus próprios dados.
- **Planos Freemium:** Grátis e Premium, com checkout (cartão / PIX / boleto) e cupons.
- **Tema enterprise:** layout claro estilo "console", barra lateral de navegação, cor central azul caneta Bic, fonte Inter e ícones SVG profissionais.

## Funcionalidades (v1)

- **Login multi‑tenant** com usuário/senha e "Entrar com Google" (simulado).
- **Home** estilo console: deck de apps central, comunicação de projetos (com anexo de imagem), mini‑dashboard e avisos.
- **Projetos de IA**: cada projeto possui Canvas próprio, Kanban (Backlog → A Fazer → Em Andamento → Concluído) com **drag‑and‑drop**, edição de cards (descrição, responsável, prazo, anexos) e etapas do AI Lab (Canvas → Piloto → Escala → Operação).
- **AI Bridge Canvas** (console de canvas + editor): ponte de Tecnologia/Negócios/Governança com 9 pilares, perguntas‑chave, **respostas em post‑its (estilo Miro) coloridos por grupo**, identificação do projeto, **geração de PDF** e **vínculo ao Kanban** (card "AI Bridge Canvas preenchido").
- **Diagnóstico de Maturidade** com radar SVG (offline), sumário executivo, padrões de risco e **relatório executivo**.
- **Trilhas de Risco**, **Compliance** (LGPD / EU AI Act), **Treinamento** (vídeos do YouTube) e **rodapé** com a citação da fonte/autor.
- **Painel de Administração**: gestão de usuários e indicadores de uso, **cupons de desconto** e **configurações de pagamento** (preço, parcelamento, conexão PagSeguro).

## Primeiro acesso

A plataforma é entregue **sem dados de demonstração**. No primeiro uso:

1. Abra a aplicação e clique em **Criar conta**.
2. O **primeiro usuário cadastrado torna‑se o administrador** (com plano Premium), podendo gerenciar usuários, cupons e pagamentos.
3. Os cadastros seguintes entram como usuários comuns no plano Grátis.

> Os dados ficam no navegador (`localStorage`). Limpar os dados do site reinicia a plataforma.

## Rodar localmente

Não há etapa de build. Basta abrir o arquivo:

```bash
# opção 1: abrir direto
# (dê duplo clique em index.html)

# opção 2: servidor local (recomendado p/ testar geração de PDF e fontes)
python3 -m http.server 8080
# acesse http://localhost:8080
```

## Publicar no GitHub

```bash
# dentro da pasta do projeto (que contém index.html, README.md, render.yaml)
git init
git add .
git commit -m "CAIO — AI Governance Platform v1.0"
git branch -M main
git remote add origin https://github.com/<SEU_USUARIO>/<SEU_REPO>.git
git push -u origin main
```

## Deploy no Render (Static Site)

O CAIO é um site estático — o deploy no Render é simples e gratuito.

**Opção A — pelo painel do Render (recomendado):**
1. Crie a conta em <https://render.com> e conecte o GitHub.
2. **New → Static Site** e selecione o repositório.
3. Configure:
   - **Build Command:** *(deixe em branco)*
   - **Publish Directory:** `.` (raiz, onde está o `index.html`)
4. **Create Static Site**. Em poucos segundos a URL pública estará no ar (HTTPS automático).

**Opção B — via Blueprint (`render.yaml`):** este repositório já inclui um `render.yaml`. Em **New → Blueprint**, aponte para o repositório e o Render criará o serviço automaticamente.

> Cada `git push` na branch `main` dispara um novo deploy automático.

## Deploy no GitHub Pages (alternativa)

1. No repositório, vá em **Settings → Pages**.
2. Em **Source**, selecione a branch `main` e a pasta `/ (root)`.
3. Salve. A URL `https://<seu_usuario>.github.io/<seu_repo>/` ficará disponível.

## Arquitetura e armazenamento

- **Stack:** HTML + CSS + JavaScript (vanilla), sem dependências de build.
- **Dependências externas (CDN):** Fonte **Inter** (Google Fonts) e **jsPDF** (cdnjs) para gerar PDF. Funcionam quando há internet; sem conexão, a fonte cai para a do sistema e a geração de PDF exibe aviso.
- **Persistência:** `localStorage` do navegador, com chaves namespaceadas por usuário (`aiGov_<usuario>_workspace`, `aiGov_<usuario>_canvasState`, `aiGov_accounts`, `aiGov_coupons`, `aiGov_billing_config`, `aiGov_trainings`).
- **Isolamento:** cada conta acessa somente seus próprios projetos, canvases e comunicados.

Consulte **[docs/INTEGRACOES.md](docs/INTEGRACOES.md)** para os detalhes técnicos de pagamento, PDF e login Google.

## Integrações

| Recurso | Estado na v1 | Para produção |
|---|---|---|
| **Gerador de PDF** | Funcional via **jsPDF** (CDN). Gera e baixa o PDF do Canvas direto. | Opcional: hospedar a lib localmente para uso offline. |
| **Pagamentos (PagSeguro)** | Checkout e credenciais **simulados** (armazenados localmente). | Exige **backend** que chame a API/Checkout do PagSeguro + **webhook** de confirmação. |
| **Login com Google** | **Simulado** (seletor de contas local). | Substituir por **Google Identity Services / OAuth** com verificação de token no backend. |
| **Banco de dados** | `localStorage` (por navegador). | Backend + banco (Postgres) para multiusuário real por empresa. |

Passo a passo de cada integração em **[docs/INTEGRACOES.md](docs/INTEGRACOES.md)**.

## Limitações e próximos passos

A v1 é um **protótipo client‑side**: os dados vivem no navegador de cada usuário e o pagamento/Google são simulados. O caminho para produção:

1. **Backend** (Node/Express, etc.) + **banco de dados** para persistência real e multiusuário por empresa.
2. **Autenticação real** (JWT/sessão) e **Google OAuth**.
3. **Gateway de pagamento** real (PagSeguro/PagBank) com webhook ativando o plano Premium.
4. **Armazenamento de arquivos** (S3/Blob) para anexos de tarefas e PDFs.

Toda a UI (preços, parcelas, cupons, métodos, planos) já está pronta para plugar esses serviços trocando apenas a camada de processamento.

## Créditos e licença

- **Método e conteúdo:** *CAIO — Chief Artificial Intelligence Officer: Manual de Governança em IA*, de **Dimitri de Melo**.
- **AI Bridge Canvas** e os 9 pilares de governança baseiam‑se na obra citada.
- Uso interno/corporativo. Defina a licença adequada antes da publicação pública (ex.: proprietária ou MIT).

---

> Fonte: CAIO — Chief Artificial Intelligence Officer: Manual de Governança em IA (Dimitri de Melo).
