---
name: dd-setup-guide
description: >
  Generate a bespoke, documentation-verified Datadog setup and configuration guide for a specific customer
  environment. Use this skill when a user wants to plan, set up, configure, or deploy any Datadog product
  and needs a tailored, accurate configuration plan. Trigger on: "set up Datadog", "configure Datadog",
  "Datadog setup guide", "Datadog config plan", "how do I install Datadog", "create a Datadog setup plan",
  "Datadog onboarding", "Datadog deployment guide". Always prefer this skill over ad-hoc answers when the
  user wants a written plan or guide they can act on.
user-invocable: true
---

# Datadog Setup Guide Skill

Produces a bespoke, documentation-verified Datadog configuration plan for a specific customer environment.
Every configuration claim, code block, and compatibility statement is sourced directly from the official
Datadog documentation GitHub repository — not from memory.

**Source of truth:** https://github.com/DataDog/documentation (`content/en/` branch: master)

---

## Phase 1 — Product Scoping

Ask the user which Datadog products they want to configure. Present this list so they can pick:

> Which Datadog products are in scope for this guide? Pick all that apply:
>
> - APM & Distributed Tracing
> - Log Management
> - Infrastructure Monitoring / Metrics
> - Real User Monitoring (RUM)
> - Session Replay
> - Synthetics
> - Database Monitoring (DBM)
> - Network Performance Monitoring
> - Network Device Monitoring
> - Cloud Security Management (CSM)
> - Application Security Management (ASM)
> - Cloud SIEM
> - CI Visibility / Test Optimization
> - Serverless Monitoring
> - Container Monitoring
> - OpenTelemetry
>
> Also: what name should I use for this customer in the output file?

One ask. Wait for the reply before proceeding.

---

## Agent Tool Usage — Fetch Strategy

All documentation fetches MUST use **synchronous (foreground) Agent calls**. Never use
`run_in_background: true` for any doc fetch task — background agents stall silently for 10+ minutes
with no error surfaced, blocking the entire workflow.

When multiple independent fetches are needed at the same phase, issue them as **parallel foreground
Agent calls in a single message** (multiple Agent tool use blocks). This gives full parallelism
without the silent-stall risk:

```
# CORRECT — parallel foreground calls in one message
Agent(fetch APM docs, foreground)
Agent(fetch Infra docs, foreground)

# WRONG — background agents for HTTP fetches
Agent(fetch APM docs, run_in_background=true)   ← never do this
```

Each Agent call should fetch 3–5 URLs maximum. Batching too many URLs in one agent increases the
chance of partial failures.

---

## Phase 2 — Documentation Discovery

For each selected product, discover what the documentation actually says is supported — do not rely on
memory or training data.

### Step A — List the product directory

For each product, look up its **Directory API URL** from `references/products.md` and fetch it. This
returns a JSON array of files and subdirectories. Read it and note the subdirectory names — these map
directly to the feature areas and deployment paths Datadog has documented.

Example for APM:
```
GET https://api.github.com/repos/DataDog/documentation/contents/content/en/tracing
```
Returns entries like: `trace_collection/`, `trace_pipeline/`, `compatibility/`, `guide/`, etc.

### Step B — Read the product `_index.md`

Fetch the raw entry point for the product from `references/products.md`. This file gives an overview
of the product and typically links to major subsections and deployment paths.

Example:
```
GET https://raw.githubusercontent.com/DataDog/documentation/master/content/en/tracing/_index.md
```

### Step C — Drill into key subdirectories

Based on what you found in Steps A and B, fetch the directory listing and `_index.md` for the
subdirectories most relevant to deployment and configuration. For APM this would be:
- `trace_collection/` — to discover installation paths by environment and language
- `trace_collection/compatibility/` — to discover which runtimes and versions are supported

For logs this would be `log_collection/`, for infrastructure it would be `configuration/`, etc.
Use your judgement about which 2-3 subdirectories are most likely to contain the deployment options.

### Step D — Surface what you found

Present your findings to the user in plain language. For each product, describe what you actually
read from the docs — the environments, runtimes, install methods, and other dimensions that branch
the setup path:

> **APM & Distributed Tracing** — Reading the documentation I can see the following:
>
> **Deployment environments:**
> - Kubernetes (with Datadog Admission Controller auto-injection, or manual sidecar)
> - ECS Fargate
> - ECS EC2
> - Bare metal / VM
> - AWS Lambda (Serverless APM)
> - Azure Container Apps
>
> **Language runtimes with auto-instrumentation support:**
> Java, Python, Ruby, Go, Node.js, .NET, PHP, C++, Swift
>
> Which environment(s) and runtime(s) apply to this customer?

Batch all products into a single message. Do not ask one product at a time.

Wait for the user's reply before proceeding to Phase 3.

---

## Phase 3 — Targeted Documentation Fetch

Now that you know the exact product + environment + runtime combinations, fetch the specific markdown
files that cover each path. Do not re-fetch top-level pages from Phase 2.

### Finding the right files

Use directory listing calls to navigate to the right subdirectory, then fetch the specific `.md` files.
Follow this approach for each combination:

1. Fetch the directory listing for the most specific subdirectory that matches the combination.
   Example for APM / Java / Kubernetes:
   ```
   GET https://api.github.com/repos/DataDog/documentation/contents/content/en/tracing/trace_collection/automatic_instrumentation
   ```
2. Identify the most relevant files from the listing (look for filenames matching the environment or runtime).
3. Fetch the raw markdown for each:
   ```
   GET https://raw.githubusercontent.com/DataDog/documentation/master/content/en/tracing/trace_collection/automatic_instrumentation/dd_libraries/java.md
   ```

Fetch at minimum:
- The install / setup guide for the specific environment
- The configuration reference (env vars, config file keys, tracer options)
- The compatibility / supported versions file if it exists separately

### Hugo shortcodes

The markdown files use Hugo shortcodes like `{{< tabs >}}`, `{{< code-block >}}`, `{{< img >}}`.
Read through these — they contain real content. Tab blocks typically show environment-specific variants
(Helm vs Operator, Docker vs VM, etc.). Extract the relevant tab for the customer's environment.

### Hugo partials — treat as UNVERIFIED

Some content is served via Hugo partials, e.g.:
```
{{< partial name="trace_collection/python/supported_runtimes.html" >}}
```
These partials render dynamically and their source is NOT in the raw markdown. You cannot read this
content from GitHub. When a doc page uses a partial for a table, version list, or configuration block:
- Do NOT invent or guess the content of the partial
- Mark the corresponding claim in the plan as `⚠️ UNVERIFIED` and add it to Open Questions
- Note the partial name so it can be looked up manually

### Build internal notes

For each product/environment/runtime combination, record:

```
Product: APM / Java / Kubernetes (Admission Controller)
Files fetched:
  - https://raw.githubusercontent.com/.../tracing/trace_collection/automatic_instrumentation/dd_libraries/java.md
  - https://raw.githubusercontent.com/.../tracing/trace_collection/single-step-apm/_index.md
Config keys found (exact names as in docs):
  - DD_ENV, DD_SERVICE, DD_VERSION (unified service tagging — required)
  - DD_TRACE_AGENT_URL (optional — only if agent is not on localhost:8126)
  - [full list from the docs, not abbreviated]
Supported versions: Java 8u40+, Java 11+, Java 17+ (exact wording from compatibility page)
Prerequisites:
  - Datadog Agent 7.x running and reachable
  - Admission Controller feature enabled
Known limitations / caveats:
  - [exact text from docs]
Deprecated options found:
  - [anything marked deprecated in the docs]
```

These notes are internal. Do not show them to the user. They are the raw material for Phase 4.

If a file returns 404 or an error, note it and try an alternative path. If still unreachable, flag it
as an open question rather than guessing at the content.

---

## Phase 4 — Plan Generation

Write the configuration plan to a file named:
`dd-setup-<customer-name>-<YYYY-MM-DD>.md`

Use today's date. Use a slugified customer name (lowercase, hyphens, no spaces).

### Output file structure

```markdown
# Datadog Setup Guide — <Customer Name>
Generated: <date>
Products in scope: <comma-separated list>
Environment summary: <one-line description of what was scoped in Phase 2>

---

## Prerequisites

[Agent version requirements, network connectivity, IAM/RBAC, firewall rules — sourced from docs]

---

## <Product Name> — <Environment> / <Runtime>

### Compatibility
[Supported versions, runtimes, OS requirements — exact from docs, not paraphrased]

### Installation
[Step-by-step with exact commands — sourced from docs]

### Configuration
[Config blocks with exact key names, types, and documented defaults]

### Validation
[Commands or UI steps to confirm the product is working]

---

[Repeat for each product/environment/runtime combination]

---

## Open Questions
[Points that could not be resolved from docs or user input]

---

## Amendment Log
[Populated by Phase 5 — leave blank]
```

### Rules for writing the plan

- **Source every claim.** Add an HTML comment after each configuration block, version statement, or
  non-obvious instruction:
  `<!-- source: https://raw.githubusercontent.com/DataDog/documentation/master/content/en/... -->`
  Use the raw GitHub URL so it can be re-fetched exactly in Phase 5.
- **Use exact key names.** Copy them verbatim from the markdown. Do not normalise case, expand
  abbreviations, or rename anything.
- **Do not state undocumented defaults.** Only include default values that appear explicitly in the
  fetched files.
- **Do not include steps that aren't in the docs for this specific environment.** If a step appears
  in a different tab or a different runtime's guide, do not include it unless the user's environment
  matches.
- **Ambiguity → Open Question.** If the docs are unclear or contradictory on a point, write it as an
  open question rather than picking one interpretation.

After writing the file, tell the user the filename and that Phase 5 (verification) is starting now.

---

## Phase 5 — Verification Pass

This phase is mandatory. Do not skip it, do not summarise it away.

Re-read the written file section by section. For each section, re-fetch the raw GitHub source URL
from the HTML comment and check the following against what the file actually says:

### What to verify

**Configuration keys**
- Is every key name character-for-character identical to the documentation? (`DD_TRACE_AGENT_URL` not
  `DD_TRACE_AGENT_HOSTNAME`, `DD_LOGS_ENABLED` not `DD_LOG_ENABLED`)
- Are value types correct? (boolean `true`/`false` vs string `"true"`, integer vs string)
- Are all required keys present? Check if the doc lists any as mandatory.
- Are any keys in the plan marked as deprecated or removed in the doc?
- Are there additional required keys in the doc that the plan omitted?

**Version and compatibility claims**
- Are version range boundaries exact? (`8u40` not `8.0`, `7.x` not `7+`)
- Are there OS, architecture, or cloud-provider constraints the plan didn't capture?

**Install steps**
- Are steps in the correct order relative to the documentation?
- Are any prerequisite steps missing?
- Are there environment-specific warnings or caveats the plan omitted?

**Hugo shortcode tabs**
- Did the plan capture the right tab for the customer's environment? (e.g., Helm tab not Operator tab
  if the customer is using Helm)

**Config syntax format (Helm vs Datadog Operator)**
- YAML blocks in the docs often use `datadog:` as the top-level key — this is **Helm values syntax**
  and is NOT valid in a `DatadogAgent` CRD manifest used with the Datadog Operator.
- For every YAML config block in the plan, verify which install method it applies to. If the customer
  is using Datadog Operator but the source doc only shows Helm syntax (`datadog:` prefix), flag it:
  `> ⚠️ This config block is in Helm values syntax. Datadog Operator CRD equivalent not confirmed.`
- Specifically watch for: `kubernetesResourcesLabelsAsTags`, `clusterAgent.useHostNetwork`,
  `datadog.apm.instrumentation`, and any block prefixed with `datadog:` at the top level.

### What to do with findings

For each issue found:
1. Fix it in the file in-place.
2. Log it in `## Amendment Log`:
   ```
   - [APM / Configuration] DD_TRACE_AGENT_HOSTNAME → DD_TRACE_AGENT_URL (source: <raw url>)
   - [APM / Compatibility] "Java 8+" corrected to "Java 8u40+" (source: <raw url>)
   ```

For claims that cannot be verified because the source URL returned an error or the content was
ambiguous:
- Mark inline: `> ⚠️ UNVERIFIED: <claim> — source unavailable or ambiguous`
- Add to `## Open Questions`.

### After the verification pass

Report to the user:
- Number of amendments made
- Final file path

Then immediately proceed to Phase 6 if there are any unverified items. If there are none, proceed
directly to Phase 7.

---

## Phase 6 — Unverified Item Resolution

If any items were marked `⚠️ UNVERIFIED` during Phase 5, do not leave them for the user to find on
their own. Work through them one at a time before declaring the guide complete.

### Resolution flow

For each unverified item, in the order they appear in the document:

1. **Present the item clearly** to the user:

   > **Unverified item (N of M):** [section name]
   >
   > The plan states: [exact claim from the document]
   >
   > This could not be verified because: [reason — e.g., content is in a Hugo partial, source URL
   > returned 404, doc only shows Helm syntax and customer is using Operator, etc.]
   >
   > To resolve this, I can:
   > - **Try an alternative source** — attempt to find the content at a different path or in a
   >   related file (describe the path you'll try)
   > - **Ask you directly** — if this is something you can answer from your environment (e.g.,
   >   which Python version is running, which CNI is in use)
   > - **Leave it flagged** — keep the `⚠️ UNVERIFIED` marker and Open Question as-is
   >
   > How would you like to proceed?

2. **If the user chooses "try an alternative source"** — fetch it synchronously and report what you
   find. If the content resolves the claim: update the plan in-place, remove the `⚠️ UNVERIFIED`
   marker, add an entry to the Amendment Log, and move to the next item.

3. **If the user provides the answer directly** — update the plan with their input, note it as
   user-confirmed (not doc-sourced) with an inline comment:
   `<!-- confirmed by customer: <their answer> -->`, remove the `⚠️ UNVERIFIED` marker, and move
   to the next item.

4. **If the user chooses "leave it flagged"** — leave the marker and Open Question unchanged and
   move to the next item.

After all unverified items have been addressed, report the final tally and proceed to Phase 7.

---

## Phase 7 — Standalone Configuration File Generation

After the guide is written and verified, generate the standalone configuration files an engineer
would actually deploy. These are separate from the guide — they are the real artefacts that get
committed to a repo or applied to a cluster.

### Step A — Identify required files

Based on the products, install method, and environment scoped in Phases 1–3, determine which
configuration files are needed. Common files include:

| File | When needed |
|------|-------------|
| `datadog-agent.yaml` | Datadog Operator install — the `DatadogAgent` CRD manifest |
| `datadog-values.yaml` | Helm install — values file passed to `helm install` |
| `datadog.yaml` | Agent main config — used for non-containerised or VM-based installs |
| `datadog-secret.yaml` | Kubernetes Secret manifest for the API key (optional, if not using `kubectl create secret`) |

Do not generate files for install methods the customer is not using. If the customer chose Datadog
Operator, generate `datadog-agent.yaml` — not `datadog-values.yaml`.

Present the list to the user before generating:

> I'll generate the following configuration files based on your setup:
> - `datadog-agent.yaml` — DatadogAgent CRD for Datadog Operator
> - `datadog-secret.yaml` — Kubernetes Secret for your API key
>
> Shall I proceed, or would you like to add or remove any files from this list?

Wait for confirmation before writing files.

### Step B — Generate each file

For each file, write it to the working directory alongside the guide document. Use the naming
convention: `<filename>` (no customer name prefix — these are deploy-ready files, not reference docs).

**Rules for generated config files:**

- **Complete and deploy-ready.** Every required field must be present. Do not use placeholder
  comments like `# add your config here`. Use `<REPLACE_ME: description>` tokens for values the
  customer must supply (API key, cluster name, site, namespace names).
- **Source every value from the verified plan.** Do not add keys that weren't in the plan. If a
  key appeared in the plan as `⚠️ UNVERIFIED`, omit it from the generated file and add a comment
  noting it was excluded pending resolution.
- **Correct syntax for the install method.** Datadog Operator files use the `DatadogAgent` CRD
  schema (top-level `spec:`). Helm values files use the `datadog:` prefix. Never mix the two.
- **Include a header comment** in every generated file:

  ```yaml
  # Generated by dd-setup-guide — <date>
  # Guide: <guide filename>
  # WARNING: Replace all <REPLACE_ME: ...> tokens before applying.
  # Source: values derived from Datadog documentation as cited in the guide.
  ```

- **One file per write call.** Write each file separately so the user can review them individually.

### Step C — Report and offer next steps

After writing all files, list what was generated:

> **Configuration files generated:**
> - `datadog-agent.yaml` — DatadogAgent CRD, ready to apply after replacing tokens
> - `datadog-secret.yaml` — Kubernetes Secret for API key
>
> **Tokens to replace before deploying:**
> - `<REPLACE_ME: Datadog API key>` — in `datadog-secret.yaml`
> - `<REPLACE_ME: cluster name>` — in `datadog-agent.yaml`
> - `<REPLACE_ME: Datadog site, e.g. datadoghq.com>` — in `datadog-agent.yaml`
>
> **What would you like to do next?**
> - **Review a generated file** — I'll walk through each field
> - **Fill in the tokens** — tell me the values and I'll update the files
> - **Add another product** to this guide
> - **Add a second environment** variant

Wait for direction.

---

## Tone & Style Guidelines

- Write for SREs, platform engineers, and DevOps. Skip hand-holding, keep it dense and precise.
- Use exact Datadog terminology from the documentation. Do not invent synonyms or shorthand.
- Be explicit about required vs optional vs recommended for every config key.
- When the docs document two approaches (e.g., Helm vs Operator, agent-based vs agentless), present
  both briefly and note which applies based on the user's Phase 2 answers.
- Flag deprecations and version-specific behaviour prominently — these are high-value signals.
- The goal: a document an engineer can open on Monday morning and execute without ambiguity.
