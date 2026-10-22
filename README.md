# NVIDIA's Nemotron 3.5 Bolt 74B on a 12GB card + 32GB of RAM: every command and every number

This is everything from the NoCodeAbode video, as promised to everyone who commented "12 gig". Free. Run it on your own PC and see what you get.

NVIDIA's model card says the 24GB GGUF fits in 24GB of VRAM. My card has 12. The first run did 2.6 tokens a second. An hour later, same card, same file: 30 tokens a second, and Claude Code running on it locally passed 24 of 24 checks. The fix had nothing to do with the model.

Measured on my PC on October 18, 2026. One run per setting. Your numbers will differ.

## My PC

- Graphics card: RTX 4070 SUPER, 12GB
- RAM: 32GB DDR5-5200
- CPU: Intel Core i5-14400F
- Drive: 2TB NVMe SSD
- Windows 11

## The model

- Repo: https://huggingface.co/nvidia/NVIDIA-Nemotron-3.5-Bolt-74B-A9B-24GB-GGUF
- File: `NVIDIA-Nemotron-3.5-Bolt-74B-A9B-24GB.gguf` (22 GB on disk)
- Text only. I skipped the vision file and the draft model.
- 74B total, about 9B active per word, 2-bit.

## The engine

- llama.cpp build **b11721**, the official Windows CUDA build: https://github.com/ggml-org/llama.cpp/releases/tag/b11721
- Download `llama-b11721-bin-win-cuda-13.4-x64.zip` plus the cudart zip from the same release and unzip them into one folder.
- Bolt support starts at b11669. Older builds cannot load the file.

## THE RULE: free about 14GB of your 32GB before you start

With a browser (40 processes, 6.3 GB), Telegram, Teams and WSL2 open, I had **188 MB** of RAM free with the model loaded. The model pulled from the SSD up to 20,316 times a second and wrote at **2.6 tok/s**.

Closed the browser and the chat apps, shrank WSL2: **3.5 GB free with the model loaded, 30.5 tok/s.** Same file, same card.

Check it in Task Manager, Performance, Memory, "Available". Do this before you launch, not after.

## The launch line (every run)

```
llama-server.exe -m NVIDIA-Nemotron-3.5-Bolt-74B-A9B-24GB.gguf --host 127.0.0.1 --port 8092 -c 32768 -fa on -ctk q8_0 -ctv q8_0 --jinja --no-webui -np 1
```

No layer flags. llama.cpp's default fit put 9.7 GB on the card and the experts of 57 of 88 layers in system RAM.

The server speaks the OpenAI chat format at `http://127.0.0.1:8092/v1/chat/completions`. For the six jobs I sent `temperature 0.2`, `top_p 1.0`, `reasoning_effort: "none"` and `chat_template_kwargs: {"enable_thinking": false}` (thinking off).

## Claude Code on it

Same launch line with `-c 65536` (64K context), then in PowerShell before you start Claude Code:

```powershell
$env:ANTHROPIC_BASE_URL = "http://127.0.0.1:8092"
$env:ANTHROPIC_AUTH_TOKEN = "local-bolt"
$env:ANTHROPIC_API_KEY = ""
$env:ANTHROPIC_MODEL = "claude-opus-5-5"
```

The base URL is the switch. The token can be any text.

Claude Code opens every session with about 27,500 tokens of its own instructions. On this PC that is 68 to 103 seconds of reading before the first turn, then about 2 seconds a turn. It writes at 27 to 29 tok/s.

## Every number (thinking off, temperature 0.2, one clean run)

| Job | Result |
|---|---|
| Writing speed, median of six jobs | **30.5 tok/s** (per job: 22.4 / 29.8 / 30.5 / 30.8 / 30.8 / 30.5) |
| All six jobs, wall time | 361 s |
| Invoice fix (6 checks) | **6/6**. Qwen 3.8 27B failed this 3 runs out of 3 on the same card |
| Snake game (6 checks) | 6/6 |
| Estimate calculator (5 checks) | 3/5. Math right, money wrong: $1,250 prints as $250,250.00. One missing pair of parentheses in one regex line |
| 20,049-token prompt, first word | **46.5 s** (430 tok/s read). Flash-Next Coder on plain llama.cpp: 85.5 s |
| 11 comment replies from a facts list | Fail. It renamed my 2-bit file "Q4" and contradicted the facts in 6 of 11 replies |
| GPU power, whole run | 109.5 W average |
| Qwen 3.8 27B (9.5 GB, all on the card) for comparison | 46 tok/s. Still the faster writer |

With thinking on, the invoice took 448 s (13,586 tokens, first answer at 302 s) and scored 5/6. With NVIDIA's own sampling (temperature 1.0, top-p 0.95) the snake game dropped to 4/6.

## Claude Code on Bolt (64K context)

| Job | Bolt, local, $0 | Opus 5.5 |
|---|---|---|
| A. Fix two invoice bugs (6 checks) | 6/6, 404 s, 0 tool errors | 6/6, 40.6 s |
| B. Build the estimate calculator (5 checks) | 5/5, 395 s. It ran its own check before finishing | 5/5 |
| C. Add a travel fee (13 hidden tests) | 13/13, 149 s | 13/13, 30.6 s |
| Total | **24/24 in 947 s (15.8 min)** | 157 s |

Strata's Flash-Next Coder on the same three jobs: 33 minutes, and it died at the 64K wall on job B.

## The prompts

The `prompts` folder has the exact text I pasted in. One new chat per job. Thinking off.

| File | What it asks for | How I graded it |
|---|---|---|
| `1-cold-lead-text.txt` | One follow-up text to a lead | Under 300 characters, one question at the end, nothing invented |
| `2-invoice-fix.txt` | Fix two bugs in an invoice page | A robot opens the page and runs 6 checks |
| `3-transcript-description.txt` | A YouTube description and chapters from a transcript | Every timestamp real, every fact from the transcript |
| `4-estimate-calculator.txt` | A job estimate calculator, one HTML file | A robot types numbers in and checks the totals, 5 checks |
| `5-comment-replies.txt` | Replies to 11 YouTube comments from a facts list | Every number from the facts list, nothing invented |
| `6-snake-game.txt` | A snake game, one HTML file | A robot plays it, 6 checks |
| `7-long-prompt-details-doc.txt` | Five questions about a 20,000-token document | All five answers in the document |

## Who it's for

- You have 12GB of VRAM and 32GB of RAM and you can close your browser while it runs: this is the strongest model I have run on this PC for code. It passed the invoice job the 27B never did.
- You keep 20 tabs open and chat apps running: you will get 2.6 tok/s and blame the model. Free the RAM first.
- You only do short writing jobs: the 27B on the card is faster and the 2-bit drift in Bolt's writing is real.

## Same PC, same six jobs, other models

- Qwen 3.8 27B vs Flash-Next: https://www.youtube.com/watch?v=YMZ8vKNqi9c
- Strata's 125B Coder, steps and prompts: https://github.com/JonDipilato/strata-coder-12gb-32gb
- Which file fits your card: https://github.com/JonDipilato/local-ai-card-cheat-sheet

Tell me your free RAM before you start and which card you run. I read every comment.
