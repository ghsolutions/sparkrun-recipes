# sparkrun-recipes (gdAI-Labs)

SparkRun-Rezept-Registry von **gdAI-Labs** für NVIDIA DGX Spark (GB10) —
produktionsreife vLLM-Lanes für 2-Knoten-Cluster (Tensor-Parallel 2) mit
revision- bzw. digest-pinned Containern und Modell-Checkpoints.

## Registry hinzufügen

**Wichtig:** `sparkrun registry add` akzeptiert nur die Schemes `https://`,
`ssh://`, `git@…:` und `file://`. Die plain-HTTP-URL des Gitea
(`http://192.168.233.30:30080/...`) wird vom CLI abgelehnt.

```bash
python3 -m pip install --upgrade sparkrun

# Variante 1: SSH gegen das Gitea (dafür muss ein SSH-Key im Gitea hinterlegt sein)
sparkrun registry add ssh://git@192.168.233.30/dgx-spark/sparkrun-recipoes.git

# Variante 2: lokale Arbeitskopie auf der Steuer-Maschine
sparkrun registry add "file:///Volumes/USR/ENTWICKLUNG/KI + AI LLM/sparkrun-recipoes"

# Variante 3: https://, sobald das Gitea hinter einem TLS-Proxy läuft
sparkrun registry add https://gitea.internal.lan/dgx-spark/sparkrun-recipoes.git
```

Optional mit Trust (Rezept-Hooks `pre_exec`/`post_exec`/`post_commands` laufen
dann ohne Rückfrage — nur empfehlen, wenn man die Rezepte geprüft hat; diese
Registry enthält aktuell keine Hooks): `--trust` anhängen.

Aktualisieren nach Repo-Änderungen:

```bash
sparkrun registry update
```

## Rezepte

| Rezept | Modell | Runtime | Nodes | Beschreibung |
|---|---|---|---|---|
| `@gdai-labs/qwen3.8-flash-next-fp8-vllm` | `Qwen/Qwen3.8-Flash-Next-FP8` (rev. `bcd9f01`) | vLLM (digest-pinned) | 2 × TP2 | Memory-safe Flash-Next-Lane: MTP3, eager, PP+Caching, Multimodal ohne Processor-Cache |
| `@gdai-labs/qwen3.8-27b-uncensored-fp8-vllm` | `orcarouter/Qwen3.8-27B-Uncensored-FP8` | vLLM (digest-pinned) | 2 × TP2 | Uncensored-Variante auf gleichem vLLM-Build |
| `@gdai-labs/minimax-m2.7-awq-vllm` | `cyankiwi/MiniMax-M2.7-AWQ-4bit` | vLLM (Ray) | 2+ | MiniMax M2.7 AWQ 4-bit, Tool-Calling + Reasoning-Parser |
| `@gdai-labs/deepseek-v4-flash-0731` | `deepseek-ai/DeepSeek-V4-Flash-0731` | vLLM | 2+ | DeepSeek V4 Flash 0731 on dual DGX Spark, TP=2, PR 41834 SM12x |

## Verwendung

```bash
sparkrun list                          # alle registrierten Rezepte
sparkrun search qwen3                  # Suche über alle Registries
sparkrun show @gdai-labs/qwen3.8-flash-next-fp8-vllm   # Defaults + VRAM-Schätzung
sparkrun run @gdai-labs/minimax-m2.7-awq-vllm --dry-run
sparkrun run @gdai-labs/qwen3.8-flash-next-fp8-vllm -H <head-ip>,<worker-ip>
sparkrun run @gdai-labs/qwen3.8-27b-uncensored-fp8-vllm -H <head-ip>,<worker-ip>
sparkrun run @gdai-labs/deepseek-v4-flash-0731 -H <head-ip>,<worker-ip>
sparkrun stop @gdai-labs/qwen3.8-flash-next-fp8-vllm
```

## Repo-Layout

```
├── .sparkrun/
│   └── registry.yaml      # Manifest — Pflicht für Auto-Discovery
└── recipes/               # YAML-Rezepte (Name-Feld == Dateistem)
    ├── minimax-m2.7-awq-vllm.yaml
    ├── qwen3.8-27b-uncensored-fp8-vllm.yaml
    ├── qwen3.8-flash-next-fp8-vllm.yaml
    └── deepseek-v4-flash-0731.yaml
```

## Neue Rezepte hinzufügen

1. `recipes/<name>.yaml` anlegen (Dateistem == `name`-Feld, Kleinschreibung)
2. `model`, `container` und `runtime` explizit setzen, `metadata.model_params`
   und `metadata.model_dtype` für die VRAM-Schätzung pflegen
3. Validieren und testen:
   ```bash
   sparkrun recipe validate recipes/<name>.yaml
   sparkrun run recipes/<name>.yaml --solo --dry-run
   ```
4. Pushen, dann bei den Clients `sparkrun registry update`

## Links

- SparkRun-Doku: <https://sparkrun.dev/> (Recipe Format, Registries, Security Model)
- Spark Arena: <https://spark-arena.com/>