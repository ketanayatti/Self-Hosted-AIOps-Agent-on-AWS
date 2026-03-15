# 📁 docs — Project Screenshots & Diagrams

This folder is the home for all **visual assets** used in the project README and documentation.

---

## 🖼️ How to Add Your Screenshots

1. Take your screenshots (see the list below for what to capture).
2. Name each file **exactly** as shown in the table — the README references these filenames directly.
3. Drop the image files into this `docs/` folder.
4. Commit and push. The images will automatically appear in the README.

```bash
git add docs/
git commit -m "docs: add project screenshots"
git push
```

---

## 📸 Recommended Screenshots

| Filename | What to Capture |
|---|---|
| `api-response.png` | A `curl` or Postman response from `POST /query` showing the agent replying to a natural-language question |
| `metrics-dashboard.png` | Output of `GET /metrics` showing live CPU & memory percentages |
| `architecture-diagram.png` | Architecture diagram (export from draw.io, Excalidraw, or a clean terminal diagram) |
| `llm-inference.png` | Terminal output showing `llama-server` running and tokens being generated |
| `ec2-instance.png` | AWS Console view of the running EC2 t2.micro instance |
| `health-check.png` | Browser or curl response from `GET /health` confirming all services are up |
| `shell-exec.png` | Response from `POST /exec` showing a command (e.g. `df -h`) being run via the API |
| `swap-config.png` | Terminal output of `free -h` confirming swap is active on the server |

---

## 💡 Tips for Great Screenshots

- **Use a terminal with a dark theme** (e.g. Dracula, One Dark) — screenshots look far more professional.
- **Use `jq`** to pretty-print JSON API responses: `curl ... | jq .`
- **Crop tightly** — remove unnecessary browser chrome or desktop clutter.
- **Consistent resolution** — aim for at least 1280 × 720 px.
- **Annotate key areas** — use a free tool like [Annotely](https://annotely.com/) or [Lightshot](https://app.prntscr.com/) to add arrows/highlights.

---

## 🛠️ Architecture Diagram Tools

If you need to create or update the architecture diagram, these free tools are recommended:

| Tool | Link |
|---|---|
| **Excalidraw** | https://excalidraw.com — hand-drawn style, great for tech diagrams |
| **draw.io / diagrams.net** | https://app.diagrams.net — professional flowcharts |
| **Mermaid Live** | https://mermaid.live — code-based diagrams |
| **Cloudcraft** | https://www.cloudcraft.co — AWS-specific 3D architecture diagrams |

---

> **Note:** Image files are intentionally excluded from `.gitignore` — commit them directly to this folder so they render in the GitHub README.
