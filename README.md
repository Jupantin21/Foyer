# Foyer · Agenda Cultural

Calendário cultural compartilhado para grupos. Permite que qualquer pessoa com o link adicione, edite e visualize eventos colaborativamente.

**Tecnologia:** HTML/CSS/JS puro · GitHub Contents API como banco de dados · GitHub Pages com GitHub Actions para publicação segura.

**Custo recorrente:** zero.

---

## Estrutura

```
foyer/
├── index.html                          ← o app (com placeholders de token)
├── data/
│   └── events.json                     ← banco de dados de eventos
├── .github/
│   └── workflows/
│       └── deploy.yml                  ← injeta secrets e publica
└── README.md                           ← este arquivo
```

O app inteiro é um único arquivo HTML que conversa com a GitHub Contents API para ler e gravar eventos no `data/events.json`. Cada salvamento vira um commit Git automático.

**Importante:** o `index.html` no repositório tem placeholders (`__GH_TOKEN__`, `__GH_OWNER__`) — o token real **nunca** aparece no código-fonte. Quando você dá push, o GitHub Actions roda automaticamente, substitui os placeholders pelos valores reais (lidos de Secrets), e publica o site no GitHub Pages.

---

## Publicação — passo a passo

### 1 · Criar o repositório

Em [github.com/new](https://github.com/new):

* **Name:** `foyer`
* **Visibility:** Public
* Marcar **"Add a README file"**
* Clicar **Create repository**

### 2 · Subir os arquivos

No repositório recém-criado, clicar em **"Add file" → "Upload files"** e arrastar:
* `index.html`
* `data/events.json` (digite `data/` no campo de caminho antes de arrastar)
* `.github/workflows/deploy.yml` (digite `.github/workflows/` antes de arrastar)
* `README.md`

Commit changes.

### 3 · Gerar o token de acesso

Em [github.com/settings/personal-access-tokens/new](https://github.com/settings/personal-access-tokens/new):

* **Token name:** `foyer-calendar-write`
* **Expiration:** 90 dias (renovar quando expirar)
* **Repository access:** Only select repositories → escolher `foyer`
* **Permissions** → **Repository permissions** → **Contents:** Read and write
* (Manter todos os outros em "No access")

Clicar **Generate token** e **copiar o valor que começa com `github_pat_`**.

### 4 · Cadastrar o token como Secret

No repositório `foyer`:

* **Settings** (na barra superior) → **Secrets and variables** → **Actions**
* Clicar **New repository secret**
* **Name:** `GH_TOKEN`
* **Secret:** colar o valor `github_pat_...` do passo 3
* Clicar **Add secret**

### 5 · Ativar GitHub Pages com Actions

* **Settings** → **Pages**
* **Source:** **GitHub Actions** (não "Deploy from a branch"!)
* Salvar (clicar em "Save" se aparecer)

### 6 · Disparar o primeiro deploy

O workflow precisa rodar pelo menos uma vez para publicar. Há duas formas de disparar:

**A — Editar qualquer arquivo:** clicar em qualquer arquivo (ex: README.md), o ícone do lápis, fazer uma alteração mínima (adicionar um espaço), e commitar. O push dispara o workflow.

**B — Disparo manual:** ir em **Actions** (na barra superior) → **Deploy Foyer Calendar** → **Run workflow** → confirmar.

### 7 · Aguardar deploy

Em **Actions**, você verá o workflow rodando (bolinha amarela girando). Após 1-2 minutos, fica verde com ✓.

O calendário estará no ar em:

```
https://SEU_USUARIO.github.io/foyer/
```

Esse é o link que você manda para os amigos.

---

## Como funciona o banco de dados

Cada vez que alguém cria, edita ou apaga um evento, o `index.html` chama a API do GitHub para reescrever o `data/events.json`. Cada operação vira um commit Git automático com mensagem descritiva:

```
add: Hamlet — Theatro Municipal (Julia)
update: Concerto OSESP — Mahler 5 (Pedro)
delete: evento abc123
```

Todo o histórico fica registrado e visível em **Commits**. Para reverter alguma mudança, basta abrir o commit e clicar em **Revert**.

### Concorrência

Se duas pessoas tentam salvar ao mesmo tempo, o app detecta o conflito de SHA, automaticamente busca a versão mais recente, mescla a alteração local em cima dela, e salva de novo. Tudo transparente para o usuário — limite de 3 tentativas.

### Polling

A cada 15 segundos, o app puxa a versão mais recente do `events.json` automaticamente. Eventos criados em outro dispositivo aparecem para todo mundo em até 15 segundos.

---

## Sobre a segurança do token

O `index.html` no repositório **não contém o token real** — só placeholders. O token real fica em GitHub Secrets, que é um mecanismo criptografado e nunca aparece em logs nem em código.

O workflow `deploy.yml` injeta o token apenas no momento do build, e o HTML publicado no GitHub Pages tem o token funcional. Quem inspecionar a página via DevTools (View Source) consegue ver o token — mas o código-fonte do repositório permanece limpo.

**Mitigantes do risco do token visível na página servida:**

1. **Escopo mínimo:** o token só consegue mexer em arquivos dentro deste repo, com Contents:Read+Write. Não pode criar repos, apagar repos, ler emails, ou mexer em outros projetos.
2. **Histórico Git:** qualquer alteração ruim pode ser revertida.
3. **Rotação fácil:** se desconfiar de uso indevido, gere um token novo (passo 3), atualize o Secret `GH_TOKEN` (passo 4), e o token antigo deixa de funcionar.

### Quando renovar o token

Tokens fine-grained com 90 dias expiram. Quando isso acontecer, o calendário mostrará "Erro de conexão". Para renovar:

1. Repetir o passo 3 (gerar novo token)
2. Repetir o passo 4 (atualizar o Secret `GH_TOKEN`) — pode editar o secret existente em **Settings → Secrets and variables → Actions** clicando no nome `GH_TOKEN`
3. Disparar o deploy: **Actions → Deploy Foyer Calendar → Run workflow**

Leva 3 minutos no total.

---

## Funcionalidades

### Calendário

* Visualização em grade (desktop) ou lista (mobile)
* Navegação 6 meses adiante a partir do mês atual
* Eventos com horário, valor do ingresso, link de compra, autor
* Filtros por categoria: Cinema, Teatro, Dança, Música, Outro
* Hoje destacado em laranja
* Memória do nome do autor

### Save the Date (Wishlist)

Para eventos sem data definida — ideias que o grupo está considerando. Quando o grupo decide a data real, edita o item e muda o status para "Confirmado": ele migra para o calendário automaticamente.

### Sincronização

* Status visível: bolinha verde "Sincronizado" / laranja pulsando "Salvando…" / vermelha "Erro"
* Polling a cada 15s
* Re-sync ao voltar para a aba

---

## Identidade visual

Símbolo: arco côncavo branco sobre quadrado laranja `#CC5500`. Para a especificação canônica do logo (SVG, paleta exata, variantes), ver `FOYER-LOGO.md` no projeto principal.

Paleta:
* Primário `#CC5500` · Laranja `#FF8C00` · Âmbar `#FFB347` · Creme `#FFF0DC`
* Tipos: Cinema `#5B8DEF` · Teatro `#CC5500` · Dança `#C44D8A` · Música `#6B9D6F` · Outro `#8C7B6B`
