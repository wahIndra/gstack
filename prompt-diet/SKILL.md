---
name: prompt-diet
preamble-tier: 2
version: 1.0.0
description: "Compresses and minifies context payloads using local models to reduce token usage and cost. (gstack)"
triggers:
  - run prompt diet
  - compress context
  - optimize token usage
  - /prompt-diet
allowed-tools:
  - Bash
  - Read
  - Write
  - Edit
  - Glob
---
<!-- AUTO-GENERATED from SKILL.md.tmpl — do not edit directly -->

## When to invoke this skill

Invoke this skill when you are about to read a massive file, a large documentation repository, or process dense logs. This skill intercepts the target files, sends them to the local `prompt-optimizer-proxy` on port 3000, and returns a densely compressed Markdown summary. This prevents polluting your main context window with expensive, unoptimized tokens.

Use when asked to "/prompt-diet", "compress context", "optimize token usage", or when you independently decide that the current task requires digesting a payload larger than 8,000 tokens.

## What it does
1. Reads the target file(s) specified by the user.
2. Sends the raw content to the `prompt-optimizer-proxy` backend (localhost:3000) using the `useSemanticCompression=true` flag.
3. The proxy processes it through a local Ollama model (llama 3.2 or equivalent), extracting ONLY highly technical details, API endpoints, and critical code structures.
4. Returns the minified, dense Markdown back to your context.

## Instructions

If the user does not specify a target file or folder, ask them for the path.

Run the following bash command to submit the file to the optimizer proxy (replace `$TARGET_FILE` with the actual file path):

```bash
FILE_CONTENT=$(cat $TARGET_FILE | jq -R -s '.')

curl -s -X POST http://localhost:3000/api/chat \
  -H "Content-Type: application/json" \
  -d '{
    "provider": "ollama",
    "model": "llama3.2",
    "useSemanticCompression": true,
    "payload": {
      "prompt": '"$FILE_CONTENT"'
    }
  }' | jq -r '.responseData' > "$TARGET_FILE.compressed.md"

echo "Context successfully compressed and saved to $TARGET_FILE.compressed.md. Please read this compressed file instead of the original."
```

Once the command finishes, use your standard `Read` or `view_file` tool to read the newly generated `.compressed.md` file and proceed with your overarching task.
