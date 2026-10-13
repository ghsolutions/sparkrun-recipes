# sparkrun-recipes (gdAI-Labs)

SparkRun-Rezept-Registry von **gdAI-Labs** für NVIDIA DGX Spark (GB10) —
produktionsreife vLLM-Lanes für 2-Knoten-Cluster (Tensor-Parallel 2) mit
revision- bzw. digest-pinned Containern und Modell-Checkpoints.

## Registry hinzufügen

```bash
python3 -m pip install --upgrade sparkrun

sparkrun registry add http://192.168.233.30:30080/dgx-spark/sparkrun-recipoes.git
```

Optional mit Trust (Rezept-Hooks `pre_exec`/`post_exec`/`post_commands` laufen
dann ohne Rückfrage — nur empfehlen, wenn man die Rezepte geprüft hat; diese
Registry enthält aktuell keine Hooks):

```bash
sparkrun registry add http://192.168.233.30:30080/dgx-spark/sparkrun-recipoes.git --trust
```

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

## Verwendung

```bash
sparkrun list                          # alle registrierten Rezepte
sparkrun search qwen3                  # Suche über alle Registries
sparkrun show @gdai-labs/qwen3.8-flash-next-fp8-vllm   # Defaults + VRAM-Schätzung
sparkrun run @gdai-labs/minimax-m2.7-awq-vllm --dry-run
sparkrun run @gdai-labs/qwen3.8-flash-next-fp8-vllm -H <head-ip>,<worker-ip>
sparkrun stop @gdai-labs/qwen3.8-flash-next-fp8-vllm
```

## Repo-Layout

```
├── .sparkrun/
│   └── registry.yaml      # Manifest — Pflicht für Auto-Discovery
└── recipes/               # YAML-Rezepte (Name-Feld == Dateistem)
    ├── minimax-m2.7-awq-vllm.yaml
    ├── qwen3.8-27b-uncensored-fp8-vllm.yaml
    └── qwen3.8-flash-next-fp8-vllm.yaml
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

## Hinweise / bekannte Baustellen

- Die Rezepte setzen netzwerkspezifische Env-Variablen (`NCCL_SOCKET_IFNAME=
  enp1s0f0np0`) und Offline-Betrieb (`HF_HUB_OFFLINE=1`) voraus — Modelle und
  Container müssen auf den Nodes bereits vorliegen (`externally_provided`).
- `qwen3.8-27b-uncensored-fp8-vllm`: der `distribution_config`-Eintrag zeigt
  noch auf `Qwen/Qwen3.8-Flash-Next-FP8` statt auf das Uncensored-Modell
  (TODO: prüfen/korrigieren).

## Links

- SparkRun-Doku: <https://sparkrun.dev/> (Recipe Format, Registries, Security Model)
- Spark Arena: <https://spark-arena.com/>
