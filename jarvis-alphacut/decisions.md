# JARVIS AlphaCut — Decisions

Defaults chosen where `framework.md` is silent. Merging this file accepts all defaults except those marked Confirmed by Gordon.

Confirmed by Gordon 2026-10-19: D1, D2, D4, D19, D21, D22, D24, D26, D27, D34, D38, D39, D40, D41. The other entries are accepted-with-plan defaults.

These are the §11 defaults. They are recorded here. They are not copied into framework §10: this change adds only this file and `plan.md`.

## D1

- Decision: `jarvis` and `jarvis-apps` are created under the GitHub user `gordoncooper` (the user that owns project-plans).
- Reason: R36 needs `admin-approved` from Gordon’s own account, and this repo is already on that user. A personal public user matches free branch protection.
- Where: P0 repo creation (`p0a-03`, `p0a-06`).
- Status: Confirmed by Gordon 2026-10-19.

## D2

- Decision: the env-01 Ansible inventory host is `bench.lan`. The env stays named env-01.
- Reason: it is the name the LAN DNS resolves, so no IP or alias is needed, and the inventory holds no IPs (R25).
- Where: `envs/env-01/inventory/hosts.yml` (`p0a-03`, `p0a-12`).
- Status: Confirmed by Gordon 2026-10-19.

## D3

- Decision: local-path data is `/var/lib/jarvis/local-path`. The backup path is `/var/lib/jarvis/backup`. When a node has a data disk, `/var/lib/jarvis` is its mount point, so the paths stay the same (D42).
- Reason: both directories are required and neither path is named.
- Where: Ansible, VolSync, etcd snapshots (`p0a-12`, `p0b-09`).
- Status: accepted-with-plan default.

## D4

- Decision: pod and service CIDRs are the built-in defaults of the pinned k3s version, written once into `cluster-vars.yaml` and `/etc/rancher/k3s/config.yaml`.
- Reason: CIDRs are set once and no values are given.
- Where: k3s role (`p0a-12`).
- Status: Confirmed by Gordon 2026-10-19.

## D5

- Decision: the k3s version is pinned in `cluster-vars.yaml` and changes only by PR. Renovate does not manage it.
- Reason: R33 requires that pin. Renovate manages image digests and does not manage the k3s host binary.
- Where: k3s role (`p0a-12`).
- Status: accepted-with-plan default.

## D6

- Decision: etcd snapshot interval and retention are the built-in defaults of that same k3s version. The snapshot directory is the backup path (D3).
- Reason: a schedule is required and no numbers are given. Snapshots are for same-node rollback only (R31).
- Where: k3s config (`p0a-12`, `p0b-10`).
- Status: accepted-with-plan default.

## D7

- Decision: §5 contract files live at these paths. Two contracts that share a folder are separate sections of that one `CONTRACT.md`. Component folders named in §6 still get their own `CONTRACT.md` when the component is built.
  - Env schema and profile schema: `services/hermes/core/schema/CONTRACT.md`
  - Code delivery provider: `docs/code-delivery/CONTRACT.md`
  - Build agent: `docs/build-agent/CONTRACT.md`
  - Exposure: `platform/traefik/CONTRACT.md` (cert-manager and lan-dns component contracts point at it)
  - Backups: `platform/backups/CONTRACT.md`
  - Login and alert path: `services/observability/CONTRACT.md`
  - Model provider, assistant API, and skill contract: `services/hermes/core/CONTRACT.md`
  - Each Hermes deploy (`services/hermes/deploy/ops`, `services/hermes/deploy/code-factory`) is a component root; its `CONTRACT.md` lists the R13 egress hosts it uses.
  - Sandbox: `services/hermes/deploy/code-factory/sandbox/CONTRACT.md`
  - Code-factory work protocol: `services/hermes/deploy/code-factory/CONTRACT.md`
  - App hosting: `platform/app-hosting/CONTRACT.md`
- Reason: §5 requires a `CONTRACT.md` from one template, and the layout names only the code-delivery and build-agent paths.
- Where: every contract task.
- Status: accepted-with-plan default.

## D8

- Decision: the R2 check treats a component root as a §6 folder the layout assigns `CONTRACT.md`. Every direct child of `platform/` is a root. Nested directories are not component roots, and a parent that holds only nested roots and no files of its own is skipped until it is built.
- Reason: the rule requires the file and does not define the folder set. A check on every directory would fail on `skills/` and `tests/`.
- Where: `tools/checks` (`p0a-08`).
- Status: accepted-with-plan default.

## D9

- Decision: a Flux path with no component yet contains one ConfigMap named `placeholder-<layer>` (unique per layer, since two layers can share a namespace), so the Kustomization is Ready. The task that adds the component removes that ConfigMap.
- Reason: P0a requires every Flux object Ready before Traefik, observability, and Hermes exist.
- Where: `envs/env-01/flux` (`p0a-14` and later component tasks).
- Status: accepted-with-plan default.

## D10

- Decision: the P0a decrypt proof is the VolSync restic Secret in namespace `platform-backups`. Other bootstrap Secrets stay in Git and are referenced by a Kustomization only once their namespace exists.
- Reason: bootstrap-secrets writes every §8 bootstrap entry in one run, and a Secret in a missing namespace would leave a Kustomization unready.
- Where: `p0a-04`, `p0a-14`.
- Status: accepted-with-plan default.

## D11

- Decision: the first Flux Operator chart pin is one release behind current, so Renovate has a PR to open. Merging that PR is allowed afterward.
- Reason: it gives Renovate a real dependency PR to open, which p3-04 can reuse as a stuck-update seed. The P0a check also accepts Renovate’s onboarding PR.
- Where: the Flux Operator pin in `p0a-14`, opened as a PR by `p0a-15`.
- Status: accepted-with-plan default.

## D12

- Decision: the P0b test app is namespace `platform-testapp`, hostname `testapp.alpha.lan`, image `docker.io/library/nginx` pinned by digest, serving a static file with a known checksum. It stays up so P1 can kill it and restore it.
- Reason: the reachability check needs a test app, and §4 has no HTTP fixture. The image is a default beyond §4 (R9); this entry is the record.
- Where: `p0b-08`, `p1-08`, `p1-11`.
- Status: accepted-with-plan default.

## D13

- Decision: Gatus checks every 60s and alerts after 2 consecutive failures.
- Reason: the P1 check requires a Telegram alert within 5 minutes, and no interval is given.
- Where: Gatus config (`p1-08`, `p1-11`).
- Status: accepted-with-plan default.

## D14

- Decision: the clock alert fires when chrony reports the clock is not synchronised.
- Reason: the alert is required and no offset threshold is given.
- Where: Grafana alert rules (`p1-10`).
- Status: accepted-with-plan default.

## D15

- Decision: restic’s weekly prune runs Sunday at 04:00 America/Los_Angeles. VolSync schedules take TZ from cluster-vars.
- Reason: retention includes a weekly prune and no clock is given (§7). env-01’s time zone is America/Los_Angeles (D21).
- Where: backup config (`p0b-09`), Hermes pod (`p2-09`).
- Status: accepted-with-plan default.

## D16

- Decision: Grafana and both Hermes dashboards use username `admin`. Passwords, the Hermes password hash, the signing secret, and the API key come from bootstrap-secrets.
- Reason: R27 requires a built-in login and names no username.
- Where: bootstrap-secrets, login contract (`p0a-04`, `p1-01`, `p2-05`).
- Status: accepted-with-plan default.

## D17

- Decision: `make verify PHASE=<phase>` (`tools/verify/<phase>`, short, no tests of its own) checks only what a machine can check from the controller as the agent user. It waits for or reads live state (for example `kubectl wait --for=condition=Ready` on Flux objects) and compares it with the repo (for example node role labels against inventory groups, D42). It exits non-zero on any miss, including when it finds zero objects, and prints no reminder or placeholder lines. Everything else is the owner checklist written in that phase’s verify step in the plan: owner runs, `[G]` checks and evidence links such as seed PRs. Gordon pastes those results with the attached script output, and his review is the gate. Steps that need the age key (a full `site.yml`, `bootstrap-secrets`) are owner runs by Gordon. The agent user keeps node SSH only for narrow runs that need no age key (for example `site.yml --tags kubeconfig`). The “another ID gets none” check uses a gateway-handler fixture for the non-allow-listed ID.
- Reason: §2 requires attached verify output, and reserves `[G]` checks and full `site.yml` runs for Gordon. A script that echoes reminders proves nothing, and no person’s name belongs in the repo (§11 code design). The age key is never on the agent user (R25, §8). No second Telegram account is available, so the non-allow-listed ID is a fixture.
- Where: every phase verify task.
- Status: accepted-with-plan default.

## D18

- Decision: the usage check asserts Hermes analytics returns usage for the ops deploy. Gordon pastes one line that the grok.com dashboard shows usage for the same period.
- Reason: the P2 check names both surfaces, and agents have no grok.com session (R23).
- Where: `p2-19`.
- Status: accepted-with-plan default.

## D19

- Decision: the revoked-login and provider-down drills use a reversible test override (an unreachable address, and a 401 fixture for the revoked or limited login), then restore `xai-oauth`. The real subscription stays signed in.
- Reason: the P2 check requires those failures (R18). Revoking the real login would destroy the device token.
- Where: `p2-18`.
- Status: Confirmed by Gordon 2026-10-19.

## D20

- Decision: the primary-blocked drill sets the primary model to a name the provider rejects and leaves the fallback model in place, then restores the tier map.
- Reason: the P2 check requires a fallback answer and does not say how to block the primary.
- Where: `p2-18`.
- Status: accepted-with-plan default.

## D21

- Decision: cron times use America/Los_Angeles. homelab stuck-updates at 06:30 daily. Morning brief at 07:00 daily. Evening wrap at 18:00 daily. Weekly review Monday at 07:30.
- Reason: env-01 runs on America/Los_Angeles, and the time zone is part of the env schema. The node and the Hermes schedules both use it. Stuck-updates runs before the brief so the brief can carry its line.
- Where: `p0a-01`, `p0a-11`, `envs/env-01/values/hermes.yaml` (`p3-03`, `p3-04`).
- Status: Confirmed by Gordon 2026-10-19.

## D22

- Decision: unanswered email means unread threads in the inbox. Rank is the cheap-tier model’s importance order. The rank is shown in chat and is not written to Gmail.
- Reason: the wrap and the inbox triage need a ranking, and no ranking rule is given (R15).
- Where: `p3-01a`, `p3-03`.
- Status: Confirmed by Gordon 2026-10-19.

## D23

- Decision: the seeded failure prints only the `::error`. homelab writes the suggested fix.
- Reason: CI may leave the error annotation. The suggested fix is homelab’s chat note (R15), not a second annotation.
- Where: `p3-04`.
- Status: accepted-with-plan default.

## D24

- Decision: during `make google-consent`, Gordon sends one message to himself with subject `jarvis-p3-seed`. The draft check uses that thread.
- Reason: the draft check needs a seeded thread. The Gmail token is read plus drafts, and the bot must not send (R15).
- Where: `p3-06`, Gordon action for `p3-01a`.
- Status: Confirmed by Gordon 2026-10-19.

## D25

- Decision: the P4a app is `jarvis-apps/apps/demo`, own code, Python standard-library HTTP on port 8080, health path `/health`.
- Reason: P4a needs one outside app and names none. Own code in jarvis-apps is the smaller path.
- Where: `p4a-04`, `p4a-07`.
- Status: accepted-with-plan default.

## D26

- Decision: one workflow invocation does builder, then checker, then on fail one builder fix with the checker’s notes, then checker, then publish draft. A second fail or the 30-minute job timeout gets `code-factory:failed`.
- Reason: R40 requires a cap. One invocation with one fix is that cap. A second checker fail stops the loop.
- Where: `p4b-07`, `p4b-13`.
- Status: Confirmed by Gordon 2026-10-19.

## D27

- Decision: the lane timeout default is 30 minutes. The timeout seed passes a 1-minute timeout.
- Reason: R42 requires a time limit and sets none. The seed must finish inside a verify run.
- Where: `p4b-06a`, `p4b-14`.
- Status: Confirmed by Gordon 2026-10-19.

## D28

- Decision: lane family strings are `composer` (Composer 2.5), `grok` (Grok 4.7), and `anthropic` (Claude for `code-factory:hard`).
- Reason: R41 compares families, and §7 names the models only.
- Where: `hermes.yaml` lane map (`p4b-04`, `p4b-08`).
- Status: accepted-with-plan default.

## D29

- Decision: homelab reads metrics through mcp-victoriametrics, the official VictoriaMetrics MCP server, read-only, as a pinned release binary on the deploy volume. Cluster and Flux reads use a digest-or-version-pinned `kubernetes-mcp-server` in read-only mode.
- Reason: the P2 homelab seeds ask which service uses the most memory. §4 names mcp-victoriametrics for those reads and the Kubernetes MCP for cluster and Flux reads.
- Where: `p2-12`.
- Status: accepted-with-plan default.

## D30

- Decision: every container starts from these memory requests and is tuned from Grafana as real usage shows up: VictoriaMetrics 512Mi, VictoriaLogs 512Mi, Grafana 256Mi, Gatus 128Mi, Fluent Bit 64Mi, vmagent 128Mi, node-exporter 64Mi, kube-state-metrics 64Mi, Hermes ops 2Gi with a 3Gi limit, Hermes code-factory 1Gi with a 2Gi limit, each platform container 128Mi request and 256Mi limit. Every container has a memory request and limit and a CPU request, and no CPU limit.
- Reason: R5 requires numbers; real usage, not up-front calculation, sets the final sizes.
- Where: every manifest task.
- Status: accepted-with-plan default.

## D31

- Decision: the jarvis-apps cluster-scope check is `jarvis-apps/.github/checks/namespaced-only.sh`, called from `ci.yml`.
- Reason: R30 requires the check, and the layout names `ci.yml` only.
- Where: `p4a-05`.
- Status: accepted-with-plan default.

## D32

- Decision: the Ingress class is the class the bundled Traefik chart already registers. The install task copies that name into `cluster-vars.yaml`.
- Reason: OQ8 requires plain Ingress and does not name the class.
- Where: `p0b-03`.
- Status: accepted-with-plan default.

## D33

- Decision: CI tests for `bootstrap-secrets` generate a throwaway age key inside the job. The real private key is never in CI.
- Reason: the script needs a test (R7, R35), and R25 keeps the real key off CI and off the repo.
- Where: `p0a-04`.
- Status: accepted-with-plan default.

## D34

- Decision: the node SSH user is an env setting, `ssh_user` in `cluster-vars.yaml`, default `ubuntu`. The default lives once, in the env schema’s resolved values; Ansible consumes the resolved value and adds no fallback of its own. Gordon installs Ubuntu with his SSH key on that user. `site.yml` sets `ansible_user` from it and runs every task as root with become; k3s is a root systemd service, and nothing installs under that user.
- Reason: the user is only a login, so the Ubuntu default needs no extra user, and an env that differs changes one value (R3, R4).
- Where: `p0a-01`, `p0a-06`, `p0a-12`, `p0a-13`.
- Status: Confirmed by Gordon 2026-10-19.

## D35

- Decision: jarvis-apps PR-code jobs (`ci`, and the namespaced-only check) run on `pull_request` only, with no secrets. A separate job named `admin-gate` runs only on `pull_request_target`, never checks out code, and is its own required check. Image publish stays in `build.yml` on push to main (§6).
- Reason: P0 requires branch protection with check `ci` on both repos. `pull_request_target` must not run PR code. `admin-gate` is metadata-only and its own required check. P0a’s `ci` job starts green. p2-24 adds the `admin-gate` job. p4a-05 adds the namespaced-only check to the `pull_request` job.
- Where: `p0a-06`, `p2-24`, `p4a-04`, `p4a-05`.
- Status: accepted-with-plan default.

## D36

- Decision: the in-repo geo MCP server uses the Hermes image’s Python standard library only.
- Reason: it runs on the image’s Python (§5). A package download at start is what pinning avoids for the in-repo geo server. Metric reads use mcp-victoriametrics (D29).
- Where: `p2-11`.
- Status: accepted-with-plan default.

## D37

- Decision: foreman stores the last reported GitHub `updated` time on its volume and skips the model call when the public issue and PR list is unchanged.
- Reason: R40 requires the skip and does not say how to detect “nothing changed”.
- Where: `p4b-09b`.
- Status: accepted-with-plan default.

## D38

- Decision: admin-gate’s bot and App actor cases are proven by a fixture event (actor type Bot, and actor type App) run in CI against the same function the workflow calls. The live PR proves a label from `gordon5417-ai` fails, and a label Gordon’s own account adds after the latest commit passes.
- Reason: the P2 check requires a bot or App actor, and the code-factory App is created at P4 (§8).
- Where: p2-24. The live App label is repeated at p4b-10.
- Status: Confirmed by Gordon 2026-10-19.

## D39

- Decision: k3s NetworkPolicy can't match hostnames. Egress is DNS plus TCP 443 to 0.0.0.0/0 except RFC1918, the pod and service CIDRs, and the node IP. CONTRACT.md lists hosts as documentation. Hostname allow-listing waits for the §9 proxy.
- Reason: a NetworkPolicy matches IPs, not names. The host list records intent and does not pretend to enforce it.
- Where: `p1-09`, `p2-09`, `p3-01a`, `p4b-05`, `p4a-07`.
- Status: Confirmed by Gordon 2026-10-19.

## D40

- Decision: Gmail and Calendar tools come from workspace-mcp (taylorwilsdon/google_workspace_mcp, MIT), pinned workspace-mcp==2.1.1 via uvx with its uv cache on the deploy volume, single-user, stdio. inbox runs it with `--single-user --permissions gmail:drafts calendar:readonly` (the drafts level never registers send_gmail_message). concierge runs it with `--single-user --read-only --tools gmail calendar`. make google-consent writes the token file into WORKSPACE_MCP_CREDENTIALS_DIR/<email>.json (writable, the server refreshes it) with USER_GOOGLE_EMAIL set.
- Reason: most-used open-source Workspace MCP, local with Gordon’s own OAuth client, one flag removes send. Hermes’s bundled google-workspace skill needs the terminal tool and asks for send and Drive scopes.
- Where: `p3-00`, `p3-01a`, `p3-01b`, `p3-05`.
- Status: Confirmed by Gordon 2026-10-19.

## D41

- Decision: GitHub tools come from github-mcp-server (github/github-mcp-server, official, MIT), release binary v2.0.2 pinned on the deploy volume. Ops (concierge `p3-02`, homelab `p3-04`) runs `stdio --read-only --toolsets pull_requests,actions` with a fine-grained token from Gordon’s account with no repos selected (public read only), in SOPS for hermes-ops. spec-writer (`p4b-09a`) runs `stdio --toolsets issues` with code-factory App auth (Issues read/write), behind the R15 confirm step.
- Reason: GitHub’s own server. Its read-only and toolset switches are the whole restriction. App auth is native. R13 allows this one token in ops.
- Where: `p3-02`, `p3-04`, `p4b-09a`.
- Status: Confirmed by Gordon 2026-10-19.

## D42

- Decision: node roles are Ansible inventory groups in `envs/<env>/inventory/hosts.yml`. A node’s optional data disk is a host var in `envs/<env>/inventory/host_vars/<host>.yml`. The env schema validates both in CI, and there is no separate nodes file.

  ```yaml
  # envs/<env>/inventory/hosts.yml
  all:
    children:
      control_plane: {hosts: {<host>: }}   # allowed groups: control_plane, workers, storage, backup, gpu
      workers:       {hosts: {<host>: }}
      storage:       {hosts: {<host>: }}
      backup:        {hosts: {<host>: }}

  # envs/<env>/inventory/host_vars/<host>.yml
  data_disk:                          # optional; omit it and data stays on the root disk
    id: /dev/disk/by-id/<stable id>   # a whole disk; never /dev/nvmeXnY or a -partN path
    wipe_confirmed: true
  ```

  - Groups: the names use underscores, never hyphens. The schema rejects an unknown group name, a host_vars file for a host that is not in the inventory, and any host_vars key other than `data_disk` (no IPs, R25). It requires exactly one host in `control_plane` (the k3s server) and exactly one in `backup`. A host may sit in several groups.
  - Plays target groups: base-os runs on `all`, node-disk only on hosts that set `data_disk`, and the k3s server on `control_plane`. The k3s config sets `node-name` from `inventory_hostname`. Labels come from each host’s `group_names` as `role.<dns_suffix>/<group>=true`, prefixed by the env’s domain (D44). A task on `control_plane` applies them with the kubernetes.core module on every run, removing `role.<dns_suffix>/` labels for groups a host has left, so group changes are reconciled. k3s `node-label` is not used, because it applies only at registration. Verify flags any drift (D17).
  - Roles are declared, never auto-detected, and site.yml asserts them against facts. Before any change, a `gpu` host must have an NVIDIA device. After node-disk and before k3s, a host with `data_disk` must have `/var/lib/jarvis` mounted as the `jarvis-data` filesystem. A fresh node passes because node-disk has just mounted the disk.
  - Boot safety: on a host with `data_disk`, node-disk installs a systemd drop-in for `k3s.service` with `RequiresMountsFor=/var/lib/jarvis`. With `nofail` the host still boots without the disk, but k3s does not start, so nothing is written to the root disk in its place.
  - Disk: the data disk mounts at `/var/lib/jarvis`, so the D3 paths do not change; without one, the same paths sit on the root disk. k3s and images stay on root. The filesystem label `jarvis-data` identifies the disk, and fstab mounts it by `LABEL=jarvis-data` with `nofail`. Layout: `local-path/` (PVC volumes), `backup/etcd-snapshots/`, `backup/restic/` (VolSync repositories). Backups on the same disk cover mistakes and rollback, not disk loss; off-site is deferred (R31).
  - Safety (`node-disk` role, after base-os, before k3s first starts): refuse a missing disk, a partition, the root disk, a disk mounted elsewhere, or a host where another disk already carries the label. A disk whose filesystem is labeled `jarvis-data` is mounted, never wiped. Any other disk is wiped only when both `wipe_confirmed: true` and the run-time `-e jarvis_wipe_confirm=<node>:<disk id>` are present. If either is missing, the role stops and prints the disk’s model, size and partitions. It never mounts over a non-empty `/var/lib/jarvis`.
  - Public repo: the by-id path contains the disk’s serial number. That is accepted as low risk.
  - env-01: bench.lan is in `control_plane`, `workers`, `storage` and `backup`. Its data disk is the 1 TB Corsair MP600 Mini, by the by-id path read from bench.lan. Gordon confirms it may be wiped at the wipe-and-rebuild.
- Reason: Gordon (2026-10-20) wants roles defined in the inventory, where Ansible already targets hosts. Groups give plays native targeting, and `group_names` gives the labels without a second list. Checking declared roles against facts catches a wrong inventory without guessing roles. A label mount survives device renames. Two keys keep a stale flag or a copied env from wiping a disk.
- Where: `p0a-11a` (before `p0a-12`), `p0a-12`, `p0a-13`, `p0a-18`; env-00 is the multi-node fixture (D43); framework §5 and §6.
- Status: Proposed; accepted when the owner merges this PR.

## D43

- Decision: `envs/env-00` is an example env, a pretend 7-node cluster that is never deployed. It is the one complete fixture for the env schema, the inventory and the Ansible role logic.
  - Hosts are invented, on the example domain `env-00.example`: `cp-1` (`control_plane`), `wk-1` to `wk-4` (`workers`, with `wk-2` and `wk-3` also in `gpu`), `st-1` (`storage`) and `bk-1` (`backup`). Every group is used.
  - Every host except `wk-4` has a `data_disk` host_vars file with a made-up by-id path, and `wipe_confirmed` is true on some and false on others. `wk-4` keeps its data on the root disk.
  - Values set every key the schema allows, using example values: `dns_suffix: env-00.example`, `flux_git_repo_url: https://github.com/example-owner/jarvis`, the owner handle `example-owner`, and an example `ssh_user`. No value is copied from a real env.
  - It has no `secrets/`, no SOPS group_vars and no `flux/`, because it never deploys and the schema does not require them. If a later schema key needs a SOPS file, it gets a clearly fake placeholder encrypted to a throwaway test key.
  - CI validates it like any env. A behavior test asserts that env-00 sets every key the schema allows, so a new schema key fails CI until env-00 sets it. CI also runs `ansible-inventory --graph` on its inventory, plus the role and fact asserts in check mode where they need no host (group and host_vars rules, the label set from `group_names`).
- Reason: Gordon (2026-10-20): Jarvis is a product, not a home-lab pet, so its core is proven against a full multi-node env, not only the one-node env-01. Requiring every key catches schema keys the example forgot.
- Where: `p0a-05a`, extended by `p0a-11a`; used by `p0a-11` and `p0a-12`; framework §5 and §6.
- Status: Proposed; accepted when the owner merges this PR.

## D44

- Decision: environment-identity values never appear in the repo’s core, and each has one source of truth.
  - Values that live in env settings are read from there. The core-names check (`tools/checks/core-names`) reads them from every `envs/<env>` except env-00, which is excluded from every source, env names included, so fixtures, tests and role asserts may name it: inventory hostnames, `ssh_user`, `dns_suffix`, and the repo slug (`<owner>/<repo>`) and owner handle from the env’s `flux_git_repo_url` (schema-validated, in `envs/<env>/values`). A new env is covered automatically.
  - The agent account handle is not repo config, so the repo never references it and core-names has nothing to check. If the repo ever needs it, it becomes an env value and is covered automatically.
  - Only values that never live in the repo come from a GitHub Actions repo variable, with an env var of the same name for local runs. Today that is just the owner’s personal name. The check fails loudly when that variable is unset in CI.
  - The check holds no values of its own. It scans `infra/`, `platform/`, `services/`, `tools/`, `docs/`, `.github/` (except `.github/CODEOWNERS`, which needs literal handles), every README and `rulebooks/`. Workflows name their own repo with `${{ github.repository }}` or local `./.github/...` refs, never a hardcoded slug. Hostnames, `dns_suffix`, the repo slug, the owner handle and the owner’s personal name match case-insensitively as whole words, so a handle inside a URL is caught and a substring of a longer word is not.
  - `ssh_user` matches only in identity contexts: `/home/<user>`, `<user>@<host>` and `~<user>`. It never matches as a bare word, so `ubuntu` the OS and `ubuntu:26.04` pass. The check holds no word lists.
  - `envs/<env>/` may hold hostnames, the repo URL with its owner handle, usernames and the domain, because that is env config. The owner’s personal name is allowed nowhere, `envs/` included.
  - The repo is matched as its slug, because the bare repo name is the product name, which core may use.
  - Code that needs an identity value at run time, such as admin-gate’s owner account or the code-factory label-actor check, reads the owner handle from the env values.
  - The LAN domain is the env’s DNS suffix: `dns_suffix` in `cluster-vars.yaml`, schema-validated and reaching manifests as `DNS_SUFFIX` through postBuild. Hostnames, Ingress, the wildcard cert and the node-label prefix `role.<dns_suffix>/<group>` all derive from it. Every nodeSelector and affinity on that prefix uses `${DNS_SUFFIX}` through Flux postBuild and is never hardcoded, so changing `dns_suffix` relabels every node and its selectors follow.
- Reason: Gordon (2026-10-20) wants core to read the same for any owner. Reading env settings, plus one variable for the name that never enters the repo, keeps one source of truth, and the check never contains the names it guards.
- Where: `p0a-05` (`dns_suffix`), `p0a-07`, `p0a-12`, `p0a-18`, `p2-24`, the P4b label-actor check; framework R3, §5, §6, §8 and §11.
- Status: Proposed; accepted when the owner merges this PR.
