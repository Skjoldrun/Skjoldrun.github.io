---
layout: page
title: Linux - llama.cpp router: GGUF dw and tuning
parent: Linux
---

# llama.cpp router: authenticated GGUF download and tuning

Serving several LLMs with the llama.cpp multi-model router means downloading large GGUF files from [HuggingFace](https://huggingface.co/), placing them where the router expects them, and telling the router how much context and parallelism each model can afford. This article walks through an authenticated download (which lifts HuggingFace's anonymous bandwidth throttling), the KV-cache math that decides the `models.ini` values, and the reload pitfalls that can silently kill the router.

## The setup: llama.cpp router

A typical deployment runs `llama-server` as a systemd service in *router mode*:

```bash
ExecStart=/opt/llama.cpp/bin/llama-server \
  --models-dir /var/lib/llama.cpp/models \
  --models-preset /etc/llama.cpp/models.ini \
  --models-max 2 \
  --host 0.0.0.0 --port 13405 \
  --n-gpu-layers 999 --flash-attn on --jinja --metrics
```

The router scans `--models-dir`, reads the tuning overrides from `--models-preset`, and spawns one child `llama-server` per model. `--models-max 2` means at most two models stay warm; the router unloads idle ones to make room.

```
                     /etc/llama.cpp/models.ini
                              |
                              v
   llama-server (router, :13405)          --models-dir /var/lib/llama.cpp/models
        |
        +--> child llama-server  Model A   (model .gguf in its own folder)
        +--> child llama-server  Model B   (model .gguf in its own folder)
```

Each model lives in its own folder named after the GGUF basename:

```
/var/lib/llama.cpp/models/
  Qwen3.6-27B-UD-Q4_K_XL/
    Qwen3.6-27B-UD-Q4_K_XL.gguf
    mmproj-F16.gguf                 # vision projector, optional
```

## Authenticated download for full bandwidth

Anonymous HuggingFace downloads are rate-limited; authenticated requests get the fast CDN routes. You need a **Read** access token from <https://huggingface.co/settings/tokens>.

Arch Linux system Python refuses global packages (externally managed environment), so install the CLI into a virtualenv:

```bash
python3 -m venv /opt/hf-venv
/opt/hf-venv/bin/pip install -U "huggingface_hub"
```

Log in once; the token is stored in `~/.cache/huggingface/token` with mode `600`, readable only by you:

```bash
/opt/hf-venv/bin/hf auth login --token hf_xxxxxxxxxxxxxxxx
# Token is valid (permission: read)
# Your token has been saved to /home/you/.cache/huggingface/token
```

> Never put the token in scripts, logs or command history. Read it from the token file inside the download script instead.

### Download a single GGUF with resume support

`curl` with the `resolve/main` URL, the Bearer token and `-C -` resumes interrupted transfers:

```bash
#!/bin/bash
HF_TOKEN="$(cat "$HOME/.cache/huggingface/token")"
DEST="/var/lib/llama.cpp/models/Qwen3.8-27B-UD-Q8_K_XL"
mkdir -p "$DEST"
BASE="https://huggingface.co/unsloth/Qwen3.8-27B-GGUF/resolve/main"
for f in Qwen3.8-27B-UD-Q8_K_XL.gguf mmproj-F16.gguf; do
  curl -L --fail --retry 10 --retry-delay 5 --retry-all-errors \
    -H "Authorization: Bearer $HF_TOKEN" \
    -C - -o "$DEST/$f" "$BASE/$f"
done
```

Run it fully detached so it survives the calling shell:

```bash
setsid /opt/scripts/download-model.sh </dev/null >/var/log/model-download.log 2>&1 &
```

`nohup ... &` alone is not enough here: some terminal/tooling setups kill the whole process group on session exit, taking the download with it. `setsid` puts the script in its own session so it keeps running. Monitor progress without blocking:

```bash
tail -c 300 /var/log/model-download.log | tr '\r' '\n' | tail -3
```

Authenticated, a 29 GiB GGUF downloads at roughly 60-70 MB/s instead of the throttled anonymous speed.

## Not every repo ships a GGUF

The upstream `Qwen/Qwen3.8-27B` repository contains only `safetensors`. Community conversions like `unsloth/Qwen3.8-27B-GGUF` publish the GGUF plus the `mmproj-F16.gguf` vision projector. Check the file list before downloading:

```bash
curl -s "https://huggingface.co/api/models/unsloth/Qwen3.8-27B-GGUF/tree/main?recursive=true"
```

Verify the result is a real GGUF and not a truncated download:

```bash
xxd -l 4 Qwen3.8-27B-UD-Q8_K_XL.gguf   # must print "GGUF"
```

## Choosing a quantization

Trade-offs for a 27B-parameter model:

| Quant | GGUF size | Quality | Notes |
|---|---|---|---|
| Q4_K_M | ~15 GiB | good | smallest practical 4-bit |
| Q4_K_XL | ~16 GiB | better | higher-bit blocks for sensitive tensors |
| Q8_K_XL | ~29 GiB | near-lossless | ~1 byte per parameter |

Pick based on free RAM plus how much context you intend to serve (next section).

## The KV-cache math that decides context and parallelism

The `models.ini` values are not guesses: context (`c`) and slots (`parallel`) multiplied by the per-token KV size must fit in RAM together with the weights.

### Full-attention models

Classic transformers keep a KV cache on every layer:

```
KV bytes per token = 2 (K+V) x layers x kv_heads x head_dim x bytes_per_element
```

### Hybrid models (Qwen3.5 / Qwen3-Next style)

These mix full-attention with linear-attention (Mamba-style) layers. Only the full-attention layers build a real KV cache, so inspect the model `config.json`:

```json
"layer_types": [
    "linear_attention", "linear_attention", "linear_attention", "full_attention",
    ...
],
"num_hidden_layers": 64,
"num_key_value_heads": 4,
"head_dim": 256
```

64 layers, every fourth is `full_attention` -> 16 KV layers. With fp16 flash attention:

```
KV bytes per token = 2 x 16 x 4 x 256 x 2 = 65536 bytes = 64 KiB/token
```

| Context per slot | Slots | KV cache total |
|---|---|---|
| 262144 | 1 | ~16 GiB |
| 131072 | 2 | ~16 GiB |
| 131072 | 1 | ~8 GiB |

So for a 29 GiB Q8 model at 2 slots x 131072 you budget roughly 29 + 16 = 45 GiB. The same model at Q4_K_XL would be about 17 + 16 = 33 GiB.

## Configuring models.ini

`models.ini` has a global `[*]` block whose values every model inherits, overridable per section:

```ini
[*]
c = 65536
n-gpu-layers = 999
flash-attn = on
jinja = true
temp = 0.7

[Qwen3.8-27B-UD-Q8_K_XL]
c = 131072
parallel = 2
temp = 0.7
```

Always add a blank line before a section header, and comment your KV math so the next person can re-validate it:

```ini
; Qwen3.8-27B (hybrid model, only 16 full-attn layers -> ~64 KiB/tok KV).
; 2 slots x 131072 = 262144 tokens (~16 GiB KV at full use + ~30 GiB weights).
[Qwen3.8-27B-UD-Q8_K_XL]
c = 131072
parallel = 2
temp = 0.7
```

If two huge models are candidates to be warm at the same time, sum their budgets: 86 GiB coder + 46 GiB model exceeds a 124 GiB machine. `--models-max 2` plus the router unloading idle models usually saves you, but keep the arithmetic honest.

## Pitfalls

- **Pasting a multi-line heredoc into a terminal often mangles it.** Instead, write the block to a file and append it in one short command:
  ```bash
  sudo tee -a /etc/llama.cpp/models.ini < /tmp/models.ini.append
  ```
- **`systemctl reload` (HUP) can kill the router.** The reload signal is delivered as `SIGHUP`; if the preset file fails to parse (e.g. a section header glued to a trailing comment), the router exits and `Type=simple` with `Restart=on-failure` does not bring a HUP-killed process back. Prefer `systemctl restart` after editing `models.ini`:
  ```bash
  sudo cp /tmp/models.ini /etc/llama.cpp/models.ini
  sudo systemctl restart llama-router
  ```
- **Comments glued to section headers break parsing.** `[Qwen3-Embedding-4B-Q8_0]; note` - the `]` must end the header line; keep `;` comments on their own lines.
- **Verify through the API**, not just `systemctl status`. The router lists all known models including the new one:
  ```bash
  curl -s -H "Authorization: Bearer $(cat /etc/llama.cpp/api-key)" \
    http://localhost:13405/v1/models
  ```
  The model loads lazily on the first request.

## Sources

- llama.cpp server / router documentation: https://github.com/ggml-org/llama.cpp/tree/master/tools/server
- HuggingFace tokens: https://huggingface.co/settings/tokens
- huggingface_hub CLI reference: https://huggingface.co/docs/huggingface_hub/main/en/guides/cli
- Qwen3.8-27B GGUF conversion: https://huggingface.co/unsloth/Qwen3.8-27B-GGUF