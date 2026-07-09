# Uso — ak-adcreative (fork do arcads-claude-code)

> Fork privado do [arcads-claude-code](https://github.com/krusemediallc/arcads-claude-code) para
> produção de **criativo de anúncio** (imagem/vídeo) via API da Arcads. Adotado 2026-07-08 para
> atacar o gargalo "criativo alto e manual" da **Oreon** (piloto 1º). Arquivo local — não conflita
> com `git pull` do upstream.

## O que é (e o que NÃO é)

- **É** um skill pack de Claude Code para gerar criativo: imagem estática de Meta ad (37 templates),
  UGC, vídeo (Seedance/Sora/Veo/Kling), e publicar como **Meta ad PAUSED**.
- **NÃO** faz site e **NÃO** usa o higgsfield MCP — usa a **API própria da Arcads** (paga, por
  crédito), que já embute os modelos. Higgsfield é concorrente separado (MCP próprio) — não usado aqui.

## Setup (uma vez, ~5 min)

```bash
cd ~/dev/portfolio/ak-adcreative
./scripts/setup.sh        # Git Bash (o script assume bash)
```
Pede a **API key da Arcads** (`app.arcads.ai/settings/api` — conta paga), grava em `.env` (gitignored),
cria `MASTER_CONTEXT.md`. Preencher nele: produto default, custo de crédito, brand voice — o agente lê
toda sessão e para de perguntar. Testar conexão: `./scripts/check-arcads-env.sh`.

Para publicar na Meta (opcional): `META_ACCESS_TOKEN` + `META_AD_ACCOUNT_ID` no `.env`.

## Piloto (a decisão: vira operação ou morre?)

Comece pelo **estático** — geradores são Python stdlib puro, sem ffmpeg/WSL. Vídeo (Seedance/Veo)
exige `ffmpeg`; deixa pra fase 2.

1. Abrir Claude Code **na pasta** `ak-adcreative` (as skills são project-level, só ativam aqui).
2. Dropar 1 referência real em `references/products/` (um criativo que você faz à mão hoje).
3. Pedir em linguagem natural, ex.:
   - `"Make me an Apple Notes-style ad for [produto]"`
   - `"Clone this comparison-table ad for our product"`
4. Ele estima o **custo em crédito → você confirma → gera**. Compara tempo/qualidade com o manual.
5. (Opcional) `"Publish this as a paused Meta ad"` — sai **PAUSED**, você lança manualmente.

**Critério:** se reproduz em minutos o que toma horas na Oreon, vira operação. Se não, mata.

## Guardrails (nativos, mantêm você no controle)

- **Custo confirmado** antes de cada geração; log em `logs/arcads-api.jsonl` (histórico de créditos).
- Meta ad sempre **PAUSED** — nada vai ao ar sem você.
- `.env` e `MASTER_CONTEXT.md` gitignored — key nunca commitada, nunca colada no chat.

## Como isto plugará no Hermes depois (modelo A)

Arcads é um **braço criativo** do mesmo modelo A do Hermes (ver `ak-agent-lab/docs/telegram-hermes.md`).
O Hermes já delega código ao Claude Code via a skill `dev-vps`. Quando o piloto provar ROI, "gera um
criativo pro produto X" pelo Telegram → Hermes → `dev-vps` → Claude Code nesta pasta → skill arcads
gera. Nenhuma ponte nova: reusa o despacho que já existe. Pré-condição: a API key da Arcads no BWS.

## Manter em dia com o upstream

```bash
git fetch upstream 2>/dev/null || git remote add upstream https://github.com/krusemediallc/arcads-claude-code.git
git fetch upstream && git merge upstream/main   # seus .env/MASTER_CONTEXT/references sobrevivem (gitignored)
```
Customização de skill do core: manter sob caminho não-rastreado pra evitar conflito no merge.
