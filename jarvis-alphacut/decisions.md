# JARVIS AlphaCut — Decisions

Defaults chosen where `framework.md` is silent. Each entry is a default, not yet accepted. Gordon accepts one by merging a change that says so.

These are the §11 defaults. They are recorded here. They are not copied into framework §10: this change adds only this file and `plan.md`.

## D1

- Decision: `jarvis` and `jarvis-apps` are created under the GitHub user `gordoncooper` (the user that owns project-plans).
- Reason: R36 needs `admin-approved` from Gordon’s own account, and this repo is already on that user. A personal public user matches free branch protection.
- Where: P0 repo creation (`p0a-03`, `p0a-06`).
- Status: default, not yet accepted.

## D2

- Decision: the env-01 Ansible hostname is `env-01`. If Ansible cannot resolve it, the IP goes in `group_vars/*.sops.yaml` only.
- Reason: the inventory lists hostnames only (R25) and no hostname is given.
- Where: `envs/env-01/inventory` (`p0a-03`, `p0a-12`).
- Status: default, not yet accepted.

## D3

- Decision: local-path data is `/var/lib/jarvis/local-path`. The backup path is `/var/lib/jarvis/backup`.
- Reason: both directories are required and neither path is named.
- Where: Ansible, VolSync, etcd snapshots (`p0a-12`, `p0b-09`).
- Status: default, not yet accepted.

## D4

- Decision: pod and service CIDRs are the built-in defaults of the pinned k3s version, written once into `cluster-vars.yaml` and `/etc/rancher/k3s/config.yaml`.
- Reason: CIDRs are set once and no values are given.
- Where: k3s role (`p0a-12`).
- Status: default, not yet accepted.

## D5

- Decision: the k3s version is pinned in `cluster-vars.yaml` and changes only by PR. Renovate does not bump it.
- Reason: R33 covers container images. The host binary has no stated updater.
- Where: k3s role (`p0a-12`).
- Status: default, not yet accepted.

## D6

- Decision: etcd snapshot interval and retention are the built-in defaults of that same k3s version. The snapshot directory is the backup path (D3).
- Reason: a schedule is required and no numbers are given. Snapshots are for same-node rollback only (R31).
- Where: k3s config (`p0a-12`, `p0b-10`).
- Status: default, not yet accepted.

## D7

- Decision: §5 contract files live at these paths. Two contracts that share a folder are separate sections of that one `CONTRACT.md`. Component folders named in §6 still get their own `CONTRACT.md` when the component is built.
  - Env schema and profile schema: `services/hermes/core/schema/CONTRACT.md`
  - Code delivery provider: `docs/code-delivery/CONTRACT.md`
  - Build agent: `docs/build-agent/CONTRACT.md`
  - Exposure: `platform/traefik/CONTRACT.md` (cert-manager and lan-dns component contracts point at it)
  - Backups: `platform/backups/CONTRACT.md`
  - Login and alert path: `services/observability/CONTRACT.md`
  - Model provider, assistant API, and skill contract: `services/hermes/core/CONTRACT.md`
  - Sandbox: `services/hermes/deploy/code-factory/sandbox/CONTRACT.md`
  - Code-factory work protocol: `services/hermes/deploy/code-factory/CONTRACT.md`
  - App hosting: `platform/app-hosting/CONTRACT.md`
- Reason: §5 requires a `CONTRACT.md` from one template, and the layout names only the code-delivery and build-agent paths.
- Where: every contract task.
- Status: default, not yet accepted.

## D8

- Decision: the R2 check treats a component root as a §6 folder the layout assigns `CONTRACT.md`. Nested directories are not component roots.
- Reason: the rule requires the file and does not define the folder set. A check on every directory would fail on `skills/` and `tests/`.
- Where: `tools/checks` (`p0a-08`).
- Status: default, not yet accepted.

## D9

- Decision: a Flux path with no component yet contains one ConfigMap named `placeholder`, so the Kustomization is Ready. The task that adds the component removes that ConfigMap.
- Reason: P0a requires every Flux object Ready before Traefik, observability, and Hermes exist.
- Where: `envs/env-01/flux` (`p0a-14` and later component tasks).
- Status: default, not yet accepted.

## D10

- Decision: the P0a decrypt proof is the VolSync restic Secret in namespace `platform-backups`. Other bootstrap Secrets stay in Git and are referenced by a Kustomization only once their namespace exists.
- Reason: bootstrap-secrets writes every §8 bootstrap entry in one run, and a Secret in a missing namespace would leave a Kustomization unready.
- Where: `p0a-04`, `p0a-14`.
- Status: default, not yet accepted.

## D11

- Decision: the first Flux Operator chart pin is one release behind current, so Renovate has a PR to open. Merging that PR is allowed afterward.
- Reason: the P0a check requires an open Renovate PR, and a fully current pin set can open none.
- Where: the Flux Operator pin in `p0a-14`, opened as a PR by `p0a-15`.
- Status: default, not yet accepted.

## D12

- Decision: the P0b test app is namespace `platform-testapp`, hostname `testapp.alpha.lan`, image `docker.io/library/nginx` pinned by digest, serving a static file with a known checksum. It stays up so P1 can kill it and restore it.
- Reason: the reachability check needs a test app, and §4 has no HTTP fixture. The image is a default beyond §4 (R9); this entry is the record.
- Where: `p0b-08`, `p1-08`, `p1-11`.
- Status: default, not yet accepted.

## D13

- Decision: Gatus checks every 60s and alerts after 2 consecutive failures.
- Reason: the P1 check requires a Telegram alert within 5 minutes, and no interval is given.
- Where: Gatus config (`p1-08`, `p1-11`).
- Status: default, not yet accepted.

## D14

- Decision: the clock alert fires when chrony reports the clock is not synchronised.
- Reason: the alert is required and no offset threshold is given.
- Where: Grafana alert rules (`p1-10`).
- Status: default, not yet accepted.

## D15

- Decision: restic’s weekly prune runs Sunday at 04:00 in the node’s local time.
- Reason: retention includes a weekly prune and no clock is given (§7).
- Where: backup config (`p0b-09`).
- Status: default, not yet accepted.

## D16

- Decision: Grafana and both Hermes dashboards use username `admin`. Passwords, the Hermes password hash, the signing secret, and the API key come from bootstrap-secrets.
- Reason: R27 requires a built-in login and names no username.
- Where: bootstrap-secrets, login contract (`p0a-04`, `p1-01`, `p2-05`).
- Status: default, not yet accepted.

## D17

- Decision: `make verify PHASE=<phase>` runs every check the agent user can run from the bastion. Commands only Gordon can run, and every `[G]` check, print a `NEEDS GORDON` line. The attached output includes his pasted result. A missing paste fails the run.
- Reason: §2 requires attached verify output, and reserves `[G]` checks and `site.yml` for Gordon. The agent user has no SSH key (R25, §8).
- Where: every phase verify task.
- Status: default, not yet accepted.

## D18

- Decision: the usage check asserts Hermes analytics returns usage for the ops deploy. Gordon pastes one line that the grok.com dashboard shows usage for the same period.
- Reason: the P2 check names both surfaces, and agents have no grok.com session (R23).
- Where: `p2-19`.
- Status: default, not yet accepted.

## D19

- Decision: the revoked-login and provider-down drills use a reversible test override (an unreachable address, and a 401 fixture for the revoked or limited login), then restore `xai-oauth`. The real subscription stays signed in.
- Reason: the P2 check requires those failures (R18). Revoking the real login would destroy the device token.
- Where: `p2-18`.
- Status: default, not yet accepted.

## D20

- Decision: the primary-blocked drill sets the primary model to a name the provider rejects and leaves the fallback model in place, then restores the tier map.
- Reason: the P2 check requires a fallback answer and does not say how to block the primary.
- Where: `p2-18`.
- Status: default, not yet accepted.

## D21

- Decision: cron times use the node’s local clock. homelab stuck-updates at 06:30 daily. Morning brief at 07:00 daily. Evening wrap at 18:00 daily. Weekly review Monday at 07:30.
- Reason: the jobs must arrive at a scheduled time and no clock is given. Stuck-updates runs before the brief so the brief can carry its line.
- Where: `envs/env-01/values/hermes.yaml` (`p3-03`, `p3-04`).
- Status: default, not yet accepted.

## D22

- Decision: unanswered email means unread threads in the inbox. Rank is the cheap-tier model’s importance order. The rank is shown in chat and is not written to Gmail.
- Reason: the wrap and the inbox triage need a ranking, and no ranking rule is given (R15).
- Where: `p3-01`, `p3-03`.
- Status: default, not yet accepted.

## D23

- Decision: the stuck-update suggested fix is the body of the `::error` annotation. The seeded failure prints `::error::suggested fix: <text>`.
- Reason: the finding needs a suggested fix, and the annotation is the signal CI is allowed to leave.
- Where: `p3-04`.
- Status: default, not yet accepted.

## D24

- Decision: during `make google-consent`, Gordon sends one message to himself with subject `jarvis-p3-seed`. The draft check uses that thread.
- Reason: the draft check needs a seeded thread. The Gmail token is read plus drafts, and the bot must not send (R15).
- Where: `p3-06`, Gordon action for `p3-01`.
- Status: default, not yet accepted.

## D25

- Decision: the P4a app is `jarvis-apps/apps/demo`, own code, Python standard-library HTTP on port 8080, health path `/health`.
- Reason: P4a needs one outside app and names none. Own code in jarvis-apps is the smaller path.
- Where: `p4a-04`, `p4a-07`.
- Status: default, not yet accepted.

## D26

- Decision: the fix cap is 2 total lane runs on an issue (the first run and one fix). The next failing checker verdict applies `code-factory:failed` and stops.
- Reason: R40 requires a cap and sets no number. Two runs keep the seeded loop short.
- Where: `p4b-07`, `p4b-13`.
- Status: default, not yet accepted.

## D27

- Decision: the lane timeout default is 30 minutes. The timeout seed passes a 1-minute timeout.
- Reason: R42 requires a time limit and sets none. The seed must finish inside a verify run.
- Where: `p4b-06`, `p4b-14`.
- Status: default, not yet accepted.

## D28

- Decision: lane family strings are `composer` (Composer 2.5), `grok` (Grok 4.7), and `anthropic` (Claude for `code-factory:hard`).
- Reason: R41 compares families, and §7 names the models only.
- Where: `hermes.yaml` lane map (`p4b-04`, `p4b-08`).
- Status: default, not yet accepted.

## D29

- Decision: homelab reads metrics through an in-repo read-only Python MCP at `services/hermes/core/skills/metrics/` against VictoriaMetrics. Cluster and Flux reads use a digest-or-version-pinned `kubernetes-mcp-server` in read-only mode.
- Reason: the P2 homelab seeds ask which service uses the most memory. §4 names the Kubernetes MCP and names no metrics MCP.
- Where: `p2-12`.
- Status: default, not yet accepted.

## D30

- Decision: container memory stays inside the §7 budgets. Starting requests: VictoriaMetrics 512Mi, VictoriaLogs 512Mi, Grafana 256Mi, Gatus 128Mi, Fluent Bit 64Mi, vmagent 128Mi, node-exporter 64Mi, kube-state-metrics 64Mi, Hermes ops 2Gi with a 3Gi limit, Hermes code-factory 1Gi with a 2Gi limit, each platform container 128Mi request and 256Mi limit. A platform container may be raised only while the platform total stays near 2Gi. Every container has a CPU request and no CPU limit.
- Reason: R5 requires numbers and §7 gives budgets, not per-container sizes.
- Where: every manifest task.
- Status: default, not yet accepted.

## D31

- Decision: the jarvis-apps cluster-scope check is `jarvis-apps/.github/checks/namespaced-only.sh`, called from `ci.yml`.
- Reason: R30 requires the check, and the layout names `ci.yml` only.
- Where: `p4a-05`.
- Status: default, not yet accepted.

## D32

- Decision: the Ingress class is the class the bundled Traefik chart already registers. The install task copies that name into `cluster-vars.yaml`.
- Reason: OQ8 requires plain Ingress and does not name the class.
- Where: `p0b-03`.
- Status: default, not yet accepted.

## D33

- Decision: CI tests for `bootstrap-secrets` generate a throwaway age key inside the job. The real private key is never in CI.
- Reason: the script needs a test (R7, R35), and R25 keeps the real key off CI and off the repo.
- Where: `p0a-04`.
- Status: default, not yet accepted.

## D34

- Decision: the Ubuntu install creates SSH user `gordon` with Gordon’s SSH key. Ansible uses that user.
- Reason: `site.yml` needs a login and none is named. Gordon’s user is the one that runs `site.yml` (§8).
- Where: `p0a-12`.
- Status: default, not yet accepted.

## D35

- Decision: `jarvis-apps` gets a green `ci` workflow in P0a so branch protection has a check to report. P4a extends that same workflow with the image build, the namespaced-object check, and the admin-gate caller.
- Reason: P0 requires branch protection with check `ci` on both repos, and jarvis-apps otherwise has no workflow until P4a.
- Where: `p0a-06`, `p4a-04`, `p4a-05`.
- Status: default, not yet accepted.

## D36

- Decision: the in-repo geo and metrics MCP servers use the Hermes image’s Python standard library only.
- Reason: they run on the image’s Python (§5). A package download at start is what pinning avoids for in-repo servers.
- Where: `p2-11`, `p2-12`.
- Status: default, not yet accepted.

## D37

- Decision: foreman stores the last reported GitHub `updated` time on its volume and skips the model call when the public issue and PR list is unchanged.
- Reason: R40 requires the skip and does not say how to detect “nothing changed”.
- Where: `p4b-09`.
- Status: default, not yet accepted.

## D38

- Decision: admin-gate’s bot and App actor cases are proven by a fixture event (actor type Bot, and actor type App) run in CI against the same function the workflow calls. The live PR proves a label from `gordon5417-ai` fails, and a label Gordon’s own account adds after the latest commit passes.
- Reason: the P2 check requires a bot or App actor, and the code-factory App is created at P4 (§8).
- Where: `p2-23`.
- Status: default, not yet accepted.
