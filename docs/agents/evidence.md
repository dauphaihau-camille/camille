# Evidence

Verification proves the code passes its checks. Evidence proves the change does
what was asked. They are different claims, and a change is not finished until
the one its class requires exists.

This file owns what evidence is required, where it lives, and how it is
published. The PR **body** shape is owned by the `pr` skill and
`.github/pull_request_template.md`; do not restate capture commands there.

Run the evidence phase after the last edit and after `./scripts/verify` is
green, never before.

## Required evidence, by change class

Gate on what the diff touches, not on how large it feels.

| Diff touches | Required evidence |
| --- | --- |
| `apps/web/apps/**`, `apps/web/packages/shared/src/components/**` | screenshot pair of the actual surface |
| `apps/api/api/src/domains/**` | request/response transcript plus relevant server log lines |
| `openapi.json`, `api-spec` | `./scripts/verify spec` and `./scripts/verify api-types` output, plus the `openapi.json` diff |
| a behavior test added or changed | failing-before output, then passing-after output |
| a perf-motivated change | before/after numbers from a micro-benchmark or scaled-down input |
| a database migration | applied migration log and resulting row state |
| `docs/**` only | none; `./scripts/verify` is skipped too |

If a UI change causes no visual difference, that is a claim: state it in the
manifest instead of attaching an identical pair.

## Where evidence lives

One directory per change, outside the repository:

```text
~/.cache/camille/evidence/<YYYY-MM-DD>-<branch>-<slug>/
├── MANIFEST.md
├── before.png
├── after.png
├── post-document.http
└── server.log
```

`MANIFEST.md` is the entrypoint and the report. One line per artifact naming the
claim it proves:

```text
before.png        -> toolbar overflows at 1280px
after.png         -> toolbar fits at 1280px
post-document.http -> POST /documents returns 201 with an id
```

Rules:

- Never commit artifacts, and never add them under the repository.
- Never use `$TMPDIR`: it changes every boot and is purged.
- Keep the directory until the pull request is merged.

## Capturing

### UI

Playwright already captures traces on first retry
(`apps/web/apps/app/playwright.config.ts:9`). For a change that needs visual
proof, override it for the run:

```bash
cd apps/web/apps/app
bunx playwright test --trace on --grep "<spec>"
cp -R test-results "$EVIDENCE/playwright"
```

`test-results/` and `playwright-report/` are gitignored under `apps/web`;
copy them out, do not rely on them in place.

For a surface no spec covers, capture against the running app
(`bun run dev` in `apps/web/apps/app`, `http://127.0.0.1:5102`) with the same
viewport on both sides, and record the viewport size in the manifest.

### API

Capture the whole exchange, including status line and headers:

```bash
curl -i -sS -X POST http://localhost:5100/documents \
  -H 'content-type: application/json' \
  -d '{"title":"evidence"}' | tee "$EVIDENCE/post-document.http"
```

Point at a local or dev instance only. Never capture a mutation from production.

### Logs

```bash
<command> 2>&1 | tee "$EVIDENCE/server.log"
```

Trim the file to the lines that prove the claim. Do not attach a full startup
log.

### Performance

```bash
hyperfine --warmup 3 --export-json "$EVIDENCE/bench.json" "<command>"
```

Report the numbers, not the intent.

### Contract

```bash
./scripts/verify spec
./scripts/verify api-types
git diff -- apps/api/api/openapi.json > "$EVIDENCE/openapi.diff"
```

## Publishing to the pull request comment

Evidence is published as a PR comment, from the same manifest that reports the
change. No second write-up.

`--attach` requires `gh` 2.99.0 or newer; check with `gh --version` before
relying on it.

```bash
gh pr comment <number> \
  -F "$EVIDENCE/MANIFEST.md" \
  --attach "$EVIDENCE/before.png#before: toolbar overflows at 1280px" \
  --attach "$EVIDENCE/after.png#after: toolbar fits at 1280px"
```

- Up to 50 files per command.
- Alt text follows `#`; without it the filename is used.
- Image references in the body resolve relative to the body file's directory and
  are rewritten to the uploaded asset URL.
- Attachments the body does not reference are appended at the end.
- Refresh in place with `--edit-last` rather than adding a second comment.

Logs, transcripts, and diffs stay inside `MANIFEST.md` as fenced text. Only
images and video need `--attach`.

## Non-negotiables

- Text before pixels: use logs and request/response wherever they can prove the
  claim; screenshots are for genuine visual change.
- Same viewport for a before/after pair.
- Local or dev instances only for state-changing calls.
- A change reported in chat names its artifacts; the PR comment is published
  from the same `MANIFEST.md`.
