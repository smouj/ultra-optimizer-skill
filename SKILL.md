---
name: Ultra Optimizer
description: AI-powered writing optimization engine for enhancing clarity, SEO, readability, and engagement
version: 1.2.1
author: OpenClaw Team <dev@openclaw.io>
tags:
  - writing
  - ai
  - automation
  - optimization
  - content
  - nlp
maintainer: dev@openclaw.io
license: MIT
dependencies:
  - python>=3.9
  - openai>=1.0.0
  - tiktoken
  - textstat>=0.7.0
  - spacy
  - markdown
  - jinja2
  - nltk
environment:
  OPENCL_ULTRA_OPENAI_API_KEY: required
  OPENCL_ULTRA_MODEL: "gpt-4-turbo-preview"
  OPENCL_ULTRA_MAX_TOKENS: 4000
  OPENCL_ULTRA_TEMPERATURE: 0.3
  OPENCL_ULTRA_REQUESTS_PER_MINUTE: 60
  OPENCL_ULTRA_CACHE_ENABLED: "true"
---

# Ultra Optimizer

AI-powered writing optimization engine that transforms raw text into polished, audience-tailored content. Integrates with OpenAI GPT models to apply sophisticated rewriting, tone adjustment, SEO enhancement, and readability improvements while preserving factual accuracy.

## Purpose

Real-world use cases:

- **SEO Optimization**: Rewrite blog posts with targeted keyword integration, meta description generation, and header restructuring to Rank higher in search engines. Example: a 1200-word article about "cloud security" optimized for keywords "zero trust architecture, cloud compliance, data protection".
- **Technical Simplification**: Convert developer documentation into user-friendly guides for non-technical audiences. Example: turning a Kubernetes deployment YAML guide into a step-by-step tutorial for product managers.
- **Marketing Copy Enhancement**: Transform feature lists into benefit-driven persuasive copy. Example: changing "Our software supports 10 databases" to "Seamlessly connect to 10+ databases, including PostgreSQL and MySQL, in minutes."
- **Executive Summarization**: Condense lengthy reports into 1-page executive summaries with key takeaways and action items.
- **Tone Uniformization**: Enforce consistent brand voice across thousands of customer support responses or blog contributors.
- **Readability Compliance**: Adjust content to meet specific grade-level targets (Flesch-Kincaid) for accessibility standards (e.g., 8th grade for government websites).
- **A/B Test Variant Generation**: Produce multiple versions of the same headline or email subject line with varying emotional triggers (curiosity, urgency, benefit).
- **Legal/Compliance Review**: Flag potentially problematic language (guarantees, absolutes) and suggest hedge phrases ("may improve" vs "will improve").

## Scope

### Primary Command
`openclaw ultra-optimizer optimize` — Main optimization pipeline

### Subcommands
- `openclaw ultra-optimizer analyze` — Quality metrics only (no changes)
- `openclaw ultra-optimizer batch` — Process directory of files
- `openclaw ultra-optimizer template` — Apply predefined template (e.g., "blog-post", "email-campaign")
- `openclaw ultra-optimizer diff` — Show unified diff between original and optimized

### Flags (optimize subcommand)
- `--input`, `-i` : Input file path or "-" for stdin
- `--output`, `-o` : Output file path or "-" for stdout (default: overwrite input if not specified with --dry-run)
- `--mode` : Optimization strategy (choices: `seo`, `simplify`, `expand`, `tone`, `grammar`, `style`, `all`) default: `all`
- `--tone` : Desired tone (`professional`, `casual`, `academic`, `persuasive`, `friendly`, `technical`, `executive`) default: inferred from style-guide
- `--audience` : Target reader (`general`, `technical`, `executive`, `beginner`, `student`) default: `general`
- `--max-length` : Maximum word count or character count (suffix w or c, e.g., `500w`, `2500c`) default: no limit
- `--min-length` : Minimum word/char count (e.g., `100w`) default: no limit
- `--keywords` : Comma-separated keywords for SEO (e.g., `"AI writing,content automation,productivity"`)
- `--style-guide` : Style guide to follow (`ap`, `chicago`, `mla`, `custom`) default: `ap`
- `--readability` : Target Flesch-Kincaid grade level (e.g., `8`, `12`) default: preserve original
- `--prompt` : Custom AI prompt template (overrides mode-specific defaults)
- `--model` : OpenAI model ID (e.g., `gpt-4-turbo-preview`, `gpt-3.5-turbo`) default: from env
- `--temperature` : AI creativity (0.0–1.0) default: `0.3` (0 for deterministic)
- `--dry-run` : Show proposed changes without writing output
- `--diff` : Output unified diff instead of full text
- `--json` : Output metrics and result as JSON with keys: `optimized_text`, `original_metrics`, `new_metrics`, `tokens_used`, `model`, `warnings`
- `--backup-original` : Create `.bak` backup before overwriting
- `--skip-grammar` : Disable grammar/spell check (faster)
- `--rate-limit` : Max API calls per minute (overrides env)

### Batch-specific flags
- `--recursive`, `-r` : Process subdirectories
- `--pattern` : Glob pattern (e.g., `"*.md"`, `"**/*.txt"`) default: `*.*`
- `--suffix` : Append to filename (e.g., `--suffix "-optimized"` creates `file-optimized.md`)
- `--fail-fast` : Stop on first error
- `--report` : Generate CSV report of all processed files with metrics

## Detailed Work Process

1. **Argument Parsing & Validation**
   - Parse CLI flags with `argparse`.
   - Validate `--max-length > --min-length` if both set.
   - Confirm input file exists and is readable.
   - If `--output` is same as input and `--backup-original` not set, abort to prevent data loss.
   - Load environment variables: `OPENCL_ULTRA_OPENAI_API_KEY` (required), others with defaults.
   - If `--json`, set `stdout` as output unless `--output` provided.

2. **Input Ingestion**
   - Read file contents (UTF-8) or stdin.
   - Strip null bytes.
   - Detect format: plain text, Markdown (`*.md`), or HTML (heuristic). Preserve formatting tokens for later reinsertion.
   - Split into paragraphs for chunking if length > model context window (e.g., >3000 tokens). Chunking respects paragraph boundaries.

3. **Pre-Analysis**
   - Compute original metrics (if `--json` or `--diff`):
     - Word count, character count.
     - Readability scores: Flesch Reading Ease, Flesch-Kincaid Grade, SMOG Index (via `textstat`).
     - Keyword density if `--keywords` provided.
     - Sentence length variance.
     - Passive voice ratio (via `spacy`).
   - Detect existing tone (heuristic: pronoun types, modal verbs, exclamation count) to compare with requested `--tone`.

4. **Prompt Engineering**
   - Construct system prompt based on `--mode` and `--tone`. Example for `seo`:
     ```
     You are an expert SEO copywriter. Optimize the following text for search engines while maintaining readability. Focus on:
     - Primary keywords: {keywords}
     - Include keywords naturally in headings (H2/H3) and first 100 words
     - Add a compelling meta description (max 160 chars) at the end marked META:
     - Use short paragraphs (max 3 sentences)
     - Target readability: Grade level {readability}
     - Maintain factual accuracy.
     ```
   - Append user content with clear delimiters `[ORIGINAL]` ... `[END]`.
   - If `--prompt` provided, use it directly (must include `{text}` placeholder).
   - Tokenize prompt+content to ensure under `--max-tokens` (model limit minus response buffer). If over, auto-chunk with carryover context.

5. **AI Invocation**
   - Call OpenAI API with `model`, `temperature`, `max_tokens` (default 2000 for optimized text), `top_p=1`, `frequency_penalty=0.1` (reduce repetition), `presence_penalty=0.1`.
   - Implement retry with exponential backoff for rate limits (429) and server errors (5xx).
   - Cache identical requests (hash of prompt+content+params) if `OPENCL_ULTRA_CACHE_ENABLED=true` to reduce cost.
   - Log `prompt_tokens`, `completion_tokens`, `total_tokens`, `model`, `latency_ms`.

6. **Post-Processing**
   - Strip ChatGPT conversational fillers ("Here's the optimized version:").
   - If `--mode seo` and `--keywords` set, extract meta description if present (between `META:` markers), place at top as HTML comment `<!-- ... -->` or separate file if `--output` is specified.
   - Enforce `--max-length`: trim from end of last sentence if exceeds.
   - Recompute metrics on optimized text.
   - If readability target not met, iterate once with adjusted temperature and second prompt (only if `mode=all`).

7. **Output Generation**
   - If `--diff` and original format detected, generate unified diff (`difflib.unified_diff`).
   - If `--json`, serialize all metrics and optimized text to JSON.
   - Else write full optimized text.
   - If `--backup-original` and output file exists, copy to `file.txt.bak` before overwriting.
   - Preserve file modification time if possible.

8. **Verification & Reporting**
   - Print summary to stderr:
     ```
     ✅ Optimization complete
     Original: 1,240 words, Grade 12.5, Passive voice 15%
     Optimized: 1,180 words, Grade 10.2, Passive voice 8%
     Keywords density: 2.1% (target 1-3%)
     Tokens used: 3,450 (prompt 2,100 + completion 1,350)
     Latency: 2.3s
     ```
   - Exit code:
     - 0: success
     - 1: validation error
     - 2: API error
     - 3: processing error (e.g., chunking failed)

## Golden Rules

1. **Never alter facts** — Numbers, dates, names, code snippets, and quotations must remain unchanged. If AI attempts to modify, revert and flag warning.
2. **No hallucinations** — Disallow addition of unsourced examples, statistics, or references. Use `--dry-run` to review if uncertain.
3. **Preserve formatting** — Keep Markdown headers, code blocks, lists intact. Only modify prose within blocks.
4. **SEO keyword bounds** — Density must stay 1–3% for primary keywords. Flag if outside range.
5. **Tone consistency** — Do not mix tones within a document. If document contains both technical and executive sections, process separately with `--mode tone` and merge.
6. **Length constraints** — When `--max-length` is set, cut from the least important section (last paragraph or summary), never from middle of critical explanation.
7. **Style guide precedence** — AP style: serial comma off; Chicago: serial comma on. Enforce aggressively.
8. **Metadata separation** — Meta descriptions, title tags, alt text suggestions must be clearly demarcated (e.g., `META:`, `TITLE:`) and not inserted into body copy.
9. **Cache by default** — Enable `OPENCL_ULTRA_CACHE_ENABLED` to avoid repeated API calls on unchanged content.
10. **Backup before overwrite** — Always use `--backup-original` when processing valuable files. The skill will not prompt; it's the user's responsibility.
11. **API key hygiene** — Never log the API key. Mask in error messages.
12. **Rate limit gracefully** — Respect `OPENCL_ULTRA_REQUESTS_PER_MINUTE`. If exceeded, sleep and retry; do not crash.

## Examples

### Example 1: SEO Blog Optimization
```bash
openclaw ultra-optimizer optimize \
  --input blog-draft.md \
  --output blog-optimized.md \
  --mode seo \
  --tone professional \
  --audience general \
  --keywords "AI writing,content automation,productivity" \
  --readability 10 \
  --backup-original \
  --diff
```

Input (`blog-draft.md`):
```markdown
# How AI Writing Tools Help

Content automation is changing how businesses create content. Many companies use AI tools to write faster.

These tools can help you produce more blog posts and social media updates. They are also good at fixing grammar.
```

Output (`blog-optimized.md`):
```markdown
<!-- META: Discover how AI writing and content automation can transform your productivity. Learn to create better content faster with intelligent tools. -->

# How AI Writing Tools Boost Productivity

**AI writing and content automation** are revolutionizing content creation for modern businesses. By leveraging intelligent tools, companies can scale their content output without sacrificing quality.

## Scale Your Content Efforts

Produce more **blog posts** and **social media updates** in less time. AI-driven platforms assist at every stage, from ideation to final polish.

## Improve Accuracy

Beyond speed, these tools excel at catching grammar errors and suggesting clearer phrasing. The result: professional, error-free content that resonates.

*Ready to boost your productivity? Explore top AI writing tools today.*
```

### Example 2: Simplify Technical Documentation
```bash
cat api-docs.txt | openclaw ultra-optimizer optimize --mode simplify --tone friendly --audience beginner > friendly-guide.txt
```

Input (`api-docs.txt`):
```
The `/v1/users` endpoint accepts POST requests with JSON payload containing `email` and `password`. Upon successful authentication, it returns a JWT token in the Authorization header. Rate limit: 100 requests per minute per IP.
```

Output (`friendly-guide.txt`):
```
Want to create a new user? Send a POST request to `/v1/users`. Include the user's email and password in JSON format. If the email isn't already taken, you'll get back a special login token called a JWT. Look for it in the Authorization header. Remember: you can only call this 100 times per minute.
```

### Example 3: Batch Processing with Report
```bash
openclaw ultra-optimizer batch --input ./marketing-copy/ --pattern "*.txt" --recursive --mode tone --tone persuasive --suffix "-persuasive" --report optimization-report.csv
```

Generates `email-persuasive.txt`, `landing-persuasive.txt`, etc., and `optimization-report.csv` with columns: `file,original_words,optimized_words,readability_improvement,tokens_used,status`.

### Example 4: Analyze Only (No Changes)
```bash
openclaw ultra-optimizer analyze --input whitepaper.pdf --readability 12 --keywords "machine learning,neural networks,deep learning"
```

Output:
```json
{
  "file": "whitepaper.pdf",
  "original_metrics": {
    "word_count": 5420,
    "flesch_reading_ease": 42.1,
    "fk_grade": 14.2,
    "passive_voice_percent": 18.5,
    "keyword_density": {"machine learning": 0.8, "neural networks": 0.4, "deep learning": 0.6}
  },
  "warnings": ["Readability grade (14.2) exceeds target 12", "Passive voice > 15%"]
}
```

## Rollback Commands

- `openclaw ultra-optimizer undo <output-file>`: Restores the `.bak` backup if it exists. Removes `.bak` after successful restore.
  ```bash
  openclaw ultra-optimizer undo blog-optimized.md  # restores blog-optimized.md.bak → blog-optimized.md
  ```

- `openclaw ultra-optimizer restore <original> <optimized>`: Explicitly copy original to optimized (manual rollback).
  ```bash
  openclaw ultra-optimizer restore draft.txt optimized.txt
  ```

- If version control is active (`.git` present in parent directory):
  ```bash
  git checkout -- blog-optimized.md  # Reverts to last committed version
  ```

- For batch operations, a `--report` CSV includes `original_path` column; you can script rollback:
  ```bash
  while IFS=, read -r file original_path; do
    cp "$original_path" "$file"
  done < <(tail -n +2 optimization-report.csv)
  ```

- To revert all files in a directory processed with `--suffix`:
  ```bash
  for f in *-persuasive.txt; do
    mv "$f" "${f/-persuasive/}"
  done
  ```

## Verification Steps

After optimization, run:

1. **Check exit code**: `echo $?` should be `0`.
2. **Validate non-empty output**: `test -s optimized-file && echo OK`.
3. **Readability check** (if `--readability` set):
   ```bash
   python -c "import textstat; print(textstat.flesch_kincaid_grade(open('optimized-file').read()))"
   ```
   Should be within ±1 of target.
4. **Keyword density verification** (if `--keywords`):
   ```bash
   words=$(tr -cs '[:alnum:]' '\n' < optimized-file | tr '[:upper:]' '[:lower:]' | sort | uniq -c)
   echo "$words" | grep -E "ai|writing|automation"
   ```
5. **Diff review** (if `--dry-run` not used):
   ```bash
   openclaw ultra-optimizer diff original.txt optimized.txt | less
   ```
6. **AI hallucination scan**: Search for added numbers/quotes not in original. Automated check: `grep -E '[0-9]{4}' optimized-file | grep -v -F -f <(grep -E '[0-9]{4}' original.txt)` should return empty.
7. **Format preservation**: Ensure no code blocks were corrupted. Manual spot-check of markdown headers (`^#+ `).

## Troubleshooting

| Error | Likely Cause | Fix |
|-------|--------------|-----|
| `Missing OPENCL_ULTRA_OPENAI_API_KEY` | Env var not set | Export: `export OPENCL_ULTRA_OPENAI_API_KEY="sk-..."` |
| `Rate limit exceeded` | Too many calls/min | Add `--rate-limit 30` or increase wait time. Cache helps. |
| `context length exceeded` | Input too large for model | Use `--mode grammar` (processes per-chunk) or split manually. |
| `Optimization produced nonsense` | Temperature too high (≥0.7) | Set `--temperature 0.2` for deterministic rewriting. |
| `Keywords not inserted` | Wrong `--mode` or missing `--keywords` flag | Use `--mode seo` and ensure keywords list non-empty. |
| `Output truncated` | `--max-length` too small | Increase limit or remove flag. |
| `Permission denied` | Output file unwritable | Use `--dry-run` first; check file write permissions. |
| `JSON decode error` | Corrupted cache file | Delete `~/.cache/openclaw/ultra-optimizer/` or set `OPENCL_ULTRA_CACHE_ENABLED=false`. |
| `Spacy model not found` | Language model missing | Run `python -m spacy download en_core_web_sm`. |
| `Empty output` | Input contained only formatting tokens | Ensure input has prose; skip files with <50 words using `--min-length`. |

## Advanced Usage

### Custom Prompt Template
```bash
openclaw ultra-optimizer optimize \
  --input article.txt \
  --prompt "Rewrite for a {audience} audience using {tone} tone. Keep jargon to minimum. Include {keywords} naturally. End with a call-to-action." \
  --audience beginner \
  --tone friendly \
  --keywords "solar energy,home panels,installation"
```

### Preserve Original With Timestamp
```bash
openclaw ultra-optimizer optimize \
  --input manuscript.txt \
  --output "manuscript-$(date +%Y%m%d-%H%M).txt" \
  --backup-original
```

### JSON Metrics for CI Integration
```bash
result=$(openclaw ultra-optimizer optimize --input README.md --json)
fk_grade=$(echo "$result" | jq -r '.new_metrics.fk_grade')
if (( $(echo "$fk_grade > 12" | bc -l) )); then
  echo "Readability too complex for general audience" >&2
  exit 1
fi
```

### Process Only Files That Need Improvement
```bash
openclaw ultra-optimizer analyze --input *.md --readability 9 > analysis.json
jq -r '.[] | select(.original_metrics.fk_grade > 10) | .file' analysis.json |
  xargs -I{} openclaw ultra-optimizer optimize --input {} --mode simplify --tone friendly
```

## Dependencies & Setup

```bash
# Install system deps (Ubuntu)
sudo apt-get install -y python3-pip build-essential

# Install Python deps
pip3 install openai textstat spacy nltk markdown jinja2

# Download spacy model
python3 -m spacy download en_core_web_sm

# Set API key
export OPENCL_ULTRA_OPENAI_API_KEY="your-key-here"

# Verify installation
openclaw ultra-optimizer --version
```

**Minimum OpenAI Account**: Requires paid API credits; GPT-3.5-turbo is cheaper but less capable. GPT-4 recommended for complex tone/style tasks.

## Notes

- The skill does **not** store content on its servers; all processing is via OpenAI API subject to their data policy. For sensitive data, use `--dry-run` and review prompts.
- Caching is keyed by SHA256 of (prompt + content + flags). Cache stored in `~/.cache/openclaw/ultra-optimizer/`. Safe to delete.
- For very large batches (1000+ files), use `--rate-limit` and consider `--model gpt-3.5-turbo` to reduce cost.
- When processing Markdown, code blocks (triple backticks) are excluded from optimization to prevent corruption.
- The skill respects `.openclawignore` patterns; files matching patterns are skipped.

---

This document describes the **Ultra Optimizer** skill as implemented in OpenClaw v1.2.1. For updates, see `docs/CHANGELOG.md`.
```