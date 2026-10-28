# JARVIS AlphaCut — Plan framework

Home: project-plans, jarvis-alphacut/framework.md. Decisions log: jarvis-alphacut/decisions.md. Once Gordon merges this file it is authoritative; the plan (jarvis-alphacut/plan.md) is derived from it alone.

Audience: Cursor, which designs the actionable plan from it. Short, checkable rules and a phase outline; not the plan: no task lists, no specific machines.

Conflict rule: a contradiction in this framework, or a rule that blocks an exit check, means stop and ask Gordon (§11).

## 1. Scope

### Goal

- A re-imagined JARVIS, built clean from Git on blank nodes. Nothing carries over from the old cluster: no data, no config, no home-grown tools.
- First build = Phases 0–3. Phase 4 (code-factory) is part of this plan: it starts only after Phase 2’s exit check passes and is live by day 90 after P0 starts.
- Public LLMs first: Hermes calls public model APIs directly. Local models come later as another OpenAI-compatible provider.
- Hermes is the star: one Hermes core runs the personal operations assistant and the code-factory as two separate deploys.
- Engineering bar: the stack rebuilds from Git on one CPU-only node and scales out by config; each service has one job and a written contract; coding agents ship small changes end to end without waiting on Gordon for routine work; every “done” has evidence.

### Capabilities and 90-day checks

- NOC (P1): Grafana shows node, pod and disk metrics and logs; the Gatus status page loads without login on LAN and tailnet; failures reach Gordon on Telegram through the alerts bot within 5 minutes, with no dependence on Hermes.
- Chat assistant (P2): Gordon chats through the Hermes dashboard and Telegram (concierge is the only Telegram-facing bot at first); quick questions that need a tool (weather, distance) get correct answers.
- Daily operations assistant (P3): the morning brief (calendar, email, weather, stuck updates) arrives on schedule every day, and the evening wrap and weekly review on theirs; inbox triages email in chat and drafts replies on request without sending; read skills for Gmail, Calendar and GitHub PRs pass their evals.
- Code-factory (P4a app hosting, P4b issue to PR): a code-factory:ready issue becomes a draft PR in jarvis-apps with a green CI check and a checker check run; one demo app runs isolated and is reachable on LAN or tailnet.
- Memory (P2): a fact told on day 1 is recalled on day 2; “forget that” removes it; a memory export produces a file.

### Starter bots

- concierge (ops): the front door and the only Telegram-facing bot (R21); owns the starter scheduled jobs: the morning brief, evening wrap and weekly review.
- inbox (ops): email triage (summarizes and ranks the unread inbox threads in chat only, applies no Gmail labels) and reply drafts on request; never sends; fewest tools (R15).
- homelab (ops): reads the cluster, Flux and metrics, runs the scheduled “watch for stuck updates” check, and proposes fixes in chat only (R15).
- foreman (code-factory): watches the code-factory queue and reports; never codes (R40).
- spec-writer (code-factory): turns ideas into issues with acceptance criteria (R40).
- builder and checker: the two lanes, run in the sandbox (R42).

### Users

- Gordon only, as admin, at launch. Multi-user later means one Hermes instance per user; any custom code keys stored data by a stable user ID (the OIDC subject once a second user exists).

### Non-goals

- Migrating data or config from the previous system, carrying over its tools, reading or copying from its archived repos, or feature parity with it.
- Designing around specific machines. No HA on env-01.
- Heavy security hardening. Small, simple wins only.
- A bot team running the build. Coding agents build; bots run JARVIS.
- A custom UI. This plan uses the Hermes dashboard, Telegram, Grafana and Gatus as they come; a custom UI is a separate, later plan.
- A fork of Hermes.
- Anything on the Deferred list (§9) beyond its named seam: no folders, CRDs or code for deferred items.
- Earlier tooling, keys or guards on the bastion: the build uses a fresh agent user and new tokens only.

### Glossary

- Hermes: Nous Research Hermes Agent, the agent runtime (not the model family).
- Hermes core: the upstream Hermes image (each deploy overlay pins its own digest, R33), plus our Kustomize base, default config, shared approved skills and small glue (profile render and sync). Not a fork, not a custom image.
- Deploy: one running Hermes instance built from the core. Two exist: ops and code-factory.
- Hermes bot: a named Hermes profile inside a deploy (its own SOUL.md, model tier from the env’s bot tier map, tools, skills and memory).
- Coding agent: Cursor agents that build JARVIS from the plan. They run on the bastion as its agent user and the gordon5417-ai GitHub account, never Gordon’s.
- Bastion: Gordon’s LAN control host, not a cluster node, and env-01’s controller; it also runs the coding agents. Gordon’s user runs the owner runs (§8 step 4); a separate agent user runs the coding agents with the kubeconfig, no age private key and no access to the previous system’s nodes.
- Controller: the host that runs make, Ansible and kubectl for an env. The repo says controller, never bastion.
- Env: one deployment target under envs/ (Example: env-01, the one-node baseline).
- Gordon (admin): the system’s admin and only human approver. Agents merge routine green PRs (R36); only he merges code-factory PRs, PRs held by admin-gate, and project-plans changes. [G] marks exit checks only he can verify; admin-approved (the label from his own account) and admin-gate (the check that requires it) are the code-level names for his approval.
- Gordon-only action: a step a coding agent cannot or must not do (§8).
- admin-approved: a PR label that counts only when Gordon’s own GitHub account added it; every new push removes it; the admin-gate check requires it on risky PRs (R36).
- Checkpoint: a sub-phase with its own exit checks (P0a, P0b, P4a, P4b).
- Lane: a sandboxed coding-agent run, either builder or checker, started by the code-factory workflow (§5).
- Code delivery provider: the §5 contract for repos, CI, the issue queue, PRs, the approval gate and the registry; GitHub on env-01.
- Build agent: the §5 contract for the coding agent a lane runs, through a thin adapter script; Cursor on env-01.
- Alerts bot: the send-only Telegram bot Gatus and Grafana use (R8, R21).

## 2. Phases and exit checks

Each phase ends with a working, checked system. Cursor may refine phases; the exit checks stay. Every check is provable by a command, URL or test output, and the live run is the proof where there is one. Agents run make verify PHASE=<phase> (tools/verify/<phase>, a short live-state check outside tools/checks/) on the controller and attach its output (R35). [G] marks a check only Gordon can verify (tailnet reachability, phone or Telegram delivery). [G] checks and Gordon-run steps are the owner checklist in the phase’s verify task, never a pass in the script (it may list them as blank lines to fill in); Gordon pastes their results with the output, and his review is the gate. Each phase’s “Gordon first” line points to that phase’s group in the §8 Gordon-only actions, which its briefs wait on. Every CI rule check is proven by a seeded bad PR that breaks a whole rule class, and lands with the first thing it guards.

### P0 — Foundation

Repos, CI rule checks, node bootstrap, cluster, storage, ingress and TLS, local DNS, GitOps, secrets, local backups, dependency updates, on env-01. LAN and Tailscale access is router-owned and not built here (§8). Order of operations: §8. Two checkpoints, P0a then P0b.

Gordon first: §8 Gordon-only actions, P0 (the router and root-cert steps fall inside P0b, per the Phase 0 order).

#### P0a — GitOps baseline

Goal: Git drives the cluster from a blank node, with CI rule checks, secrets and Renovate working.

Contracts first (§5): Env schema, Code delivery provider.

Gordon first: §8 Gordon-only actions, P0 (see above).

Done: make verify PHASE=p0a output attached.

- From a blank node, every Flux object reports Ready; a second run of bootstrap-secrets and site.yml changes nothing.
- A SOPS-encrypted secret decrypts in-cluster.
- One required check on main, ci, runs every tools/checks/ rule check (admin-gate joins at the P2 exit, R36), and a seeded bad PR fails each rule check as it lands (Examples: a hardcoded DNS suffix, a plaintext secret, a missing CONTRACT.md, an unknown code_delivery or build_agent value, a deny list missing a whole deny class).
- Renovate is installed and has opened at least one PR (its onboarding PR counts).
- Rollback drill: reverting a commit restores the prior state.

#### P0b — Reachable and recoverable

Goal: services answer over HTTPS by name on LAN and tailnet, and a volume restores.

Contracts first (§5): Exposure, Backups.

Gordon first: §8 Gordon-only actions, P0 (see above).

Done: make verify PHASE=p0b output attached.

- A test app answers over HTTPS on <svc>.<DNS suffix> (env-01: <svc>.alpha.lan) with a cert from the internal CA, from LAN and from tailnet [G], through Traefik Ingress.
- Restore drill: a deleted volume is restored by VolSync (or the restic CronJob fallback, per the Phase 0 test) and its checksum matches; an etcd snapshot exists for same-node rollback (R31).

### P1 — See the system

Metrics, logs, dashboards, synthetic checks and alerts. None of it depends on Hermes.

Gordon first: §8 Gordon-only actions, P1.

Goal: Gordon sees the system and gets alerts without Hermes.

Contracts first (§5): Login, Alert path.

Done: make verify PHASE=p1 output attached.

- Grafana shows node, pod and disk metrics, and logs for every namespace.
- Gatus checks every exposed endpoint; its status page (env-01: gatus.alpha.lan) loads without login on LAN and on tailnet [G].
- Killing a test endpoint sends a Telegram alert within 5 minutes; the disk, node-memory and clock-not-synchronised alert rules fire in a test with thresholds lowered.
- A forced backup failure (VolSync or the CronJob fallback) sends a Telegram alert.

### P2 — Talk to JARVIS

Hermes core and the ops deploy with concierge, inbox and homelab; chat through the Hermes dashboard (all three bots) and Telegram (concierge only); weather and maps read skills; built-in memory; golden-task evals.

Gordon first: §8 Gordon-only actions, P2.

Goal: Gordon chats with the ops bots through the dashboard and Telegram, with memory and evals.

Contracts first (§5): Model provider, Assistant API, Skill contract, Profile schema, Login.

Done: make verify PHASE=p2 output attached.

- A scripted prompt through the assistant API (/api/<profile>/v1) to each of the three bots gets a non-error answer; a Telegram message from Gordon’s ID to concierge gets a reply [G], and one from another ID gets none.
- Weather and distance questions get correct answers through the geo MCP server (weather: Open-Meteo; distance: Nominatim plus the OSRM demo, typical drive time) with attribution; the geo tool’s evals in concierge pass.
- Memory: a fact is recalled after a pod restart in a new session; “forget that” is staged, and approving it removes the fact; a memory export produces a file. Memory and skill writes (R12) wait for approval; a test approves one and rejects one.
- Each bot’s evals (≥3) pass against the deployed bots (make verify, R38); usage is readable in Hermes analytics and the grok.com dashboard [G]; with the subscription login revoked or capped (a 401 test fixture stands in; the real login stays signed in), the dashboard and Telegram show an error within ~30 s and make verify fails (R18).
- With the primary model blocked, chat answers from the fallback model; with the provider blocked, the dashboard and Telegram show an error within 30 s and Gatus flags it (R18).
- A test Hermes bump PR shows auto-merge is off, its deploy waited for the digest-keyed pre-upgrade backup, and the evals then ran against the deployed bot (R33).
- The Hermes ops dashboard (hermes-ops.<DNS suffix>) shows up in Gatus (R5) and requires its login (R27).
- After a pod restart, Git config is re-applied; restoring the Hermes volume from backup works.
- Branch protection requires admin-gate (R36): seeded PRs that change a protected path or remove a Namespace fail it without the label and when admin-approved comes from gordon5417-ai or a fixture Bot/App actor, and pass once Gordon’s own account adds it after the latest commit; a seeded PR fails the render check (a stale Hermes render, R4).

### P3 — Daily butler

Scheduled jobs and read skills for Gmail, Calendar and GitHub PRs, delivered through Telegram and visible in the Hermes dashboard. Writes stay deferred, except inbox’s email drafts.

Gordon first: §8 Gordon-only actions, P3.

Goal: scheduled briefs and read skills deliver the daily butler, with no writes beyond inbox drafts.

Contracts first (§5): none new (Skill contract from P2).

Done: make verify PHASE=p3 output attached.

- The morning brief arrives at its scheduled time with calendar, email and weather (Open-Meteo, attributed; “weather unavailable” if it is down), and reaches Telegram [G].
- The evening wrap arrives at its scheduled time with the day’s unread inbox threads, ranked by importance in chat only, and tomorrow’s calendar, and reaches Telegram [G].
- The weekly review arrives at its scheduled time with the past week’s calendar and email highlights and the coming week’s calendar, and reaches Telegram [G].
- homelab’s scheduled “watch for stuck updates” check reads Renovate PRs and their check runs and annotations from the public repos (the no-repos public-read-only token, R13; CI reports each failure as an ::error annotation): a seeded Renovate PR whose required check failed on its last 2 runs, or that has been open more than 7 days, produces one finding with the error and a suggested fix in homelab’s dashboard session, and the next morning brief carries a “stuck updates” line pointing to it.
- Each read skill is covered by an eval in its bot’s evals/.
- Asked in chat to draft a reply to a seeded email, inbox creates one Gmail draft in that thread and nothing is sent (the Sent folder is unchanged).
- An eval asserts no bot can send, delete, apply Gmail labels, write to calendar or reach Drive.
- A forced bot cron failure sends a Telegram alert without going through Hermes (a log-based Grafana alert on Hermes cron errors).

### P4 — Code-factory

Code-factory deploy (foreman, spec-writer), builder and checker sandbox lanes, the publisher script, isolated app hosting. Starts after the P2 exit; live by day 90 after P0 starts. Two checkpoints: P4a (app hosting, no bots needed) then P4b (the issue-to-draft-PR loop).

Gordon first: §8 Gordon-only actions, P4.

#### P4a — App hosting

Goal: one apps.yaml line deploys an outside app, isolated from JARVIS.

Contracts first (§5): App hosting.

Gordon first: §8 Gordon-only actions, P4 (see above).

Done: make verify PHASE=p4a output attached.

- After Gordon adds one line to apps.yaml, the outside app deploys to its own namespace, is reachable at app-<name>.<DNS suffix> on LAN and on tailnet [G], shows up in Gatus (R5), and a network-policy test shows it cannot reach core services.
- A seeded jarvis-apps PR with a cluster-scoped object fails CI (R30).

#### P4b — Issue to draft PR

Goal: a code-factory:ready issue becomes a checked draft PR in jarvis-apps.

Contracts first (§5): Build agent, Sandbox, Code-factory work protocol.

Gordon first: §8 Gordon-only actions, P4 (see above).

Done: make verify PHASE=p4b output attached.

- A toy code-factory:ready issue becomes a draft PR in jarvis-apps with a green CI check and a checker check run (pass or fail with notes) from a different model family.
- A seeded code-factory:ready label added by the App, not by Gordon’s account, is ignored, the workflow comments why, and foreman reports it (R40).
- The code-factory App adding admin-approved fails admin-gate.
- spec-writer turns an idea into an issue with acceptance criteria and without code-factory:ready; a failing checker verdict keeps the PR a draft and sends it back through the fix loop until the fix cap (R40).
- Config validation fails when the builder and checker are from the same model family (seeded test, R41).
- Token scopes show no agent holds push credentials and the code-factory cannot write to jarvis.
- A forced timeout gets the issue the code-factory:failed label, and foreman says so.

## 3. Rules

Numbered, each stated once. Format: rule. Why. Enforced by. A merged rule keeps its number as a one-line stub pointing to the rule that absorbed it; 34 rules are active.

### Architecture and config

R1. Four layers, one folder each: infra (nodes, OS, k3s with its server flags, etcd snapshots and host folders, and the one-time Flux Operator and FluxInstance apply; Ansible only) → platform (only what Flux reconciles: the Flux Operator itself after bootstrap, routing, certs, local DNS, backups, app hosting) → services (Hermes, observability) → apps (hosted apps, in jarvis-apps). Each layer uses only the contracts of the layer below. Each env’s Flux entry point, envs/<env>/flux/, applies them in order: platform controllers → platform configs → services → apps, chained with dependsOn, so no config lands before its CRDs. Why: swap a part without touching the rest; no first-boot races. Enforced by: review against CONTRACT.md; Flux dependsOn.

R2. One home per component: code, config schema, CONTRACT.md and tests in one folder, in the repo it belongs to. Why: nothing to hunt for. Enforced by: CI checks each component folder has a CONTRACT.md.

R3. Core is env-agnostic. Env specifics (nodes, role labels, sizes, enabled features, bots, cron, model map, backup target, DNS suffix, code delivery provider, build agent) live only under envs/<env>; roles map to nodes by labels, so moving a role or adding a node touches only envs/. Why: one core, many envs. Enforced by: CI fails if core names an env, an identity value such as the DNS suffix (D44), or, from P2, a model.

R4. Env values live in envs/<env>/values/: cluster-vars.yaml is a flat ConfigMap that reaches manifests only through Flux postBuild substitution, and Ansible reads the same file for host settings (the inventory keeps each host’s node_roles and data_disk, D42); hermes.yaml is rendered by one checked-in script into envs/<env>/rendered/ (generated, never hand-edited); apps.yaml feeds app hosting (R30). Other env-specific manifest changes are Kustomize patches. No other templating; Helm only to consume an upstream chart. Resources with their own ${…} syntax opt out of substitution (kustomize.toolkit.fluxcd.io/substitute: disabled). Flux variables are the lowercase cluster-vars keys, and substitution is strict, so a missing variable fails (D45). Why: Kustomize has no values file; one mechanism avoids invented renderers; non-strict substitution would blank unknown ${…}. Enforced by: env schema validation; CI fails if a fresh render differs from rendered/ or a ${…} is not a cluster-vars key.

R5. Every service has liveness and readiness probes, logs to stdout (JSON in our own code), Prometheus /metrics where the component offers it, and a Gatus check per exposed endpoint. Every container sets a memory request and limit and a CPU request, with no CPU limit. Why: one standard way to see health; no pod can starve the node. Enforced by: CI manifest lint.

R6 — merged into R31 (backups).

R7. Deploys are idempotent, and the stack rebuilds from Git plus the §8 secrets on a blank node, with data restored from VolSync, including the Hermes xAI device logins, which Gordon re-approves if stale (R31, §8). Why: reproducibility. Enforced by: P0 exit check.

R8. Observability and alerting never depend on Hermes; alerts go straight to Gordon through a send-only Telegram alerts bot. Why: the NOC must stay up when the assistant is down. Enforced by: P1 exit check.

R9. One tool per job, from the approved stack (§4). Adding a tool is a PR that adds it to the stack with a one-line reason and a swap boundary. Why: avoid sprawl. Enforced by: review.

R10. Languages: Python for services and glue, TypeScript for any web UI, Kustomize YAML for manifests, Ansible for nodes; one language per component; exceptions need a written reason. Why: fewer moving parts. Enforced by: review.

### Hermes

R11. Both deploys come from the Hermes core, each with one replica, its own volume, secrets and memory; they share nothing. A custom image needs a written exception. Why: one codebase, isolated runtimes. Enforced by: CI checks digest pins and that no volume or secret is shared.

R12. Behavior comes from Git, learned state from the volume. Config, SOUL.md and approved skills come from Git and win on every start (runtime edits are overwritten). skills.write_approval and memory.write_approval are on for every bot; agent-written skills are promoted by PR. Why: no drift; no memory rot. Enforced by: config in core defaults; startup step.

R13. Least privilege. Profiles are not a security boundary: terminal, file, code-execution, browser and delegation tools stay off unless the bot’s profile gives a written reason, and the root-starting image is offset by seccomp, no service-account token (except a read-only one in the ops pod: get, list and watch, no Secrets, MCP servers in read-only mode; the code-factory pod gets none), network policy (each deploy’s egress allows DNS plus TCP 443 to the internet, excluding private (RFC1918), 100.64.0.0/10 (CGNAT and tailnet), pod and service ranges and the node IP; its CONTRACT.md lists the hosts it uses, as documentation; hostname allow-listing waits for the §9 proxy) and memory limits. Never YOLO mode in ops; unattended sessions deny risky actions; YOLO only inside a throwaway code-factory sandbox. The ops deploy holds no code-factory credentials and no GitHub credential except a fine-grained token from Gordon’s account with no repos selected (public read only) for the read-only GitHub MCP server; its PR read skill reads the public repos (private-repo token: §9). Why: all bots in a deploy share one user and filesystem, and ops reads untrusted email and web content. Enforced by: CI toolset policy check; config in core defaults.

R14 — merged into R13 (least privilege).

R15. Anything that sends, deletes, spends or changes shared state asks for confirmation first. In this plan, bots only read, plus inbox email drafts and spec-writer’s jarvis-apps issues (filed after Gordon confirms in chat, never with code-factory:ready); homelab proposes fixes only as chat messages (what is wrong, a likely cause, a suggested fix) and never files issues or applies a fix; other writes are deferred (§9). Why: prompt injection and mistakes. Enforced by: evals.

R16. Skills use standard contracts: MCP for tools (versions pinned, read-only first) and SKILL.md for procedures. Why: skills survive a runtime swap. Enforced by: CI pin check and tool read/write tag check.

R17. Any UI talks only to the assistant API, never to Hermes internals. Memory stays exportable. Why: Hermes may be replaced. Enforced by: interface contract (§5).

R18. Degrade visibly: each deploy has a fallback model on the same provider login; if the provider is down, Telegram and the dashboard show an error within ~30 s, not a hang; Gatus checks provider reachability (api.x.ai); a reached subscription limit or a revoked login counts as provider down: it shows the same error, and make verify’s evals fail. Dashboard and Grafana logins work offline. Why: the internet will drop; one provider login per deploy (R23). Enforced by: Gatus check; P2 exit check.

### Bots

R19. Bots are code. Each bot is a Hermes profile: one folder under the deploy’s profiles/ (SOUL.md, profile.yaml, skills/, evals/ with ≥3 evals), started from the core profile template and added by PR, only when separating the work buys something; no placeholder folders. The starter set ships pre-baked; after that each user adds and removes their own bots by PR. SOUL.md stays generic: personal facts go in memory or SOPS-encrypted env values. envs/<env>/values/hermes.yaml lists which bots run and which cron jobs are enabled: listed cron jobs start enabled (the PR was the approval), cron jobs of bots not yet listed arrive paused, and reminders Gordon asks for in chat run as asked. Retiring is by hand after removal from Git: back up the bot’s memory, delete its running profile and revoke its Telegram token, if it has one. Rename = add the new bot, retire the old one. Why: a few small bots, fully described by Git, nothing personal in a public repo, no forgotten runbook. Enforced by: CI validates profile.yaml against the profile schema; startup step.

R20 — merged into R19 (bots are code).

R21. One Telegram bot token per Telegram-facing bot; every Telegram gateway accepts only allow-listed user IDs (Gordon at launch). Start minimal: concierge is the only Telegram-facing bot at first; other bots are reached through the Hermes dashboard, and more Telegram-facing bots are added by PR once proven in practice. The send-only alerts bot used by Gatus and Grafana has no gateway and no allow-list and does not count as Telegram-facing. Why: a token is a public front door. Enforced by: CI checks the allow-list is set.

R22 — merged into R19 (bots are code).

### Models and keys

R23. Models and keys. Model tiers are frontier, mid and cheap; profiles name only tiers, and envs/<env>/values/hermes.yaml holds the bot tier map and maps tiers to real models, so model names live only under envs/ (env-01 tier map: §7); hermes.yaml also maps the code-factory lanes (builder, hard, checker) to models with their family, which the lane job reads. One provider login per deploy, shared by its bots, never per bot: each Hermes deploy signs in once to xAI through Hermes’s xai-oauth provider on Gordon’s SuperGrok/X Premium+ subscription (a device login stored on the deploy’s volume), and the lanes use Gordon’s Cursor subscription through one Cursor user API key. No xAI API keys, team or prepaid balance; usage is read in the grok.com dashboard and per bot in Hermes analytics. Subscriptions are Gordon’s only; login tokens are never put in CI or copied between hosts. Hermes calls providers directly (no gateway). Why: swap models in one place; few credentials, nothing to tend. Enforced by: CI grep; env schema plus the R11 no-shared-secret check.

R24 — merged into R23 (models and keys).

### Security and access

R25. Repos are public for now, so nothing sensitive is in plain text: secrets are SOPS-encrypted; LAN IPs, user IDs and other env specifics are encrypted or kept out (the Ansible inventory holds hostnames with their roles and data_disk, never IPs; inventory and group_vars never need the age key, and secrets live only in SOPS files that bootstrap-secrets and Flux decrypt, D42). Workflows that hold secrets never run on fork PRs. The age private key is the only offline secret and the one bootstrap secret: Gordon holds it offline (custody: §8), it is never in the repo, and the Ansible bootstrap applies it once as flux-system/sops-age, which Flux never reconciles; k3s secrets encryption protects the in-cluster copy. While the repos are public, Flux reads them over HTTPS with no key; going private is a separate step (§9). Why: public repos; losing the key means no rebuild from Git, and Flux cannot decrypt the secret that lets it start. Enforced by: CI secret scan; Gordon-only action.

R26 — merged into R25 (public repos and the age key).

R27. Each UI uses its own built-in login: the Hermes dashboards (username and password) and Grafana (anonymous access off). The read-only Gatus page and hosted apps are login-free on LAN and tailnet. The assistant API needs the deploy’s Hermes API key. Hostnames: §6 Naming. Why: one admin on a private network; Hermes requires its own login anyway, so a front door would only add layers. Enforced by: CI checks that the Hermes dashboards have auth configured and Grafana anonymous access is off.

R28. Nothing is public, and names are internal. Every endpoint, hosted demos included, is reachable only via the LAN or Tailscale. Both are router-owned: Tailscale runs on the router and advertises a route to the LAN, and the router handles LAN and tailnet DNS and forwards the env’s zone to JARVIS DNS (external dependency, §8); nothing for remote access is installed on the nodes. Hostnames are <svc>.<DNS suffix>; the suffix is an env setting, never hardcoded in core (env-01: alpha.lan). cert-manager’s internal CA issues a wildcard cert for *.<DNS suffix>; its root key is SOPS-encrypted so rebuilds reuse it, and the root cert is trusted once per device (mechanics: §5 Exposure). Why: small, simple security; .lan is not public, so public ACME is impossible. Enforced by: CI allows Services of type LoadBalancer only for Traefik and JARVIS DNS; the router has no port forwards (Gordon, Standing); CI fails if core hardcodes a DNS suffix (R3); P0 exit check.

R29 — merged into R28 (access and names).

R30. Hosted apps run in app-<name> namespaces on the env’s cluster with default-deny network policy, a quota, their own secrets, no JARVIS secrets and no route to core. Gordon enables and sizes an app with one line in envs/<env>/values/apps.yaml; app deploy folders hold only namespaced objects, and hosted apps are stateless in this plan; an env can turn app hosting off (mechanics: §5 App hosting). Why: outside code must not touch JARVIS; cluster-scoped objects stay human-owned in jarvis. Enforced by: network-policy test (P4a); CI rejects cluster-scoped objects and PVCs in jarvis-apps.

### Backup and upgrade

R31. Backups. Prefer stateless services. Every PVC has a VolSync backup entry or is marked backup: none (rebuildable), which only observability volumes (VictoriaMetrics, VictoriaLogs) may use. VolSync (restic) backs up to a dedicated local backup path set per env (on the node’s data disk when it has one), where k3s also writes etcd snapshot files; retention per env (§7); fallback and setup: §4 and the §8 order. The rebuild path is Git plus VolSync: a fresh node rebuilds from Git and the §8 secrets, then restores data from VolSync; etcd snapshots serve only same-node rollback and get no restore drill. Restore drills: P0b proves a volume restore; P2 proves the Hermes volume restore. On one disk, backups protect only against logical loss (deleted volume, bad upgrade), not disk failure; off-site is deferred. Why: state without a tested backup is lost state; backing up rebuildable metrics and logs would roughly double their disk use. Enforced by: CI checks each PVC has a backup entry or the rebuildable mark; P0b and P2 restore drills.

R32 — merged into R31 (backups).

R33. Upgrades: images are pinned by digest and upgraded by Renovate PRs. k3s is pinned in cluster-vars.yaml and changes only by PR; Renovate does not manage it. Renovate opens minor and patch PRs after a ~3-day hold; auto-merge stays off until the P2 exit check passes, then minor and patch PRs auto-merge with CI green (protected paths per R36); majors are human-reviewed. Hermes bumps never auto-merge; Gordon merges them. Each deploy overlay pins its own Hermes digest, so there is one PR per deploy, code-factory first, adopted after a ~3–5 day soak from the upstream release, and the ops PR follows after a 24 h watch. Evals run after each deploy (R38); the pre-upgrade backup runs in-cluster: the deploy’s Flux Kustomization depends on a VolSync backup manually triggered by the new digest and waits until it completes. Why: Hermes ships weekly and breaks; SQLite sessions can corrupt. Enforced by: CI digest check; Renovate config; P2 exit check.

### Build, deploy and gates

R34. Git on the env’s code delivery provider (GitHub, §5) is the one source of truth. No hand edits in the cluster; coding agents change it only through Git and make targets; any mirror is read-only. Why: one main. Enforced by: Flux owns cluster state; the Cursor CLI deny list (.cursor/cli.json) blocks direct kubectl writes, helm, ssh and secret decryption (a guardrail that shell wrappers can bypass).

R35. Coding agents build. Every plan task is written in the brief template (goal, why, hard limits, done checks). Done means evidence attached (test output, health check, commit). A brief may set a timebox; a task past its brief’s timebox stops and reports a failed state. Why: outcome-focused, no hung work. Enforced by: PR template.

R36. Until the P2 exit, agents merge their own PRs when required checks pass. From the P2 exit, branch protection on the default branch of jarvis and jarvis-apps requires green CI plus one gate check, admin-gate: the reusable .github/workflows/admin-gate.yml sparse-checks out only tools/checks/admin-gate at the pinned jarvis SHA and runs it on the event. It fails any PR that touches a protected path (jarvis: envs/*/secrets/, envs/*/values/apps.yaml, platform/backups/, infra/, services/hermes/core/config/, tools/checks/, tools/verify/, rulebooks/, docs/code-delivery/, docs/build-agent/, AGENTS.md, CLAUDE.md, .cursor/, .github/ and .sops.yaml; jarvis-apps: .github/, AGENTS.md and CLAUDE.md), removes a Namespace, PVC or StorageClass, or comes from the code-factory App, unless it carries an admin-approved label added by Gordon’s own GitHub account, the repository owner (the R40 label-actor pattern; github.repository_owner assumes user-owned repos, and a move to an org makes the approver a repo variable). The gate runs on pull_request_target (opened, synchronize, reopened, labeled, unlabeled), reads PR metadata only and never checks out PR code, and removes admin-approved on every new push; the jarvis-apps caller uses the same trigger. Everything else still merges on green. Code-factory PRs stay draft; Gordon labels and merges them. In project-plans (framework and plan, §11), only Gordon merges; gordon5417-ai may open PRs. Why: gate only what is risky, once there is something to protect; agents never edit the rules they follow; works on free GitHub, which does not enforce CODEOWNERS review. Enforced by: branch protection requiring admin-gate (seeded P2 checks); on project-plans, only Gordon merges (by convention; gordon5417-ai opens PRs only).

R37. After deploy, each changed component runs its own small smoke check; failures are fixed forward or reverted. Rollback = revert the commit; data comes from backups. Why: fast, simple recovery. Enforced by: each component’s smoke test runs in make verify; the P0 drills.

R38. Tests are small, modular and live next to the code. Golden-task evals are pytest: ≥3 per bot, about 10 per deploy, graded by a cheap-tier model called through the same deploy’s assistant API (that deploy’s subscription login and Hermes API key; no separate key), run by make verify against the deployed bots after Hermes bumps, model-map changes and profile changes, code-factory first. Why: low upkeep, low cost; no API key in CI. Enforced by: the P2 exit check.

R39. Rulebooks: one for coding agents (rulebooks/coding-agents.md); each bot’s rulebook is its SOUL.md; deploy-wide bot rules are config in core defaults. A lean AGENTS.md in jarvis points to them and never copies them. One exception: jarvis-apps/AGENTS.md is itself the app rulebook, because the code-factory builder sees only jarvis-apps. Why: rules stated once do not drift. Enforced by: review.

### Code-factory

R40. The code-factory targets only jarvis-apps at first, never jarvis. The loop runs in the code delivery provider’s CI (GitHub Actions): a code-factory:ready label added by Gordon’s account starts one lane run at a time (concurrency queue), with a fix cap; a label from any other actor is ignored with a comment. foreman watches and reports and never codes. spec-writer turns ideas into issues with acceptance criteria and never adds code-factory:ready. Gordon approves twice: the code-factory:ready label and the merge. A run that times out or hits the fix cap gets code-factory:failed, and foreman reports it. Credentials: §8. Why: humans gate intent and result, not steps; core stays human-built. Enforced by: the workflow’s label-actor condition (seeded test in P4b); GitHub App installed on jarvis-apps only; the admin-gate check (R36).

R41. Builder: Composer through the headless Cursor CLI. Issues labeled code-factory:hard use Claude through Cursor. Checker: Grok through the Cursor CLI in read-only (ask) mode; it reads the diff and the acceptance criteria and returns pass or fail with notes. Checker family always differs from builder family. Why: independent review. Enforced by: config validation fails if the families match.

R42. Each task runs in a throwaway sandbox behind the sandbox interface (§5), with a time limit: GitHub Actions first; the in-cluster gVisor sandbox comes later (trigger: §9). Agents never hold push credentials: the lane job (builder and checker) holds only the Cursor user API key and a read-only token, never the App key; a separate publish job opens the draft PR and posts the checker check run with a short-lived App token (§5 work protocol), and the App cannot edit workflows. The code-factory machinery lives in jarvis (§6) and runs pinned to a jarvis commit SHA, so the code-factory cannot change what judges it. GitHub-hosted runners cannot restrict egress (accepted limit, §10 Risks); offsets: no JARVIS secrets in the runner, a short-lived one-repo token, pinned CLI versions and the time limit; egress allow-listing arrives with in-cluster gVisor. Why: isolation Hermes does not provide. Enforced by: token scopes; the admin-gate check on .github/ (R36).

R43 — merged into R40 (code-factory scope).

R44 — merged into R42 (offsets) and §10 Risks (accepted limit).

## 4. Approved stack

Format: tool — role — swap boundary.

### Platform

- Ubuntu 26.04 Server, minimal install, fresh install on every node (OQ10); time from chrony with NTS (the Ubuntu default; outbound 123/udp and 4460/tcp); Ubuntu never upgrades itself (unattended-upgrades and the apt-daily-upgrade timer off); Gordon runs upgrades and reboots (§8) — node OS — Ansible inventory and playbooks.
- k3s (embedded etcd via --cluster-init, secrets encryption on, network CIDRs set once; bundled containerd runtime, flannel, network-policy controller, CoreDNS for cluster DNS and ServiceLB for node-IP exposure) — cluster — Kubernetes APIs.
- Ansible — node bootstrap, OS config, k3s and Flux Operator install, bootstrap secrets — playbooks plus a per-env inventory.
- Flux via Flux Operator (ResourceSet for app onboarding) — GitOps sync and SOPS decryption — Git repo and manifests.
- Kustomize (Helm only to consume upstream charts, via Flux HelmRelease) — manifests and env overlays — Kustomize bases and patches.
- GitHub — the code delivery provider (§5): Git hosting, PRs, issues — Git remote and API.
- GitHub Actions on GitHub-hosted runners — CI and the first code-factory sandbox — workflows and the sandbox interface.
- gitleaks (pinned CLI) — CI secret scan (R25) — ci rule check.
- GHCR (GitHub Container Registry) — registry for images built by CI (jarvis-apps apps as public packages, any excepted custom image); upstream images are pulled from their own registries, all pinned by digest — OCI registry.
- Renovate (hosted GitHub App) — dependency and image upgrades as PRs — config in the repo.
- Traefik (bundled with k3s, tuned by HelmChartConfig) with plain Kubernetes Ingress; the wildcard is served through Traefik’s default TLSStore (OQ8) — routing — Ingress.
- cert-manager — internal CA and a wildcard cert for *.<DNS suffix> (env-01: *.alpha.lan) — Certificate resources.
- CoreDNS with the k8s_gateway plugin (a second CoreDNS, separate from k3s cluster DNS) — JARVIS local DNS inside the env: serves the env’s DNS-suffix zone (env-01: alpha.lan), answering <svc>.alpha.lan from Ingress routes, exposed on the edge nodes (every env assigns edge to at least one node, D42); assumed setup: the router forwards alpha.lan lookups to it (router config is Gordon-owned, §8) — DNS on port 53.
- k3s local-path — persistent volumes — storage classes.
- VolSync (restic) — volume backups to a restic repository on the env’s dedicated local backup path (a Phase 0 test decides; only fallback: one restic CronJob mounting it, OQ7); k3s writes etcd snapshot files to the same path, set by Ansible (same-node rollback only, R31) — restic repository.
- SOPS with age, plus one checked-in bootstrap script (make bootstrap-secrets: create-if-missing, never overwrites, so a second run changes nothing, with a test) for the §8 entries marked bootstrap-secrets, and make seal for secrets created in a UI — secrets in Git, one set per env — encrypted files.
- Default-deny NetworkPolicies (enforced by k3s’s network-policy controller) — isolation for hosted apps — NetworkPolicy.

### Services

- Hermes (core as defined in §1) — agent runtime with memory and skills — assistant API and MCP.
- MCP servers and SKILL.md skills — tools and procedures, read-only first (Example: kubernetes-mcp-server read-only); third-party servers are pinned to exact versions with their npm/uv cache, or a pinned upstream release binary (Example: mcp-victoriametrics), on the deploy volume, so pods do not download them at every start — MCP, SKILL.md.
- geo MCP server (in-repo, Python, read-only) — weather via Open-Meteo, distance via Nominatim plus the OSRM public demo, all keyless — MCP.
- VictoriaMetrics and VictoriaLogs, single-node — metric and log storage with retention — Prometheus query API, LogsQL.
- mcp-victoriametrics (official, read-only, pinned binary on the deploy volume) — homelab metric reads — MCP.
- workspace-mcp (taylorwilsdon/google_workspace_mcp, pinned uv package on the deploy volume; no send tool, drafts on inbox only) — Gmail and Calendar reads, inbox drafts — MCP.
- github-mcp-server (official, pinned release binary on the deploy volume; read-only in ops, issues toolset for spec-writer with the code-factory App) — PR, check-run and annotation reads; spec-writer issue filing — MCP.
- Fluent Bit (DaemonSet) — log shipper: container and node logs to VictoriaLogs — stdout logs.
- vmagent with node-exporter and kube-state-metrics — metric collection: node, Kubernetes-object and /metrics targets to VictoriaMetrics — Prometheus exporters and scrape config.
- Grafana — dashboards and Grafana-managed metric and log alerts to Telegram — data sources and alert rules in Git.
- Gatus — synthetic checks and the login-free status page — checks as YAML in Git.
- Telegram alerts bot (send-only; Gatus telegram provider, Grafana Telegram contact point) — alerts from Gatus and Grafana — Bot API.
- pytest — tests and golden-task evals — Hermes API.

### Apps and code-factory

- Hermes dashboard plus Telegram (Bot API through Hermes’s built-in gateway) — chat and daily butler surfaces — assistant API.
- Grafana home dashboard linking Gatus — NOC — observability data only.
- GitHub Issues and labels — code-factory queue — GitHub API.
- Cursor CLI, headless, on Gordon’s Cursor subscription — builder and checker lanes — sandbox interface.
- xAI Grok through Hermes’s xai-oauth provider on Gordon’s subscription — Hermes model provider — xAI API (Responses).
- Code-factory GitHub App plus the publisher script (Python) — opens draft PRs so agents never hold push credentials — GitHub API.

### Avoid

- OpenClaw; “AI employee” agent swarms; memory graphs or vaults on day one.
- Longhorn as a default; Argo CD for one cluster and one operator.
- Open WebUI (license has a branding clause).
- The Hermes agent-batch plugin as-is (unpinned npm, skips permissions); Kanban auto-decompose.
- Temporal, Hatchet or a Docker socket for the code-factory.
- Tailscale, Headscale or any VPN agent on the nodes; access is router-owned (§8).
- A local S3 server (Example: MinIO, Garage) for backups.
- A volume snapshot controller: local-path has no volume snapshots; pre-upgrade backups use VolSync (R33).

## 5. Interfaces and contracts

Each gets a CONTRACT.md from one template: offers, expects, health, config, proven by (a CI check or a §2 check). Written before the code that uses it.

- Model provider (P2) — per deploy: one provider login (env-01: xai-oauth) or one OpenAI-compatible base_url plus key (R23). hermes.yaml maps tiers to models, including a primary and a fallback model on the same login (R18); profiles name only tiers (R23). A local LLM or a gateway is just another base_url. Proven by: the P2 fallback check, Gatus provider reachability and the R23 CI grep.
- Assistant API (P2, per deploy) — on the deploy’s hostname (hermes-ops or hermes-code-factory): the dashboard at /, and each profile’s OpenAI-compatible API at /api/<profile>/v1, routed by path to that profile’s port; behind the deploy’s Hermes API key; the dashboard uses its own login (R27). Session history, including briefs, is readable through it; memory export produces a file (R17); /v1/audio/* paths are reserved and not built (the voice seam). The seam for any later UI. Proven by: the P2 scripted-prompt and memory-export checks.
- Skill contract (P2) — MCP servers (pinned) and SKILL.md. MCP servers run as pinned processes inside the deploy’s pod (no sidecars); each tool is tagged read or write in the profile’s toolsets; write tools run only behind a confirm step (R15); this plan ships read tools plus inbox drafts. The in-repo geo server (services/hermes/core/skills/geo/) runs on the Hermes image’s Python and keeps to the providers’ usage policies: at most 1 request per second to Nominatim and OSRM, cached results, a User-Agent with Gordon’s contact email, and attribution in answers (Open-Meteo; OSRM, © OpenStreetMap contributors). Home is a rounded SOPS value and is never looked up; only the destination is sent. Its CONTRACT.md records the deliberate decision Nominatim’s policy asks for. Proven by: the R16 pin and tag check, the R13 toolset policy check and the P3 no-write eval.
- Profile schema (P2) — profile.yaml: job, toolsets with reasons and read/write tags, cron, evals, telegram: true/false (allow-list required when true, R21); bot names that collide with the dashboard’s own /api/ paths (status, cron, memory, skills) are rejected (tiers come from the env’s bot tier map, R23); validated in CI.
- Env schema (P0) — what an env may set: nodes with their roles and optional data disk (inventory/hosts.yml, D42), the data paths (D3), the Git repos and AI provider host (D45), sizes, enabled features, bots, enabled cron, time zone, bot tier map and tier-to-model map, backup target and retention, DNS suffix (dns_suffix; env-01: alpha.lan), the node SSH user (ssh_user, default ubuntu; Ansible logs in as it and runs every task as root), app hosting on or off and the enabled apps, the code delivery provider (github only), the build agent (cursor only). It validates values/cluster-vars.yaml, values/hermes.yaml, values/apps.yaml and inventory/hosts.yml in CI for every env, including the env-00 example (D43). Its CONTRACT.md sections are listed in D45.
- Login (P1, P2) — each UI’s built-in login (R27); bootstrap-secrets creates the Grafana admin password and each Hermes dashboard’s password hash and signing secret. Dashboard and Grafana logins work offline (R18). Multi-user seam: a second user adds one OIDC issuer for the UIs’ native OIDC (§9). Proven by: the R27 CI check and the P2 dashboard-login check.
- Exposure (P0) — a component gets <svc>.${dns_suffix} by adding one Ingress (OQ8). Ports 80/443 and 53 land on the edge nodes; every env gives edge to at least one node, with no fallback to control_plane (D42). JARVIS DNS answers the name automatically, the wildcard cert covers it (R28), and the component adds its own Gatus check (R5). Gatus resolves the env’s zone through cluster DNS (k3s coredns-custom forwards it to JARVIS DNS) and trusts the internal CA’s public root cert, committed in plain text, so its HTTPS checks see what a browser sees. Proven by: the R5 lint and the P0b, P2 and P4a reachability checks.
- Backups (P0) — each PVC either uses the shared VolSync component from platform/backups/ (a ReplicationSource with an on-demand trigger, which make hermes-backup uses) or is marked backup: none (rebuildable) (R31). The target path (backup_path; volumes sit under local_path_data, D3), retention and weekly prune come from cluster-vars.yaml. Restore is a ReplicationDestination with a written procedure. Proven by: the R31 CI check and the P0b and P2 restore drills.
- Alert path (P1) — Gatus checks and Grafana metric and log alerts (disk full, node memory above 80%, node not ready, clock not synchronised, backup failures, bot cron failures) to one Telegram chat through the send-only alerts bot; never through Hermes. The observability network policy follows R13’s egress rule; the hosts it uses are api.telegram.org and, for Gatus’s provider probe, api.x.ai.
- Code delivery provider (P0) — one contract for six capabilities: repo hosting (the Flux source), CI runs, the issue and label queue, PR open and merge, the approval-label check (admin-gate, R36) and the image registry. Each env picks its provider in envs/<env>/values/cluster-vars.yaml (code_delivery: github, the only accepted value, validated by the env schema; no manifest reads it until a second provider exists); services, code-factory and app hosting depend only on this contract. GitHub provider (the only one built): GitHub repos, GitHub Actions, Issues and labels, PRs (coding agents through the agent account’s token (env-01: gordon5417-ai), code-factory drafts through the code-factory GitHub App), branch protection plus admin-gate, and GHCR (§4). Forgejo is the planned second provider (§9). Proven by: env schema validation and the P0a, P4a and P4b checks.
- Build agent (P4b) — one capability contract (docs/build-agent/CONTRACT.md) for the coding agent a lane runs; each agent is a thin adapter script the lane calls (lanes/agents/<name>.sh). Each env picks it in envs/<env>/values/cluster-vars.yaml (build_agent: cursor, the only accepted value, validated by the env schema). An adapter runs headless, with an exit code that means provider down (R18) and never a silent fallback to pay-as-you-go; writes one standard result file; uses one subscription-backed secret; loads AGENTS.md; takes its deny list from an admin-gate protected file (R36); runs the checker read-only; never pushes or opens PRs (the publish job does, R42); maps builder, checker and hard to models or marks a role unsupported; and installs a pinned version, called by full path. Cursor adapter (the only one built): the headless Cursor CLI on the Cursor user API key, deny list .cursor/cli.json, checker in ask mode, models per R41. grok-build and opencode-local are §9 stubs. Proven by: env schema validation and the P4b checks.
- Sandbox (P4b) — the lane workflow’s inputs (issue, repo, timeout) and outputs (status, patch, verdict, logs); same contract for gVisor later.
- Code-factory work protocol (P4b) — GitHub Issues in jarvis-apps is the queue: code-factory:ready (Gordon applies it; the workflow checks the label actor, R40), code-factory:hard, code-factory:failed (the publish job applies it); code-factory.yml runs on issues: labeled, one run at a time through a queued concurrency group; the lane job works on branch code-factory/<issue>; the publish job opens a draft PR with a short-lived App token and posts the checker’s verdict as a check run named checker (pass or fail, with notes); foreman reads the public repo on a cron job that skips the model call when nothing changed, and reports finished, failed and ignored runs; Gordon merges. Proven by: the P4b checks.
- App hosting (P4a) — one ResourceSet in platform/app-hosting/ fed by envs/<env>/values/apps.yaml (a Kustomize patch that sets its inputs, R4): namespace, quota, default-deny policy, service account, a per-namespace GitRepository and an impersonated Flux Kustomization that deploys apps/<app>/deploy (no cross-namespace source refs; no secret while the repos are public), LAN/tailnet exposure (R30); app images are public GHCR packages, so no pull secret; app.yaml in jarvis-apps gives source, port and health path only. The ResourceSet also writes the app’s Gatus endpoint as a labeled ConfigMap in observability, which a Gatus config sidecar loads (the pattern Grafana uses for dashboards). Order for a new app: merge it, build.yml publishes it, Gordon makes the package public, Renovate pins the digest in deploy/ (auto-merged), then Gordon adds the apps.yaml line.

## 6. Repo layout, env model and naming

Two repos: jarvis (infra, platform, services) and jarvis-apps (apps JARVIS hosts, one folder each; outside apps referenced, not copied). Planning lives outside both, in project-plans (§11). Stubs only for things being built. One command surface: the root Makefile wraps tools/; each CI rule check is one script in tools/checks/.

```
jarvis/
  AGENTS.md                    lean pointer to the rulebooks
  Makefile                     render | check | verify | hermes-backup | bootstrap-secrets | seal | google-consent (wraps tools/)
  .sops.yaml                   age recipients; one creation rule per env, keyed on envs/<env>/secrets/ (a new env adds its own rule, D45)
  .cursor/cli.json             Cursor CLI permission list for coding agents (deny beats allow)
  renovate.json
  docs/                        README.md (points to project-plans for the framework, plan and decisions; never copies them), CONTRACT.template.md, code-delivery/CONTRACT.md (provider contract; github only), build-agent/CONTRACT.md (agent contract; cursor only)
  rulebooks/coding-agents.md
  schema/                      env schema: the validator for every env, its tests and CONTRACT.md (D7)
  tools/                       render-profiles, bootstrap-secrets, seal, google-consent, hermes-backup, checks/ (one script per CI rule, including admin-gate), verify/ (one short checklist per phase)
  infra/                       layer 1: nodes (Ansible only)
    inventory/roles.yml/       the core constructed inventory, passed as the second -i: the constructed config turns node_roles into groups, and groups.yml keeps an empty data_disk group (D42)
    ansible/                   ansible.cfg, site.yml, roles/{base-os, node-disk, k3s, kubeconfig, flux-operator}/ (values from envs/<env>/values/cluster-vars.yaml)
  platform/                    layer 2: only what Flux reconciles; each folder has CONTRACT.md + tests/
    flux-operator/             operator HelmRelease (Flux-managed after the one-time Ansible apply, R1)
    traefik/                   HelmChartConfig (applied in kube-system); routes are plain Ingress
    cert-manager/              controller/ + config/ (internal CA, wildcard *.${dns_suffix})
    lan-dns/                   CoreDNS + k8s_gateway
    backups/                   controller/ (VolSync) + config/ (policies)
    app-hosting/               ResourceSet template: ns, quota, netpol, SA, GitRepository, impersonated Kustomization
  services/                    layer 3; each component folder has CONTRACT.md + tests/
    observability/             victoria-metrics, victoria-logs, fluent-bit, vmagent, grafana/{dashboards,alerts}, gatus
    hermes/
      core/                    base/ (Kustomize), config/ (defaults, write_approval, toolset policy),
                               skills/ (geo/: in-repo read-only MCP server), glue/ (render lib, startup sync), schema/ (profile schema and its CONTRACT.md),
                               templates/profile/, testing/ (mock MCP, grader), CONTRACT.md
      deploy/ops/              overlay of core/base; profiles/{concierge,inbox,homelab}/{SOUL.md,profile.yaml,skills/,evals/}
      deploy/code-factory/     overlay of core/base; profiles/{foreman,spec-writer}/; lanes/{builder,checker}/, lanes/agents/cursor.sh (build agent adapter);
                               publisher/; sandbox/ (lane workflow contract)
  envs/
    env-00/                    never-deployed 7-node example env that sets every schema key (D43)
    env-01/
      README.md                purpose, what deploys, promotion
      values/                  cluster-vars.yaml (flat, lowercase keys, postBuild; dns_suffix, flux_git_repo_url, apps_git_repo_url, local_path_data, backup_path, llm_provider_host, code_delivery: github, build_agent: cursor, ssh_user, age_key_file), hermes.yaml (bots, cron, tier map, models), apps.yaml (enabled apps and sizes)
      inventory/               hosts.yml (one block per node: node_roles and an optional data_disk, D42; never IPs)
      flux/                    FluxInstance sync path: platform-controllers -> platform-configs -> services -> apps (dependsOn)
      patches/                 Kustomize patches (sizes, node selectors on role labels)
      rendered/                GENERATED Hermes profile config, CI diff-checked
      secrets/                 <component>-<purpose>.sops.yaml, one Secret per file
    # more envs: copy env-01 when hardware exists
  .github/
    workflows/                 ci.yml (the one required ci check), admin-gate.yml (pull_request_target; reusable, also called by jarvis-apps), code-factory-lane.yml (reusable, called by jarvis-apps)
    pull_request_template.md   brief template + evidence
jarvis-apps/
  AGENTS.md                    the app rulebook (the builder reads only this repo)
  renovate.json
  apps/
    _template/                 CONTRACT.md, app.yaml (source, port, health path), deploy/ (namespaced objects only), Dockerfile or src/
    <app>/                     own code, or a pinned upstream ref built to GHCR
  .github/
    workflows/                 code-factory.yml (issues: labeled caller of jarvis code-factory-lane.yml@<sha>, passing jarvis_sha as an input for the lane checkout), build.yml (GHCR), ci.yml (calls jarvis admin-gate.yml@<sha> on pull_request_target, passing the same SHA as an input for the admin-gate checkout)
```

### Env model

- An env folder only sets values and turns features on or off; it never holds core logic.
- env-01 is the baseline and prod: one node, 14 GiB RAM, CPU only, a 256 GB NVMe OS disk with 100 GB on /, and no data disk: Jarvis data stays on root on purpose (D42).
- Each env has a written purpose, what deploys to it and how changes are promoted, in envs/<env>/README.md.

### Naming

- Namespaces: kube-system (k3s-owned; only HelmChartConfig and coredns-custom overrides go there), flux-system, platform-<name>, observability, hermes-ops, hermes-code-factory, app-<name>.
- Hostnames: <svc>.<DNS suffix>, matching the namespace where there is one; the suffix is set per env in envs/<env> (env-01: alpha.lan, so it never clashes with existing .lan names). All sit under the one *.<DNS suffix> wildcard cert. Examples on env-01: gatus.alpha.lan, grafana.alpha.lan, hermes-ops.alpha.lan and hermes-code-factory.alpha.lan (each deploy’s dashboard at / and assistant API at /api/<profile>/v1), app-<name>.alpha.lan (hosted apps).
- Node role labels: role.<DNS suffix>/<role>=true, one per role the node declares; the prefix is reserved for them (D42).
- Labels: code-factory:ready, code-factory:hard, code-factory:failed on issues; admin-approved on PRs (R36).
- Branches: agent/<topic> for coding agents, code-factory/<issue> for the code-factory.
- Secrets: <component>-<purpose>, one Secret per SOPS file in envs/<env>/secrets/. Bots: lowercase, one word.

## 7. env-01 defaults and budget

- Schedules (env-01, America/Los_Angeles): stuck-updates 06:30, morning brief 07:00, evening wrap 18:00, weekly review Mon 07:30. Stuck-updates runs first so the brief carries its line.
- Lane limits (per repo, in the lane workflow, not env values): 30-minute run timeout; fix cap 2 runs (first try plus one fix).
- Bot tier map (env-01; R23): mid = concierge, inbox reply drafting and homelab chat replies; cheap = cron jobs, inbox triage, homelab checks, foreman reports and spec-writer drafts; frontier = on demand only. Hermes does not run coding agents: code-factory:hard is a Cursor lane setting in the Actions lane job, not a Hermes tier. Lane models (builder Composer 2.5, checker Grok 4.7, code-factory:hard builder Claude, all through Cursor) and their families are set in hermes.yaml (R23, R41).
- RAM: every container sets a memory request and limit and a CPU request, no CPU limit (R5), starting from the short D30 defaults list and tuned from Grafana as real usage shows up. Ansible sets kubelet system and kube reserves (~1 GB each) and eviction thresholds; app sizes map to quotas (small = 512 Mi and 0.5 CPU, medium = 2 Gi and 1 CPU); a Grafana alert fires when node memory use passes 80%. No coding-sandbox RAM while the sandbox is GitHub Actions. CPU floor: 4 cores.
- Disk: about 500 GB or more for data and backups, on the node’s data disk when it has one (D42; env-01 has no data disk, so data, OS and images share the 100 GB root), split roughly OS and images ~60 GB, metrics ~10–15 GB, logs ≤20 GB, backups ≤150 GB, the rest free. Alert at 80% full.
- Disk caps, each in the writer’s own config: VictoriaMetrics 30-day retention plus -storage.minFreeDiskSpaceBytes; VictoriaLogs 14-day retention plus -retention.maxDiskSpaceUsageBytes (~20 GB); restic keep 7 daily / 4 weekly / 3 monthly, pruned weekly; journald SystemMaxUse 2 GB; images by kubelet image-GC defaults.

## 8. Bootstrap

### Secrets and accounts (honest count: 16 entries; Gordon creates 13; two Telegram bot tokens at launch: concierge and the send-only alerts bot)

Format: item — created by — stored in — used by. SOPS files are envs/<env>/secrets/<component>-<purpose>.sops.yaml; an Actions secret names its repo.

- age keypair — Gordon — private key offline (custody below), public key in .sops.yaml; applied once by the Ansible bootstrap as flux-system/sops-age (R25) — Flux and Ansible (SOPS decryption).
- Internal CA root (10-year expiry) — bootstrap-secrets — SOPS; root cert trusted per device by Gordon — platform-cert-manager.
- VolSync restic password — bootstrap-secrets — SOPS — platform-backups.
- Other component secrets (Grafana admin, and for each deploy the Hermes API key and the dashboard password hash and signing secret) — bootstrap-secrets — SOPS — their own namespace only.
- Renovate GitHub App install — Gordon — no stored secret — jarvis and jarvis-apps.
- Hermes xAI login, ops deploy (xai-oauth device login on Gordon’s subscription) — Gordon approves the device code — the deploy’s volume (VolSync-backed), never Git or CI — hermes-ops only.
- Hermes xAI login, code-factory deploy (xai-oauth device login on Gordon’s subscription) — Gordon approves the device code — the deploy’s volume (VolSync-backed), never Git or CI — hermes-code-factory only.
- Cursor user API key (Gordon’s subscription) — Gordon — Actions secret in jarvis-apps — the jarvis-apps lane caller, which passes it explicitly to the jarvis lane workflow (builder and checker).
- Bastion Cursor login (agent login as the agent user, on Gordon’s subscription) — Gordon — bastion agent user only — coding agents.
- Code-factory GitHub App (jarvis-apps only: contents RW, pull requests RW, issues RW, checks RW; never workflows) — Gordon, by hand from that four-permission checklist — SOPS and an Actions secret in jarvis-apps — the publish job (Actions secret) and spec-writer (SOPS in hermes-code-factory).
- gordon5417-ai classic token (public_repo and workflow scopes; the bastion’s own token, not shared with any other tool) — Gordon, signed in as gordon5417-ai, pasted onto the bastion — bastion agent user only — coding agents.
- Telegram bot token for concierge, the only Telegram-facing bot at first (one more token per Telegram-facing bot added later, R21), plus Gordon’s user ID — Gordon — SOPS — hermes-ops.
- Google OAuth client and refresh token for Gmail and Calendar read plus Gmail drafts — Gordon publishes the OAuth app as In production (unverified, personal use; accepts the warning screen) and runs make google-consent once on the bastion — SOPS — hermes-ops only.
- No-repos public-read-only GitHub token (fine-grained, Gordon’s account, no repos selected) — Gordon — SOPS — hermes-ops only (read-only GitHub MCP server, R13).
- Telegram alerts bot token (send-only) plus Gordon’s chat ID — Gordon, who sends the bot /start once — SOPS — observability (Gatus, Grafana).
- Home location (rounded to 2 decimals, ~1 km) and Gordon’s contact email for the geo User-Agent — Gordon — SOPS env value — hermes-ops only; personal data, not a key.
- Custody and rotation: the age key is the only offline secret, in two copies (password manager plus USB or paper), and is on the bastion only during bootstrap runs. The Hermes xAI logins live on each deploy’s volume; after a restore, Gordon redoes the device login if the token is stale. Keys do not expire by default: rotate one on a suspected leak, and a bot’s Telegram token when the bot is retired (R19). The only scheduled expiry is the internal CA root (10 years; Gordon keeps a calendar reminder a month before). The Google refresh token does not expire because the OAuth app is In production; a Google password change or six months unused still revokes it, so Gordon re-runs make google-consent if the brief reports invalid_grant.

### Gordon-only actions

- Before P0: only Gordon merges in project-plans (gordon5417-ai may open PRs); merge jarvis-alphacut/framework.md, then jarvis-alphacut/plan.md (§11); archive the previous system’s repos read-only. Every token, key and webhook for the new build is new; none is reused.
- P0: create the two repos under gordoncooper (the account that adds admin-approved) and branch protection requiring the ci check; set the owner-name repo variable on jarvis (D44); invite gordon5417-ai (write) to jarvis and jarvis-apps only, accept the two invites signed in as gordon5417-ai, and set up the bastion’s agent user with a Cursor login and the gordon5417-ai token, which Gordon creates signed in as gordon5417-ai and pastes onto the bastion; generate the age key, install Renovate; run make bootstrap-secrets on the bastion and commit its output; wipe the env-01 node (a separate spare compute node; never a node of the previous system), fresh-install Ubuntu 26.04 Server (minimal install) with a login user that has sudo, make the node’s name resolve on the router’s DNS, run tools/prep-node (step 3) and run the first and second site.yml (owner runs, step 4); once JARVIS DNS and the CA exist, set router forwarding of alpha.lan and trust the root cert on each device.
- P1: create the send-only alerts bot with BotFather, send it /start once and record his chat ID.
- P2: create concierge’s Telegram token and record Gordon’s user ID (the chat ID from P1) for concierge’s allow-list; approve the ops deploy’s xAI device login; send a Telegram message during the provider-down and 401 drills; at the exit, create the admin-approved label and add admin-gate to branch protection on jarvis and jarvis-apps.
- P3: create the Google Cloud OAuth client and enable the Gmail and Calendar APIs; create the no-repos public-read-only GitHub token; publish the Google OAuth app as In production and run make google-consent, then send himself one email with subject jarvis-p3-seed (the draft check’s seed; the bot never sends).
- P4: approve the code-factory deploy’s xAI device login; create the Cursor user API key and set Cursor billing to auto top-up off with a monthly limit (on the plan tier that includes Claude for code-factory:hard); create the GitHub App by hand from its four-permission checklist (contents, pull requests, issues, checks; no manifest flow); for P4a, merge the app’s line in envs/<env>/values/apps.yaml and set its GHCR package to public on first publish; for P4b, apply code-factory:ready and merge code-factory PRs.
- Standing: own router-side access (external dependency: Tailscale runs on the router and advertises a route to the LAN; the router handles LAN and tailnet DNS; nothing for it is installed or configured on the nodes); verify the [G] exit checks (§2); review PRs held by admin-gate and add admin-approved from his own account; merge Hermes bump PRs (R33); re-run site.yml after merging an infra/ PR; run OS package upgrades by hand and reboot after kernel updates; keep the previous system frozen (no new work, but not shut down, wiped or destroyed).

### Phase 0 order of operations

1. [Gordon] Repos and branch protection, gordon5417-ai invited to jarvis and jarvis-apps, the bastion’s agent user, age key and .sops.yaml, Renovate install.
2. CI: the one required ci check runs the tools/checks/ rule checks (secret scan included); admin-gate is added at the P2 exit.
3. [Gordon] Fresh Ubuntu 26.04 Server minimal install on the node, with a login user that has sudo. The node’s name must already resolve on the router’s DNS; tools/prep-node checks that first and changes nothing otherwise. tools/prep-node, run from the controller, confirms the host key, installs the controller user’s SSH key, sets up the Ansible user (the login user or a new one, recorded as ssh_user) with passwordless sudo, can change the hostname, can pin the DHCP address as static (or Gordon keeps a DHCP reservation on the router), can turn SSH password login off, and checks the clock is synced. It records the host, ssh_user and age_key_file in the env and re-renders the env’s generated cluster-vars ConfigMap. The age private key is a per-env file on the controller: age_key_file in cluster-vars.yaml, or SOPS_AGE_KEY_FILE, default ~/.config/sops/age/keys.txt under the running user’s HOME, never in the repo. prep-node, bootstrap-secrets and site.yml use the same file; prep-node can create a missing key and warns when an existing key does not match the env’s .sops.yaml recipient. Then make bootstrap-secrets on the controller; commit the env diff and the SOPS output.
4. [Gordon] The first site.yml on env-01 (infra/ansible; it logs in as ssh_user and runs every task as root): OS config (journald capped at 2 GB; chrony with NTS; no automatic upgrades or reboots), the data disk if the node has one (D42), k3s (configured in /etc/rancher/k3s/config.yaml, not INSTALL_K3S_* variables, which Ubuntu’s sudo-rs drops; fixed CIDRs, --cluster-init, secrets encryption, kubelet system and kube reserves and eviction thresholds, the etcd snapshot schedule, and the node role labels and edge selection, D42), the local-path data folder and the backup path, the age key (R25), and the one-time Flux Operator and FluxInstance apply (Flux manages both afterwards, R1). site.yml refuses any host that already runs k3s unless it carries this env’s marker file (/etc/jarvis/env), which the first run writes, and writes the kubeconfig to the running user’s ~/.kube/config (backup: true keeps a timestamped copy of the file it replaces). Owner runs: a full site.yml and anything else that needs the age key are Gordon’s; the agent user keeps node SSH only for narrow runs that need no age key (Example: site.yml --tags kubeconfig).
5. FluxInstance syncs envs/env-01/flux; Renovate opens PRs; P0a checks (make verify PHASE=p0a).
6. Traefik with plain Ingress (OQ8). Then cert-manager’s internal CA (wildcard *.alpha.lan), then JARVIS local DNS serving the alpha.lan zone.
7. [Gordon] Router forwarding of alpha.lan to JARVIS DNS; trust the root cert on each device.
8. VolSync: first a ~10-minute backup-and-restore test to the backup path, and if it fails, one restic CronJob instead, no further searching (OQ7); then backups with the env’s retention.
9. P0b checks (make verify PHASE=p0b); P0 exit.

Steps 6–8 are the order components are added to the platform layer; Flux dependsOn handles the apply order (R1).

## 9. Deferred

Not built in this plan. Core builds only the named seam. Format: item — seam now — trigger.

- Custom UI, themes, voice — assistant API (reserved /v1/audio/* paths, §5), one theme file later — separate later plan once core is established.
- Confirm-first writes (calendar, Drive, email send, SSH) — R15 and write-tagged MCP tools behind a confirm step (Skill contract, §5) — Gordon asks.
- Local LLMs — another base_url in the Model provider contract (§5) — GPU nodes exist.
- Model gateway (lean: Bifrost; LiteLLM only for missing provider coverage) — another base_url in the Model provider contract (§5) — something besides Hermes needs models, shared local models, or central budgets wanted.
- Multi-user and RBAC — Login contract (§5): one self-hosted OIDC issuer (lean: Pocket ID) for each UI’s native OIDC, data keyed by a stable user ID — a second user.
- Knowledge vault, external memory providers — exportable built-in memory — revisit after P2.
- Larger envs, hardware pinning, replicated storage (lean: Piraeus; NAS-backed democratic-csi if a NAS exists) — envs/ and role labels — an env needs it.
- k3s embedded registry mirror (Spegel) — the embedded-registry flag plus registries.yaml mirrors, one flag per env — an env has two or more nodes.
- Preview env — another env folder — core works and promotion needs a gate.
- In-cluster gVisor sandbox and domain egress allow-list (lean: small forward proxy; Cilium later/if) — sandbox interface (R42; an env switch and a gVisor RuntimeClass arrive when built, ~4 GB per sandbox) — a second node exists or GitHub Actions limits or cost actually hurt, whichever comes first.
- Code-factory PRs to jarvis — GitHub App scope — code-factory proven on jarvis-apps.
- foreman orchestrating the code-factory loop (polling ready issues and dispatching lane runs) instead of the Actions loop — the lane and publish jobs and the Sandbox contract (§5) stay — revisit once the Actions loop works.
- Forgejo as a second code delivery provider (self-hosted on LAN: repos, issues, Forgejo Actions runners, built-in container registry) — the code delivery contract (§5); switching takes a Forgejo service and runners, registry trust, ported gate and lane workflows, self-run Renovate and a bootstrap order that does not host the Flux source inside the cluster it builds — Gordon decides.
- grok-build as a second build agent — the build agent contract (§5); its adapter stub exits “not implemented”. Trigger: xAI documents an approved subscription login for CI, or Gordon accepts a self-hosted runner with a device login.
  - Only a pay-as-you-go XAI_API_KEY works in CI; copying ~/.grok/auth.json is discouraged.
  - Grok models only; Grok already runs in the lane through Cursor.
  - It auto-approves git push unless denied: set its own deny list (.grok/config.toml).
  - Its `agent` binary clashes with Cursor’s: install each by full path.
  - When enabled, .grok/ and .claude/ join admin-gate’s protected paths (R36).
- opencode-local as a third build agent — the build agent contract (§5); its adapter stub exits “not implemented”. Trigger: a GPU node with enough VRAM serves a local coding model, and Gordon decides.
  - opencode on local models served on a GPU node: no API keys, no public dependency.
  - Quality on 16 GB local models is unproven: it passes the lane evals before it replaces Cursor.
  - When enabled, .opencode/ and opencode.json join admin-gate’s protected paths (R36).
- Bot plumbing: Kanban, shared multi-bot Telegram gateway, parallel code-factory issues, home automation, a bot in its own pod — profiles and config by PR — a real need.
- One-command bot retire and orphan-profile flagging — R19’s retire-by-hand steps — more than a handful of bots.
- More Telegram-facing bots (inbox, homelab, spec-writer) — one token each and the allow-list (R21) — a bot proven in practice.
- Off-site backups (Example: a B2 bucket) — backup target setting — JARVIS holds data worth keeping.
- Local read-only Git mirror — Flux source — offline rebuilds wanted.
- LLM tracing (Langfuse, hermes_otel) — Hermes analytics meanwhile — more RAM or a cost puzzle.
- Postgres (CloudNativePG) — none; Hermes uses SQLite — a component needs it.
- Private repos — R25 keeps them ready; while repos are public the ops deploy holds only the no-repos public-read-only GitHub token (R13), and only once they go private is that token widened to read-only on the private repos for the PR read skill, jarvis is set to let jarvis-apps call its reusable workflows (R42), and Flux gets read-only deploy keys and a pull secret (R25); free GitHub enforces branch protection, and so admin-gate, only on public repos, so going private needs GitHub Pro or accepting an unenforced gate; handle later — Gordon decides.
- promptfoo — swap boundary in §4 — a stated need.

## 10. Open questions and risks

### Open questions (all closed; OQ7 resolves as a Phase 0 check)

OQ1. Closed 2026-10-19, approved by Gordon: approval gates (the admin-gate check, R36) switch on at the P2 exit.

OQ2. Closed 2026-10-19, approved by Gordon: Gordon is the only user at launch (§1 Users).

OQ3. Closed 2026-10-19, approved by Gordon: the initial build runs on a separate spare compute node (env-01); the previous system is not shut down or destroyed, and all work on it is frozen from 2026-10-19; only the env-01 node is wiped (§8).

OQ4. Closed 2026-10-19, approved by Gordon: homelab proposes fixes only as chat messages (what is wrong, a likely cause, a suggested fix) and never files issues; the ops deploy holds no GitHub credential (R13, R15); spec-writer creates issues in jarvis-apps through the code-factory GitHub App, without code-factory:ready. (amended 2026-10-19: ops now holds the no-repos public-read-only token, R13)

OQ5. Closed 2026-10-19, approved by Gordon: start minimal; concierge is the only Telegram-facing bot at first, and more are added once proven in practice (R21). Bot tier map: inbox reply drafting and homelab chat replies run on mid (R23).

OQ6. Closed 2026-10-19, approved by Gordon: hosted Renovate app; a send-only Telegram alerts bot; Grafana-managed metric alerts; evals after deploy (R38); internal CA root key kept SOPS-encrypted; spec-writer files issues without code-factory:ready; Hermes soak and Renovate auto-merge (R33); retention (§7); the bot tier map (R23). Remote access is router-owned (R28).

OQ7. Closed 2026-10-19 as a Phase 0 check (resolved by the test; the fallback is the default): Phase 0’s first backup step is a ~10-minute VolSync backup-and-restore test to the local backup path; if it fails, use the single restic CronJob fallback, no further searching (R31). No local S3 server.

OQ8. Closed 2026-10-19, approved by Gordon: plain Kubernetes Ingress on k3s-bundled Traefik; no Gateway API (§4).

OQ9. Closed 2026-10-19, approved by Gordon 2026-10-19: logs = Fluent Bit → VictoriaLogs; metrics = vmagent + node-exporter + kube-state-metrics → VictoriaMetrics; Grafana. No fallbacks (recorded in §4).

OQ10. Closed 2026-10-19, approved by Gordon: node OS = Ubuntu 26.04 Server, minimal install, fresh install; JARVIS local DNS = CoreDNS with the k8s_gateway plugin on the edge nodes (D42), assuming the router forwards alpha.lan lookups (env-01’s DNS suffix, R28) to it (router config is Gordon-owned); GHCR as the registry for images CI builds from jarvis-apps (recorded in §4).

### Risks

- Hermes upgrade breakage — R33; the runtime stays swappable.
- SQLite corruption and a root-starting image — one volume per deploy, pre-upgrade backup, R13 offsets.
- Prompt injection through email or web — read-only tools, R13, R15.
- Subscription pool use (code-factory shares Gordon’s personal weekly pool; accepted) — Hermes loop hard stops, cheap tiers, Cursor auto top-up off with a monthly limit.
- xAI returns 403 to some accounts on xai-oauth — documented fix: one xAI API key per deploy; a contingency, not built.
- Memory rot (small memory cap) — memory.write_approval, short facts.
- age key loss means no rebuild — R25.
- Internal CA root loss means re-trusting every device — root kept in SOPS.
- Router outage or misconfiguration cuts LAN and tailnet access and DNS — external dependency, Gordon-owned (§8); nothing on the nodes to fix.
- Single disk full or failed, local-only backups — retention, 80% alert, honest R31 limits; off-site deferred.
- Telegram token as an access path — R21 allow-list.
- Telegram or internet outage stops alerts — accepted; the Gatus page stays readable on LAN.
- Gmail token scope (gmail.compose for drafts) also permits send — offsets: no send tool, the P3 no-send eval.
- Previous and new systems run side by side — the previous system stays up but frozen; the new build uses only its own new tokens, keys and webhooks (§8).
- Public repos leak something — R25, secret scan.
- GitHub outage blocks deploys (running JARVIS is unaffected) — accepted; mirror deferred.
- Free public data APIs (Open-Meteo, Nominatim, OSRM demo) have no SLA — answers degrade to “unavailable”; usage stays within their policies.
- A registry outage or rate limit during a rebuild delays it — accepted; retry.
- Sandbox spend (Cursor pool and Claude usage, Actions minutes) — job time limits; Cursor auto top-up off with a monthly limit.
- code-factory:hard (Claude through Cursor) may bill at API price outside the subscription pool, depending on Gordon’s Cursor plan — check the plan before P4b; offset: Cursor’s monthly limit with auto top-up off.
- Unrestricted runner egress — accepted limit: GitHub-hosted runners cannot restrict egress; offsets in R42, egress allow-listing arrives with in-cluster gVisor (§9).
- Vendor shifts — swap boundaries; plain open-source licenses.

## 11. Rules for the plan

- Plan: Cursor writes jarvis-alphacut/plan.md in project-plans, one section per checkpoint (P0a, P0b, P1, P2, P3, P4a, P4b, as outlined in §2), with every task in the brief template; Gordon merges it, and coding agents build from the merged plan.
- Must require: every plan task written in the coding-agent brief template (goal, why, hard limits, done checks), and a single list of Gordon-only actions (§8).
- Brief template: the core above is required. Also add an ID, depends-on (task IDs) and a short “touches” (rough area or paths; a best guess is fine). These are guidance, not rigid fields.
- Traceability: each task cites the exit check(s) it advances.
- Each checkpoint: the §5 contracts tagged with it are written first as CONTRACT.md with “proven by”; its exit checks stay verbatim, with [G] kept; done = attached make verify PHASE=<checkpoint> output. Rules are cited by surviving number.
- Task size: each task fits one PR (split anything over ~1 day).
- Lean standard (the coding-agent rulebook carries it): build only what the task’s done check needs, and build it well. Tests encode a rule or a behavior, never a copied file or task list, and never re-run what CI already runs. The live run is the proof where there is one. Ansible behavior is proven by localhost plays with fake binaries, not container tests, and hardware proofs (Example: chrony NTS) are checks in the live owner run. Prefer a config file’s own validation (Ansible validate:). No local tool that duplicates a hosted service. Hardening ideas go in the PR report as notes, not code. Keep every cheap check that guards a rule already in force.
- Code design (the same rulebook carries it): Ansible uses modules and facts; shell only where no module exists. Required config fails loudly; a default lives once, in the env schema, and role defaults/ hold nothing that looks like env config (product constants go in vars/, D46). Tags sit at role or play level. Tests are named for the behavior they prove and never match file text. Verify scripts check and fail; they never echo. No person’s name and no environment-identity value appears in core (D44); env-00 is the multi-node fixture (D43).
- Review: design is judged first. A bad design (wrong data model, logic in the wrong layer, duplicated config, echo-only checks) is redone, not patched. A sound design gets one fix round; after it, only real bugs block and the rest become notes. Reviewers block an unexplained file move and the removal of a check for a rule in force. Small low-risk PRs merge on reviewer approval plus green CI; docs, plan, design, infra and risky PRs go to Gordon first (R36 adds admin-gate from the P2 exit).
- Fix order for repeated agent mistakes: fix it in the code first (clear boundaries, typed contracts, make the easy path the right path), then in automated checks (lint, tests, CI), then in prose (AGENTS.md, only for what can’t be enforced), and use human review last. Most effort goes at the top, least at the bottom.
- Build on this framework and the §4 stack; don’t import picks from earlier systems. Anything new the plan needs beyond §4 is fine, recorded as a default (next point).
- Conflicts and gaps: a contradiction in this framework, or a rule that blocks an exit check, means stop and ask Gordon: list it under “Stop and ask” at the top of plan.md, and still deliver the rest of the plan in the PR. Where the framework is silent, choose the simplest option that fits the rules and §4, record it in jarvis-alphacut/decisions.md (by PR; Gordon merges) as “default, not yet accepted”, and list it in §10 Open questions.
- Plan done: every exit check is covered by at least one task, every Gordon-only action is tied to the task that needs it, and every §5 contract has a task.
- Done when: Gordon merges this framework, every open question is decided or has an accepted default, and Cursor can derive a phased plan with tasks and verification steps from it alone.

Example task (placeholders only):

```text
ID:          <checkpoint>-<nn>
Goal:        <one outcome, in a sentence>
Why:         <the need it serves; cite R<n>>
Advances:    <checkpoint> exit check: “<check text, verbatim>”
Depends on:  <task IDs, or none>
Touches:     <rough area or paths, best guess>
Hard limits: <rules and boundaries it must not cross>
Done checks: <the one proof that shows the outcome: a live run, a seeded CI rule check, or a small behavior test>
```

## Appendix — Eval seeds

B2’s tagged example interactions, carried over verbatim; personal names are placeholders (<contact>, <doc>, <trip>) because project-plans is public.

Tag: (phase, bot, read / draft / memory / reminder / refuse / later).

- “What’s the weather in Dallas today?” (P2, concierge, read)
- “How far is it to LAX from here?” (P2, concierge, read; home from the rounded SOPS value; typical OSRM drive time without traffic, with attribution)
- “What do I need to know?” “Brief me on today.” “Prep me for my 2pm meeting.” (P3, concierge, read; also the scheduled brief, wrap and weekly review)
- “What’s important in my inbox today?” (P3, inbox, read: ranks unread email in chat; applies no labels)
- “Draft a reply to that email, but don’t send it.” (P3, inbox, draft)
- “Send an email to <contact> with the Drive doc called <doc>.” (P3, inbox, draft: drafts it without the doc, says Drive isn’t connected, and sends nothing; confirm-first send is later)
- “Create a doc outlining JARVIS’s features and drop it into Google Drive.” (P3, concierge, refuse: says Drive isn’t connected)
- “Add <contact>’s doctor appointment tomorrow at 5pm.” (P3, concierge, refuse: says calendar writes aren’t available yet)
- “Remind me to take out the trash tomorrow.” (P2, concierge, reminder)
- “What did I ask you to remember about the <trip>?” “Forget that.” (P2, concierge, memory)
- “Anything wrong with the cluster today?” “Which service uses the most memory?” “What changed this week?” (P2, homelab, read; proposes a fix in chat, never files or applies it)
- Unprompted: “The Renovate PR bumping Grafana keeps failing CI: <error>. Suggested fix: <fix>.” (P3, homelab, read; scheduled “watch for stuck updates” check on the public repos with the no-repos public-read-only token; lands in homelab’s dashboard session and as a “stuck updates” line in concierge’s morning brief)
- “What PRs need my attention?” (P3, concierge, read)
- “What did the code-factory finish overnight?” (P4b, concierge, read)
- “Turn this idea into a code-factory issue.” (P4b, spec-writer, draft: files the issue after Gordon confirms, without code-factory:ready; Gordon labels it)
- “Deploy Gods Eye View as an app.” (P4b, spec-writer then foreman, draft: spec-writer files the issue after Gordon confirms; once Gordon labels it code-factory:ready, the lane builds it and the publish job opens a draft PR adding apps/gods-eye-view to jarvis-apps; foreman reports the PR and asks Gordon to merge it and enable it in apps.yaml; reports app-gods-eye-view.alpha.lan once live)
- “Spin up a demo site for this business.” (P4b, spec-writer then foreman, draft: same flow; reports app-<name>.alpha.lan once live; LAN or tailnet only)
