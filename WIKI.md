# Clicknator XP — Wiki do Projeto

## 1) Visão geral
Clicknator XP é um sistema operacional fictício em HTML/CSS/JavaScript com interface estilo Windows XP.

Principais capacidades:
- Desktop com ícones de apps e arquivos.
- Janelas arrastáveis/redimensionáveis.
- Barra de tarefas, menu iniciar e menu de contexto.
- Sistema de arquivos virtual (VFS) via `localStorage`.
- Conexões entre apps (entrada/saída/storage) com roteamento de recursos.
- Apps internos (navegador, paint com camadas, compilador, storage, processadora, conversor, etc.).

---

## 2) Como executar
Não há build/lint/test configurados no repositório (site estático).

Formas de rodar:
- Abrir `index.html` no navegador; ou
- Servir via host estático (GitHub Pages, Nginx, etc.).

Domínio configurado:
- `CNAME`: `clicknator.com.br`

---

## 3) Arquitetura

### 3.1 Camadas
- **UI base**: `index.html` + `style.css`
- **Runtime principal**: `script.js`
- **Catálogo de apps/sites**: `apps.json` e `sites.json`
- **Apps/sites em iframes**: `apps/*.html`, `sites/*.html`
- **Persistência local**: `localStorage`

### 3.2 Persistência (localStorage)
Chaves principais em `script.js`:
- `winxp-state` (`SAVE_KEY`): estado geral (ex.: wallpaper, session_time)
- `winxp-unlocks` (`UNLOCK_KEY`): desbloqueios de apps
- `winxp-files` (`FILE_KEY`): arquivos do VFS
- `winxp-upgrades` (`UPGRADES_KEY`): upgrades de RAM/armazenamento

### 3.3 Recursos de sistema
`window.systemInfo`:
- `username`
- `ram` (base 256 MB + upgrades)
- `space` (base 2048 MB + upgrades)
- `session_time`

### 3.4 APIs globais expostas para apps/iframes
Em `script.js`:
- `window.getFiles()`, `window.saveFiles()`
- `window.getUpgrades()`, `window.saveUpgrades()`
- `window.addFile(file)`, `window.deleteFile(id)`
- `window.renderDesktopFiles()`
- `window.openImageViewer(file)`
- `window.createShortcut(name, url)`
- `window.applyUpgrades()`, `window.unlockApp(appId)`
- `window.getUsedRAM()`, `window.getRAMInfo()`
- `window.emitResource(appId, resource)`
- `window.pescar(appId)`
- API para automação/compilador:
  - `window.getOpenWindows()`
  - `window.OpenWindow(id)`
  - `window.closeWindowById(id)`

### 3.5 Eventos de mensagem (`postMessage`)
Mensagens tratadas no runtime:
- `download-app`
- `open-app`
- `delete-file`
- `create-shortcut`
- `emit-from-window`
- `emit-resource`
- `navigate`

### 3.6 Modelo de conexão entre apps
Apps podem declarar portas em `apps.json`:
- `out` (saída), `in` (entrada), `storage`
- com `format` (ex.: `fish`, `dinheiro`, `any`)

O runtime:
- valida compatibilidade de direção e formato,
- desenha cabos no canvas `#cables`,
- roteia recursos entre janelas,
- faz auto-feed de storage em intervalos quando aplicável.

---

## 4) Catálogo de apps (`apps.json`)
Campos comuns:
- `id`, `nome`, `icone`, `html`
- opcionais: `ram`, `instantiable`, `unlocked`, `ports`, `appType`

Apps com arquivo presente:
- `computer` → `apps/computer.html`
- `notepad` → `apps/notas.html`
- `gonetgo` → `apps/browser.html`
- `paint` → `apps/paint.html`
- `credits` → URL externa (GitHub)
- `pescaria` → `apps/pescaria.html`
- `processadora` → `apps/processadora.html`
- `loja` → `apps/loja.html` (atenção ao case do nome do arquivo no repo: `Loja.html`)
- `conversor` → `apps/conversor.html`
- `inventario` → `apps/storage.html`
- `compiler` → `apps/compiler.html`
- `music` → `apps/music.html`

Referências atualmente sem arquivo local correspondente:
- `dungeonclash` → `apps/dungeon.html` (não existe)
- `mail` → `apps/mail.html` (não existe)
- `clicknator`/`money` → `apps/clicknator.html` (não existe)

---

## 5) Catálogo de sites (`sites.json`)
Mapeia domínios `.gogo` do navegador interno para páginas locais.

Sites com arquivo presente:
- `gogoshop` → `sites/GoGoShop.html`
- `bloggy` → `sites/Bloggy.html`
- `godownloader` → `sites/GoDownloader.html`
- `MSN` → `sites/Msn.html`
- `gocode` → `sites/GoCode.html`

Referência sem arquivo local correspondente:
- `googlar` → `../sites/googlar.html` (não existe)

---

## 6) Documentação por pasta e arquivo

## Raiz

### `index.html`
Shell principal do sistema:
- desktop (`#desktop`), taskbar, menu iniciar.
- menu de contexto com ações de abrir/renomear/excluir/fixar/desbloquear.
- canvas de cabos (`#cables`).
- carrega `script.js`.

### `style.css`
Tema visual XP-like:
- desktop, ícones, janelas, title-bar.
- barra de tarefas, start menu, quick launch.
- estilos de conexão e barras de status (RAM/disco).

### `script.js`
Cérebro do sistema.

Blocos principais:
1. **Estado/Persistência** (localStorage + `systemInfo`).
2. **Layout/UX base**:
   - posicionamento de ícones em grade,
   - relógio,
   - drag & drop,
   - resize de janelas,
   - seleção múltipla com rubber-band.
3. **Sistema de conexões**:
   - definição/validação de portas,
   - `tryConnect`, `drawConnections`, `routeResourceToWindow`, `emitResource`.
4. **VFS (arquivos)**:
   - `addFile`, `deleteFile`, mover entre desktop/pastas,
   - render de ícones de arquivo,
   - handlers por tipo (`text`, `folder`, `image`, `shortcut`, `fish`, `dinheiro`).
5. **Janelas/apps**:
   - criação de janela, taskbar item, consumo de RAM,
   - abertura por id (`openAppById`) e fechamento (`closeWindow`).
6. **Features utilitárias**:
   - visualizador de imagem/peixe/dinheiro,
   - bloco de notas simples,
   - pastas com drag/drop e refresh,
   - downloads de app (`baixar`).
7. **UI global**:
   - contexto, start menu, wallpaper, shutdown.
8. **Integração por `postMessage`** entre iframes.
9. **Bootstrap**: `applySavedState()`, `applyUpgrades()`, `initSessionTime()`, `loadApps()`.

### `apps.json`
Registro de apps instaláveis/desbloqueáveis e metadados de portas/RAM.

### `sites.json`
Registro de roteamento de domínios `.gogo` para páginas locais.

### `README.md`
Resumo curto do projeto (agora com link para esta wiki).

### `CNAME`
Domínio customizado para GitHub Pages.

### `ads.txt`
Configuração de autorização AdSense.

### `txt.txt`
Arquivo texto grande com conteúdo de desenvolvimento/rascunho relacionado ao runtime (inclui cópia/variante de trechos de `script.js` e nota inicial de bug).

### `Bg.jpg`
Asset de fundo do sistema.

### `download.jpg`
Imagem usada em páginas de conteúdo (`apps/creditos.html` e `apps/notas.html`).

---

## Pasta `apps/`

### `apps/browser.html` (GoNetGo)
Navegador interno com:
- múltiplas abas,
- input de URL/domínio `.gogo`,
- resolução via `sites.json`,
- iframes por aba,
- favorito rápido para blog.

### `apps/compiler.html` (GOGO Compiler)
Editor + console + interpretador custom.

Recursos:
- syntax highlight,
- numeração de linhas,
- auto-indent,
- execução (`Ctrl+Enter`),
- salvar/importar código.

Linguagem interpretada:
- variáveis e expressões,
- `para ... de ... ate ...:`,
- `enquanto ...:`,
- `se ...:`,
- funções `f nome(args):` + `retorna`.

Builtins com integração ao SO:
- `mostre(...)`
- `abrir(...)`
- `fechar(...)`
- `listar_janelas()`
- `fechar_todas()`
- `contar(tipo)`

### `apps/computer.html`
Painel “Meu Computador”:
- usuário,
- tempo de sessão,
- consumo RAM/disco com barras estilo XP,
- atualização periódica via `window.parent`.

### `apps/conversor.html`
Conversor de arquivos em dinheiro:
- filtra arquivos do VFS por tipo,
- calcula total por taxa,
- seleção por quantidade,
- emite recurso `dinheiro` por conexão ou salva no VFS,
- opção de remover arquivos fonte após conversão,
- painel de log interno.

### `apps/creditos.html`
Conteúdo textual estilo “diário”/história, com imagem `download.jpg`.

### `apps/Loja.html`
Arquivo atualmente praticamente vazio (apenas espaços/linhas).

### `apps/music.html` (TYR4N)
DAW/sequenciador musical no browser.

Recursos principais:
- transport (play/stop/rec/loop/BPM/volume),
- channel rack (instrumento, volume, pitch, mute/solo),
- piano roll com draw/select/erase,
- snap de grade, velocity, seleção e edição de notas,
- preview por teclado,
- gravação live de notas,
- export/import MIDI, export WAV, gravação de áudio.

### `apps/css/music.css`
Estilos completos do TYR4N (tema dark, transport, channel rack, piano roll, menus/context).

### `apps/notas.html`
Conteúdo textual semelhante ao de créditos (mensagem narrativa + imagem `download.jpg`).

### `apps/paint.html` (GoPaint)
Editor de imagem com camadas.

Funcionalidades:
- ferramentas: lápis, pincel, borracha, balde, conta-gotas, linha, retângulo, elipse,
- presets de tamanho e memória de tamanho por ferramenta (`localStorage`),
- importação de imagem como nova camada,
- zoom com `Ctrl+wheel`,
- painel de camadas (visibilidade, opacidade, rename, delete, reorder por drag),
- undo por camada,
- fill e eyedropper,
- redimensionamento de canvas,
- salvar no VFS como `image` e export PNG.

### `apps/pescaria.html`
Mini app simples com botão `Pescar`, chamando `window.parent.pescar('pescaria')`.

### `apps/processadora.html`
Processadora de peixes:
- recebe recursos `fish` via `postMessage`,
- buffer de entrada (lotes),
- converte para `dinheiro` com bônus por raridade,
- cooldown entre ciclos,
- envio por `emit-from-window`,
- flush manual e logs/estatísticas.

### `apps/storage.html` (Inventário)
Explorador de armazenamento:
- lista arquivos do VFS em tabela,
- visual de uso de disco,
- estrutura de pastas (expand/collapse),
- miniatura para imagens,
- exibição de info de shortcuts,
- exclusão de arquivos,
- integração com visualizador de imagens do parent.

---

## Pasta `sites/`

### `sites/Bloggy.html`
Página estilo blog com changelog/atualizações do projeto.

### `sites/GoCode.html`
Página simples de documentação de sintaxe da linguagem do compilador.

### `sites/GoDownloader.html`
“Loja” de downloads com botões que enviam `postMessage` (`download-app`) para instalar apps no sistema.

### `sites/GoGoShop.html`
Página minimalista com botões estáticos de compra (sem lógica implementada no arquivo).

### `sites/Msn.html`
Arquivo minimalista com botão “Compre RAM/HD” (sem handlers implementados localmente).

---

## 7) Convenções de dados no VFS
Estrutura típica de arquivo:

- `id`: numérico (timestamp)
- `name`: nome exibido
- `type`: `text | folder | image | shortcut | fish | dinheiro | app | ...`
- `size`: tamanho lógico (MB)
- `data`: payload do tipo

Exemplos de `data`:
- `text`: `{ content }`
- `folder`: `{ children: [fileId...] }`
- `image`: `{ src: dataURL }`
- `shortcut`: `{ url }`
- `fish`: `{ raridade }`
- `dinheiro`: `{ valor, origem, qtd, taxa, ... }`
- `app`: `{ appId }`

---

## 8) Fluxos importantes

### Download de app
`sites/GoDownloader.html` → `postMessage(download-app)` → `script.js:baixar(id)` → cria `.exe` no VFS + `unlockApp`.

### Pescar e processar
`apps/pescaria.html` → `window.parent.pescar()` → recurso `fish` → conexão para `processadora` → saída `dinheiro`.

### Conversão manual
`apps/conversor.html` seleciona arquivos no VFS → converte para `dinheiro` → emite por conexão ou salva no VFS.

### Navegação `.gogo`
`apps/browser.html` resolve domínio em `sites.json` e carrega página local em iframe.

---

## 9) Lacunas e observações técnicas
- Alguns apps/sites em JSON não têm arquivo correspondente (listados acima).
- `apps/Loja.html` está vazio.
- `sites/Msn.html` e `sites/GoGoShop.html` são páginas mínimas sem lógica completa.
- `txt.txt` contém material de rascunho/diagnóstico; não parece usado em runtime.
- Não há pipeline de teste/lint/build no repositório atual.

---

## 10) Estrutura completa (referência rápida)

```text
/
├── index.html
├── style.css
├── script.js
├── apps.json
├── sites.json
├── README.md
├── WIKI.md
├── CNAME
├── ads.txt
├── txt.txt
├── Bg.jpg
├── download.jpg
├── apps/
│   ├── Loja.html
│   ├── browser.html
│   ├── compiler.html
│   ├── computer.html
│   ├── conversor.html
│   ├── creditos.html
│   ├── music.html
│   ├── notas.html
│   ├── paint.html
│   ├── pescaria.html
│   ├── processadora.html
│   ├── storage.html
│   └── css/
│       └── music.css
└── sites/
    ├── Bloggy.html
    ├── GoCode.html
    ├── GoDownloader.html
    ├── GoGoShop.html
    └── Msn.html
```
