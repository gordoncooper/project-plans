# JARVIS AlphaCut — Decisions

Defaults chosen where `framework.md` is silent. Merging this file accepts all defaults except those marked Confirmed by Gordon.

Confirmed by Gordon 2026-10-19: D1, D2, D4, D19, D21, D22, D24, D26, D27, D34, D38, D39, D40, D41. Confirmed by Gordon 2026-10-20: D3, D44. Confirmed by Gordon 2026-10-31: D17, D42, D43, D45, D46, D47. Confirmed by Gordon 2026-11-02: D48–D67. Confirmed by Gordon 2026-11-04: D68, D69. The other entries are accepted-with-plan defaults.

These are the §11 defaults. They are recorded here. They are not copied into framework §10: this change adds only this file and `plan.md`.

## D1

- Decision: the product repo `jarvis` is created under the GitHub user `gordoncooper` (the user that owns project-plans). env-01’s instance repo is `gordoncooper/jarvis-internal` (D60). `jarvis-apps` is created under the same user at P4a.
- Reason: R36 needs `admin-approved` from Gordon’s own account, and this repo is already on that user.
- Where: P0 repo creation (`p0a-03`, `p0a-06`); instance repo `p0b-13`; apps repo `p4a-04`.
- Status: Confirmed by Gordon 2026-10-19.

## D2

- Decision: the env-01 Ansible inventory host is `bench.lan`. The env stays named env-01.
- Reason: it is the name the LAN DNS resolves, so no IP or alias is needed, and the inventory holds no IPs (R25).
- Where: `envs/env-01/inventory/hosts.yml` (`p0a-03`, `p0a-12`).
- Status: Confirmed by Gordon 2026-10-19.

## D3

- Decision: the data paths are env settings in `cluster-vars.yaml`: `local_path_data` (default `/var/lib/jarvis/local-path`) and `backup_path` (default `/var/lib/jarvis/backup`). The defaults live only in the env schema. A data disk mounts at `/var/lib/jarvis`, so the defaults sit on it (D42).
- Reason: both directories are required and neither path is named. An operator who changes a path means it; the k3s mount guard follows the configured paths (D42).
- Where: Ansible, VolSync, etcd snapshots (`p0a-11b`, `p0a-12`, `p0b-09`).
- Status: Confirmed by Gordon 2026-10-20.

## D4

- Decision: pod and service CIDRs are the built-in defaults of the pinned k3s version, written once into `cluster-vars.yaml` and `/etc/rancher/k3s/config.yaml`.
- Reason: CIDRs are set once and no values are given.
- Where: k3s role (`p0a-12`).
- Status: Confirmed by Gordon 2026-10-19.

## D5

- Decision: the k3s version is pinned in `cluster-vars.yaml` and changes only by PR. Renovate does not manage it. A k3s bump is also a Traefik bump: the Traefik CONTRACT.md records the bundled chart version and the k3s version it was checked against, and ci fails until a bump PR re-checks `platform/traefik` and updates that line.
- Reason: R33 requires that pin. Renovate manages image digests and does not manage the k3s host binary. The bundled Traefik chart comes only from the k3s pin, so a bump could silently break the HelmChartConfig.
- Where: k3s role (`p0a-12`), Traefik (`p0b-04`).
- Status: accepted-with-plan default.

## D6

- Decision: etcd snapshot interval and retention take the built-in default values of that same k3s version, written as product constants in the k3s role’s `vars/` (D46). The snapshot directory is under the backup path (D3).
- Reason: a schedule is required and no numbers are given. Snapshots are for same-node rollback; a restic copy on the rest-server adds retention, not disk-failure protection (R31, D47).
- Where: k3s config (`p0a-12`, `p0b-09b`, `p0b-10`).
- Status: accepted-with-plan default.

## D7

- Decision: §5 contract files live at these paths. Two contracts that share a folder are separate sections of that one `CONTRACT.md`. Component folders named in §6 still get their own `CONTRACT.md` when the component is built.
  - Env schema: `schema/CONTRACT.md` (repo-root `schema/` holds the env validator and its contract)
  - Profile schema: its own `services/hermes/core/schema/CONTRACT.md`, created at `p2-04` (it stays with Hermes, D69)
  - Code delivery provider: `docs/code-delivery/CONTRACT.md`
  - Build agent: `docs/build-agent/CONTRACT.md`
  - Exposure: `platform/traefik/CONTRACT.md` (cert-manager and app-dns component contracts point at it)
  - Backups: `platform/backups/CONTRACT.md`
  - Login and alert path: `services/observability/CONTRACT.md`
  - Chat-bot slot contract: `services/slots/chat-bot/CONTRACT.md`. It covers hosting, storage, backups, secrets and URL/ingress, and holds the model provider, assistant API and skill contract sections (D69).
  - Hermes core: `services/hermes/core/CONTRACT.md` says how Hermes meets the chat-bot slot. The profile schema and the deploy contracts below stay with Hermes.
  - Each Hermes deploy (`services/hermes/deploy/ops`, `services/hermes/deploy/code-factory`) is a component root; its `CONTRACT.md` lists the R13 egress hosts it uses.
  - Sandbox: `services/hermes/deploy/code-factory/sandbox/CONTRACT.md`
  - Code-factory work protocol: `services/hermes/deploy/code-factory/CONTRACT.md`
  - App hosting: `platform/app-hosting/CONTRACT.md`
- Reason: §5 requires a `CONTRACT.md` from one template, and the layout names only the code-delivery and build-agent paths.
- Where: every contract task.
- Status: accepted-with-plan default.

## D8

- Decision: the R2 check treats a component root as a §6 folder the layout assigns `CONTRACT.md`. Every direct child of `platform/` is a root. Nested directories are not component roots, and a parent that holds only nested roots and no files of its own is skipped until it is built.
- Reason: the rule requires the file and does not define the folder set. A check on every directory would fail on `skills/`.
- Where: `ci/checks` (`p0a-08`).
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

- Decision: the P0b test app is namespace `platform-testapp`, hostname `testapp.alpha.lan`, image `docker.io/nginxinc/nginx-unprivileged` (the non-root build: uid 101, port 8080) pinned by digest, serving a static file with a known checksum. It stays up so P1 can kill it and restore it.
- Reason: the reachability check needs a test app, and §4 has no HTTP fixture. The image is a default beyond §4 (R9); this entry is the record. `library/nginx` runs its master process as root.
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

- Decision: the backup cron is read in the VolSync pod’s time zone, and that pod’s `TZ` is `time_zone` from cluster-vars. restic prunes weekly through `pruneIntervalDays: 7` on the backup run, an interval, not a clock; there is no Sunday cron. The Hermes pod takes TZ from cluster-vars too.
- Reason: retention includes a weekly prune (§7), and VolSync’s cron has no time-zone field (the ReplicationSource CRD rejects `CRON_TZ`). env-01’s time zone is America/Los_Angeles (D21).
- Where: backup config (`p0b-09`, `p0b-09a`, `p0b-09b`, `p0b-09c`), Hermes pod (`p2-09`).
- Status: accepted-with-plan default.

## D16

- Decision: Grafana and both Hermes dashboards use username `admin`. Passwords, the Hermes password hash, the signing secret, and the API key come from bootstrap-secrets.
- Reason: R27 requires a built-in login and names no username.
- Where: bootstrap-secrets, login contract (`p0a-04`, `p1-01`, `p2-05`).
- Status: accepted-with-plan default.

## D17

- Decision: `make verify ENV=<env>` is a read-only, fail-closed check of that env’s standing state, usable by any adopter at any time. It runs `tools/verify/`, a short script outside `ci/` with no tests of its own. The operator (framework §2) runs it on the controller as the agent user, with no age key, and compares live state with the repo (Example: `kubectl wait --for=condition=Ready` on Flux objects; node labels against the inventory, D42). It exits non-zero if anything is unchecked, including zero objects found, and does not re-run CI checks or list owner items. Seeded runs and write attempts are their own runs (seeded PRs, drills), attached beside it, never inside the script. It may read standing state (Example: that admin-gate is a required check), not cause it. `make drill ENV=<env>` runs the restore drill in the repo (a scratch volume, checksum before and after). Drills and evals are their own runs. Owner runs (the runs that need the age key: the first `site.yml` on a fresh node and an age-key swap or rotation; not `bootstrap-secrets`, which needs only the `.sops.yaml` public recipient, except the one run that recovers `envs/<env>/ca.crt` from an existing CA secret), `[G]` checks and evidence links stay as the phase’s owner checklist in the plan, outside the script; Gordon pastes them with the output, and his review is the gate. The flux-operator role refuses to overwrite a differing `flux-system/sops-age`, so a swap means deleting that secret by hand, re-encrypting the env’s secrets to the new recipient, and running `site.yml` with the new key file. The agent user never holds a real age key and keeps node SSH for runs that need none: any later `site.yml`, full or tagged, once `flux-system/sops-age` exists (Example: `site.yml --tags kubeconfig`). The “another ID gets none” check uses a gateway-handler fixture.
- Reason: §2 requires attached verify output for standing state, and a separate in-repo drill for restore. A script that lists owner items or parses a transcript proves nothing. `[G]` checks and Gordon-run steps stay outside the script. The age key never reaches the agent user (R25, §8), and `site.yml` needs it only while `flux-system/sops-age` is absent. No second Telegram account exists.
- Where: every phase verify task, `p0a-13`, `p0a-14`, `p0b-10`.
- Status: Confirmed by Gordon 2026-10-31.

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
- Where: `p0a-01`, `p0a-11`, the env-01 instance repo `values/hermes.yaml` (`p3-03`, `p3-04`).
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

- Decision: during the Hermes add-on’s `google-consent` (D69), Gordon sends one message to himself with subject `jarvis-p3-seed`. The draft check uses that thread.
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

- Decision: every container starts from these memory requests and is tuned from Grafana as real usage shows up: VictoriaMetrics 512Mi, VictoriaLogs 512Mi, Grafana 256Mi, Gatus 128Mi, Fluent Bit 64Mi, vmagent 128Mi, node-exporter 64Mi, kube-state-metrics 64Mi, Hermes ops 2Gi with a 3Gi limit, Hermes code-factory 1Gi with a 2Gi limit, each platform container 128Mi request and 256Mi limit. Every long-running container (controllers, apps, DNS) has a memory request and limit and a CPU request, and no CPU limit. Short-lived chart hooks, Jobs, init containers and test pods keep the chart’s defaults; a chart default is overridden only when it causes a real problem.
- Reason: R5 requires numbers; real usage, not up-front calculation, sets the final sizes.
- Where: every manifest task.
- Status: accepted-with-plan default.

## D31

- Decision: the jarvis-apps cluster-scope check is `jarvis-apps/.github/checks/namespaced-only.sh`, called from `ci.yml`.
- Reason: R30 requires the check, and the layout names `ci.yml` only.
- Where: `p4a-05`.
- Status: accepted-with-plan default.

## D32

- Decision: the Ingress class is the class the bundled Traefik chart registers, written as a constant in manifests, not a cluster-vars setting (D46).
- Reason: OQ8 requires plain Ingress and does not name the class.
- Where: `p0b-03`.
- Status: accepted-with-plan default.

## D33

- Decision: CI tests for `bootstrap-secrets` generate a throwaway age key inside the job. The real private key is never in CI.
- Reason: the script needs a test (R7, R35), and R25 keeps the real key off CI and off the repo.
- Where: `p0a-04`.
- Status: accepted-with-plan default.

## D34

- Decision: the node SSH user is an env setting, `ssh_user` in `cluster-vars.yaml`, default `ubuntu`. The default lives only in the env schema; Ansible adds no fallback. `tools/prep-node` installs the controller user’s SSH key on that user (the login user or a new one it creates), grants it passwordless sudo and records it as `ssh_user`. `site.yml` sets `ansible_user` from it and runs every task as root with become; k3s is a root systemd service, and nothing installs under that user.
- Reason: the user is only a login, so the Ubuntu default needs no extra user, and an env that differs changes one value (R3, R4).
- Where: `p0a-01`, `p0a-05`, `p0a-12`, `p0a-13`.
- Status: Confirmed by Gordon 2026-10-19.

## D35

- Decision: jarvis-apps PR-code jobs (`ci`, and the namespaced-only check) run on `pull_request` only, with no secrets. A separate job named `admin-gate` runs only on `pull_request_target`, checks out no PR code, and is its own required check. Image publish stays in `build.yml` on push to main (§6).
- Reason: `pull_request_target` must not run PR code. `admin-gate` is metadata-only and its own required check when branch protection is on (D56, D61). P0a’s `ci` job starts green. p2-24 adds the `admin-gate` job. p4a-05 adds the namespaced-only check to the `pull_request` job. Paid branch protection is optional, not the deploy gate.
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

- Decision: admin-gate’s bot and App actor cases are proven by a fixture event (actor type Bot, and actor type App) run in CI against `ci/checks/admin-gate`, the same script the workflow runs. The live PR proves a label from `gordon5417-ai` fails, and a label Gordon’s own account adds after the latest commit passes.
- Reason: the P2 check requires a bot or App actor, and the code-factory App is created at P4 (§8).
- Where: p2-24. The live App label is repeated at p4b-10.
- Status: Confirmed by Gordon 2026-10-19.

## D39

- Decision: k3s NetworkPolicy can't match hostnames. Egress is DNS plus TCP 443 to 0.0.0.0/0 except RFC1918, 100.64.0.0/10, the pod and service CIDRs, and the node IP. CONTRACT.md lists hosts as documentation. Hostname allow-listing waits for the §9 proxy.
- Reason: a NetworkPolicy matches IPs, not names. The host list records intent and does not pretend to enforce it.
- Where: `p1-09`, `p2-09`, `p3-01a`, `p4b-05`, `p4a-07`.
- Status: Confirmed by Gordon 2026-10-19.

## D40

- Decision: Gmail and Calendar tools come from workspace-mcp (taylorwilsdon/google_workspace_mcp, MIT), pinned workspace-mcp==2.1.1 via uvx with its uv cache on the deploy volume, single-user, stdio. inbox runs it with `--single-user --permissions gmail:drafts calendar:readonly` (the drafts level never registers send_gmail_message). concierge runs it with `--single-user --read-only --tools gmail calendar`. The Hermes add-on’s google-consent (D69) writes the token file into WORKSPACE_MCP_CREDENTIALS_DIR/<email>.json (writable, the server refreshes it) with USER_GOOGLE_EMAIL set.
- Reason: most-used open-source Workspace MCP, local with Gordon’s own OAuth client, one flag removes send. Hermes’s bundled google-workspace skill needs the terminal tool and asks for send and Drive scopes.
- Where: `p3-00`, `p3-01a`, `p3-01b`, `p3-05`.
- Status: Confirmed by Gordon 2026-10-19.

## D41

- Decision: GitHub tools come from github-mcp-server (github/github-mcp-server, official, MIT), release binary v2.0.2 pinned on the deploy volume. Ops (concierge `p3-02`, homelab `p3-04`) runs `stdio --read-only --toolsets pull_requests,actions` with a fine-grained token from Gordon’s account with no repos selected (public read only), in SOPS for hermes-ops. spec-writer (`p4b-09a`) runs `stdio --toolsets issues` with code-factory App auth (Issues read/write), behind the R15 confirm step.
- Reason: GitHub’s own server. Its read-only and toolset switches are the whole restriction. App auth is native. R13 allows this one token in ops.
- Where: `p3-02`, `p3-04`, `p4b-09a`.
- Status: Confirmed by Gordon 2026-10-19.

## D42

- Decision: each node declares its roles (`node_roles`) and optional data disk in its own block in `envs/<env>/inventory/hosts.yml`. One constructed inventory in core, the folder `infra/inventory/roles.yml` (the constructed config plus `groups.yml`), passed as a second `-i` after the env inventory, turns the roles into Ansible groups. No env copies it. There are no host_vars and no separate nodes file.

  ```yaml
  # envs/<env>/inventory/hosts.yml
  all:
    hosts:
      <host>:
        node_roles: [control_plane, workers, storage, backup]   # known: control_plane, workers, storage, backup, gpu, edge
        data_disk: true   # optional; the admin mounted a disk at the data path. Omit when the node has no second disk.

  # infra/inventory/roles.yml/constructed.yml (core; one copy for every env)
  plugin: ansible.builtin.constructed
  strict: true
  keyed_groups:
    - key: node_roles
      prefix: ""
      separator: ""
      leading_separator: false
  groups:
    data_disk: data_disk is defined
  ```

  - Groups: the empty prefix and separator, with `leading_separator: false`, name each group after its role (never `_<role>`), so `--limit storage` and `hosts: storage` work. `data_disk` is a constructed group of the hosts that declare one; it is not a role and gets no label. `groups.yml` keeps it as an empty group when no host declares one, so its play matches nothing without a warning.
  - Schema: `node_roles` (not Ansible’s reserved `roles`) from the known set; exactly one `control_plane` host and one `backup` host, and at least one `edge` host; `data_disk` is the plain flag `true` or absent, with no id and no wipe confirmation; no other host keys (no IPs, R25). One control plane, amd64 only; GPU hosts join as agents (D59). Nodes without the `workers` role carry a NoSchedule taint; platform pods and tools tolerate it; agents restart one at a time.
  - Secrets: inventory and group_vars never need the age key. Secrets live only in SOPS files that bootstrap-secrets writes and Flux decrypts, never loaded through the community.sops vars plugin, so later `site.yml` runs work without the key (D17).
  - Plays: base-os on `all`, node-disk on `data_disk`, the k3s server on `control_plane`, the k3s agent on `all:!control_plane`, then kubeconfig and flux-operator on `control_plane`; `node-name` comes from `inventory_hostname`.
  - Join: every host that is not `control_plane` joins as a k3s agent; `workers`, `storage`, `backup`, `gpu` and `edge` are scheduling labels only, and the `control_plane` host runs the server only. An agent’s config sets `server: https://<control_plane inventory_hostname>:6443`, its own `node-name` and the agent token, with no `cluster-init` or `node-label`. The server token is sealed with the env’s secrets; agents join with a separate agent token (D49). Neither token is printed, and neither lives in the inventory or cluster-vars.
  - Labels: `role.jarvis/<role>=true` for each role; the prefix is the fixed product prefix and is reserved for them. A task on `control_plane` sets them with kubernetes.core on every run and removes prefixed labels for dropped roles. No k3s `node-label`, which applies only at registration.
  - Edge: 80/443 (Traefik) and 53 (App DNS) land on the `edge` nodes. Every env must give `edge` to at least one node; there is no fallback to `control_plane`, and the schema rejects an env with no `edge` host. The same task reconciles servicelb’s `svccontroller.k3s.cattle.io/enablelb=true` label onto exactly that set on every run.
  - Asserts: roles are declared, never detected. Before any change, the first play asserts amd64 hosts (D46) and a `gpu` host must have an NVIDIA device. On a host with `data_disk: true`, Ansible checks that the data path exists, is a real mount, and is empty or already holds Jarvis data, and refuses otherwise. It does not format, wipe, label or mount the disk.
  - Disk: the hardware admin prepares and mounts any data disk at `/var/lib/jarvis`. `data_disk: true` means that mount is in place; a node with no second disk leaves `data_disk` out. Example admin commands live in `docs/deploy.md`. Ansible checks that the data path exists, is a real mount when a data disk is declared, and is empty or already holds Jarvis data, and refuses otherwise. k3s and images stay on root. The k3s role creates `local_path_data`, `backup_path` and `etcd-snapshots` under the data path at 0700; node-disk does not. Layout under the data path: `local-path/`, `backup/etcd-snapshots/` (k3s writes the snapshots; a CronJob copies them into the rest-server), and `backup/restic/`, the rest-server’s data directory, with one restic repository per PVC at `ns-<namespace>/pvc-<pvc>` and no repo PV (D47). Same-disk backups cover mistakes and rollback, not disk loss (R31).
  - Boot safety: the k3s role adds a `k3s.service` drop-in with `RequiresMountsFor=` the configured `local_path_data` and `backup_path` (D3), so k3s waits for the mount. On a server/agent switch the k3s role disables the opposite k3s unit.
  - env-01: bench.lan has `control_plane`, `workers`, `storage`, `backup` and `edge`. It has no `data_disk` and keeps its data on lv-data on the OS disk.
- Reason: one block reads as the whole node, and the constructed config gives plays real groups without a second list. Fact asserts catch a wrong inventory without guessing roles. The hardware admin owns the disk; Ansible only refuses a data path that is missing, not a real mount, or neither empty nor already Jarvis data.
- Where: `p0a-11a`, `p0a-11b`, `p0a-12`, `p0a-13`, `p0a-14`, `p0a-18`, `p0b-03`, `p0b-05`; framework §5 and §6.
- Status: Confirmed by Gordon 2026-10-31.

## D43

- Decision: `envs/env-00` is a never-deployed example env, a pretend 7-node cluster, and the one complete fixture for the env schema, the inventory and the Ansible roles.
  - Hosts on `env-00.example`: `cp-1` (`control_plane`), `wk-1` to `wk-4` (`workers`; `wk-2` and `wk-3` also `gpu`, `wk-1` also `edge`), `st-1` (`storage`) and `bk-1` (`backup`). Every host but `wk-4` sets `data_disk: true`; `wk-4` leaves `data_disk` out and keeps its data on root.
  - Values set every schema key with example values (Example: `dns_suffix: env-00.example`, repo URLs under `example-owner`). Nothing is copied from a real env.
  - No `secrets/`, SOPS files or `flux/`.
  - CI validates it like any env, fails when the schema allows a key env-00 does not set, and runs `ansible-inventory --graph` and `--syntax-check` against it.
- Reason: Jarvis is a product, so core is proven against a multi-node env, not only the one-node env-01.
- Where: `p0a-05a`, `p0a-11`, `p0a-11a`; framework §5 and §6.
- Status: Confirmed by Gordon 2026-10-31.

## D44

- Decision: no environment-identity value appears in core, and each has one source.
  - Sources: `ci/checks/core-names` reads them from every `envs/<env>` except env-00, a fixture excluded from every source: env names (the folder names), inventory hostnames, `ssh_user`, `dns_suffix`, and the owner handle in `flux_git_repo_url` and `apps_git_repo_url`. A new env is covered automatically. The owner’s personal name never lives in the repo; it comes from a GitHub Actions repo variable, not a secret (an env var of the same name locally), it matches only as the full name, and the check fails when it is unset in CI. The agent account is not repo config, so the repo never names it.
  - Scan: `infra/`, `platform/`, `services/`, `tools/`, `ci/`, `docs/`, `.github/` except `CODEOWNERS`, every README and `rulebooks/`. Env names are allowed in `envs/**` and `docs/**`, the other values in `envs/<env>/`, and the personal name nowhere. Root files are not scanned, so the per-env `.sops.yaml` rules (D45) need no exemption.
  - Match: case-insensitive whole words, so a handle inside a URL is caught and a longer word is not. Bare repo names are not identity values; a repo URL outside `envs/<env>/` fails on its owner handle. `ssh_user` matches only as `/home/<user>`, `<user>@<host>` or `~<user>`, so `ubuntu:26.04` passes. The check holds no values and no word lists.
  - Product rules in the same check: the word `bastion` and any hardcoded `/home/` path are banned.
  - Workflows name their own repo with `${{ github.repository }}` or local `./.github/` refs. admin-gate and the code-factory label-actor check trust the repository owner from the event (`github.repository_owner`) and check out no PR code.
  - The LAN domain is `dns_suffix`. Hostnames, Ingress and the wildcard cert derive from it. Node role labels use the fixed product prefix `role.jarvis/` (D42); selectors use that prefix, not `${dns_suffix}`.
- Reason: core reads the same for any owner. Reading env settings keeps one source of truth, and the check never contains the names it guards.
- Where: `p0a-05`, `p0a-07`, `p0a-12`, `p0a-18`, `p2-24`, `p4b-07`; framework R3, R36, §5 and §11.
- Status: Confirmed by Gordon 2026-10-20.

## D45

- Decision: how env settings are defined and reach the cluster.
  - `apps_git_repo_url`: absent until the step that adds its consumer (`p4a-02`, `p4b-09a`, `p4b-09b`), and that step adds the key and its validation. When set, it is https, validated like `flux_git_repo_url`. It names the apps repo for app hosting and the code-factory. No rule ties envs to one GitHub owner (D55).
  - `llm_provider_host`: absent until `p2-18`, which adds the key and its validation. The AI provider’s API host, read by Hermes and the Gatus provider probe; the p2-18 drill overrides it.
  - `hermes.yaml`, `apps.yaml`, their validators and `build_agent` are absent until the step that adds their consumer (D54): Hermes at `p2-07`, apps at P4a, build agents as an adapter list with the code factory (`p4b-04`). `code_delivery` stays absent until a consumer reads it; when set, the only accepted value is `github`. `hermes.*` validation lands with the P2 hermes steps that consume those keys (`p2-07`, `p2-13`). A `.sops.yaml` rule for an env that does not exist fails.
  - The rendered cluster-vars ConfigMap publishes an allow-list of cluster keys. Host-only keys stay in Ansible and are not published. The render step (`p0a-14`) decides which keys are published and which stay host-only, and lists them in the env schema CONTRACT.
  - Flux variables are the lowercase cluster-vars keys (`${dns_suffix}`), and lists are JSON strings. kustomize-controller’s `StrictPostBuildSubstitutions` is on, so a missing variable fails instead of going empty. A literal `${...}` that is not a cluster var (shell in ConfigMaps, dashboards, Gatus templates) is written `$${...}`, and `$${}` is that literal escape. `ci/checks/flux-substitute` fails unless every `${var}` a Flux Kustomization substitutes is a key in that env’s cluster-vars.
  - `.sops.yaml` has one age creation rule per env, keyed on `envs/<env>/secrets/`.
  - `age_key_file`: optional, the age private key path on the controller, default `~/.config/sops/age/keys.txt` under the running user’s HOME, never in the repo; `SOPS_AGE_KEY_FILE` overrides it. Each env uses its own key file, and `prep-node`, `bootstrap-secrets` and `site.yml` use the same one. `prep-node` warns when an existing key does not match the env’s `.sops.yaml` recipient.
  - Tools that act on one env (bootstrap-secrets, seal, the cluster-vars render, verify, drill, restore, restore-all) require `ENV=` or `--env` and fail without it. The stale-render test in `ci/tests/test_flux_entry.py` skips envs with no `flux/`. `tools/prep-node` re-renders `envs/<env>/flux/generated/cluster-vars-configmap.yaml` after it edits cluster-vars.
  - The env schema CONTRACT.md has sections for node roles and labels (D42), identity values and product rules (D44), env-00 as the every-key reference, where a new key is set in env-00 in the same PR (D43), and cluster-vars to postBuild (this entry).
- Reason: one source per value, loud failures instead of empty ones, and no tool that guesses the env. Keys with no consumer yet stay optional, and the cluster-vars ConfigMap publishes only an allow-list of cluster keys.
- Where: `p0a-03`, `p0a-04`, `p0a-05`, `p0a-05a`, `p0a-11a`, `p0a-14`, `p2-07`, `p2-13`, `p2-18`, `p4a-02`, `p4b-04`, `p4b-09a`, `p4b-09b`; framework R3, R4, §5 and §6. Pre-consumer keys: D54. Owner rule: D55.
- Status: Confirmed by Gordon 2026-10-31.

## D46

- Decision: product constants, not env settings.
  - The etcd snapshot schedule and retention (D6) and journald SystemMaxUse 2G (§7) live in role `vars/`. Role `defaults/` hold nothing that looks like env config; only the env schema holds defaults.
  - Ingress is always the bundled Traefik class; there is no `ingress_class` setting (D32).
  - Blocked egress always includes 100.64.0.0/10 (D39).
  - The first play asserts amd64 hosts, before any change.
  - Added only when an env needs one: monitoring retention, `ssh_port`, a registry mirror (§9).
- Reason: a setting no env changes is a constant; making it look configurable invites drift.
- Where: `p0a-11`, `p0a-12`, `p0b-03`, the D39 egress tasks; framework §11.
- Status: Confirmed by Gordon 2026-10-31.

## D47

- Decision: backups enroll themselves, and restore is one command.
  - Enrollment: a Kyverno ClusterPolicy (classic, `failurePolicy: Ignore`, `generateExisting`, `synchronize`) generates each PVC’s restic Secret and VolSync ReplicationSource (`copyMethod: Direct`) in the PVC’s namespace, and sets `volsync.backube/privileged-movers` on that namespace. It skips PVCs annotated `backup: none` and VolSync’s own claims (`app.kubernetes.io/created-by: volsync`). No ReplicationSource is written in Git; the CI check guards the policy and the opt-out allow-list.
  - Storage: one restic rest-server in `platform-backups`, pinned to the backup node, serves every repository from `backup_path/restic`, one per PVC at `ns-<namespace>/pvc-<pvc>` (the prefixes keep every path segment off rest-server’s reserved names, such as `data`, `keys`, `locks`, `snapshots` and `config`). Each namespace has its own restic credentials and its own private repository (D51). The off-host copy of every repository is D48.
  - Restore: `make restore ENV=<env> APP=<app>` and `make restore-all ENV=<env>`. The app is suspended and scaled to 0, then one Direct ReplicationDestination with `trigger.manual`, `restoreAsOf` before the loss and `enableFileDeletion` fills the app’s own PVC and is deleted. `make restore` fails on a PVC with no eligible snapshot. `make restore-all ENV=<env>` fails on any selected PVC with no eligible snapshot unless `ACCEPT_EMPTY` names that PVC. This holds with or without a rebuild hold, including for a PVC that is not in the hold. It never silently skips a PVC. `ACCEPT_EMPTY` and `RETIRE` are only valid with `make restore-all`. `ACCEPT_EMPTY` names a selected PVC that is accepted empty. `RETIRE` names a held repository that has no PVC.
  - Rebuild hold: a build that finds no Flux root source and restic repositories already on the backup path writes ConfigMap `kube-system/backup-rebuild-hold`; the policy keeps every generated ReplicationSource paused while it exists. `make restore-all ENV=<env>` lifts the hold only when every repository in the hold is restored, accepted empty, or retired by name. A run that leaves any listed repository unaccounted for keeps the hold. One `make restore` never lifts it. Sources also keep 7 days (`retain.within: 7d`).
  - No volume populator and no PVC `dataSourceRef` while volumes are local-path.
  - etcd: a CronJob on the server node backs up `backup_path/etcd-snapshots` into one host-scope repository (`host-etcd-snapshots`), with that host-scope credential and the same retention, after each k3s snapshot in `time_zone`. Restore is owner-run: restic restore to the node, then k3s’s cluster reset from that file.
  - Running volumes are backed up nightly as a file-by-file copy, not a snapshot, so a database quiesces or dumps first (Hermes: `p2-09`, `p2-20`).
- Reason: a new app is backed up just by having a PVC, with no per-app manifests to forget. One server removes the per-volume repo PV and PVC, its CI exception and most of the NFS rework (§9: only the server’s volume changes). The populator needs snapshot-based restores, which local-path cannot provide; its PVC stays Pending and cannot be fixed without deleting it. After a rebuild the new source backs up the fresh, empty app at once, so restoring the latest snapshot would restore the empty app; `restoreAsOf` skips that backup and `enableFileDeletion` clears the fresh files. Those empty backups would also push the good snapshots out of retention, so a rebuild holds every backup until `make restore-all` has restored, accepted empty, or retired by name every listed repository; a run that leaves any listed repository unaccounted for keeps the hold; keep-within counts from the newest snapshot, so it only delays that loss. Backup objects and repository URLs come from the one policy, and CI rejects them hand-written under `services/` or `apps/`, so a service never carries its own copy that drifts. A fail-open webhook never blocks a PVC, and the background scan backfills one missed during an outage. Velero with Kopia or K8up would replace VolSync, a larger change for one node. Kyverno 1.19 is tested upstream only to Kubernetes 1.35 and builds against 1.36, so its gate is admission Ready. The etcd snapshots are the cluster’s state, so they get the same retention, dedup, later NFS path and the off-host copy as the volumes (D48, p0b-09e); same-disk rest-server data covers mistakes and rollback, not disk loss.
- Where: `p0b-02`, `p0b-09a`, `p0b-09b`, `p0b-09c`, `p0b-09d`, `p0b-09e`, `p0b-10`, `p2-09`, `p2-20`, `p2-21`; framework R31, §4, §5, §6, §8. Per-namespace credentials: D51. Off-host copy: D48. Rebuildable cache: D67.
- Status: Confirmed by Gordon 2026-10-31.

## D48

- Decision: a nightly `restic copy` of every repository (each volume and the etcd copy) to a per-env off-host target, any restic backend; verify reports the copy’s freshness. The target is env config; core stays bastion-agnostic. env-01’s target is, for now, a temporary test target on the bastion that Operator sets up with its own user and append-only access (add, never delete), to prove the flow; a better target comes later as a setting change. The bastion copy covers a dead disk, not loss of the site. The off-host password is on the keep-safe list (§8). It lands before the first real data volume (`p2-09`).
- Reason: same-disk backups cover mistakes, not disk loss (R31). The off-host copy is that disk-loss path.
- Where: `p0b-09e`, `p0b-11`, `p2-09`; framework R31, §5 Backups, §8, §9.
- Status: Confirmed by Gordon 2026-11-02.

## D49

- Decision: the k3s server token is sealed with the env’s secrets, and agents join with a separate agent token. Sealing an existing env’s token is an owner step (never a coding agent; the value is never printed). The rebuild path (“rebuild a dead node”) is written and rehearsed. For a single-node env whose backups share the OS disk, the rehearsal is a planned outage on the owner’s go: a k3s uninstall and reinstall with the OS and `/var/lib/jarvis` kept. The new cluster starts from an empty etcd datastore and restores no snapshot; Git rebuilds everything. The sealed `server_token` is the short, passphrase-only form: a `K10<ca-hash>::server:<passphrase>` token carries the old CA’s hash, so it fails on a fresh CA. The passphrase is unchanged, so older etcd snapshots stay restorable. The rehearsal never wipes the data path, so it proves recovery from OS loss, not disk loss (disk loss is the off-host copy’s job, D48).
- Reason: the join token is a cluster secret; a printed or unsealed copy cannot be rotated from Git.
- Where: `p0a-12`, `p0a-14`, `p0b-13`; framework R7, R25, §8. D60 holds what env-01’s rebuild does.
- Status: Confirmed by Gordon 2026-11-02.

## D50

- Decision: two copies of one age key, the second offline; each proven by a repo command that test-decrypts one secret and prints only pass or fail; a second recipient is optional. A keep-safe list in the owner’s password manager holds the age key, the off-host backup password and, once a coding lane uses one, the lane token’s expiry date. Recovery reseals live Secrets when the cluster is alive and regenerates only when it’s gone. Tools refuse paths that would strand backups.
- Reason: losing the only copy means no rebuild from Git (R25); a printed secret is a leak.
- Where: `p0a-03`, `p0a-04`, `p0b-12`; framework R25, §8 custody.
- Status: Confirmed by Gordon 2026-11-02.

## D51

- Decision: the trusted zone is core (the `jarvis` repo’s k3s and Flux foundation, D69) plus the Jarvis add-ons, including the chat bot. It is one zone with a short credential list: the age key, the k3s server token (plus a separate agent token from the D2 rebuild rehearsal), one restic password and one storage login for all core backups, and the GitHub deploy key. Hosted apps from `jarvis-apps` are untrusted: each `app-<name>` namespace with data gets its own restic credential and its own private repository, before that app stores data. An app credential cannot read, write or prune any core repository, including the chat bot’s personal data, or another app’s.
- Reason: core and its add-ons are trusted, so more credentials inside the zone add work without much safety; apps are the untrusted side, so their backups are kept apart from core and from each other.
- Where: `p0b-09d`, `p0b-09e`, `p2-09`; framework R31, §5 Backups, §8.
- Status: Confirmed by Gordon 2026-11-02.

## D52

- Decision: nodes take security-only unattended upgrades with no automatic reboot; the owner is told when a reboot is pending; a drain-and-reboot runbook exists. GPU nodes use Ubuntu’s signed `linux-modules-nvidia-*` packages so kernel and driver stay matched.
- Reason: an unattended reboot drops the control plane; an unmatched GPU driver drops the GPU.
- Where: `p0a-11`; framework §4 Platform, §8 Standing.
- Status: Confirmed by Gordon 2026-11-02.

## D53

- Decision: services land as per-service Flux Kustomizations from a named core source. Only `flux-system` decrypts, with the env key; tenant secrets live in the instance under `envs/<env>/secrets/<namespace>/`; a tenant namespace never gets the age key. Flux runs with multi-tenancy lockdown; every Kustomization and HelmRelease names its service account; CI enforces it. A separate apps repo and the P4a ResourceSet come at P4a.
- Reason: one decryptor; tenants cannot read each other’s secrets or apply cluster-scoped objects.
- Where: `p0a-13`, `p0a-14`, `p0b-13`, `p2-09`; framework R1, R25, §5, §6.
- Status: Confirmed by Gordon 2026-11-02.

## D54

- Decision: `hermes.yaml`, `apps.yaml`, their validators and `build_agent` are absent from the env schema until the step that adds their consumer. Hermes re-adds `hermes.yaml` at `p2-07`; apps re-add `apps.yaml` at P4a; build agents arrive as an adapter list with the code factory (`p4b-04`).
- Reason: a key with no consumer is a required file that no step reads.
- Where: `p0a-01`, `p0a-03`, `p0a-05`, `p2-07`, `p4a-02`, `p4b-04`; framework R3, R4, §5 Env schema.
- Status: Confirmed by Gordon 2026-11-02.

## D55

- Decision: no rule ties envs to one GitHub owner. The `core-names` sources that read the owner handle from `flux_git_repo_url` and `apps_git_repo_url` stay (D44). A `.sops.yaml` rule for an env that does not exist fails.
- Reason: core stays owner-agnostic; one handle across envs is not a product rule.
- Where: `p0a-05`, `p0a-07`; framework R3, D44, D45.
- Status: Confirmed by Gordon 2026-11-02.

## D56

- Decision: the product repo is public; each instance repo is private; Flux can use an optional pull secret. Once the gate lands, Flux deploys only tags signed by owner-held keys, verified by Flux itself (not the forge’s web-merge key), mode `Tag`, on both the root sync (the instance repo) and the named core source. Because the FluxInstance `sync` block has no verify field, the env’s root source is a self-managing `GitRepository` that Ansible applies (FluxInstance `sync` dropped). Once the gate lands, nothing deploys until the owner pushes a signed release tag: core releases, Renovate bumps and env edits alike. Coding agents never create or push tags. Paid branch protection is optional, not the gate. The signing key’s public fingerprint and where the signing key lives go on the keep-safe list; “signing key lost or compromised: rotate the trusted key in Flux” is a Gordon break-glass action. The gate lands in `p2-08a`, after the foundation build and before the chat bot holds Gordon’s real data (D68). Until then, the env-01 move (`p0b-13`) deploys both sources from `main` with no signature check.
- Reason: the deploy gate is a signature Flux checks, not a forge merge setting.
- Where: `p0a-13`, `p2-08a`, `p2-24`; framework R25, R34, R36, §6, §8.
- Status: Confirmed by Gordon 2026-11-02.

## D57

- Decision: a new env’s internal CA is name-constrained to its DNS suffix; an existing CA gains the constraint at its next planned rotation. The CA, restic-rotation and age-key recovery runbooks are written; tools refuse paths that would replace the CA or strand backups.
- Reason: a CA that signs any name is a cluster-wide impersonation path.
- Where: `p0a-04`, `p0b-04`; framework R28, §5 Exposure, §8.
- Status: Confirmed by Gordon 2026-11-02.

## D58

- Decision: each coding-agent adapter’s deny list (Cursor, Claude Code) is checked against one category corpus for parity. The guard files are protected and change only in an isolated PR the owner approves.
- Reason: two adapters with different deny classes is a hole the other one closed.
- Where: `p0a-10`; framework R34, R36.
- Status: Confirmed by Gordon 2026-11-02.

## D59

- Decision: one control plane, amd64 only; GPU hosts join as agents. arm64 and HA stay deferred, with “an arm64 or second control-plane node joins” as the trigger.
- Reason: the first env is one amd64 node; a second control plane or an arm64 host is a new topology.
- Where: `p0a-11a`, `p0a-12`; framework D42, D46, §9.
- Status: Confirmed by Gordon 2026-11-02.

## D60

- Decision: core is consumed through a named core source; each env lives in its own instance repo; upgrading is a core ref bump. env-01 moves to its own private repo after the foundation fixes and before the chat-bot phase writes instance data. GitHub is the source of truth: the product repo is `gordoncooper/jarvis`, env-01’s instance repo is `gordoncooper/jarvis-internal`, and Gitea is retired for Jarvis. At the split, env-01 is rebuilt fresh from the instance repo, not migrated in place, and Flux deploys both sources from `main` with no signature check until the signed-tag gate lands (D56, D68). The rebuild is the D49 rehearsal: the CA is minted fresh with name constraints (D57), the agent token is rotated to its own value, the server token is kept in the short, passphrase-only form, and the restic password, the rest-server login and the off-host credentials move unchanged. The restic-password rotation rehearsal is its own later task (`p0b-13a`). bench is test-only; its data disk holds only test data but is still never wiped without the owner’s explicit say-so. Later-phase steps that write instance data (`p2-07` renders, `p2-10` bot profiles, P4a’s `apps.yaml`) write to the env’s instance repo.
- Reason: one public product, one private instance per env; a core bump is the upgrade.
- Where: `p0b-13`, `p0b-13a`, `p2-07`, `p2-09`, `p2-10`, `p4a-02`; framework R1, R34, §6.
- Status: Confirmed by Gordon 2026-11-02.

## D61

- Decision: the repo owner merges every PR. An agent merges only when its brief explicitly says to, after required checks pass and the reviewer has ticked every Ready to merge box that applies. Once branch protection requires `admin-gate`, a gated PR also needs an `admin-approved` label the owner added after the latest push, the owner merges it whatever the brief says, and an agent never adds that label, even when it runs under the owner’s account. `admin-gate` stays the PR-risk check on the product repo and on jarvis-apps (R36); it is not the deploy gate (D56).
- Reason: merge authority is the owner’s; the deploy gate is Flux’s signature check (D56). Matches `rulebooks/coding-agents.md` (jarvis `524f0e3`, tag v0.1.1).
- Where: every PR; framework R36, glossary, §8, §11.
- Status: Confirmed by Gordon 2026-11-02.

## D62

- Decision: voice stays deferred, but the GPU and network groundwork goes in now (the GPU runtime and the HTTPS path) so a first voice version over HTTPS fits later. UDP or SIP exposure is a recorded trigger.
- Reason: a later HTTPS voice path needs the GPU node and the TLS path; UDP and SIP are a new exposure.
- Where: `p0a-11`, `p0b-03`; framework §4, §5 Assistant API, §9.
- Status: Confirmed by Gordon 2026-11-02.

## D63

- Decision: hosted apps are stateless by default; stateful apps are allowed, with backup enrollment, once that app has its own backup credential (D51).
- Reason: an app PVC without its own restic credential would share core’s backup secret.
- Where: `p4a-01`, `p4a-05`, `p4a-06`; framework R30, R31.
- Status: Confirmed by Gordon 2026-11-02.

## D64

- Decision: code-factory lanes target running inside Jarvis; GitHub-hosted runners serve only the initial setup.
- Reason: GitHub-hosted runners cannot restrict egress (R42); the in-cluster lane is the product path.
- Where: `p4b-02`, `p4b-06a`; framework R42, §9.
- Status: Confirmed by Gordon 2026-11-02.

## D65

- Decision: lane logins are an adapter list (a Cursor key, a Claude subscription token); each adopter brings their own; parallel unattended lanes are capped; token expiry is on the keep-safe list (D50).
- Reason: one vendor login is not a product constant; an expired token is an outage.
- Where: `p4b-04`, `p4b-06a`; framework R23, R42, §8.
- Status: Confirmed by Gordon 2026-11-02.

## D66

- Decision: Hermes’s model login is a swappable provider (subscription login, a local model, or an API key), not tied to one vendor.
- Reason: the model-provider contract is a login or a base_url plus key (§5); env-01’s first login is still xai-oauth (R23).
- Where: `p2-01`, `p2-13`; framework R23, §5 Model provider.
- Status: Confirmed by Gordon 2026-11-02.

## D67

- Decision: backups sit on the data disk, plus the off-host copy (D48), plus a rebuildable-cache class (model weights, build caches) that is never backed up. `backup: none` remains the annotation; observability volumes stay on that allow-list, and the rebuildable-cache class joins it.
- Reason: backing up rebuildable caches doubles their disk and buys no recovery.
- Where: `p0b-02`, `p0b-09c`; framework R31, §5 Backups.
- Status: Confirmed by Gordon 2026-11-02.

## D68

- Decision: moving env-01 to `gordoncooper/jarvis-internal` and rebuilding it fresh is one step (`p0b-13`). In it Flux deploys both sources from `main` with no signature check: the instance root over one read-only deploy key (D51), core with no credential. The signed-tag gate (D56: Flux `verify`, mode `Tag`, both sources, only Gordon’s keys) is a separate, later step (`p2-08a`). It lands after the foundation build and before the chat bot holds Gordon’s real data, so `p2-09`, the first real data volume, depends on it.
- Reason: the move and the rebuild are a large change on their own; the gate only has to hold before real data arrives.
- Where: `p0b-13`, `p2-08a`, `p2-09`; framework R7, R25, R33, R36, §1, §4, §5, §6, §8; D56, D60.
- Status: Confirmed by Gordon 2026-11-04.

## D69

- Decision: core stands alone, Jarvis add-ons are optional, and the chat bot is a slot.
  - Core stands alone. Core is the k3s and Flux foundation: nodes, k3s, Flux, certs, ingress, local DNS, backups, secrets and CI. It installs and runs as a general k3s foundation. An env with no add-ons enabled is fully healthy, and core health is `make verify`.
  - Jarvis add-ons are optional. Add-ons are Jarvis-owned apps and services. They live in the `jarvis` repo under `services/`, are off by default, and each env enables them in its own instance repo.
  - Add-on wiring (built in jarvis #177; contract at `services/CONTRACT.md`, slots at `services/slots/<slot>/CONTRACT.md`). The services stage runs as the platform account and applies only each add-on’s wiring: its namespaces, tenant account and Flux Kustomizations. Each add-on’s own objects run as its tenant account, `addon-<name>` in `flux-system`. Add-ons take no substitution sources: a value they need is an inline `postBuild.substitute` entry filled from a key the env’s `cluster-vars` already publishes. Disabling an add-on prunes its workloads; its namespace, tenant account, Roles and secrets stay (`prune: disabled`) until the cluster admin deletes them by hand. An add-on’s standing checks are `services/<name>/verify.py`.
  - The chat bot is a slot. It sits behind a slot contract (`services/slots/chat-bot/CONTRACT.md`, D7) covering hosting, storage, backups, secrets, URL/ingress, and later the assistant API and model provider. Hermes is its first implementation. Core and other add-ons depend on the slot, never on Hermes by name, so swapping Hermes replaces one add-on with no core edits. Tools and checks that serve Hermes (profile render, on-demand backup, Google consent, the Hermes verify checks) live in the add-on’s folder, and the verify checks run only when the env enables it.
  - Observability (VictoriaMetrics, VictoriaLogs, Fluent Bit, vmagent, Grafana, Gatus, alerting) is an optional add-on, not core. A rule or step that uses it applies when the env enables it; core’s own health is `make verify`.
  - The code factory is its own add-on. Its shape is decided at P4b.
  - `jarvis-apps` stays the untrusted zone (D51, D63, R30).
- Reason: core is useful on its own as a k3s foundation, and the chat-bot runtime may be replaced later.
- Where: `p0a-04`, `p0b-01`, `p0b-06`, the P1 phase note, `p1-13`, `p2-01`, `p2-05`, `p2-07`, `p2-09`, `p2-12`, `p2-14`, `p2-18`, `p2-19`, `p2-20`, `p2-21`, `p2-25`, `p3-00`, `p3-08`, `p4a-03`, `p4a-07`, `p4b-15`; D7, D24, D40, D51; framework R1, R3, R5, R7, R8, §1, §2, §4, §5, §6, §8.
- Status: Confirmed by Gordon 2026-11-04.
