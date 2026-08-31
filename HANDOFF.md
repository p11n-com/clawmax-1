# Session Handoff — 2026-08-30

## What was done

Fixed: a verified OpenAI-compatible endpoint (vLLM on DGX Spark at
`http://172.16.1.70:8000/v1`) with BYOK's optional "Default model" box left empty was
treated everywhere as having no model, so AI generation refused with
"OpenAI-compatible AI generation requires a default model", and the wizard showed a
readiness warning saying the same.

Commit `5171df3b` — pushed to `fork` (p11n-com/clawmax-1). `origin`
(Maximilien-ai/clawmax) rejected the push: the authenticated `p11n-com` account has no
write access there (403). A PR from the fork is the remaining step if upstream is wanted.

## The change

An endpoint with no typed default model resolves to whichever chat model it advertises —
the same model endpoint validation completes its test prompt on.

- `server/lib/model-discovery.ts` — new `resolveOpenAiCompatibleDefaultModel` (async) and
  `getCachedOpenAiCompatibleDefaultModel` (sync, cache-only). Discovery cache is keyed per
  endpoint AND credential, preserves the endpoint's response order (display sorting moved
  into `discoverModels`), coalesces in-flight `/models` requests, evicts expired entries.
- `server/lib/ai-generator.ts` — `resolveOpenAiCompatibleGenerationDefaults` now selects a
  whole endpoint (base URL + its own key + its own model) instead of merging fields across
  sources. `setRequestByokKeys` stays synchronous; the async warm-up is a separate exported
  `warmOpenAiCompatibleGenerationModel` that routes await first.
- `server/routes/{agents,ai,ai-builder,skills,templates,workflows}.ts` — warm before set.
- `server/routes/chat.ts` — `resolveChatOpenAiCompatibleEndpoint` pairs endpoint/model;
  `warmChatOpenAiCompatibleModel` runs before readiness in both chat routes.
- `server/lib/agent-default-model.ts`, `getAvailableModelsCached` — same fallback.
- `client/src/lib/byok.ts` — a base URL alone counts as a hosted path; the
  "needs a default model in BYOK" warning is gone (unverified endpoints still get the
  existing "not been verified yet" warning).

## Verification performed

Headless (puppeteer-core + system Chrome) against a throwaway container from the same
RC46 image with the local `dist` bind-mounted, on port 3402, pointed at a copy of
`~/.clawmax/workspace`. The live dashboard on 3201 was never touched.

- Wizard step 1 preselects `openai-compatible/deepseek-ai/DeepSeek-V4-Flash-0731`
- No readiness warning; Generate with AI returns 200 with real IDENTITY/SOUL/TOOLS
- Agent provisions on that model; chat readiness `available:true`
- Three prompts answered correctly (marker echo, ConnectX reasoning, 17*23=391)
- Re-run in full after each code revision, on a freshly re-provisioned agent

Test sweep: 287 suites. 5 non-zero exits, all confirmed pre-existing at HEAD —
`lib/ai-builder`, `lib/safe-env`, `lib/skills`, `routes/skills` fail identically at
baseline; `lib/agent-runtime` has one flaky timing test ("a turn cancelled after
streaming keeps its partial text") failing 1/12 at baseline and 1/12 with the change.

Two `/droid-review` passes (GPT-5.6 Sol); all findings verified in code and fixed except
the ones listed below.

## Next steps

1. **`_requestByokKeys` is a module-level global** in `ai-generator.ts` — concurrent
   generations can overwrite each other's BYOK tuple, which undermines the credential
   pairing this commit establishes. Pre-existing and not widened here (the setter is
   synchronous again). Droid's call: convert to `AsyncLocalStorage`, wrapping the seven
   route handlers with `withRequestByokKeys(keys, fn)` the way generation attribution
   already does. Deferred as a separate change.
2. **Group/community chat has no model fallback at all** — `callAgent` in
   `routes/channels.ts` derives `useOpenAiCompatible` from the agent's own provider, so a
   modelless agent discards the endpoint and key entirely. Pre-existing and broader than
   the default-model box.
3. **`generatedBy` is always null** — attribution is recorded in `getAIClient`, but the
   generation path uses `currentClient()` → `createAiGenerationClient`, which never
   records. The wizard therefore cannot show which model wrote the files.
4. Open a PR from `p11n-com/clawmax-1` to `Maximilien-ai/clawmax` if upstream is wanted.
