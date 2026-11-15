# agente-que-aprende

[![agente-que-aprende](guia/assets/banner.jpg)](https://inematds.github.io/agente-que-aprende/guia/)

**ðŸ‡§ðŸ‡· [PortuguÃªs](README.md) Â· ðŸ‡ºðŸ‡¸ [English](README.en.md) Â· ðŸ‡ªðŸ‡¸ [EspaÃ±ol](README.es.md)**

## O que Ã©

O agente-que-aprende Ã© um kit de quatro scripts em Python para quem trabalha com agentes de IA de programaÃ§Ã£o, como o Claude Code e o Codex. Ele lÃª as suas sessÃµes e os arquivos onde vocÃª anota falhas e liÃ§Ãµes (FALHAS.md, LESSONS.md), mostra onde vocÃª corrigiu o agente e quais erros voltam, e ajuda a transformar cada liÃ§Ã£o numa trava automÃ¡tica em vez de mais uma regra escrita. Precisa sÃ³ de Python 3; nÃ£o usa internet nem API, e nenhuma regra Ã© adotada sem a sua decisÃ£o.

## ðŸ“– Guia de uso

Guia completo (landing + passo a passo): **https://inematds.github.io/agente-que-aprende/guia/**

ðŸŽ¬ VÃ­deo: https://www.youtube.com/watch?v=bnLvmYcP4PI

O seu agente de IA erra, vocÃª corrige no chat, e na sessÃ£o seguinte ele erra igual. Este kit fecha esse ciclo: acha onde
vocÃª corrigiu o agente, acha o erro que volta (no mesmo projeto ou em outro) e transforma a liÃ§Ã£o em **trava** â€” um
gancho que impede o comando, um validador que gera o que falta, um teste â€” em vez de mais uma linha de texto que o modelo
pode pular. Funciona com Claude Code e Codex, em Python puro, sem API e sem rede.


## A ideia em uma frase

O modelo Ã© o mesmo; o que melhora Ã© o sistema em volta dele. E a correÃ§Ã£o mais forte vem primeiro:
**arquitetura** (o erro fica impossÃ­vel) > **gancho/validador** (bloqueia) > **teste** (pega) > **texto** (pode ser pulado).

## O que tem

| Ferramenta | O que faz | Degrau |
|---|---|---|
| `retro.py` | A retrospectiva de 1 minuto: lÃª a sessÃ£o (`.jsonl`) e lista onde vocÃª corrigiu o agente e que erro de ferramenta se repetiu, com o degrau sugerido para cada um | memÃ³ria com portÃ£o |
| `degrau.py` | LÃª FALHAS.md / LESSONS.md de um projeto e mostra o que ainda Ã© sÃ³ texto **e se repete**, com o degrau para onde deve subir | erro impossÃ­vel |
| `jardineiro.py` | O mesmo em todos os projetos de uma vez, toda semana: taxa de reincidÃªncia, erros que "mudaram de casa", ranking | loop de fora |
| `gancho.py` + `ganchos.json` | Hook PreToolUse: cada liÃ§Ã£o vira uma linha de JSON (padrÃ£o â†’ mensagem) e o comando que jÃ¡ deu errado Ã© bloqueado com a explicaÃ§Ã£o | erro impossÃ­vel |
| `hooks/fim-de-sessao.sh` | Hook SessionEnd: roda a retro sozinho quando a sessÃ£o fecha | automaÃ§Ã£o |
| `modelos/` | Validador de passo (gera o que falta ou para), timer semanal do jardineiro, exemplos de ganchos (`ganchos-exemplo.json`) | â€” |

Integra com o [SIL Loop R](https://github.com/inematds/sil-loop-r) (`--sil <projeto>`): as propostas entram como
ocorrÃªncia + liÃ§Ã£o **proposta**; sÃ³ viram regra com a sua decisÃ£o.

## O processo, do inÃ­cio ao fim

ðŸ§  sessÃ£o com vocÃª Â· âš™ï¸ script/timer Â· ðŸ¤– agente

| # | Passo | Comando | Quem |
|---|---|---|---|
| 1 | Registrar cada falha numa linha (data, o que quebrou, menor correÃ§Ã£o, prompt/infra) | `FALHAS.md` / `LESSONS.md` | ðŸ§  |
| 2 | No fim da sessÃ£o, a retro roda sozinha e guarda o relatÃ³rio se houve correÃ§Ã£o ou erro repetido | `hooks/fim-de-sessao.sh` (SessionEnd) | âš™ï¸ |
| 3 | Ler a retro e escolher o que vira liÃ§Ã£o (portÃ£o humano) | `python3 retro.py <sessao.jsonl> [--sil P]` | ðŸ§  |
| 4 | Toda semana, o jardineiro mede a reincidÃªncia e lista os candidatos | `python3 jardineiro.py --saida jardim/AAAA-MM-DD.md` (timer) | âš™ï¸ |
| 5 | Para cada candidato, subir o degrau: gancho no `ganchos.json`, validador na cadeia ou teste | editar `ganchos.json` / `modelos/validador-passo.sh` | ðŸ§  + ðŸ¤– |
| 6 | Testar a trava (o comando errado Ã© bloqueado, o certo passa) | `python3 -m unittest discover -s tests` | âš™ï¸ |
| 7 | Na semana seguinte, a reincidÃªncia tem que cair; o candidato tratado some da lista | `jardineiro.py` de novo | âš™ï¸ |

## ComeÃ§ar

```bash
git clone https://github.com/inematds/agente-que-aprende.git && cd agente-que-aprende
python3 -m unittest discover -s tests -v            # 10 testes, sem rede
python3 degrau.py caminho/do/projeto/FALHAS.md      # o que subir de degrau neste projeto
python3 jardineiro.py --raiz ~/projetos             # todos os projetos de uma vez
python3 retro.py ~/.claude/projects/<pasta>/<id>.jsonl --pergunta   # retro + pedido para colar no agente
```

Ligar no Claude Code (`~/.claude/settings.json`):

```json
"hooks": {
  "PreToolUse": [{"matcher": "Bash", "hooks": [{"type": "command", "command": "python3 /caminho/agente-que-aprende/gancho.py"}]}],
  "SessionEnd": [{"hooks": [{"type": "command", "command": "bash /caminho/agente-que-aprende/hooks/fim-de-sessao.sh"}]}]
}
```

O `ganchos.json` do kit traz sÃ³ a trava universal (`pkill -f`). Travas da sua mÃ¡quina vÃ£o num arquivo seu, ligado por `AQA_GANCHOS=/caminho/meus.json` antes do comando do hook. O gancho funciona no Claude Code; no Codex, a retro e o jardineiro jÃ¡ funcionam.

Timer semanal: copiar `modelos/systemd/jardineiro.{service,timer}` para `~/.config/systemd/user/`, ajustar os caminhos e
`systemctl --user enable --now jardineiro.timer`.

## Formatos que ele lÃª

- **FALHAS.md**: tabela `| data | o que quebrou | menor correÃ§Ã£o | prompt \| infra |`
- **LICOES.md**: tabela `| data | o que quebrou | proteÃ§Ã£o |`
- **LESSONS.md**: lista `- regra em uma linha (dd/mm/aaaa)`
- **SessÃµes**: Claude Code (`~/.claude/projects/*/*.jsonl`) e Codex (`~/.codex/sessions/**/rollout-*.jsonl`). SÃ³ as mensagens
  digitadas por vocÃª contam como correÃ§Ã£o (avisos de tarefa, resumos de compactaÃ§Ã£o e continuaÃ§Ãµes automÃ¡ticas ficam fora).

## Limites

- O degrau atual e a sugestÃ£o sÃ£o **palpites por palavras** (transparentes, ajustÃ¡veis em `aprende/comum.py`), nÃ£o uma
  anÃ¡lise por modelo. A decisÃ£o Ã© sua.
- O jardineiro junta erros parecidos por semelhanÃ§a de palavras; erros iguais escritos de jeitos muito diferentes podem
  ficar separados.
- Um loop que roda sem ninguÃ©m olhando tambÃ©m erra sem ninguÃ©m olhando: nada aqui adota regra sozinho.

LicenÃ§a MIT.

---

Projeto adaptado da comunidade INEMA TDS - Marca Arckhe