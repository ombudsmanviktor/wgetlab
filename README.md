# WgetLAB

Ferramenta web para **baixar em lote imagens, áudios e vídeos listados num CSV**, direto para uma pasta do seu computador. Você arrasta a planilha, indica a coluna com as URLs e o app faz o resto. Não há instalação nem servidor, e seus dados não saem do seu computador.

Inspirado no fluxo `wget` em R muito usado em pesquisas com dados de plataformas digitais. O app também **gera o script R equivalente**, para reproduzir a mesma operação no RStudio.

**[▶ Abrir o app](https://wgetlab.colab.meme/)**

---

## O que faz

- Carrega um CSV via drag-and-drop e lê o arquivo no navegador, com PapaParse
- Detecta automaticamente as colunas que contêm URLs e já sugere a mais provável
- Aceita **uma ou várias colunas** de URL ao mesmo tempo, por exemplo `music.url` e `music.cover`
- Baixa **todas as linhas** ou uma **amostra aleatória de X linhas**, com semente reproduzível
- Deixa você escolher ou criar a pasta de destino **numa janela do próprio sistema**, sem digitar caminhos
- Nomeia os arquivos por uma coluna de ID (`id`, `data.id`…) ou pelo número da linha
- Detecta a extensão pela URL (`.jpg`, `?format=png`, `mime_type=video_mp4`) ou pelos bytes do arquivo (JPEG, PNG, GIF, WebP, MP4, M4A, MP3, WebM…)
- Grava em streaming: vídeos grandes vão direto para o disco, sem ocupar a memória
- Mostra o progresso e permite pausar e cancelar. Pode **pular arquivos já existentes**, o que permite retomar um download interrompido
- Ao final, grava um manifesto `wgetlab_manifest_….csv` (linha, id, coluna, URL, arquivo, status, erro) e oferece a lista de falhas para exportar
- Gera um **script R** (tidyverse + `wget`) com os mesmos parâmetros, que baixa as mesmas linhas com os mesmos nomes de arquivo

## Formatos de célula reconhecidos

| Formato | Exemplo | Origem típica |
|---|---|---|
| Uma URL por célula | `https://p16-sg.tiktokcdn.com/…/capa.jpeg` | TikTok (`music.url`, `music.cover`) |
| Várias URLs separadas por `\|`, `;`, vírgula ou espaço | `https://pbs.twimg.com/media/A?format=jpg \| https://video.twimg.com/…/v.mp4` | TweetLAB (`midias`) |
| JSON com URLs dentro | `[{"type":"video","url":"https://lookaside.facebook.com/…"}]` | Meta Content Library (`multimedia`) |

Células vazias, `NA` e `NULL` são ignoradas. Quando uma célula tem várias URLs, os arquivos recebem os sufixos `_1`, `_2`…

---

## Navegadores

| Navegador | Modo |
|---|---|
| **Chrome, Edge, Opera, Brave** (desktop) | Grava direto na pasta escolhida (File System Access API) |
| Safari, Firefox, celulares | **Modo ZIP**: os arquivos são reunidos num `.zip` salvo em Downloads. Tudo fica na memória até o fim, então evite milhares de vídeos de uma vez |

## Limitações importantes

- **CORS.** O navegador só consegue baixar arquivos de servidores que permitem esse acesso. As CDNs do X/Twitter (`pbs.twimg.com`, `video.twimg.com`) e do TikTok (`*.tiktokcdn.com`) permitem. Servidores que não permitem aparecem como "bloqueado pelo servidor (CORS)". Nesse caso, use o script R gerado.
- **Meta Content Library.** Os links `lookaside.facebook.com/mcl/…` só funcionam com login no ambiente da Meta Content Library. Fora dele, o servidor responde "não autorizado" (401), aqui e no `wget`.
- **Links com prazo de validade.** URLs assinadas (`x-expires`, `expire`, `x-signature`…) expiram algumas horas ou dias após a coleta. Se a base for antiga, espere erros 403 ou 404.
- **Páginas não são mídia.** Links para páginas (como `tiktok.com/player/…` ou `x.com/…/status/…`) não são arquivos, e o app os marca como falha.

---

## Script R gerado

O script segue o fluxo clássico:

```r
download_folder <- "~/Downloads/minha_pasta/"
dir.create(download_folder, recursive = TRUE, showWarnings = FALSE)
for (i in seq_len(nrow(tarefas))) {
  system(paste("wget -q", shQuote(url), "-O", shQuote(tmp)))
  ...
}
```

Ele lê o CSV com todas as colunas como texto, para não perder dígitos de IDs longos, e extrai as URLs com as mesmas regras do app. Se você usou amostra, o script inclui o **vetor exato das linhas sorteadas**, para reproduzir a mesma amostra. Ele detecta a extensão, pula arquivos já existentes e grava um manifesto. Precisa do pacote `tidyverse` e do `wget` instalado (`brew install wget` no macOS).

---

## Hospedar no GitHub Pages (fork em 2 minutos)

1. **Faça um fork** deste repositório
2. Vá em **Settings → Pages**
3. Em **Source**, selecione `Deploy from a branch`, branch `main`, pasta `/ (root)`
4. Clique em **Save** e aguarde cerca de 1 minuto. O app estará em `https://SEU-USUARIO.github.io/NOME-DO-REPO/`

O app é um único arquivo `index.html`: não há build nem dependências para instalar.

## Usar localmente

```bash
python3 -m http.server 8080
# Abra http://localhost:8080 no Chrome ou Edge
```

> A escolha de pasta só funciona em `https://` ou `http://localhost`. Abrir o arquivo com `file://` não funciona.

---

## Privacidade

O CSV é lido apenas no seu navegador. O app só se comunica com os servidores das próprias URLs listadas, para baixar os arquivos. Não há servidor intermediário, telemetria nem cookies de rastreamento.
