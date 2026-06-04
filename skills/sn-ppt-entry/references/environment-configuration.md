# PPT Skills — Environment Configuration

> Captured session: 2026-06-01 — PPT skills smoke test
> File: `references/environment-configuration.md`

## Overview

The PPT skill family consists of interconnected skills under `sn-ppt-*`. Before dispatching to any mode, the agent must confirm environment readiness. This file documents the credential layout, key paths, and known failure modes.

## Credential Layout

### Required (hard block if missing)

| Purpose | Env var | Backend endpoint |
|---------|---------|-----------------|
| LLM text chat | `SN_API_KEY` (preferred) OR `SN_TEXT_API_KEY` | `https://token.sensenova.cn/v1` |
| Vision chat | `SN_VISION_API_KEY` (preferred) OR `SN_CHAT_API_KEY` | `https://token.sensenova.cn/v1` |
| Text-to-image | `U1_API_KEY` | `https://u1-api.sensenova.cn/model` |

### Industry standards appear as `OPENAI_API_KEY`**

The SenseNova API defaults;** the `SENSENOVA` prefix is required. `OPENAI_API_KEY` is **not** read by `model_client.py` — it explicitly checks `SN_TEXT_API_KEY`, `SN_CHAT_API_KEY`, `SN_API_KEY`.

### Image search

| Purpose | Env var | Backend |
|---------|---------|---------|
| Image search (standard mode) | `SERPER_API_KEY` | Serper.dev |

## Models

| Component | Default model | Env override |
|-----------|--------------|--------------|
| LLM chat | `sensenova-6.7-flash-lite` | `SN_TEXT_MODEL` / `SN_CHAT_MODEL` |
| Vision | `sensenova-6.7-flash-lite` | `SN_VISION_MODEL` |
| T2I | `SenseNova-U1-Preview` | `U1_IMAGE_GEN_MODEL` |

Base URLs: `SN_TEXT_BASE_URL`, `SN_VISION_BASE_URL`, `SN_CHAT_BASE_URL`, `U1_IMAGE_GEN_BASE_URL`.

## Running the Diagnostics

### Correct path to sn-ppt-doctor

```bash
python3 /home/chenhaodong/ppt-api/ppt-superpower-store/SenseNova-Skills/skills/sn-ppt-doctor/ppt_doctor/check_environment.py --non-interactive
```

⚠️ The script is at `$SKILL_DIR/ppt_doctor/check_environment.py` — **not** `$SKILL_DIR/check_environment.py` (no `ppt_doctor/` subdirectory).

## Dependency Checklist

| Package | Purpose | Install command |
|---------|---------|----------------|
| `python-dotenv` | `.env` loading in `model_client.py` | `pip3 install python-dotenv` |
| `openai` | HTTP client for SenseNova API | `pip3 install openai` |
| `litellm` | Alternative chat client | `pip3 install litellm` |
| `python-pptx` | PPTX assembly (creative mode `build_pptx.py`) | `pip3 install python-pptx` |
| `pillow` | Image preprocessing | `pip3 install pillow` |
| `pypdf` | PDF text extraction | `pip3 install pypdf` |
| `python-docx` | DOCX text extraction | `pip3 install python-docx` |
| `playwright` + Chromium | PPTX export (`html_to_pptx.mjs` headless browser) | `pip3 install playwright && npx playwright install chromium` |

## Known Pitfalls

### 1. `OPENAI_API_KEY` ≠ `SN_API_KEY`

The `model_client.py` in `sn-ppt-standard/lib/` reads env vars exclusively as:
```python
_env("SN_TEXT_API_KEY", "SN_CHAT_API_KEY", "SN_API_KEY")
```
If the SenseNova token is only available as `OPENAI_API_KEY` (it may be stored that way in `.env`), the PPT pipeline will fail at dispatch. Fix: copy the value to `SN_API_KEY` in `.env`.

### 2. Two separate billing accounts

`U1_API_KEY` (image generation via SenseNova U1) and `SN_API_KEY` (LLM chat via SensorNova) may live on different billing systems. A working U1 key does not imply a working SN key. Always test both independently.

### 3. PPTX export is optional

Standard mode HTML pages are the primary deliverable. PPTX (`--export`) is a bonus — if Chromium is unavailable, the skill degrades gracefully to HTML-only output. Never fail the whole pipeline over missing browser.

### 4. Model defaults only set if env is configured

`model_client.py` uses default base URL `https://token.sensenova.cn/v1` and model `sensenova-6.7-flash-lite` when no override is set. These defaults are correct for SenseNova users but will fail with other providers.

### 5. U1 API returns 401 with invalid format

The U1 endpoint (`u1-api.sensenova.cn/model`) validates API keys separately from `token.sensenova.cn/v1`. If you get a 401 ("Invalid API key in request"), confirm the key hasn't been rotated and matches the U1 billing plan.
