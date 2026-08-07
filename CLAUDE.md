# JARVIS (Mark-L) — contexto do projeto

Fork de [FatihMakes/Mark-L](https://github.com/FatihMakes/Mark-L), assistente de
voz para desktop. Licença CC BY-NC 4.0: uso pessoal, não comercial, mantendo o
crédito ao autor original.

O dono é o Murilo. **Fale português com ele.** Ele está aprendendo a programar —
explique o que está fazendo e por quê, sem assumir jargão. Um comando por vez:
ele roda no terminal do Windows, e comandos colados juntos já grudaram numa
linha só várias vezes.

## Como rodar

```bash
python main.py          # abre o assistente (precisa de microfone)
```

Instalação em `C:\Users\Pichau\Mark-L-novo`. Python 3.12, dependências já
instaladas. A chave do Gemini está em `config/api_keys.json`, que é ignorado
pelo git — **este repositório é público, nunca commite esse arquivo.**

## Arquitetura

O cérebro é a **Gemini Live API** (`main.py`, `JarvisLive.run`, ~linha 1400):
áudio bidirecional nativo, onde voz entra e sai pelo mesmo stream do Google.
Não há STT/LLM/TTS separados nesse caminho.

- `main.py` (1550 linhas) — laço principal, sessão Live, despacho de 23 ferramentas
- `ui.py` (3350 linhas) — HUD em PyQt6, desenhado à mão com QPainter
- `actions/` — 23 módulos: abrir apps, controlar sistema, buscar, arquivos, visão
- `memory/long_term.json` — memória persistente (ignorado pelo git, tem dados pessoais)
- `core/prompt.txt` — personalidade, **em português**

### Código órfão

`core/llm_client.py` (cliente Ollama completo), `core/stt.py` (Whisper) e
`core/tts.py` (Kokoro/EdgeTTS) existem e funcionam, mas **nenhum arquivo os
importa** — são restos da arquitetura local do Mark XL. Religá-los ao `main.py`
é o caminho para rodar offline, e exige reescrever o laço de áudio.

## O que foi modificado neste fork

| Arquivo | Mudança |
|---|---|
| `core/prompt.txt` | traduzido para português |
| `actions/gemini_model.py` | **novo** — descobre o modelo Gemini em runtime |
| `claude_backend.py` | **novo** — roteia o agente de código para o Claude Code |
| `jarvis_mcp_server.py` | **novo** — expõe 8 ações do JARVIS como ferramentas MCP |
| `jarvis/` | instalador, diagnóstico, temas, teste da ponte |
| `main.py` | filtros de warning; usa o resolvedor de modelo |
| `ui.py` | lê `ui_accent`/`ui_accent2` do config (realces de tema) |
| `actions/*.py` | 14 nomes de modelo cravados → resolvedor |

### Por que o resolvedor de modelo existe

`gemini-2.5-flash` estava cravado em 14 lugares e saiu do ar para chaves novas
(404). Cravar um sucessor só adiaria o problema — essa família já foi renomeada
várias vezes. `actions/gemini_model.py` pergunta à API o que a chave enxerga e
escolhe o flash mais novo. Nesta máquina resolve para `gemini-3.6-flash`.

**Cuidado ao mexer:** `models.list()` devolve um pager preguiçoso. O cliente
precisa ficar numa variável durante toda a iteração, senão é coletado no meio e
dá "Cannot send a request, as the client has been closed".

### Por que o claude_backend faz fallback

`dev_agent` e `code_helper` rodavam em `gemini-2.5-flash`, ruim para código. Vão
para o `claude -p` quando ele existe. Se não existir, **caem de volta no
Gemini** — trocar um motor por um ausente deixaria o assistente pior do que
estava. `encontrar_claude()` procura no PATH e também nos binários embutidos na
extensão do VS Code e no Claude Desktop.

## Convenções

- **Quebras de linha CRLF.** Os fontes são CRLF; ler e regravar pelo modo texto
  padrão do Python converte tudo para LF e transforma um patch de 10 linhas num
  diff de 1168. Use `newline=""` na leitura e na escrita.
- **Comentários explicam o porquê**, não o quê. Em português, como o resto.
- **Nada destrutivo no MCP.** Apagar arquivo, desligar e reiniciar existem em
  `actions/` e continuam disponíveis por voz, mas não são expostos como
  ferramenta automática. Só exponha com autorização explícita do Murilo.
- Antes de mexer em `ui.py`, **abra o assistente e olhe**. É layout desenhado à
  mão; mudanças não se verificam por leitura de código.

## Ferramentas de apoio

```bash
python jarvis/diagnostico.py      # SO, RAM, GPU, modelos do Ollama, pacotes
python jarvis/testar_claude.py    # a ponte com o Claude Code funciona?
python jarvis/modelos.py          # que modelos Gemini esta chave pode usar
python jarvis/temas.py            # temas, com amostra de cor no terminal
python jarvis/instalar.py --desfazer   # reverte o kit inteiro
```

## O que o Murilo quer a seguir

1. **Abas laterais no HUD**, no espírito de um painel de chat, para agrupar
   funções úteis. Requer trabalhar em `ui.py` com o app aberto na frente.
2. **Integração com Instagram** para publicar posts. O caminho curto é o JARVIS
   delegar ao Claude Code, não falar com a API da Meta diretamente.
3. **Ollama como cérebro**, para rodar offline e sem depender do Gemini. É o
   maior dos três: exige reescrever o laço de áudio do `main.py`.
