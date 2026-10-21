# JARVIS AlphaCut — Plan

Derived only from jarvis-alphacut/framework.md. Defaults the framework leaves open are in jarvis-alphacut/decisions.md and cited here as D numbers. One PR per task. Timeboxes are upper bounds, not estimates; most tasks need far less. Done checks follow the framework §11 lean standard: the live run is the proof where there is one.

## Stop and ask

- None (OQ9 confirmed by Gordon 2026-10-19: §4 logs and metrics stack stands).

## Gordon-only actions

One list, from §8. Each action names the task that waits on it.

- Before P0: only Gordon merges in project-plans (gordon5417-ai may open PRs). Task: p0a-01.
- Before P0: merge jarvis-alphacut/framework.md, then jarvis-alphacut/plan.md (§11). Task: p0a-01.
- Before P0: archive the previous system’s repos read-only. Task: p0a-01.
- Before P0: every token, key and webhook for the new build is new; none is reused. Task: p0a-01.
- P0: create the two repos under gordoncooper (D1), before anything writes to jarvis. Task: p0a-01.
- P0: after p0a-06 merges, set branch protection requiring the ci check. Task: p0a-05.
- P0: invite gordon5417-ai (write) to jarvis and jarvis-apps only. Task: p0a-03.
- P0: accept the two invites signed in as gordon5417-ai. Task: p0a-03.
- P0: set up the bastion’s agent user with a Cursor login and the gordon5417-ai token, which Gordon creates signed in as gordon5417-ai and pastes onto the bastion. Task: p0a-03.
- P0: generate the age key (public key in .sops.yaml, private key offline). Task: p0a-03.
- P0: install Renovate. Task: p0a-15.
- P0: run make bootstrap-secrets on the bastion and commit its output. Task: p0a-04.
- P0: wipe the env-01 node (a separate spare compute node; never a node of the previous system). Task: p0a-12.
- P0: fresh-install Ubuntu 26.04 Server (minimal install) with Gordon’s SSH key on the env’s SSH user (ssh_user, default ubuntu; D34). Task: p0a-12.
- P0: set a DHCP reservation. Task: p0a-12.
- P0: run the first site.yml, then the second site.yml and the second make bootstrap-secrets, from the controller (each needs the age key, so these are owner runs). Task: p0a-14.
- P0: for a node with a secondary data disk, record its stable disk id in nodes.yaml and confirm the wipe at run time (D42). Task: p0a-11a.
- P0: once JARVIS DNS and the CA exist, set router forwarding of alpha.lan. Task: p0b-08.
- P0: trust the root cert on each device. Task: p0b-08.
- P1: create the send-only alerts bot with BotFather. Task: p1-09.
- P1: send it /start once and record his chat ID. Task: p1-09.
- P2: create concierge’s Telegram token. Task: p2-15.
- P2: record Gordon’s user ID (the chat ID from P1) for concierge’s allow-list. Task: p2-15.
- P2: approve the ops deploy’s xAI device login. Task: p2-13.
- P2: during the provider-down and capped-login drills, send a Telegram message. Task: p2-18.
- P2: at the exit, create the admin-approved label. Task: p2-24.
- P2: add admin-gate to branch protection on jarvis and jarvis-apps. Task: p2-24.
- P3: publish the Google OAuth app as In production. Task: p3-00.
- P3: create the Google Cloud OAuth client and enable the Gmail and Calendar APIs. Task: p3-00.
- P3: create the no-repos public-read-only GitHub token. Task: p3-02.
- P3: run make google-consent. Task: p3-00.
- P3: send himself one email with subject jarvis-p3-seed. Task: p3-01a.
- P4: approve the code-factory deploy’s xAI device login. Task: p4b-05.
- P4: create the Cursor user API key. Task: p4b-06a.
- P4: set Cursor billing to auto top-up off with a monthly limit (on the plan tier that includes Claude for code-factory:hard). Task: p4b-06a.
- P4: create the GitHub App by hand from its four-permission checklist (contents, pull requests, issues, checks; no manifest flow). Task: p4b-06b.
- P4a: merge the app’s line in envs/<env>/values/apps.yaml. Task: p4a-07.
- P4a: set its GHCR package to public on first publish. Task: p4a-04.
- P4b: apply code-factory:ready. Task: p4b-11.
- P4b: merge code-factory PRs. Task: p4b-11.
- Standing: own router-side access (Tailscale on the router advertises a route to the LAN; the router handles LAN and tailnet DNS; nothing for remote access is installed on the nodes). Task: p0b-11.
- Standing: verify the [G] exit checks (§2). Tasks: p0b-11, p1-13, p2-15, p2-25, p3-03, p3-08, p4a-07.
- Standing: review PRs held by admin-gate and add admin-approved from his own account. Task: p2-24 (also any later protected PR, including p4a-05, p4a-07, p4b-01, p4b-15).
- Standing: merge Hermes bump PRs (R33). Task: p2-20.
- Standing: re-run site.yml after merging an infra/ PR. Task: p0a-13.
- Standing: run OS package upgrades by hand (Ubuntu never upgrades itself) and reboot after kernel updates. Task: p0a-11.
- Standing: keep the previous system frozen (no new work, but not shut down, wiped or destroyed). Task: p0a-01.
- §8 secrets: home location (rounded to 2 decimals) and Gordon’s contact email, SOPS env value for the geo User-Agent. Task: p2-11.
- §8 custody: after a Hermes volume restore, Gordon redoes the xAI device login if the token is stale. Task: p2-21.
- §8 custody: Gordon keeps a calendar reminder a month before the internal CA root expires. Task: p0b-04.

## P0a

Goal: Git drives the cluster from a blank node, with CI rule checks, secrets and Renovate working.

### Contracts

```text
ID:          p0a-01
Goal:        Write the Env schema CONTRACT.md from the shared template, with proven by.
Why:         P0a is env values before any manifest (R3, R4). The contract is written before the validator (§5, §11).
Advances:    P0a exit check: “One required check on main, ci, runs every tools/checks/ rule check (admin-gate joins at the P2 exit, R36), and a seeded bad PR fails each rule check as it lands (Examples: a hardcoded DNS suffix, a plaintext secret, a missing CONTRACT.md, an unknown code_delivery or build_agent value, a deny list missing a whole deny class; a check added later is seeded in the task that adds it, such as the PVC backup-entry check at P0b and the stale render at P2).”
Depends on:  none
Touches:     docs/CONTRACT.template.md, services/hermes/core/schema/CONTRACT.md
Hard limits: R3; R4; R23 (tiers and model names only under envs/). Allowed code_delivery is github and allowed build_agent is cursor (§5). The time zone is part of the env schema. env-01 runs on America/Los_Angeles, and the node and the Hermes schedules both use it. The schema includes ssh_user, the node SSH login, default ubuntu (D34). Gordon creates jarvis and jarvis-apps under gordoncooper before this PR writes to jarvis (D1). D7. Profile schema is a later section of this file (p2-04).
Done checks: The file has offers, expects, health, config, and proven by, and the schema includes the time zone and ssh_user. One PR.
Timebox:     4h
Proven by:   Env schema validation in CI, and the P0a seeded unknown code_delivery or build_agent value.
```

```text
ID:          p0a-02
Goal:        Write the Code delivery provider CONTRACT.md for the six capabilities, GitHub only, with proven by.
Why:         Services, the code-factory, and app hosting depend on this contract and not on GitHub by name (§5, R34).
Advances:    P0a exit check: “One required check on main, ci, runs every tools/checks/ rule check (admin-gate joins at the P2 exit, R36), and a seeded bad PR fails each rule check as it lands (Examples: a hardcoded DNS suffix, a plaintext secret, a missing CONTRACT.md, an unknown code_delivery or build_agent value, a deny list missing a whole deny class; a check added later is seeded in the task that adds it, such as the PVC backup-entry check at P0b and the stale render at P2).”
Depends on:  p0a-01
Touches:     docs/code-delivery/CONTRACT.md
Hard limits: R34; R36 (admin-gate is named here and enforced at the P2 exit). Forgejo stays a §9 seam: this contract names it as the planned second provider and adds no Forgejo code. D7.
Done checks: CONTRACT.md lists repo hosting, CI, the issue queue, PR open and merge, the approval-label check, and the registry, and says which checkpoint proves each. One PR.
Timebox:     4h
Proven by:   Env schema validation and the P0a, P4a, and P4b checks (§5).
```

### Build

```text
ID:          p0a-03
Goal:        Create the jarvis repo skeleton, env-01 values, and the hostname-only inventory.
Why:         Later tasks need one tree that matches §6, with env specifics only under envs/env-01 (R3).
Advances:    P0a exit check: “From a blank node, every Flux object reports Ready; a second run of bootstrap-secrets and site.yml changes nothing.”
Depends on:  p0a-01, p0a-02
Touches:     jarvis Makefile, docs/README.md, .sops.yaml, envs/env-01/, rulebooks/ left for p0a-17
Hard limits: R3; R10; R25 (hostnames only; the age public key is the recipient Gordon provides). D1; D2. Create only paths this checkpoint uses. Hermes deploy folders, app-hosting, and observability wait for their phases. docs/README.md points at project-plans and does not copy the framework (§6).
Done checks: cluster-vars.yaml sets code_delivery: github, build_agent: cursor, DNS suffix alpha.lan, time zone America/Los_Angeles, the §7 retention, and the §7 env-01 memory budgets and headroom. hermes.yaml holds the §7 tier map and an empty bot list. apps.yaml has app hosting off. inventory/hosts.yml lists bench.lan (D2). envs/env-01/README.md states purpose, what deploys, and promotion. One PR.
Timebox:     6h
```

```text
ID:          p0a-04
Goal:        Add make bootstrap-secrets and make seal: create-if-missing, second run changes nothing.
Why:         Git has to hold the §8 bootstrap secrets before Flux can decrypt one (R7, R25).
Advances:    P0a exit check: “From a blank node, every Flux object reports Ready; a second run of bootstrap-secrets and site.yml changes nothing.”
Depends on:  p0a-03
Touches:     tools/bootstrap-secrets, tools/seal, Makefile
Hard limits: R7; R25; R35. The script creates the internal CA, the VolSync restic password, the Grafana admin password, and each deploy’s Hermes API key, dashboard password hash, and signing secret (D16). It never overwrites. The real age private key stays out of CI (D33). D10: only the restic secret is referenced in-cluster at P0a. There is no default env: with one env under envs/ both tools pick it, and once there are two, ENV= (or --env) is required.
Done checks: A unit test with a throwaway age key shows a second run leaves the files byte-identical. Gordon runs make bootstrap-secrets on the controller and commits the SOPS output (see Gordon-only actions). One PR.
Timebox:     6h
```

```text
ID:          p0a-06
Goal:        Add the required ci workflow, the gitleaks secret scan, a seeded plaintext secret, and a green jarvis-apps ci.
Why:         P0a’s required check is ci, and §4 names the pinned gitleaks CLI as the CI secret scan (R25, §8 phase 0 order).
Advances:    P0a exit check: “One required check on main, ci, runs every tools/checks/ rule check (admin-gate joins at the P2 exit, R36), and a seeded bad PR fails each rule check as it lands (Examples: a hardcoded DNS suffix, a plaintext secret, a missing CONTRACT.md, an unknown code_delivery or build_agent value, a deny list missing a whole deny class; a check added later is seeded in the task that adds it, such as the PVC backup-entry check at P0b and the stale render at P2).”
Depends on:  p0a-03
Touches:     jarvis/.github/workflows/ci.yml, tools/checks/secret-scan, jarvis-apps/.github/workflows/ci.yml
Hard limits: R25; R36 (admin-gate is not part of ci; it joins at the P2 exit). The secret scan is the pinned gitleaks CLI (§4). jarvis-apps ci runs on pull_request and holds no secrets (D35). Workflows that would hold secrets do not run on fork PRs. D1; D35.
Done checks: ci runs every script in tools/checks/. A seeded PR with a plaintext secret fails gitleaks. jarvis-apps ci reports success on an empty tree. One PR.
Timebox:     6h
```

```text
ID:          p0a-05
Goal:        Implement the env schema validator and seed one PR with an unknown provider value.
Why:         Fix order (§11): the schema is the boundary; CI only runs it. This is the code that makes the easy values the legal values (R4).
Advances:    P0a exit check: “One required check on main, ci, runs every tools/checks/ rule check (admin-gate joins at the P2 exit, R36), and a seeded bad PR fails each rule check as it lands (Examples: a hardcoded DNS suffix, a plaintext secret, a missing CONTRACT.md, an unknown code_delivery or build_agent value, a deny list missing a whole deny class; a check added later is seeded in the task that adds it, such as the PVC backup-entry check at P0b and the stale render at P2).”
Depends on:  p0a-01, p0a-03, p0a-06
Touches:     services/hermes/core/schema/, tools/checks/env-schema
Hard limits: R3; R4; R10 (Python). Validates cluster-vars.yaml, hermes.yaml, apps.yaml, and inventory/hosts.yml (§5). Memory budgets and the headroom target are schema fields whose defaults are the full-cluster §7 numbers (D30). Gordon sets branch protection requiring ci after p0a-06 merges, before the seeded PR. Unknown code_delivery or build_agent fails. A missing ssh_user means ubuntu (D34). Core naming an env, a model, or a DNS suffix is p0a-07.
Done checks: pytest covers a valid env-01, a bad build_agent, a malformed memory budget, and ssh_user defaulting to ubuntu. A seeded PR that sets code_delivery to a value other than github fails ci. One PR.
Timebox:     6h
```

```text
ID:          p0a-07
Goal:        Add the core-names check and seed a PR that hardcodes a DNS suffix.
Why:         Core stays env-agnostic only if CI rejects an env name or a DNS suffix in core (R3). Model names join this check at p2-23, when the env model maps hold names.
Advances:    P0a exit check: “One required check on main, ci, runs every tools/checks/ rule check (admin-gate joins at the P2 exit, R36), and a seeded bad PR fails each rule check as it lands (Examples: a hardcoded DNS suffix, a plaintext secret, a missing CONTRACT.md, an unknown code_delivery or build_agent value, a deny list missing a whole deny class; a check added later is seeded in the task that adds it, such as the PVC backup-entry check at P0b and the stale render at P2).”
Depends on:  p0a-06
Touches:     tools/checks/core-names
Hard limits: R3; R28. envs/** and docs/** are exempt; contracts use placeholders such as <env> and <dns_suffix>. The check also rejects inventory hostnames, ssh_user logins, home-directory paths and controller or owner account names outside envs/, so core works from any controller.
Done checks: A seeded PR that writes alpha.lan under platform/ or services/ fails ci. One PR.
Timebox:     4h
```

```text
ID:          p0a-08
Goal:        Add the CONTRACT.md check and seed a PR that drops one.
Why:         Fix order (§11): the folder layout is the boundary (R2); this script only reports a missing contract. D8 defines which folders count.
Advances:    P0a exit check: “One required check on main, ci, runs every tools/checks/ rule check (admin-gate joins at the P2 exit, R36), and a seeded bad PR fails each rule check as it lands (Examples: a hardcoded DNS suffix, a plaintext secret, a missing CONTRACT.md, an unknown code_delivery or build_agent value, a deny list missing a whole deny class; a check added later is seeded in the task that adds it, such as the PVC backup-entry check at P0b and the stale render at P2).”
Depends on:  p0a-06, p0a-01
Touches:     tools/checks/contract-md
Hard limits: R2; D8. The seeded folder is a throwaway component root on the §6 list, removed when the PR closes.
Done checks: A seeded PR whose component root has no CONTRACT.md fails ci. Existing contract files pass. One PR.
Timebox:     4h
```

```text
ID:          p0a-10
Goal:        Add the Cursor CLI deny list and its rule check, so agents change the cluster only through Git.
Why:         Fix order (§11): the deny list is the automated guard for R34. The rulebook comes after the checks (p0a-17) and does not repeat this list.
Advances:    P0a exit check: “One required check on main, ci, runs every tools/checks/ rule check (admin-gate joins at the P2 exit, R36), and a seeded bad PR fails each rule check as it lands (Examples: a hardcoded DNS suffix, a plaintext secret, a missing CONTRACT.md, an unknown code_delivery or build_agent value, a deny list missing a whole deny class; a check added later is seeded in the task that adds it, such as the PVC backup-entry check at P0b and the stale render at P2).”
Depends on:  p0a-06
Touches:     .cursor/cli.json, tools/checks/cli-deny
Hard limits: R34. Deny direct kubectl and flux writes, helm, ssh, secret decryption, and reading the age private key. Reads stay allowed. The list is a guardrail, not a hard boundary (shell wrappers can bypass it), so keep patterns narrow enough not to block reads, and never pass --force or --yolo (P4b adapter).
Done checks: tools/checks/cli-deny prints one dry-run table showing kubectl apply, helm, ssh, sops decrypt and an age-key read blocked and kubectl get allowed. A seeded PR that removes a whole deny class (Example: the age-key read deny) fails ci. One PR.
Timebox:     4h
```

```text
ID:          p0a-11
Goal:        Add the Ansible base-os role: journald cap, chrony with NTS, the time zone, and no automatic upgrades or reboots.
Why:         The blank node has to match the §4 OS before k3s is installed (R1, R10).
Advances:    P0a exit check: “From a blank node, every Flux object reports Ready; a second run of bootstrap-secrets and site.yml changes nothing.”
Depends on:  p0a-03
Touches:     infra/ansible/roles/base-os, infra/ansible/ansible.cfg, infra/ansible/site.yml
Hard limits: R1 (Ansible only in infra/); R10; §4 Ubuntu 26.04 minimal. journald SystemMaxUse 2G (§7). The node time zone comes from the env schema, the same value the Hermes schedules use. Ubuntu never upgrades itself: unattended-upgrades and the apt-daily-upgrade timer are off, and the package-list refresh stays on. Our tooling may install packages. Upgrades and reboots stay Gordon’s (§8). Outbound 123/udp and 4460/tcp stay available for chrony. Config files use Ansible validate: where the tool can check its own file, rather than block/rescue. Ansible is the pinned PyPI ansible package (it bundles community.general; ansible-core alone is not enough). No Tailscale or VPN on the node (R28).
Done checks: ansible-playbook --syntax-check site.yml passes. Small tests assert the rules in the rendered files (journald cap, NTS pools, automatic upgrades off). The first live site.yml (p0a-14) is the proof for the running services: chronyc -N authdata shows NTS, and no upgrade timer is active; a container cannot prove chrony (it needs CAP_SYS_TIME and outbound NTS). One PR.
Timebox:     6h
```

```text
ID:          p0a-11a
Goal:        Declare each node’s roles and optional data disk in env values, and have Ansible prepare the disk before k3s first starts (D42).
Why:         k3s, its etcd snapshots, volumes and backups write under /var/lib/jarvis, and k3s applies node labels only when a node first registers, so disks and roles come before k3s (R3, R31).
Advances:    P0a exit check: “From a blank node, every Flux object reports Ready; a second run of bootstrap-secrets and site.yml changes nothing.”
Advances:    P0b exit check: “Restore drill: a deleted volume is restored by VolSync (or the restic CronJob fallback, per the Phase 0 test) and its checksum matches; an etcd snapshot exists for same-node rollback (R31).”
Depends on:  p0a-05, p0a-11
Touches:     envs/env-01/values/nodes.yaml, services/hermes/core/schema/, infra/ansible/roles/node-disk, infra/ansible/site.yml
Hard limits: R1; R3; R4; R31. D3; D42. node-disk runs after base-os and before k3s. The disk is named by a stable /dev/disk/by-id path, never a kernel name. Ansible refuses to wipe without both the committed confirm flag and a matching run-time confirmation, and never touches a disk that holds the root filesystem or is mounted. A disk already marked as this env’s Jarvis data disk is mounted, never wiped. local_path_data and backup_path do not change: they sit under the data mount, or on the root disk when a node has none. Only one server node (control-plane) is built; other roles are labels, which p0a-12 writes into the k3s config.yaml.
Done checks: The env schema rejects an unknown role, a node not in the inventory, a data disk without a by-id path, and an env without exactly one control-plane node (small tests). A narrow node-disk run without the run-time confirmation stops before touching the disk and prints its model, size and partitions (no age key needed). Gordon’s first live site.yml (p0a-14, an owner run) wipes and mounts the data disk before k3s starts, and his second changes nothing. One PR.
Timebox:     6h
```

```text
ID:          p0a-12
Goal:        Add the k3s role: config.yaml, fixed CIDRs, secrets encryption, reserves, local-path, backup path, etcd snapshots, marker file.
Why:         Phase 0 installs k3s from config.yaml because Ubuntu sudo-rs drops INSTALL_K3S_* (§8, R1).
Advances:    P0a exit check: “From a blank node, every Flux object reports Ready; a second run of bootstrap-secrets and site.yml changes nothing.”
Advances:    P0b exit check: “Restore drill: a deleted volume is restored by VolSync (or the restic CronJob fallback, per the Phase 0 test) and its checksum matches; an etcd snapshot exists for same-node rollback (R31).”
Depends on:  p0a-11a
Touches:     infra/ansible/roles/k3s, envs/env-01/values/cluster-vars.yaml
Hard limits: R1; §7 (kube and system reserves about 1 GB each, eviction thresholds); R28; R31; R33. D2; D3; D4; D5; D6; D34. The k3s version and binary checksum are pinned in cluster-vars.yaml and change only by PR. Renovate does not manage them. Ansible logs in as ssh_user and runs every task as root with become (D34). site.yml refuses a host that already runs k3s unless /etc/jarvis/env holds this env’s name; the first run writes that marker. site.yml writes the kubeconfig only to the running user’s ~/.kube/config; copying it to another controller user is a controller setup step outside the repo. The node is the spare env-01 box, wiped and freshly installed by Gordon before this role runs.
Done checks: A small test of the rendered config.yaml shows --cluster-init, secrets encryption, the CIDRs, the reserves, the snapshot directory, the node-label entries from nodes.yaml (role.jarvis.lan/<role>=true, D42), and no INSTALL_K3S_* variable. One container test, on the node OS image (ubuntu:26.04), shows site.yml refuses a host that runs k3s without the marker (the only safe proof of that guard); the first live site.yml proves the fresh-node path. One PR.
Timebox:     8h
```

```text
ID:          p0a-13
Goal:        Add the Flux Operator Ansible role: one-time apply, sops-age, then Flux manages the operator.
Why:         The age key and the first Flux apply are the bootstrap bridge (R1, R25). After that, Git is the source (R34).
Advances:    P0a exit check: “From a blank node, every Flux object reports Ready; a second run of bootstrap-secrets and site.yml changes nothing.”
Depends on:  p0a-12, p0a-04
Touches:     infra/ansible/roles/flux-operator, infra/ansible/site.yml
Hard limits: R1; R25. The private age key is applied once as flux-system/sops-age, which Flux never reconciles, with no_log on every task that handles it. A full site.yml needs SOPS_AGE_KEY_FILE, so it is an owner run by Gordon; the agent user never holds the age key. The agent user keeps SSH to the node only for narrow runs that need no age key (for example site.yml --tags kubeconfig). site.yml sets ansible_user from cluster-vars ssh_user and become: true (D34). The operator chart pin in the role and in Git have one source of truth, so Renovate bumps both.
Done checks: site.yml runs the four roles in §8 order (base-os, node-disk, k3s, flux-operator). Gordon’s first and second live site.yml (p0a-14) prove the apply and its idempotence; no container re-run of this role. One PR.
Timebox:     6h
```

```text
ID:          p0a-14
Goal:        Add the Flux entry point: four Kustomizations with dependsOn, the operator HelmRelease, and one decrypted Secret.
Why:         Git has to drive a blank node, with platform before configs before services before apps (R1, R7).
Advances:    P0a exit check: “From a blank node, every Flux object reports Ready; a second run of bootstrap-secrets and site.yml changes nothing.”
Advances:    P0a exit check: “A SOPS-encrypted secret decrypts in-cluster.”
Depends on:  p0a-13, p0a-02, p0a-08
Touches:     envs/env-01/flux/, platform/flux-operator/, platform/backups/ namespace only
Hard limits: R1; R4 (cluster-vars through postBuild substitution only); R25. D9 (one uniquely named placeholder ConfigMap per empty layer); D10; D11 (the operator chart pin starts one release behind). Resources that use their own ${…} syntax set substitute: disabled. The cluster-vars ConfigMap is rendered from values/cluster-vars.yaml by one script, and CI fails if the committed render is stale. No hand-edited cluster state (R34).
Done checks: CI runs kustomize build plus a schema check on every Flux Kustomization path, read from the Kustomization files so a new path is never skipped. After Gordon’s first site.yml, every Flux object reports Ready and the restic Secret exists in platform-backups with its value unprinted. Gordon’s second site.yml and second bootstrap-secrets change nothing; both are owner runs because they need the age key (D17). One PR.
Timebox:     8h
```

```text
ID:          p0a-15
Goal:        Add Renovate on jarvis and jarvis-apps, with minor and patch auto-merge held for about 3 days.
Why:         P0a is done only after Renovate has opened a PR (R33).
Advances:    P0a exit check: “Renovate is installed and has opened at least one PR (its onboarding PR counts).”
Depends on:  p0a-14
Touches:     renovate.json in both repos
Hard limits: R33. Majors are not auto-merged. Hermes digests are not auto-merged (the digest rule is enforced again in p2-20). Renovate does not manage the k3s version (D5). The image digest check itself lands with the first third-party image (p0b-07). Gordon installs the hosted Renovate GitHub App; no local Renovate or config-validator tooling (§11 lean standard).
Done checks: Renovate has opened its onboarding PR or a dependency PR. The config shows the 3-day hold for minor and patch and the k3s exclusion. One PR.
Timebox:     4h
```

```text
ID:          p0a-16
Goal:        Run the rollback drill: change a throwaway ConfigMap through Git, revert the commit, and show Flux restored the prior value.
Why:         Rollback is revert-the-commit (R37). P0a has to prove it once.
Advances:    P0a exit check: “Rollback drill: reverting a commit restores the prior state.”
Depends on:  p0a-14
Touches:     a throwaway ConfigMap under the platform-configs path
Hard limits: R34; R37. The drill uses a ConfigMap, not a Secret. It needs only git revert and kubectl reads; no helper script. The ConfigMap is removed once the drill passes, and the log stays with the P0a evidence outside the repo. Data restore is P0b (R31).
Done checks: The attached log shows the new value, the revert, and the prior value back on the cluster, and the ConfigMap is gone afterward. A few tiny PRs (baseline, change, revert, remove).
Timebox:     4h
```

```text
ID:          p0a-17
Goal:        Write the coding-agent rulebook, a lean AGENTS.md, and the PR template.
Why:         Fix order (§11): prose only for what the checks and the deny list cannot enforce (R39). This task comes after p0a-06 through p0a-10.
Advances:    P0a exit check: “One required check on main, ci, runs every tools/checks/ rule check (admin-gate joins at the P2 exit, R36), and a seeded bad PR fails each rule check as it lands (Examples: a hardcoded DNS suffix, a plaintext secret, a missing CONTRACT.md, an unknown code_delivery or build_agent value, a deny list missing a whole deny class; a check added later is seeded in the task that adds it, such as the PVC backup-entry check at P0b and the stale render at P2).”
Depends on:  p0a-06, p0a-07, p0a-08, p0a-10
Touches:     rulebooks/coding-agents.md, AGENTS.md, .github/pull_request_template.md
Hard limits: R35; R39. AGENTS.md points at the rulebook and does not copy it or the framework. The rulebook carries the §11 lean standard and review rules in a few lines. The PR template is the brief template plus an evidence line and a short ready-to-merge list; its only seed rule is that each new or changed rule check fails on a seeded bad value. Branch names for coding agents are agent/<topic> (§6).
Done checks: A reviewer can point at one rule that exists only in the rulebook because it cannot be a check. The template contains the §11 fields. One PR.
Timebox:     4h
```

```text
ID:          p0a-18
Goal:        Add make verify PHASE=p0a as a short checklist and attach its run.
Why:         Done for the checkpoint is attached evidence that Gordon reviews (R35, §2).
Advances:    P0a exit check: “From a blank node, every Flux object reports Ready; a second run of bootstrap-secrets and site.yml changes nothing.”
Advances:    P0a exit check: “A SOPS-encrypted secret decrypts in-cluster.”
Advances:    P0a exit check: “One required check on main, ci, runs every tools/checks/ rule check (admin-gate joins at the P2 exit, R36), and a seeded bad PR fails each rule check as it lands (Examples: a hardcoded DNS suffix, a plaintext secret, a missing CONTRACT.md, an unknown code_delivery or build_agent value, a deny list missing a whole deny class; a check added later is seeded in the task that adds it, such as the PVC backup-entry check at P0b and the stale render at P2).”
Advances:    P0a exit check: “Renovate is installed and has opened at least one PR (its onboarding PR counts).”
Advances:    P0a exit check: “Rollback drill: reverting a commit restores the prior state.”
Depends on:  p0a-04, p0a-05, p0a-14, p0a-15, p0a-16, p0a-17
Touches:     Makefile, tools/verify/p0a
Hard limits: R35; R37. D17. About 50 lines, outside tools/checks/ so CI never runs it: live kubectl reads (every Flux object Ready, the restic Secret present in platform-backups with its value unprinted, the role labels on each node) and the evidence links (ci seed PRs, the Renovate PR, the rollback log). It does not re-run CI checks and has no tests of its own. Gordon’s second site.yml and second bootstrap-secrets are owner runs: it prints NEEDS GORDON for each, and the attached output includes his pastes. It stays in the repo so a rebuild re-runs it.
Done checks: make verify PHASE=p0a exits 0 on the live cluster with the Flux Ready list and the decrypt proof, lists the evidence links, and prints NEEDS GORDON for the two owner runs; the attached output includes Gordon’s pastes. One PR.
Timebox:     2h
```

### Exit checks

- From a blank node, every Flux object reports Ready; a second run of bootstrap-secrets and site.yml changes nothing.
- A SOPS-encrypted secret decrypts in-cluster.
- One required check on main, ci, runs every tools/checks/ rule check (admin-gate joins at the P2 exit, R36), and a seeded bad PR fails each rule check as it lands (Examples: a hardcoded DNS suffix, a plaintext secret, a missing CONTRACT.md, an unknown code_delivery or build_agent value, a deny list missing a whole deny class; a check added later is seeded in the task that adds it, such as the PVC backup-entry check at P0b and the stale render at P2).
- Renovate is installed and has opened at least one PR (its onboarding PR counts).
- Rollback drill: reverting a commit restores the prior state.

Done = attached make verify PHASE=p0a output

## P0b

Goal: services answer over HTTPS by name on LAN and tailnet, and a volume restores.

### Contracts

```text
ID:          p0b-01
Goal:        Write the Exposure CONTRACT.md with proven by.
Why:         A hostname is one Ingress, and the contract is written before Traefik, the CA, and DNS (§5, §11).
Advances:    P0b exit check: “A test app answers over HTTPS on <svc>.<DNS suffix> (env-01: <svc>.alpha.lan) with a cert from the internal CA, from LAN and from tailnet [G], through Traefik Ingress.”
Depends on:  p0a-18
Touches:     platform/traefik/CONTRACT.md
Hard limits: R5; R28. D7. The contract states JARVIS DNS answers the name, the wildcard covers it, and the component adds its own Gatus check. Gatus trusts the public root and resolves the zone through cluster DNS.
Done checks: CONTRACT.md has offers, expects, health, config, and proven by. One PR.
Timebox:     4h
Proven by:   The R5 manifest lint and the P0b, P2, and P4a reachability checks (§5).
```

```text
ID:          p0b-02
Goal:        Write the Backups CONTRACT.md with proven by.
Why:         Every PVC uses one shared backup contract before VolSync is installed (§5, R31).
Advances:    P0b exit check: “Restore drill: a deleted volume is restored by VolSync (or the restic CronJob fallback, per the Phase 0 test) and its checksum matches; an etcd snapshot exists for same-node rollback (R31).”
Depends on:  p0a-18
Touches:     platform/backups/CONTRACT.md
Hard limits: R31. D7. Target path, retention, and weekly prune come from cluster-vars.yaml. Restore is a ReplicationDestination with a written procedure. The opt-out is the unprefixed annotation backup: none on a PVC or on a volumeClaimTemplate’s metadata, allowed only under services/observability/victoria-metrics/ and services/observability/victoria-logs/. A StatefulSet’s volumes get one ReplicationSource per <template>-<statefulset>-<ordinal> PVC.
Done checks: CONTRACT.md has offers, expects, health, config, and proven by. One PR.
Timebox:     4h
Proven by:   The R31 CI check (added with the first PVC, p0b-09) and the P0b and P2 restore drills (§5).
```

### Build

```text
ID:          p0b-03
Goal:        Install Traefik’s HelmChartConfig for plain Ingress and the default TLSStore.
Why:         P0b reachability is HTTPS through Traefik Ingress (OQ8, R28).
Advances:    P0b exit check: “A test app answers over HTTPS on <svc>.<DNS suffix> (env-01: <svc>.alpha.lan) with a cert from the internal CA, from LAN and from tailnet [G], through Traefik Ingress.”
Depends on:  p0b-01
Touches:     platform/traefik/
Hard limits: R1; R4; R5; R9; R30 waits for P4a. D30; D32. kube-system receives only this HelmChartConfig. No Gateway API. The platform-controllers Kustomization points at one platform controllers list from here on, so later controllers join that list rather than add Flux paths. The placeholder ConfigMap on this path is removed (D9).
Done checks: The HelmChartConfig is applied and Traefik is Ready, probes and memory limits are set, and no Gateway API object exists. One PR.
Timebox:     6h
```

```text
ID:          p0b-04
Goal:        Install cert-manager, the internal CA, and the wildcard certificate. Commit the public root.
Why:         The test app’s cert has to come from the internal CA, and rebuilds have to reuse the root key (R28).
Advances:    P0b exit check: “A test app answers over HTTPS on <svc>.<DNS suffix> (env-01: <svc>.alpha.lan) with a cert from the internal CA, from LAN and from tailnet [G], through Traefik Ingress.”
Depends on:  p0b-03
Touches:     platform/cert-manager/controller, platform/cert-manager/config
Hard limits: R5; R25; R28. The root key is the SOPS secret from bootstrap-secrets. The public root is plain text for Gatus and for devices. The CA lifetime is 10 years. Gordon keeps the calendar reminder (§8 custody). D30.
Done checks: A Certificate for *.alpha.lan is Ready and the cert chains to the committed root. One PR.
Timebox:     6h
```

```text
ID:          p0b-05
Goal:        Install JARVIS DNS (CoreDNS with k8s_gateway) and the coredns-custom forward.
Why:         Names under alpha.lan have to resolve from the cluster and, once Gordon forwards the zone, from the LAN (R28, §4).
Advances:    P0b exit check: “A test app answers over HTTPS on <svc>.<DNS suffix> (env-01: <svc>.alpha.lan) with a cert from the internal CA, from LAN and from tailnet [G], through Traefik Ingress.”
Depends on:  p0b-04
Touches:     platform/lan-dns/, kube-system coredns-custom
Hard limits: R5; R28. D30. This CoreDNS is separate from k3s cluster DNS. It is exposed on the node IP, port 53. LoadBalancer Services are only Traefik and this DNS (the lint is p0b-06). No VPN agent on the node.
Done checks: From a pod, testapp is not required yet; a fixture Ingress name returns the node address from JARVIS DNS. One PR.
Timebox:     6h
```

```text
ID:          p0b-06
Goal:        Add the manifest lint for probes, resources, and stdout, and the LoadBalancer allow-list, each with a seeded PR.
Why:         Fix order (§11): components already set the fields (p0b-03 through p0b-05); these checks only catch a later miss (R5, R28).
Advances:    P0b exit check: “A test app answers over HTTPS on <svc>.<DNS suffix> (env-01: <svc>.alpha.lan) with a cert from the internal CA, from LAN and from tailnet [G], through Traefik Ingress.”
Depends on:  p0b-03, p0b-05
Touches:     tools/checks/manifest-lint, tools/checks/loadbalancer
Hard limits: R5 (memory request and limit, CPU request, no CPU limit, probes, stdout logs, Gatus when the endpoint exists). R28.
Done checks: A seeded manifest missing a memory limit fails. A seeded LoadBalancer besides Traefik and JARVIS DNS fails. Current platform manifests pass. One PR.
Timebox:     6h
```

```text
ID:          p0b-07
Goal:        Add the image digest pin check and seed an unpinned image.
Why:         Images are pinned by digest so Renovate is the upgrade path (R33).
Advances:    P0b exit check: “A test app answers over HTTPS on <svc>.<DNS suffix> (env-01: <svc>.alpha.lan) with a cert from the internal CA, from LAN and from tailnet [G], through Traefik Ingress.”
Depends on:  p0b-06
Touches:     tools/checks/digest-pin
Hard limits: R33. The k3s host binary stays a cluster-vars pin (D5), not an image digest.
Done checks: A seeded image: tag with no digest fails ci. Traefik, cert-manager, and CoreDNS manifests are digest-pinned. One PR.
Timebox:     4h
```

```text
ID:          p0b-08
Goal:        Deploy the test app and its Ingress, and prove HTTPS from the LAN.
Why:         P0b’s reachability check needs one named HTTPS service on the internal CA (R28).
Advances:    P0b exit check: “A test app answers over HTTPS on <svc>.<DNS suffix> (env-01: <svc>.alpha.lan) with a cert from the internal CA, from LAN and from tailnet [G], through Traefik Ingress.”
Depends on:  p0b-04, p0b-05, p0b-06, p0b-07
Touches:     platform/testapp/
Hard limits: R5; R28. D12. Hostname testapp.alpha.lan. The app is a platform fixture, not an app-<name> hosted app (that isolation is P4a, R30).
Done checks: Gordon sets router forwarding of alpha.lan and trusts the root cert on the controller before the LAN curl. curl from the LAN to https://testapp.alpha.lan returns the known file and the cert chains to the internal CA. The tailnet half is NEEDS GORDON (D17). One PR.
Timebox:     6h
```

```text
ID:          p0b-09
Goal:        Add the R31 PVC backup-entry check, run the VolSync backup-and-restore test (if it fails, install the single restic CronJob), then apply retention and the weekly prune.
Why:         OQ7 is a Phase 0 test with one fallback and no further searching (R31, §4).
Advances:    P0b exit check: “Restore drill: a deleted volume is restored by VolSync (or the restic CronJob fallback, per the Phase 0 test) and its checksum matches; an etcd snapshot exists for same-node rollback (R31).”
Depends on:  p0b-02, p0a-14
Touches:     tools/checks/pvc-backup, platform/backups/controller, platform/backups/config
Hard limits: R9; R31. No local S3 server. No volume snapshot controller. D3; D15; D30; D42. VolSync schedules take TZ from cluster-vars. The ~10-minute test writes the choice into the backup CONTRACT.md config section. The R31 check lands in this task, no later than the test volume, and stays small (about 60 lines): every PVC has a ReplicationSource or the p0b-02 backup: none annotation inside the allow-list. StatefulSet template handling waits for the first StatefulSet (P1).
Done checks: A seeded PR with a PVC that has neither a backup entry nor backup: none fails ci. The log shows the VolSync test result and either a working VolSync ReplicationSource or the one CronJob, writing to backup_path. Retention in cluster-vars is 7 daily / 4 weekly / 3 monthly (§7). One PR.
Timebox:     8h
```

```text
ID:          p0b-10
Goal:        Run the checksum restore drill and show an etcd snapshot file on the backup path.
Why:         P0b has to prove a deleted volume comes back with the same checksum, and that an etcd snapshot exists for same-node rollback (R31).
Advances:    P0b exit check: “Restore drill: a deleted volume is restored by VolSync (or the restic CronJob fallback, per the Phase 0 test) and its checksum matches; an etcd snapshot exists for same-node rollback (R31).”
Depends on:  p0b-09, p0a-12
Touches:     platform/backups/ (drill volume)
Hard limits: R31. The etcd snapshot is not restored in this drill. The drill runs by hand with kubectl reads and its log is P0b evidence; no drill helper. The volume drill uses the mechanism p0b-09 kept.
Done checks: The log shows the original checksum, the delete, the restore, and the matching checksum, plus an etcd snapshot filename on the backup path. One PR.
Timebox:     6h
```

```text
ID:          p0b-11
Goal:        Add make verify PHASE=p0b as a short checklist and attach its run, including the [G] tailnet line.
Why:         Done for the checkpoint is the attached verify output (§2, R35).
Advances:    P0b exit check: “A test app answers over HTTPS on <svc>.<DNS suffix> (env-01: <svc>.alpha.lan) with a cert from the internal CA, from LAN and from tailnet [G], through Traefik Ingress.”
Advances:    P0b exit check: “Restore drill: a deleted volume is restored by VolSync (or the restic CronJob fallback, per the Phase 0 test) and its checksum matches; an etcd snapshot exists for same-node rollback (R31).”
Depends on:  p0b-08, p0b-10, p0b-07
Touches:     Makefile, tools/verify/p0b
Hard limits: R28; R35. D17. Router forwarding of alpha.lan and the trusted root cert are already Gordon's steps on p0b-08. This checklist, about 50 lines like p0a-18’s, re-checks LAN and collects the tailnet paste. Nothing for Tailscale is installed on the node. §2: make verify prints NEEDS GORDON for each [G] check and each Gordon-run step; the attached output includes his pasted result. The checklist reads live state and links evidence; it does not re-run CI checks (D17).
Done checks: make verify PHASE=p0b prints NEEDS GORDON for the tailnet [G] check, and exits 0 with the LAN curl, the cert issuer, the restore checksum, and Gordon’s tailnet paste. One PR.
Timebox:     4h
```

### Exit checks

- A test app answers over HTTPS on <svc>.<DNS suffix> (env-01: <svc>.alpha.lan) with a cert from the internal CA, from LAN and from tailnet [G], through Traefik Ingress.
- Restore drill: a deleted volume is restored by VolSync (or the restic CronJob fallback, per the Phase 0 test) and its checksum matches; an etcd snapshot exists for same-node rollback (R31).

Done = attached make verify PHASE=p0b output

## P1

Goal: Gordon sees the system and gets alerts without Hermes.

### Contracts

```text
ID:          p1-01
Goal:        Write the Login and Alert path sections of the observability CONTRACT.md with proven by.
Why:         Grafana’s built-in login and the send-only alert path are one contract, written before either is built (R8, R27, §5).
Advances:    P1 exit check: “Grafana shows node, pod and disk metrics, and logs for every namespace.”
Advances:    P1 exit check: “Killing a test endpoint sends a Telegram alert within 5 minutes; the disk, node-memory and clock-not-synchronised alert rules fire in a test with thresholds lowered.”
Advances:    P1 exit check: “A forced backup failure (VolSync or the CronJob fallback) sends a Telegram alert.”
Depends on:  p0b-11
Touches:     services/observability/CONTRACT.md
Hard limits: R8; R18 (logins work offline: no external IdP); R27. D7; D16. The Hermes dashboard half is added in p2-05. Multi-user OIDC stays the §9 seam. The alert list is disk full, node memory above 100% less the env headroom (70% on env-01), node not ready, clock not synchronised, backup failures, and bot cron failures. Bot cron alerts are delivered in P3 (p3-07); the contract names them now. Egress to api.telegram.org is required. The api.x.ai probe is added in P2 (R18).
Done checks: The Login section and the Alert path section each have offers, expects, health, config, and proven by. One PR.
Timebox:     6h
Proven by:   The R27 CI check and the P2 dashboard-login check (§5). Grafana anonymous-off is the P1 half. The P1 kill-endpoint alert, the lowered-threshold alerts, and the backup-failure alert.
```

### Build

```text
ID:          p1-03
Goal:        Deploy VictoriaMetrics with 30-day retention and the disk floor.
Why:         Grafana’s node, pod, and disk panels need a metrics store that caps itself (§4, §7, R8).
Advances:    P1 exit check: “Grafana shows node, pod and disk metrics, and logs for every namespace.”
Depends on:  p1-01
Touches:     services/observability/victoria-metrics/
Hard limits: R5; R8; R31 (backup: none, the observability exception). D30. Retention 30 days and -storage.minFreeDiskSpaceBytes (§7). Remove the services placeholder ConfigMap when this path becomes real (D9).
Done checks: The pod is Ready, /metrics answers, and the retention flags are in the manifest. One PR.
Timebox:     4h
```

```text
ID:          p1-04
Goal:        Deploy VictoriaLogs with 14-day retention and the disk cap.
Why:         Grafana has to show logs for every namespace, with a disk cap so logs stay inside §7.
Advances:    P1 exit check: “Grafana shows node, pod and disk metrics, and logs for every namespace.”
Depends on:  p1-01
Touches:     services/observability/victoria-logs/
Hard limits: R5; R8; R31 (backup: none). D30. 14-day retention and -retention.maxDiskSpaceUsageBytes around 20 GB (§7).
Done checks: The pod is Ready and a test line written to stdout is queryable. One PR.
Timebox:     4h
```

```text
ID:          p1-05
Goal:        Deploy vmagent, node-exporter, and kube-state-metrics.
Why:         Node, pod, and disk metrics have to be scraped into VictoriaMetrics (§4).
Advances:    P1 exit check: “Grafana shows node, pod and disk metrics, and logs for every namespace.”
Depends on:  p1-03
Touches:     services/observability/vmagent/
Hard limits: R5; R8. D30. Scrape node-exporter, kube-state-metrics, and pod /metrics where a component offers it.
Done checks: vmagent targets for the node and kube-state-metrics are up. One PR.
Timebox:     6h
```

```text
ID:          p1-06
Goal:        Deploy Fluent Bit as a DaemonSet shipping container and node logs to VictoriaLogs.
Why:         Logs for every namespace have to arrive without a second log stack (§4, OQ9 as recorded in §4).
Advances:    P1 exit check: “Grafana shows node, pod and disk metrics, and logs for every namespace.”
Depends on:  p1-04
Touches:     services/observability/fluent-bit/
Hard limits: R5; R8. D30. No fallback shipper.
Done checks: A log line from kube-system and from platform-testapp is queryable in VictoriaLogs. One PR.
Timebox:     4h
```

```text
ID:          p1-07
Goal:        Deploy Grafana with anonymous access off, Git-managed datasources, and the home dashboard.
Why:         Gordon has to see node, pod, and disk metrics and logs for every namespace (R8, R27).
Advances:    P1 exit check: “Grafana shows node, pod and disk metrics, and logs for every namespace.”
Depends on:  p1-05, p1-06
Touches:     services/observability/grafana/dashboards, services/observability/grafana/
Hard limits: R5; R18; R27. D16; D30. The home dashboard links to Gatus (§4). The admin password is the bootstrap Secret. Login does not call an external IdP.
Done checks: Opening grafana.alpha.lan asks for login, and the home dashboard shows node, pod, disk, and a logs panel that can filter by namespace. One PR.
Timebox:     6h
```

```text
ID:          p1-08
Goal:        Deploy Gatus with a check for every exposed endpoint, the public CA root, and the login-free status page.
Why:         The status page is the NOC surface that loads without a login on the LAN and the tailnet (R5, R27).
Advances:    P1 exit check: “Gatus checks every exposed endpoint; its status page (env-01: gatus.alpha.lan) loads without login on LAN and on tailnet [G].”
Depends on:  p1-07, p0b-08
Touches:     services/observability/gatus/
Hard limits: R5; R8; R27; R28. D12; D13; D30. Gatus resolves alpha.lan through the coredns-custom forward and trusts the committed public root (§5). Checks include testapp, grafana, and gatus, plus TCP/53 for JARVIS DNS.
Done checks: https://gatus.alpha.lan loads with no login from the LAN and lists those checks. The tailnet load is NEEDS GORDON (D17). One PR.
Timebox:     6h
```

```text
ID:          p1-09
Goal:        Point Gatus and Grafana at the send-only Telegram alerts bot.
Why:         Alert delivery has to work while Hermes is absent (R8).
Advances:    P1 exit check: “Killing a test endpoint sends a Telegram alert within 5 minutes; the disk, node-memory and clock-not-synchronised alert rules fire in a test with thresholds lowered.”
Depends on:  p1-01, p1-08
Touches:     services/observability/ network policy, Gatus provider, Grafana contact point, SOPS secret
Hard limits: R8. D39. The alert-path contract (§5) lists api.telegram.org as documentation, from observability only. Egress is DNS plus TCP 443 to 0.0.0.0/0 except RFC1918, the pod and service CIDRs, and the node IP. Hostname allow-listing waits for the §9 proxy. The bot token and chat ID are the SOPS values Gordon records after /start. The alerts bot has no gateway and no allow-list (R21).
Done checks: A Grafana test notification returns success from the Telegram API. The token value is not printed. One PR.
Timebox:     4h
```

```text
ID:          p1-10
Goal:        Add the disk, node-memory, node-not-ready, clock, and backup-failure alert rules, and fire them with the thresholds lowered.
Why:         The P1 check requires those rules to fire in a test, at the §7 thresholds in normal operation (R8).
Advances:    P1 exit check: “Killing a test endpoint sends a Telegram alert within 5 minutes; the disk, node-memory and clock-not-synchronised alert rules fire in a test with thresholds lowered.”
Advances:    P1 exit check: “A forced backup failure (VolSync or the CronJob fallback) sends a Telegram alert.”
Depends on:  p1-09
Touches:     services/observability/grafana/alerts/
Hard limits: R8; §7 (disk at 80% full, node memory at 100% less the env headroom: 70% on env-01). D14. The lowered-threshold test is reverted at the end of the run. Backup failure covers the mechanism p0b-09 kept.
Done checks: The verify log shows each rule firing under the lowered threshold and the committed rules back at 80% and 70%. One PR.
Timebox:     6h
```

```text
ID:          p1-11
Goal:        Kill the test endpoint and force a backup failure; both send a Telegram alert.
Why:         These are the two live alert proofs that do not depend on Hermes (R8).
Advances:    P1 exit check: “Killing a test endpoint sends a Telegram alert within 5 minutes; the disk, node-memory and clock-not-synchronised alert rules fire in a test with thresholds lowered.”
Advances:    P1 exit check: “A forced backup failure (VolSync or the CronJob fallback) sends a Telegram alert.”
Depends on:  p1-10
Touches:     tools/verify/p1, platform/testapp scale
Hard limits: R8. D12; D13. Restore the test app and the backup path when the check finishes. The verify script reads send success and does not print the bot token.
Done checks: The log shows the testapp alert arriving within 5 minutes and the backup-failure alert arriving. testapp is Ready again afterward. One PR.
Timebox:     6h
```

```text
ID:          p1-12
Goal:        Add the R27 check that Grafana anonymous access is off.
Why:         Fix order (§11): Grafana is already configured (p1-07); CI keeps a later edit from turning anonymous access on (R27).
Advances:    P1 exit check: “Grafana shows node, pod and disk metrics, and logs for every namespace.”
Depends on:  p1-07
Touches:     tools/checks/ui-auth
Hard limits: R27. The Hermes dashboard half of this check is added in p2-23, when that dashboard exists.
Done checks: A seeded Grafana manifest with anonymous access on fails ci. The live manifest passes. One PR.
Timebox:     4h
```

```text
ID:          p1-13
Goal:        Wire make verify PHASE=p1 and attach a full run.
Why:         Done for the checkpoint is the attached verify output (§2, R35).
Advances:    P1 exit check: “Grafana shows node, pod and disk metrics, and logs for every namespace.”
Advances:    P1 exit check: “Gatus checks every exposed endpoint; its status page (env-01: gatus.alpha.lan) loads without login on LAN and on tailnet [G].”
Advances:    P1 exit check: “Killing a test endpoint sends a Telegram alert within 5 minutes; the disk, node-memory and clock-not-synchronised alert rules fire in a test with thresholds lowered.”
Advances:    P1 exit check: “A forced backup failure (VolSync or the CronJob fallback) sends a Telegram alert.”
Depends on:  p1-11, p1-12, p1-08
Touches:     Makefile, tools/verify/p1
Hard limits: R8; R35. D17. The tailnet Gatus load is the [G] line. §2: make verify prints NEEDS GORDON for each [G] check and each Gordon-run step; the attached output includes his pasted result. The checklist reads live state and links evidence; it does not re-run CI checks (D17).
Done checks: make verify PHASE=p1 prints NEEDS GORDON for the tailnet [G] check, and exits 0 with dashboard evidence, the Gatus page, both alert proofs, and Gordon’s tailnet paste. One PR.
Timebox:     4h
```

### Exit checks

- Grafana shows node, pod and disk metrics, and logs for every namespace.
- Gatus checks every exposed endpoint; its status page (env-01: gatus.alpha.lan) loads without login on LAN and on tailnet [G].
- Killing a test endpoint sends a Telegram alert within 5 minutes; the disk, node-memory and clock-not-synchronised alert rules fire in a test with thresholds lowered.
- A forced backup failure (VolSync or the CronJob fallback) sends a Telegram alert.

Done = attached make verify PHASE=p1 output

## P2

Goal: Gordon chats with the ops bots through the dashboard and Telegram, with memory and evals.

### Contracts

```text
ID:          p2-01
Goal:        Write the Model provider, Assistant API, and Skill contract sections of the Hermes core CONTRACT.md with proven by.
Why:         Tiers, the assistant API, and the skill rules are one core contract, written before any bot is deployed (R15, R17, R18, R23, §5).
Advances:    P2 exit check: “With the primary model blocked, chat answers from the fallback model; with the provider blocked, the dashboard and Telegram show an error within 30 s and Gatus flags it (R18).”
Advances:    P2 exit check: “A scripted prompt through the assistant API (/api/<profile>/v1) to each of the three bots gets a non-error answer; a Telegram message from Gordon’s ID to concierge gets a reply [G], and one from another ID gets none.”
Advances:    P2 exit check: “Memory: a fact is recalled after a pod restart in a new session; “forget that” is staged, and approving it removes the fact; a memory export produces a file. Memory and skill writes (R12) wait for approval; a test approves one and rejects one.”
Advances:    P2 exit check: “Weather and distance questions get correct answers through the geo MCP server (weather: Open-Meteo; distance: Nominatim plus the OSRM demo, typical drive time) with attribution; the geo tool’s evals in concierge pass.”
Advances:    P3 exit check: “An eval asserts no bot can send, delete, apply Gmail labels, write to calendar or reach Drive.”
Depends on:  p1-13
Touches:     services/hermes/core/CONTRACT.md
Hard limits: R13; R15; R16; R17; R18; R23; R27. D7. One xai-oauth login per deploy. A local LLM or a gateway is another base_url and is not built (§9). No xAI API key. Paths are / for the dashboard and /api/<profile>/v1 per profile. /v1/audio/* is reserved and this task adds no audio handler (§5, §9). Memory export produces a file. MCP servers run inside the deploy pod, pinned, no sidecars. Write tools run only behind a confirm step. This plan’s writes are inbox drafts and spec-writer’s issues. The geo section records the Nominatim policy decision: at most 1 request per second, cache, User-Agent with Gordon’s contact email, attribution, home taken from SOPS and never looked up.
Done checks: The three sections each have offers, expects, health, config, and proven by, including the Nominatim decision. One PR.
Timebox:     6h
Proven by:   The P2 fallback check, Gatus provider reachability, and the R23 CI grep (§5). The P2 scripted-prompt check and the memory-export check (§5). The R16 pin and tag check, the R13 toolset policy check, and the P3 no-write eval (§5).
```

```text
ID:          p2-04
Goal:        Add the Profile schema section to the schema CONTRACT.md with proven by.
Why:         A bot is a profile.yaml, validated before the three profiles are added (R19, §5).
Advances:    P2 exit check: “A scripted prompt through the assistant API (/api/<profile>/v1) to each of the three bots gets a non-error answer; a Telegram message from Gordon’s ID to concierge gets a reply [G], and one from another ID gets none.”
Depends on:  p2-01, p0a-01
Touches:     services/hermes/core/schema/CONTRACT.md
Hard limits: R19; R21; R23. Fields: job, toolsets with reasons and read/write tags, cron, evals, telegram true/false with allow-list required when true. Names that collide with status, cron, memory, or skills are rejected. Tiers come from the env bot tier map, not from the profile.
Done checks: The profile section has offers, expects, health, config, and proven by. One PR.
Timebox:     4h
Proven by:   CI validation of profile.yaml (§5) and the three P2 bots loading.
```

```text
ID:          p2-05
Goal:        Add the Hermes dashboard half of the Login contract, with proven by.
Why:         The ops dashboard requires its own login, and that login works offline (R18, R27).
Advances:    P2 exit check: “The Hermes ops dashboard (hermes-ops.<DNS suffix>) shows up in Gatus (R5) and requires its login (R27).”
Depends on:  p2-01, p1-01
Touches:     services/observability/CONTRACT.md, services/hermes/core/CONTRACT.md
Hard limits: R18; R27. D7; D16. Username admin, password hash and signing secret from bootstrap-secrets. The assistant API uses the deploy API key, separate from the dashboard login.
Done checks: The Login contract names both UIs and the Hermes proven-by line. One PR.
Timebox:     4h
Proven by:   The R27 CI check and the P2 dashboard-login check (§5).
```

### Build

```text
ID:          p2-06a
Goal:        Add the Hermes core base: digest pin, probes, memory limit, and seccomp.
Why:         Both deploys are overlays of one core image (R11).
Advances:    P2 exit check: “After a pod restart, Git config is re-applied; restoring the Hermes volume from backup works.”
Depends on:  p2-01
Touches:     services/hermes/core/base, services/hermes/core/templates/profile
Hard limits: R5; R11. D30. No custom image. The upstream image is pinned by digest. /v1/audio/* is not implemented.
Done checks: kustomize build of the base sets the probes, the memory limit, seccomp, write_approval, and a digest. One PR.
Timebox:     4h
```

```text
ID:          p2-06b
Goal:        Add Hermes startup sync, write_approval, and the toolset policy.
Why:         Git-sourced config wins on every start, and risky tools stay off unless a profile gives a reason (R12, R13).
Advances:    P2 exit check: “After a pod restart, Git config is re-applied; restoring the Hermes volume from backup works.”
Depends on:  p2-06a
Touches:     services/hermes/core/config, services/hermes/core/glue
Hard limits: R12; R13. skills.write_approval and memory.write_approval are on. Terminal, file, code-execution, browser, and delegation stay off unless a profile gives a written reason. Startup sync overwrites runtime edits.
Done checks: A unit test shows a runtime edit is replaced by the Git copy. One PR.
Timebox:     4h
```

```text
ID:          p2-07
Goal:        Add the hermes.yaml render script and the CI diff check, and seed a stale render.
Why:         One checked-in script renders profiles, and a hand-edited rendered/ must fail (R4).
Advances:    P2 exit check: “Branch protection requires admin-gate (R36): seeded PRs that change a protected path or remove a Namespace fail it without the label and when admin-approved comes from gordon5417-ai or a fixture Bot/App actor, and pass once Gordon’s own account adds it after the latest commit; a seeded PR fails the render check (a stale Hermes render, R4).”
Advances:    P0a exit check: “One required check on main, ci, runs every tools/checks/ rule check (admin-gate joins at the P2 exit, R36), and a seeded bad PR fails each rule check as it lands (Examples: a hardcoded DNS suffix, a plaintext secret, a missing CONTRACT.md, an unknown code_delivery or build_agent value, a deny list missing a whole deny class; a check added later is seeded in the task that adds it, such as the PVC backup-entry check at P0b and the stale render at P2).”
Depends on:  p2-06b, p0a-05
Touches:     tools/render-profiles, envs/env-01/rendered/, tools/checks/render-diff
Hard limits: R4. rendered/ is generated and never hand-edited. The stale-render seed lands with this render.
Done checks: A fresh render matches rendered/. A seeded PR that edits rendered/ without rerendering fails ci. One PR.
Timebox:     6h
```

```text
ID:          p2-08
Goal:        Validate profile.yaml in CI against the profile schema.
Why:         Fix order (§11): the schema is the boundary (p2-04); CI rejects a profile that breaks it (R19).
Advances:    P2 exit check: “A scripted prompt through the assistant API (/api/<profile>/v1) to each of the three bots gets a non-error answer; a Telegram message from Gordon’s ID to concierge gets a reply [G], and one from another ID gets none.”
Depends on:  p2-04, p0a-06
Touches:     services/hermes/core/schema/, tools/checks/profile-schema
Hard limits: R19; R21. A telegram: true profile without an allow-list fails. A bot name of memory, skills, status, or cron fails.
Done checks: pytest covers one valid profile and one colliding name. A seeded bad profile fails ci. One PR.
Timebox:     4h
```

```text
ID:          p2-09
Goal:        Deploy the ops overlay: its own volume, secrets, read-only ServiceAccount, network policy, Ingress, and dashboard login.
Why:         Gordon reaches the three bots through the ops dashboard, which Gatus watches and which requires a login (R11, R13, R27).
Advances:    P2 exit check: “A scripted prompt through the assistant API (/api/<profile>/v1) to each of the three bots gets a non-error answer; a Telegram message from Gordon’s ID to concierge gets a reply [G], and one from another ID gets none.”
Advances:    P2 exit check: “The Hermes ops dashboard (hermes-ops.<DNS suffix>) shows up in Gatus (R5) and requires its login (R27).”
Depends on:  p2-06b, p2-05, p1-08
Touches:     services/hermes/deploy/ops/, envs/env-01 secrets references
Hard limits: R5; R11; R13; R27; R28. D30; D39. Namespace hermes-ops. No shared volume or secret with the other deploy. ServiceAccount: get, list, watch, no Secrets. Egress is DNS plus TCP 443 to 0.0.0.0/0 except RFC1918, the pod and service CIDRs, and the node IP. The core CONTRACT.md lists hosts as documentation. Hostname allow-listing waits for the §9 proxy. The pod sets TZ from cluster-vars. The API key, password hash, and signing secret are the bootstrap Secrets, applied now that the namespace exists (D10). Hostname hermes-ops.alpha.lan.
Done checks: The pod is Ready, https://hermes-ops.alpha.lan asks for login, and Gatus lists the endpoint. One PR.
Timebox:     8h
```

```text
ID:          p2-10
Goal:        Add concierge, inbox, and homelab from the profile template, with generic SOUL.md files.
Why:         The starter bots are code, and only concierge faces Telegram (R19, R21).
Advances:    P2 exit check: “A scripted prompt through the assistant API (/api/<profile>/v1) to each of the three bots gets a non-error answer; a Telegram message from Gordon’s ID to concierge gets a reply [G], and one from another ID gets none.”
Depends on:  p2-07, p2-08, p2-09
Touches:     services/hermes/deploy/ops/profiles/{concierge,inbox,homelab}/
Hard limits: R13; R15; R19; R21. No placeholder bots. SOUL.md has no personal facts. inbox has the fewest tools at this checkpoint (no Gmail until P3). homelab proposes fixes only as chat. Brief crons are not listed yet, so they are absent rather than running. Built-in reminders stay available for the concierge reminder seed.
Done checks: Three profile.yaml files validate. hermes.yaml lists the three bots. A rendered diff is clean. One PR.
Timebox:     6h
```

```text
ID:          p2-11
Goal:        Add the in-repo geo MCP server and concierge’s geo evals.
Why:         Weather and distance have to be correct, attributed, and inside the providers’ usage rules (§5, R16).
Advances:    P2 exit check: “Weather and distance questions get correct answers through the geo MCP server (weather: Open-Meteo; distance: Nominatim plus the OSRM demo, typical drive time) with attribution; the geo tool’s evals in concierge pass.”
Depends on:  p2-01, p2-10
Touches:     services/hermes/core/skills/geo/
Hard limits: R15; R16. D36. At most 1 request per second to Nominatim and OSRM, cached, User-Agent set from the SOPS contact email, home from the rounded SOPS value and never geocoded. Attribution: Open-Meteo, and OSRM © OpenStreetMap contributors. Gordon commits the home and email SOPS values before the evals run.
Done checks: Unit tests show a cached weather answer and a cached distance answer with attribution and at most one Nominatim or OSRM request per second. The deployed eval pass is p2-17. One PR.
Timebox:     8h
```

```text
ID:          p2-12
Goal:        Pin kubernetes-mcp-server read-only and pin mcp-victoriametrics for homelab metric reads.
Why:         homelab has to read the cluster, Flux, and metrics, and has to answer the memory question from the appendix. §4 names mcp-victoriametrics for those reads (R13, R16).
Advances:    P2 exit check: “A scripted prompt through the assistant API (/api/<profile>/v1) to each of the three bots gets a non-error answer; a Telegram message from Gordon’s ID to concierge gets a reply [G], and one from another ID gets none.”
Depends on:  p2-10, p1-05
Touches:     ops deploy volume, ops profile toolsets, the pinned kubernetes-mcp-server and mcp-victoriametrics binaries
Hard limits: R13; R15; R16. D29. The Kubernetes MCP is read-only and cannot read Secrets. mcp-victoriametrics is the official VictoriaMetrics MCP server, read-only, a pinned release binary on the deploy volume, and it queries VictoriaMetrics only. homelab’s written reason for these tools is in profile.yaml. No terminal tool. No in-repo metrics server.
Done checks: The deploy volume holds the pinned mcp-victoriametrics binary, and a fixture query returns the workload with the highest memory. The deployed memory prompt is p2-17. One PR.
Timebox:     6h
```

```text
ID:          p2-13
Goal:        Set the env-01 tier map, the fallback model, and the ops xai-oauth device login.
Why:         Chat has to survive a blocked primary model on the same subscription login (R18, R23).
Advances:    P2 exit check: “With the primary model blocked, chat answers from the fallback model; with the provider blocked, the dashboard and Telegram show an error within 30 s and Gatus flags it (R18).”
Depends on:  p2-09, p2-07
Touches:     envs/env-01/values/hermes.yaml, ops volume
Hard limits: R18; R23. Mid: concierge, inbox reply drafting, homelab chat. Cheap: cron, inbox triage, homelab checks. Frontier: on demand only. The device login is Gordon’s approval of the code the pod prints. The token stays on the volume, never in Git or CI.
Done checks: hermes.yaml maps each tier to a primary and a fallback on xai-oauth. Gordon has approved the device login and a short chat answers. One PR.
Timebox:     4h
```

```text
ID:          p2-14
Goal:        Route /api/<profile>/v1 to each bot and run the scripted non-error prompt.
Why:         The assistant API is the only supported way to talk to a bot (R17).
Advances:    P2 exit check: “A scripted prompt through the assistant API (/api/<profile>/v1) to each of the three bots gets a non-error answer; a Telegram message from Gordon’s ID to concierge gets a reply [G], and one from another ID gets none.”
Depends on:  p2-10, p2-13
Touches:     services/hermes/deploy/ops/ routing, tools/verify/p2
Hard limits: R17. The API requires the deploy API key. The three profile names are concierge, inbox, and homelab.
Done checks: A scripted prompt to each of the three URLs returns a non-error answer. One PR.
Timebox:     4h
```

```text
ID:          p2-15
Goal:        Turn on concierge’s Telegram gateway with the allow-list, and prove another ID gets no reply.
Why:         Concierge is the only Telegram-facing bot, and the gateway accepts only Gordon’s ID (R21).
Advances:    P2 exit check: “A scripted prompt through the assistant API (/api/<profile>/v1) to each of the three bots gets a non-error answer; a Telegram message from Gordon’s ID to concierge gets a reply [G], and one from another ID gets none.”
Depends on:  p2-14
Touches:     concierge profile.yaml, SOPS telegram token
Hard limits: R21. The token and Gordon’s user ID are the SOPS values he records (the chat ID from P1). inbox and homelab stay telegram: false. The alerts bot is unchanged and has no gateway.
Done checks: A gateway-handler fixture for an ID outside the allow-list produces no reply (D17). Gordon’s own reply is NEEDS GORDON (D17). One PR.
Timebox:     6h
```

```text
ID:          p2-16
Goal:        Prove memory recall after restart, staged forget, export, and one approval plus one rejection.
Why:         Learned state lives on the volume, and writes wait for approval (R12, R17).
Advances:    P2 exit check: “Memory: a fact is recalled after a pod restart in a new session; “forget that” is staged, and approving it removes the fact; a memory export produces a file. Memory and skill writes (R12) wait for approval; a test approves one and rejects one.”
Depends on:  p2-09, p2-14
Touches:     ops memory volume, verify script
Hard limits: R12; R15; R17. The test approves one memory write and rejects one. Skill writes use the same approval switch; the test covers one skill-write rejection.
Done checks: The log shows the fact surviving a pod restart in a new session, the forget staying staged until approval, the fact gone after approval, and an export file on disk. One PR.
Timebox:     6h
```

```text
ID:          p2-17
Goal:        Add the pytest eval harness and at least three evals per ops bot, graded by the cheap tier.
Why:         Evals run against the deployed bots through the assistant API, on that deploy’s login (R38).
Advances:    P2 exit check: “Each bot’s evals (≥3) pass against the deployed bots (make verify, R38); usage is readable in Hermes analytics and the grok.com dashboard [G]; with the subscription login revoked or capped (a 401 test fixture stands in; the real login stays signed in), the dashboard and Telegram show an error within ~30 s and make verify fails (R18).”
Depends on:  p2-11, p2-12, p2-14
Touches:     services/hermes/core/testing/, profiles/*/evals/
Hard limits: R38. The grader uses the ops assistant API and the deploy API key. No separate grader key and no key in GitHub Actions. Cover the appendix P2 seeds: weather, distance, reminder, memory and forget, and the three homelab read prompts. Inbox’s three evals stay inside P2 scope (no Gmail yet): a non-error reply, a refusal to send mail, a refusal to label mail. About 10 evals on the ops deploy.
Done checks: make verify runs the evals and they pass. One PR.
Timebox:     8h
```

```text
ID:          p2-18
Goal:        Run the fallback, provider-down, and capped-login drills, and add the api.x.ai Gatus check.
Why:         A dead provider has to show an error within about 30 seconds instead of hanging (R18).
Advances:    P2 exit check: “Each bot’s evals (≥3) pass against the deployed bots (make verify, R38); usage is readable in Hermes analytics and the grok.com dashboard [G]; with the subscription login revoked or capped (a 401 test fixture stands in; the real login stays signed in), the dashboard and Telegram show an error within ~30 s and make verify fails (R18).”
Advances:    P2 exit check: “With the primary model blocked, chat answers from the fallback model; with the provider blocked, the dashboard and Telegram show an error within 30 s and Gatus flags it (R18).”
Depends on:  p2-13, p2-15, p2-17, p1-08
Touches:     Gatus provider check, verify script, cluster-vars provider host
Hard limits: R8; R18. D19; D20; D39. One provider-host value in cluster-vars is read by both Hermes and the Gatus provider probe, so one override trips both. The capped-login drill is a 401 fixture. The real login stays signed in. The drills restore xai-oauth before the task ends. The Telegram half is NEEDS GORDON: Gordon sends a message during the drill. Hosts in the contract are documentation.
Done checks: The log shows a fallback-model answer, a dashboard error within 30 seconds when the provider host is overridden, the same dashboard error within about 30 seconds for the 401 fixture, and make verify failing during that window and passing after restore. The Telegram half is Gordon’s paste from a message he sends during the drill. One PR.
Timebox:     8h
```

```text
ID:          p2-19
Goal:        Record usage from Hermes analytics and the grok.com confirmation line.
Why:         Subscription use has to be visible per bot and on the subscription dashboard (R23).
Advances:    P2 exit check: “Each bot’s evals (≥3) pass against the deployed bots (make verify, R38); usage is readable in Hermes analytics and the grok.com dashboard [G]; with the subscription login revoked or capped (a 401 test fixture stands in; the real login stays signed in), the dashboard and Telegram show an error within ~30 s and make verify fails (R18).”
Depends on:  p2-14, p2-17
Touches:     tools/verify/p2
Hard limits: R23. D18. No usage data is copied into Git.
Done checks: The verify log quotes a non-empty Hermes analytics usage field. The grok.com dashboard line is [G]: make verify prints NEEDS GORDON and the attached output includes Gordon’s paste. One PR.
Timebox:     4h
```

```text
ID:          p2-20
Goal:        Make a Hermes bump wait on a digest-keyed VolSync backup, keep auto-merge off, and run evals after deploy.
Why:         Hermes upgrades break, and the pre-upgrade backup has to finish before the new digest rolls (R33, R38).
Advances:    P2 exit check: “A test Hermes bump PR shows auto-merge is off, its deploy waited for the digest-keyed pre-upgrade backup, and the evals then ran against the deployed bot (R33).”
Depends on:  p2-17, p0b-09
Touches:     services/hermes/deploy/ops/ Flux dependency, tools/hermes-backup, renovate.json
Hard limits: R31; R33; R38. The ops Kustomization depends on a VolSync backup triggered by the new digest. Renovate does not auto-merge the Hermes digest. Gordon merges Hermes bump PRs. The code-factory deploy does not exist yet; this task covers the ops pin. The code-factory pin repeats the same pattern in p4b-05.
Done checks: A test bump PR shows auto-merge off, the backup Job complete for that digest before the rollout, and the evals passing against the deployed bot. One PR.
Timebox:     8h
```

```text
ID:          p2-21
Goal:        After a pod restart, prove Git config was re-applied, and restore the Hermes volume from backup.
Why:         Runtime edits must not survive a restart, and the Hermes volume needs its own restore drill (R12, R31).
Advances:    P2 exit check: “After a pod restart, Git config is re-applied; restoring the Hermes volume from backup works.”
Depends on:  p2-06b, p2-16, p0b-10
Touches:     ops volume, verify script
Hard limits: R12; R31. If the restored token is stale, Gordon redoes the device login (§8 custody) before chat is called again.
Done checks: The log shows a planted runtime edit gone after restart, and the restored volume’s checksum matching the backup. One PR.
Timebox:     6h
```

```text
ID:          p2-22
Goal:        Add CI guards for digest pins, unshared volumes and secrets, the toolset policy, and MCP pin and read/write tags.
Why:         Fix order (§11): the core defaults already set these boundaries; the checks stop a later PR from slipping (R11, R13, R16).
Advances:    P2 exit check: “A scripted prompt through the assistant API (/api/<profile>/v1) to each of the three bots gets a non-error answer; a Telegram message from Gordon’s ID to concierge gets a reply [G], and one from another ID gets none.”
Depends on:  p2-08, p2-09, p2-10
Touches:     tools/checks/
Hard limits: R11; R13; R16. Each new script gets one seeded failing PR.
Done checks: Three seeded PRs fail ci (shared secret, a terminal tool with no written reason, an unpinned MCP). The live ops manifests pass. One PR.
Timebox:     6h
```

```text
ID:          p2-23
Goal:        Add CI guards for the Telegram allow-list, model names outside envs/, and Hermes dashboard auth.
Why:         Fix order (§11): profiles and the dashboard already require these; CI keeps a later edit from dropping them (R21, R23, R27).
Advances:    P2 exit check: “The Hermes ops dashboard (hermes-ops.<DNS suffix>) shows up in Gatus (R5) and requires its login (R27).”
Depends on:  p2-10, p2-15, p1-12
Touches:     tools/checks/ui-auth, tools/checks/allow-list, tools/checks/core-names (model-name half)
Hard limits: R21; R23; R27. A telegram: true profile with an empty allow-list fails. A model name in core fails: core-names reads the names from the env tier and lane maps, which hold names from p2-13. Grafana anonymous-off stays in the same ui-auth script from p1-12.
Done checks: Three seeded PRs fail ci, and the live ops dashboard manifest has auth configured. One PR.
Timebox:     6h
```

```text
ID:          p2-24
Goal:        Add admin-gate, seed the protected-path and label-actor PRs, and have Gordon turn the check on.
Why:         From this exit, risky PRs need a label from Gordon’s own account (R36). Fix order (§11): the check is the automated gate; the rulebook does not replace it.
Advances:    P2 exit check: “Branch protection requires admin-gate (R36): seeded PRs that change a protected path or remove a Namespace fail it without the label and when admin-approved comes from gordon5417-ai or a fixture Bot/App actor, and pass once Gordon’s own account adds it after the latest commit; a seeded PR fails the render check (a stale Hermes render, R4).”
Depends on:  p2-07, p2-22, p2-23
Touches:     .github/workflows/admin-gate.yml, tools/checks/admin-gate, jarvis-apps/.github/workflows/ci.yml
Hard limits: R36. D38. pull_request_target on opened, synchronize, reopened, labeled, unlabeled. The job reads PR metadata only and does not check out PR code. It removes admin-approved on every new push. Protected paths are the R36 lists for jarvis and jarvis-apps. A Namespace removal fails closed. jarvis-apps ci runs on pull_request only and holds no secrets (D35). A separate job named admin-gate runs only on pull_request_target, never checks out code, calls admin-gate.yml at a pinned jarvis SHA, and is its own required check. p4a-05 adds the namespaced-only check to the pull_request ci job. Image publish stays in build.yml.
Done checks: Seeded PRs fail without the label, fail when gordon5417-ai adds admin-approved, fail when the fixture actor is a Bot or an App, and pass after Gordon’s own account adds the label on the latest commit. A seeded removal of a PVC or a StorageClass fails without the label (R36). Branch protection on both repos requires admin-gate as its own check, separate from ci. The jarvis-apps admin-gate job checks out no code. One PR.
Timebox:     8h
```

```text
ID:          p2-25
Goal:        Wire make verify PHASE=p2 and attach a full run.
Why:         Done for the checkpoint is the attached verify output (§2, R35). P4 starts only after this exit passes.
Advances:    P2 exit check: “A scripted prompt through the assistant API (/api/<profile>/v1) to each of the three bots gets a non-error answer; a Telegram message from Gordon’s ID to concierge gets a reply [G], and one from another ID gets none.”
Advances:    P2 exit check: “Weather and distance questions get correct answers through the geo MCP server (weather: Open-Meteo; distance: Nominatim plus the OSRM demo, typical drive time) with attribution; the geo tool’s evals in concierge pass.”
Advances:    P2 exit check: “Memory: a fact is recalled after a pod restart in a new session; “forget that” is staged, and approving it removes the fact; a memory export produces a file. Memory and skill writes (R12) wait for approval; a test approves one and rejects one.”
Advances:    P2 exit check: “Each bot’s evals (≥3) pass against the deployed bots (make verify, R38); usage is readable in Hermes analytics and the grok.com dashboard [G]; with the subscription login revoked or capped (a 401 test fixture stands in; the real login stays signed in), the dashboard and Telegram show an error within ~30 s and make verify fails (R18).”
Advances:    P2 exit check: “With the primary model blocked, chat answers from the fallback model; with the provider blocked, the dashboard and Telegram show an error within 30 s and Gatus flags it (R18).”
Advances:    P2 exit check: “A test Hermes bump PR shows auto-merge is off, its deploy waited for the digest-keyed pre-upgrade backup, and the evals then ran against the deployed bot (R33).”
Advances:    P2 exit check: “The Hermes ops dashboard (hermes-ops.<DNS suffix>) shows up in Gatus (R5) and requires its login (R27).”
Advances:    P2 exit check: “After a pod restart, Git config is re-applied; restoring the Hermes volume from backup works.”
Advances:    P2 exit check: “Branch protection requires admin-gate (R36): seeded PRs that change a protected path or remove a Namespace fail it without the label and when admin-approved comes from gordon5417-ai or a fixture Bot/App actor, and pass once Gordon’s own account adds it after the latest commit; a seeded PR fails the render check (a stale Hermes render, R4).”
Depends on:  p2-11, p2-15, p2-16, p2-17, p2-18, p2-19, p2-20, p2-21, p2-24
Touches:     Makefile, tools/verify/p2
Hard limits: R35; R36; R38. D17. This PR touches tools/verify/, so it waits for admin-approved. The [G] lines are concierge’s Telegram reply and the grok.com dashboard. §2: make verify prints NEEDS GORDON for each [G] check and each Gordon-run step; the attached output includes his pasted result. The checklist reads live state and links evidence; it does not re-run CI checks (D17).
Done checks: make verify PHASE=p2 prints NEEDS GORDON for the Telegram [G] check and the grok.com dashboard [G] check, and exits 0 with the scripted prompts, geo evals, memory log, eval summary, degrade log, bump log, dashboard login, restore log, admin-gate seeds, and Gordon’s Telegram paste. One PR.
Timebox:     6h
```

### Exit checks

- A scripted prompt through the assistant API (/api/<profile>/v1) to each of the three bots gets a non-error answer; a Telegram message from Gordon’s ID to concierge gets a reply [G], and one from another ID gets none.
- Weather and distance questions get correct answers through the geo MCP server (weather: Open-Meteo; distance: Nominatim plus the OSRM demo, typical drive time) with attribution; the geo tool’s evals in concierge pass.
- Memory: a fact is recalled after a pod restart in a new session; “forget that” is staged, and approving it removes the fact; a memory export produces a file. Memory and skill writes (R12) wait for approval; a test approves one and rejects one.
- Each bot’s evals (≥3) pass against the deployed bots (make verify, R38); usage is readable in Hermes analytics and the grok.com dashboard [G]; with the subscription login revoked or capped (a 401 test fixture stands in; the real login stays signed in), the dashboard and Telegram show an error within ~30 s and make verify fails (R18).
- With the primary model blocked, chat answers from the fallback model; with the provider blocked, the dashboard and Telegram show an error within 30 s and Gatus flags it (R18).
- A test Hermes bump PR shows auto-merge is off, its deploy waited for the digest-keyed pre-upgrade backup, and the evals then ran against the deployed bot (R33).
- The Hermes ops dashboard (hermes-ops.<DNS suffix>) shows up in Gatus (R5) and requires its login (R27).
- After a pod restart, Git config is re-applied; restoring the Hermes volume from backup works.
- Branch protection requires admin-gate (R36): seeded PRs that change a protected path or remove a Namespace fail it without the label and when admin-approved comes from gordon5417-ai or a fixture Bot/App actor, and pass once Gordon’s own account adds it after the latest commit; a seeded PR fails the render check (a stale Hermes render, R4).

Done = attached make verify PHASE=p2 output

## P3

Goal: scheduled briefs and read skills deliver the daily butler, with no writes beyond inbox drafts.

### Contracts

No new §5 contract. The skill contract remains the one written in P2.

### Build

```text
ID:          p3-00
Goal:        Add make google-consent, the OAuth loopback flow on the controller.
Why:         The Gmail and Calendar token has to be sealed before a bot reads mail (R15, R25).
Advances:    P3 exit check: “The morning brief arrives at its scheduled time with calendar, email and weather (Open-Meteo, attributed; “weather unavailable” if it is down), and reaches Telegram [G].”
Depends on:  p0a-04, p2-25
Touches:     tools/google-consent, Makefile
Hard limits: R15; R25. D40. Loopback flow on the controller. Scopes are gmail.readonly, gmail.compose, and calendar.readonly only. The token is written through make seal into SOPS, then into WORKSPACE_MCP_CREDENTIALS_DIR/<email>.json with USER_GOOGLE_EMAIL set. That credentials file stays writable so the server can refresh it.
Done checks: A fixture loopback run seals a token whose scopes are only those three. One PR.
Timebox:     4h
```

```text
ID:          p3-01a
Goal:        Add concierge’s Gmail and Calendar read through workspace-mcp.
Why:         The daily butler reads mail and calendar (R15, §5 skill contract).
Advances:    P3 exit check: “The morning brief arrives at its scheduled time with calendar, email and weather (Open-Meteo, attributed; “weather unavailable” if it is down), and reaches Telegram [G].”
Depends on:  p3-00, p2-25
Touches:     ops profiles, deploy volume uv cache
Hard limits: R13; R15; R16. D22; D39; D40. workspace-mcp==2.1.1 via uvx, uv cache on the deploy volume, single-user, stdio. Concierge runs `--single-user --read-only --tools gmail calendar`. No extra tool allowlist. Egress is DNS plus TCP 443 to 0.0.0.0/0 except RFC1918, the pod and service CIDRs, and the node IP. The ops CONTRACT.md lists the Google hosts as documentation. Hostname allow-listing waits for the §9 proxy.
Done checks: It runs on a token with only gmail.readonly, gmail.compose, and calendar.readonly, and registers no send tool. A read-only call returns one inbox thread and one calendar event after Gordon’s consent step. One PR.
Timebox:     6h
```

```text
ID:          p3-01b
Goal:        Add inbox’s draft tool, and no send.
Why:         Inbox drafts a reply without sending (R15).
Advances:    P3 exit check: “Asked in chat to draft a reply to a seeded email, inbox creates one Gmail draft in that thread and nothing is sent (the Sent folder is unchanged).”
Depends on:  p3-01a
Touches:     inbox profile toolset
Hard limits: R13; R15; R16. D40. Inbox runs workspace-mcp with `--single-user --permissions gmail:drafts calendar:readonly`. That drafts level never registers send_gmail_message. No extra tool allowlist.
Done checks: The draft tool is registered on inbox only, and send_gmail_message is not registered. One PR.
Timebox:     4h
```

```text
ID:          p3-02
Goal:        Add concierge’s public GitHub PR read through github-mcp-server.
Why:         “What PRs need my attention?” is a concierge read. R13 allows this one public-read token in ops.
Advances:    P3 exit check: “Each read skill is covered by an eval in its bot’s evals/.”
Depends on:  p2-25
Touches:     ops deploy volume, concierge profile, SOPS token
Hard limits: R13; R15; R16. D41. github-mcp-server v2.0.2 is a pinned release binary on the deploy volume. Concierge runs `stdio --read-only --toolsets pull_requests,actions`. The token is fine-grained, from Gordon’s account, with no repos selected, in SOPS for hermes-ops. No extra tool allowlist.
Done checks: An eval lists an open public PR. The ops manifest contains no GitHub secret other than the public-read-only token. One PR.
Timebox:     4h
```

```text
ID:          p3-03
Goal:        Enable the morning brief, evening wrap, and weekly review crons on concierge.
Why:         Listed crons start enabled; the PR is the approval (R19).
Advances:    P3 exit check: “The morning brief arrives at its scheduled time with calendar, email and weather (Open-Meteo, attributed; “weather unavailable” if it is down), and reaches Telegram [G].”
Advances:    P3 exit check: “The evening wrap arrives at its scheduled time with the day’s unread inbox threads, ranked by importance in chat only, and tomorrow’s calendar, and reaches Telegram [G].”
Advances:    P3 exit check: “The weekly review arrives at its scheduled time with the past week’s calendar and email highlights and the coming week’s calendar, and reaches Telegram [G].”
Depends on:  p3-01a
Touches:     envs/env-01/values/hermes.yaml, concierge profile
Hard limits: R15; R19. D21; D22. Cron times are America/Los_Angeles, the env schema time zone, the same value the node uses. The brief includes calendar, email, and Open-Meteo weather with attribution, or the words “weather unavailable” when Open-Meteo is down. The wrap includes the day’s unread inbox threads, ranked by importance in chat only, and tomorrow’s calendar. The weekly review includes the past week’s highlights and the coming week’s calendar. Telegram delivery is through concierge.
Done checks: A manual run of each cron produces the three bodies in the dashboard session. The cron expressions in hermes.yaml match D21, and the next run time is in America/Los_Angeles. Telegram arrival is NEEDS GORDON (D17). One PR.
Timebox:     8h
```

```text
ID:          p3-04
Goal:        Add homelab’s stuck-updates cron and the morning-brief line that points at its finding.
Why:         Stuck Renovate work has to surface in homelab’s session and in the next brief (R13, R15).
Advances:    P3 exit check: “homelab’s scheduled “watch for stuck updates” check reads Renovate PRs and their check runs and annotations from the public repos (the no-repos public-read-only token, R13; CI reports each failure as an ::error annotation): a seeded Renovate PR whose required check failed on its last 2 runs, or that has been open more than 7 days, produces one finding with the error and a suggested fix in homelab’s dashboard session, and the next morning brief carries a “stuck updates” line pointing to it.”
Depends on:  p3-02, p3-03
Touches:     homelab profile, ci annotation line, concierge brief
Hard limits: R13; R15. D21; D23; D41. homelab uses the same github-mcp-server command as concierge. It recognizes Renovate PRs by author renovate[bot] or a renovate/ branch prefix. The seed is the real D11 Flux Operator PR or a renovate/ branch. The seeded failure prints only the ::error. homelab writes the suggested fix (R15). homelab does not file an issue and does not apply a fix.
Done checks: The seeded PR produces one dashboard finding with homelab’s suggested fix, and the next brief contains a stuck-updates line that points at it. One PR.
Timebox:     8h
```

```text
ID:          p3-05
Goal:        Add one eval per read skill and the no-write eval.
Why:         Each read skill needs an eval, and no bot may send, delete, label, write calendar, or reach Drive (R15, R38).
Advances:    P3 exit check: “Each read skill is covered by an eval in its bot’s evals/.”
Advances:    P3 exit check: “An eval asserts no bot can send, delete, apply Gmail labels, write to calendar or reach Drive.”
Depends on:  p3-01b, p3-02, p2-17
Touches:     profiles/*/evals/
Hard limits: R15; R38. D40. P3 evals are required by this exit and sit on top of the P2 set. Appendix seeds included: inbox ranking, the draft-shaped send request that must not send, the Drive refusal, the calendar-write refusal, and the PR question. No extra tool allowlist.
Done checks: pytest for these evals passes against the deployed bots. The no-write eval fails the run if send_gmail_message or any other forbidden tool is registered. One PR.
Timebox:     6h
```

```text
ID:          p3-06
Goal:        Ask inbox to draft a reply to the seeded email, and show the Sent folder unchanged.
Why:         Drafts are the one Gmail write in this plan, and they must not send (R15).
Advances:    P3 exit check: “Asked in chat to draft a reply to a seeded email, inbox creates one Gmail draft in that thread and nothing is sent (the Sent folder is unchanged).”
Depends on:  p3-01b, p3-05
Touches:     inbox draft tool, verify script
Hard limits: R15. D24. The seed subject is jarvis-p3-seed. The draft is created in that thread. No send tool is called.
Done checks: The log shows one new draft in the seed thread and a Sent-folder id list that matches the pre-check list. One PR.
Timebox:     4h
```

```text
ID:          p3-07
Goal:        Add a Grafana log alert on Hermes cron errors that pages Telegram without Hermes.
Why:         A failed brief still has to reach Gordon when the assistant is the thing that failed (R8).
Advances:    P3 exit check: “A forced bot cron failure sends a Telegram alert without going through Hermes (a log-based Grafana alert on Hermes cron errors).”
Depends on:  p3-03, p1-10
Touches:     services/observability/grafana/alerts/
Hard limits: R8. The alert queries VictoriaLogs for Hermes cron errors and uses the existing alerts-bot contact point. It does not call the ops deploy.
Done checks: A planted cron error produces a Telegram send success in the Grafana log, and the ops pod is not in that path. One PR.
Timebox:     4h
```

```text
ID:          p3-08
Goal:        Wire make verify PHASE=p3 and attach a full run.
Why:         Done for the checkpoint is the attached verify output (§2, R35).
Advances:    P3 exit check: “The morning brief arrives at its scheduled time with calendar, email and weather (Open-Meteo, attributed; “weather unavailable” if it is down), and reaches Telegram [G].”
Advances:    P3 exit check: “The evening wrap arrives at its scheduled time with the day’s unread inbox threads, ranked by importance in chat only, and tomorrow’s calendar, and reaches Telegram [G].”
Advances:    P3 exit check: “The weekly review arrives at its scheduled time with the past week’s calendar and email highlights and the coming week’s calendar, and reaches Telegram [G].”
Advances:    P3 exit check: “homelab’s scheduled “watch for stuck updates” check reads Renovate PRs and their check runs and annotations from the public repos (the no-repos public-read-only token, R13; CI reports each failure as an ::error annotation): a seeded Renovate PR whose required check failed on its last 2 runs, or that has been open more than 7 days, produces one finding with the error and a suggested fix in homelab’s dashboard session, and the next morning brief carries a “stuck updates” line pointing to it.”
Advances:    P3 exit check: “Each read skill is covered by an eval in its bot’s evals/.”
Advances:    P3 exit check: “Asked in chat to draft a reply to a seeded email, inbox creates one Gmail draft in that thread and nothing is sent (the Sent folder is unchanged).”
Advances:    P3 exit check: “An eval asserts no bot can send, delete, apply Gmail labels, write to calendar or reach Drive.”
Advances:    P3 exit check: “A forced bot cron failure sends a Telegram alert without going through Hermes (a log-based Grafana alert on Hermes cron errors).”
Depends on:  p3-04, p3-05, p3-06, p3-07
Touches:     Makefile, tools/verify/p3
Hard limits: R35. D17. This PR touches tools/verify/ and waits for admin-approved (R36). The [G] lines are the morning brief, the evening wrap, and the weekly review on Telegram. §2: make verify prints NEEDS GORDON for each [G] check and each Gordon-run step; the attached output includes his pasted result. The checklist reads live state and links evidence; it does not re-run CI checks (D17).
Done checks: make verify PHASE=p3 prints NEEDS GORDON for each of the three Telegram [G] checks, and exits 0 with the three brief bodies, Gordon’s three Telegram pastes, the stuck-update finding, the eval list, the draft proof, and the cron alert. One PR.
Timebox:     4h
```

### Exit checks

- The morning brief arrives at its scheduled time with calendar, email and weather (Open-Meteo, attributed; “weather unavailable” if it is down), and reaches Telegram [G].
- The evening wrap arrives at its scheduled time with the day’s unread inbox threads, ranked by importance in chat only, and tomorrow’s calendar, and reaches Telegram [G].
- The weekly review arrives at its scheduled time with the past week’s calendar and email highlights and the coming week’s calendar, and reaches Telegram [G].
- homelab’s scheduled “watch for stuck updates” check reads Renovate PRs and their check runs and annotations from the public repos (the no-repos public-read-only token, R13; CI reports each failure as an ::error annotation): a seeded Renovate PR whose required check failed on its last 2 runs, or that has been open more than 7 days, produces one finding with the error and a suggested fix in homelab’s dashboard session, and the next morning brief carries a “stuck updates” line pointing to it.
- Each read skill is covered by an eval in its bot’s evals/.
- Asked in chat to draft a reply to a seeded email, inbox creates one Gmail draft in that thread and nothing is sent (the Sent folder is unchanged).
- An eval asserts no bot can send, delete, apply Gmail labels, write to calendar or reach Drive.
- A forced bot cron failure sends a Telegram alert without going through Hermes (a log-based Grafana alert on Hermes cron errors).

Done = attached make verify PHASE=p3 output

## P4a

Goal: one apps.yaml line deploys an outside app, isolated from JARVIS.

### Contracts

```text
ID:          p4a-01
Goal:        Write the App hosting CONTRACT.md with proven by.
Why:         One apps.yaml line is the whole enable path, and the contract comes before the ResourceSet (§5, R30).
Advances:    P4a exit check: “After Gordon adds one line to apps.yaml, the outside app deploys to its own namespace, is reachable at app-<name>.<DNS suffix> on LAN and on tailnet [G], shows up in Gatus (R5), and a network-policy test shows it cannot reach core services.”
Depends on:  p2-25
Touches:     platform/app-hosting/CONTRACT.md
Hard limits: R4; R30. D7. The contract covers namespace, quota, default-deny, service account, per-namespace GitRepository, impersonated Kustomization, and the Gatus ConfigMap. Hosted apps are stateless. An env can leave app hosting off.
Done checks: CONTRACT.md has offers, expects, health, config, and proven by. One PR.
Timebox:     4h
Proven by:   The P4a reachability, Gatus, and network-policy checks, and the seeded cluster-scoped PR (R30).
```

### Build

```text
ID:          p4a-02
Goal:        Add the app-hosting ResourceSet fed by apps.yaml, with app hosting still off on env-01.
Why:         Fix order (§11): one line in apps.yaml is the easy path; CI rejects the dangerous objects later (R30, R4).
Advances:    P4a exit check: “After Gordon adds one line to apps.yaml, the outside app deploys to its own namespace, is reachable at app-<name>.<DNS suffix> on LAN and on tailnet [G], shows up in Gatus (R5), and a network-policy test shows it cannot reach core services.”
Depends on:  p4a-01, p0a-05
Touches:     platform/app-hosting/, env schema apps.yaml rules
Hard limits: R1; R4; R30. D30 for the quota map: small is 512Mi and 0.5 CPU, medium is 2Gi and 1 CPU (§7). No cross-namespace source refs. No secret while the repos are public. No pull secret (public GHCR). The env schema accepts hosting on or off.
Done checks: With apps.yaml empty, no app namespace exists. A fixture render with one line produces the namespace, quota, default-deny policy, service account, GitRepository, and impersonated Kustomization. One PR.
Timebox:     8h
```

```text
ID:          p4a-03
Goal:        Have the ResourceSet write the app’s Gatus endpoint ConfigMap, and load it from a Gatus sidecar.
Why:         A hosted app has to show up in Gatus without a hand-edited Gatus file (R5, §5).
Advances:    P4a exit check: “After Gordon adds one line to apps.yaml, the outside app deploys to its own namespace, is reachable at app-<name>.<DNS suffix> on LAN and on tailnet [G], shows up in Gatus (R5), and a network-policy test shows it cannot reach core services.”
Depends on:  p4a-02, p1-08
Touches:     platform/app-hosting/, services/observability/gatus/
Hard limits: R5. The ConfigMap is labeled in the observability namespace. The sidecar is the same pattern Grafana uses for dashboards.
Done checks: A fixture app line produces a Gatus check for app-<name>.alpha.lan. One PR.
Timebox:     4h
```

```text
ID:          p4a-04
Goal:        Add apps/demo, its Dockerfile, and the jarvis-apps GHCR build.
Why:         P4a needs one real app whose image CI publishes as a public GHCR package (§4, R30).
Advances:    P4a exit check: “After Gordon adds one line to apps.yaml, the outside app deploys to its own namespace, is reachable at app-<name>.<DNS suffix> on LAN and on tailnet [G], shows up in Gatus (R5), and a network-policy test shows it cannot reach core services.”
Depends on:  p4a-02
Touches:     jarvis-apps/apps/demo/, jarvis-apps/.github/workflows/build.yml
Hard limits: R5; R10 (Python); R33 (digest). D25; D35. app.yaml holds source, port, and health path only. deploy/ holds namespaced objects only. The image is a public package; Gordon sets the package public on first publish.
Done checks: build.yml publishes the image. The deploy manifests pin a digest. /health answers in a local container run. One PR.
Timebox:     6h
```

```text
ID:          p4a-05
Goal:        Reject cluster-scoped objects and PVCs in jarvis-apps, and seed one cluster-scoped PR.
Why:         Fix order (§11): the ResourceSet already limits what an app is; CI is the backstop (R30).
Advances:    P4a exit check: “A seeded jarvis-apps PR with a cluster-scoped object fails CI (R30).”
Depends on:  p4a-04, p2-24
Touches:     jarvis-apps/.github/checks/namespaced-only.sh, jarvis-apps/.github/workflows/ci.yml
Hard limits: R30; R36. D31; D35. The pull_request ci job runs the namespaced-only check and holds no secrets. The admin-gate job added in p2-24 stays on pull_request_target, checks out no code, and remains its own required check. Image publish stays in build.yml on push to main (§6). .github/ on jarvis-apps is protected, so this PR waits for admin-approved.
Done checks: A seeded PR that adds a ClusterRole fails ci. A seeded PVC fails ci. apps/demo passes. One PR.
Timebox:     6h
```

```text
ID:          p4a-06
Goal:        Write jarvis-apps/AGENTS.md as the app rulebook.
Why:         Fix order (§11): the namespaced-only check already enforces the boundary; this file states only what that check cannot (R39).
Advances:    P4a exit check: “After Gordon adds one line to apps.yaml, the outside app deploys to its own namespace, is reachable at app-<name>.<DNS suffix> on LAN and on tailnet [G], shows up in Gatus (R5), and a network-policy test shows it cannot reach core services.”
Depends on:  p4a-05
Touches:     jarvis-apps/AGENTS.md
Hard limits: R30; R39. The builder sees only this repo. The file says apps are stateless, namespaced, and free of JARVIS secrets. It does not copy the jarvis rulebook. This path is protected (R36).
Done checks: AGENTS.md is the only rule file in the repo and names the health path and the namespace rule. One PR.
Timebox:     4h
```

```text
ID:          p4a-07
Goal:        Wire make verify PHASE=p4a: Gordon’s apps.yaml line, reachability, Gatus, and the network-policy test.
Why:         Done for the checkpoint is the attached verify output (§2). The line in apps.yaml is Gordon’s merge (R36).
Advances:    P4a exit check: “After Gordon adds one line to apps.yaml, the outside app deploys to its own namespace, is reachable at app-<name>.<DNS suffix> on LAN and on tailnet [G], shows up in Gatus (R5), and a network-policy test shows it cannot reach core services.”
Advances:    P4a exit check: “A seeded jarvis-apps PR with a cluster-scoped object fails CI (R30).”
Depends on:  p4a-03, p4a-04, p4a-05, p4a-06
Touches:     envs/env-01/values/apps.yaml, tools/verify/p4a
Hard limits: R28; R30; R35; R36. D12 stays the platform fixture; the hosted app is app-demo. The network-policy test shows the app pod cannot reach Grafana, the ops API, DNS-to-core service ClusterIPs, or the node IP on 443 (Traefik). Tailnet reachability is the [G] check. Gordon’s apps.yaml merge is a Gordon-run step. apps.yaml and tools/verify/ are protected. D17. §2: make verify prints NEEDS GORDON for each [G] check and each Gordon-run step; the attached output includes his pasted result. The checklist reads live state and links evidence; it does not re-run CI checks (D17).
Done checks: make verify PHASE=p4a prints NEEDS GORDON for the tailnet [G] check and for Gordon’s apps.yaml line, and exits 0 with the app Ready in namespace app-demo, LAN curl to https://app-demo.alpha.lan, the Gatus row, the denied-egress log including the node IP on 443, the seeded cluster-scoped failure, and Gordon’s tailnet paste. One PR.
Timebox:     6h
```

### Exit checks

- After Gordon adds one line to apps.yaml, the outside app deploys to its own namespace, is reachable at app-<name>.<DNS suffix> on LAN and on tailnet [G], shows up in Gatus (R5), and a network-policy test shows it cannot reach core services.
- A seeded jarvis-apps PR with a cluster-scoped object fails CI (R30).

Done = attached make verify PHASE=p4a output

## P4b

Goal: a code-factory:ready issue becomes a checked draft PR in jarvis-apps.

### Contracts

```text
ID:          p4b-01
Goal:        Write the Build agent CONTRACT.md, cursor only, with proven by.
Why:         A lane calls a thin adapter, and the contract is written before the script (§5, R42).
Advances:    P4b exit check: “A toy code-factory:ready issue becomes a draft PR in jarvis-apps with a green CI check and a checker check run (pass or fail with notes) from a different model family.”
Advances:    P4b exit check: “Config validation fails when the builder and checker are from the same model family (seeded test, R41).”
Depends on:  p4a-07
Touches:     docs/build-agent/CONTRACT.md
Hard limits: R18; R36; R41; R42. D7. The contract requires a headless run, a provider-down exit code with no silent pay-as-you-go fallback, one result file, one subscription-backed secret, loading AGENTS.md, the deny list from an admin-gate protected file, a read-only checker, no push and no PR from the adapter, a model map or an unsupported role, and a pinned binary called by full path. grok-build and opencode-local are named as §9 stubs. This path is protected (R36).
Done checks: CONTRACT.md has offers, expects, health, config, and proven by. One PR.
Timebox:     4h
Proven by:   Env schema validation and the P4b checks (§5).
```

```text
ID:          p4b-02
Goal:        Write the Sandbox CONTRACT.md with proven by.
Why:         The lane’s inputs and outputs are the seam gVisor will reuse later, and gVisor itself is not built (§5, §9, R42).
Advances:    P4b exit check: “A toy code-factory:ready issue becomes a draft PR in jarvis-apps with a green CI check and a checker check run (pass or fail with notes) from a different model family.”
Advances:    P4b exit check: “A forced timeout gets the issue the code-factory:failed label, and foreman says so.”
Depends on:  p4b-01
Touches:     services/hermes/deploy/code-factory/sandbox/CONTRACT.md
Hard limits: R42. Inputs: issue, repo, timeout. Outputs: status, patch, verdict, logs. The contract names GitHub Actions as the implementation and gVisor as a later env switch. No RuntimeClass and no gVisor code.
Done checks: CONTRACT.md has offers, expects, health, config, and proven by. One PR.
Timebox:     4h
Proven by:   The P4b toy-issue, timeout, and fix-loop checks.
```

```text
ID:          p4b-03
Goal:        Write the Code-factory work protocol CONTRACT.md with proven by.
Why:         The queue, the label actor, the publish job, and foreman’s report are one protocol (R40, §5).
Advances:    P4b exit check: “A toy code-factory:ready issue becomes a draft PR in jarvis-apps with a green CI check and a checker check run (pass or fail with notes) from a different model family.”
Advances:    P4b exit check: “A seeded code-factory:ready label added by the App, not by Gordon’s account, is ignored, the workflow comments why, and foreman reports it (R40).”
Advances:    P4b exit check: “spec-writer turns an idea into an issue with acceptance criteria and without code-factory:ready; a failing checker verdict keeps the PR a draft and sends it back through the fix loop until the fix cap (R40).”
Depends on:  p4b-02
Touches:     services/hermes/deploy/code-factory/CONTRACT.md
Hard limits: R40; R42. D7. Labels: code-factory:ready, code-factory:hard, code-factory:failed. The workflow runs on issues: labeled, one run at a time. The lane branch is code-factory/<issue>. The publish job opens the draft and posts the checker check run. foreman reports finished, failed, and ignored runs. Gordon merges.
Done checks: CONTRACT.md has offers, expects, health, config, and proven by. One PR.
Timebox:     4h
Proven by:   The P4b checks (§5).
```

### Build

```text
ID:          p4b-04
Goal:        Add lanes/agents/cursor.sh, the family map, and the grok-build and opencode-local stubs that exit not implemented.
Why:         Cursor is the only built adapter. The other two adapters exist only as the §9 stubs.
Advances:    P4b exit check: “A toy code-factory:ready issue becomes a draft PR in jarvis-apps with a green CI check and a checker check run (pass or fail with notes) from a different model family.”
Advances:    P4b exit check: “Config validation fails when the builder and checker are from the same model family (seeded test, R41).”
Depends on:  p4b-01
Touches:     services/hermes/deploy/code-factory/lanes/agents/, hermes.yaml lane map
Hard limits: R23; R41; R42. D28. Builder is Composer 2.5, checker is Grok 4.7 in ask mode, code-factory:hard builder is Claude. The checker family differs from the builder family. Each stub exits with the text not implemented. admin-gate’s protected paths stay the R36 list; .grok/ and .opencode/ join that list only when that adapter is enabled (§9).
Done checks: cursor.sh dry run prints the pinned path and the role map. Both stubs exit non-zero with the text not implemented. The live lane map records different families. One PR.
Timebox:     6h
```

```text
ID:          p4b-05
Goal:        Deploy the code-factory Hermes overlay with foreman and spec-writer, and no ServiceAccount token.
Why:         Foreman reports and spec-writer files issues. Neither one codes, and the pod holds no lane credentials (R13, R40).
Advances:    P4b exit check: “A seeded code-factory:ready label added by the App, not by Gordon’s account, is ignored, the workflow comments why, and foreman reports it (R40).”
Advances:    P4b exit check: “spec-writer turns an idea into an issue with acceptance criteria and without code-factory:ready; a failing checker verdict keeps the PR a draft and sends it back through the fix loop until the fix cap (R40).”
Depends on:  p4b-03, p2-20
Touches:     services/hermes/deploy/code-factory/, profiles/foreman, profiles/spec-writer
Hard limits: R5; R11; R12; R13; R15; R19; R33. D30. Its own volume, secrets, and digest pin. Pre-upgrade backup matches the ops pattern. No ServiceAccount token. D39: egress is DNS plus TCP 443 to 0.0.0.0/0 except RFC1918, the pod and service CIDRs, and the node IP. The CONTRACT.md lists api.x.ai and api.github.com as documentation. Hostname allow-listing waits for the §9 proxy. YOLO stays off on this pod; YOLO is only inside the throwaway lane (R13, R42). Gordon approves this deploy’s xAI device login. Evals for these bots are p4b-09a and p4b-09b.
Done checks: The pod is Ready, the two profiles validate, and the manifest has no ServiceAccount token and no Cursor key. One PR.
Timebox:     6h
```

```text
ID:          p4b-06a
Goal:        Add the reusable lane workflow and the jarvis-apps caller pinned to a jarvis SHA.
Why:         The lane holds a read-only token and the Cursor key, and never the App key (R42).
Advances:    P4b exit check: “A toy code-factory:ready issue becomes a draft PR in jarvis-apps with a green CI check and a checker check run (pass or fail with notes) from a different model family.”
Advances:    P4b exit check: “Token scopes show no agent holds push credentials and the code-factory cannot write to jarvis.”
Depends on:  p4b-02, p4b-04
Touches:     jarvis/.github/workflows/code-factory-lane.yml, jarvis-apps/.github/workflows/code-factory.yml
Hard limits: R40; R42. D27. The lane job receives the Cursor user API key and a read-only token, passed explicitly, and never the App key. The caller passes jarvis_sha. Permissions on the lane job are contents: read. .github/ is protected (R36).
Done checks: A workflow dry review shows the lane job without the App token. One PR.
Timebox:     6h
```

```text
ID:          p4b-06b
Goal:        Add the publisher script that opens the draft PR and posts the checker check run.
Why:         Agents never hold push credentials; the publish job uses a short-lived App token (R42).
Advances:    P4b exit check: “A toy code-factory:ready issue becomes a draft PR in jarvis-apps with a green CI check and a checker check run (pass or fail with notes) from a different model family.”
Advances:    P4b exit check: “Token scopes show no agent holds push credentials and the code-factory cannot write to jarvis.”
Depends on:  p4b-06a
Touches:     services/hermes/deploy/code-factory/publisher/
Hard limits: R10 (Python); R40; R42. The App cannot edit workflows. The publish job is the only PR writer. .github/ is protected (R36).
Done checks: A workflow dry review shows the publish job as the only PR writer. One PR.
Timebox:     4h
```

```text
ID:          p4b-07
Goal:        Enforce the label actor, the one-run queue, the 30-minute timeout, and the fix cap of 2 runs.
Why:         A ready label counts only from Gordon’s account. A run that times out or hits the fix cap gets code-factory:failed, and foreman reports it (R40).
Advances:    P4b exit check: “A seeded code-factory:ready label added by the App, not by Gordon’s account, is ignored, the workflow comments why, and foreman reports it (R40).”
Advances:    P4b exit check: “A forced timeout gets the issue the code-factory:failed label, and foreman says so.”
Depends on:  p4b-06b
Touches:     code-factory workflow
Hard limits: R40; R42. D26; D27. One workflow invocation does builder, then checker, then on fail one builder fix with the checker’s notes, then checker, then publish draft. A second fail or the 30-minute job timeout gets code-factory:failed, and foreman reports it. Any other actor is ignored with a comment. Concurrency is a queue, one lane at a time.
Done checks: A unit fixture of the actor check accepts Gordon’s login and rejects an App login. The workflow file shows that one invocation, the 30-minute job timeout, and code-factory:failed on a second fail. One PR.
Timebox:     6h
```

```text
ID:          p4b-08
Goal:        Fail config validation when the builder and checker families match, and seed that config.
Why:         The checker has to be a different model family from the builder (R41).
Advances:    P4b exit check: “Config validation fails when the builder and checker are from the same model family (seeded test, R41).”
Depends on:  p4b-04, p0a-05
Touches:     tools/checks/lane-families, hermes.yaml
Hard limits: R23; R41. D28. The live env-01 map stays composer versus grok. The seed is a fixture file, not the live env file.
Done checks: The seeded same-family file fails ci. env-01’s lane map passes. One PR.
Timebox:     4h
```

```text
ID:          p4b-09a
Goal:        Teach spec-writer to file an issue only after a chat confirm, with at least three evals.
Why:         spec-writer never adds code-factory:ready (R15, R40).
Advances:    P4b exit check: “spec-writer turns an idea into an issue with acceptance criteria and without code-factory:ready; a failing checker verdict keeps the PR a draft and sends it back through the fix loop until the fix cap (R40).”
Depends on:  p4b-05, p4b-07
Touches:     profiles/spec-writer
Hard limits: R15; R19; R40. D41. The issue has acceptance criteria and does not carry code-factory:ready. spec-writer runs github-mcp-server `stdio --toolsets issues` with code-factory App auth, behind the confirm step. No extra tool allowlist. Evals cover the appendix line for “turn this idea into a code-factory issue” and at least two more. This task holds spec-writer’s evals from the deploy task.
Done checks: A scripted confirm files one issue without the ready label, and a second run with the confirm denied files nothing. spec-writer’s evals pass. One PR.
Timebox:     6h
```

```text
ID:          p4b-09b
Goal:        Teach foreman to report finished, failed, and ignored runs, with at least three evals.
Why:         Foreman watches and reports, and never codes (R40).
Advances:    P4b exit check: “A seeded code-factory:ready label added by the App, not by Gordon’s account, is ignored, the workflow comments why, and foreman reports it (R40).”
Depends on:  p4b-05, p4b-06b, p4b-07
Touches:     profiles/foreman
Hard limits: R19; R40. D37. Foreman’s cron skips the model call when nothing changed. Evals cover “what did the code-factory finish overnight?” and at least two more. Together with p4b-09a the deploy has about 10 evals.
Done checks: Foreman’s session lists an ignored fixture run. Foreman’s evals pass. One PR.
Timebox:     4h
```

```text
ID:          p4b-10
Goal:        Show the lane token scopes and show the code-factory cannot write to jarvis.
Why:         The lane holds no push credential, and the App is installed on jarvis-apps only (R42, R40).
Advances:    P4b exit check: “Token scopes show no agent holds push credentials and the code-factory cannot write to jarvis.”
Advances:    P2 exit check: “Branch protection requires admin-gate (R36): seeded PRs that change a protected path or remove a Namespace fail it without the label and when admin-approved comes from gordon5417-ai or a fixture Bot/App actor, and pass once Gordon’s own account adds it after the latest commit; a seeded PR fails the render check (a stale Hermes render, R4).”
Advances:    P4b exit check: “The code-factory App adding admin-approved fails admin-gate.”
Depends on:  p4b-06b
Touches:     tools/checks/token-scopes, both workflows, jarvis-apps/.github/workflows/seed-app.yml
Hard limits: R36; R40; R42. The check fails if the lane job’s permissions include write, or if the App token secret is passed into the lane job. seed-app.yml is one workflow_dispatch workflow, admin-gate protected. It uses the App secret to add code-factory:ready, add admin-approved, and try a jarvis write. It checks out no PR code.
Done checks: make verify prints the lane permission set and the rejected write to jarvis. Dispatching seed-app.yml makes the code-factory App adding admin-approved fail admin-gate, and the jarvis write fails. One PR.
Timebox:     4h
```

```text
ID:          p4b-11
Goal:        Run a toy code-factory:ready issue through to a draft PR with green CI and a checker check run.
Why:         This is the P4b happy path: one issue becomes a checked draft in jarvis-apps (R40, R41).
Advances:    P4b exit check: “A toy code-factory:ready issue becomes a draft PR in jarvis-apps with a green CI check and a checker check run (pass or fail with notes) from a different model family.”
Depends on:  p4b-06b, p4b-08, p4b-09a, p4b-09b, p4a-04
Touches:     jarvis-apps issue, code-factory/<issue> branch
Hard limits: R40; R41; R42. Gordon applies code-factory:ready. The checker uses Grok, a different family from Composer. The PR stays draft. The check run is named checker and carries pass or fail with notes.
Done checks: The draft PR exists, ci is green, and the checker check run is on the PR. One PR (the machinery). The draft itself stays for Gordon to merge.
Timebox:     6h
```

```text
ID:          p4b-12
Goal:        Seed a code-factory:ready label added by the App, and show it ignored, commented, and reported.
Why:         A ready label from anyone but Gordon’s account must not start a lane (R40).
Advances:    P4b exit check: “A seeded code-factory:ready label added by the App, not by Gordon’s account, is ignored, the workflow comments why, and foreman reports it (R40).”
Depends on:  p4b-07, p4b-09b, p4b-10
Touches:     jarvis-apps issue, foreman session, jarvis-apps/.github/workflows/seed-app.yml
Hard limits: R40. seed-app.yml is the same workflow_dispatch workflow as p4b-10. It uses the App secret to add code-factory:ready, add admin-approved, and try a jarvis write. This task dispatches the ready label. The workflow comments why it was ignored. No lane branch is pushed.
Done checks: Dispatching seed-app.yml adds the label as the App. The issue has the workflow comment, no code-factory branch, and a foreman report of the ignored run. One PR.
Timebox:     4h
```

```text
ID:          p4b-13
Goal:        Seed a failing checker verdict and show the draft staying a draft until the fix cap.
Why:         A failed check sends the work back through the fix loop. Hitting the fix cap of 2 runs applies code-factory:failed, and foreman reports it (R40).
Advances:    P4b exit check: “spec-writer turns an idea into an issue with acceptance criteria and without code-factory:ready; a failing checker verdict keeps the PR a draft and sends it back through the fix loop until the fix cap (R40).”
Depends on:  p4b-07, p4b-11
Touches:     lane workflow, a toy issue whose acceptance criteria fail once
Hard limits: R40. D26. spec-writer’s half of this exit is proven in p4b-09a. This task proves the fix loop. The PR remains draft. After 2 runs, the publish job applies code-factory:failed and foreman reports it.
Done checks: The log shows one invocation: the first checker fail, one builder fix with the checker’s notes, a second checker fail, the draft still draft, and code-factory:failed. Foreman’s session reports that failure. One PR.
Timebox:     6h
```

```text
ID:          p4b-14
Goal:        Force a lane timeout and show code-factory:failed plus foreman’s report.
Why:         A run that times out gets code-factory:failed, and foreman reports it. The same label and report apply when the fix cap of 2 runs is hit (R40).
Advances:    P4b exit check: “A forced timeout gets the issue the code-factory:failed label, and foreman says so.”
Depends on:  p4b-07, p4b-09b
Touches:     sandbox timeout input, foreman
Hard limits: R40; R42. D27. The seed sets the timeout to 1 minute against an issue the adapter is told to wait out. Real issues use the 30-minute limit.
Done checks: The issue carries code-factory:failed, and foreman’s session says the run timed out. One PR.
Timebox:     4h
```

```text
ID:          p4b-15
Goal:        Wire make verify PHASE=p4b and attach a full run.
Why:         Done for the checkpoint is the attached verify output (§2, R35).
Advances:    P4b exit check: “A toy code-factory:ready issue becomes a draft PR in jarvis-apps with a green CI check and a checker check run (pass or fail with notes) from a different model family.”
Advances:    P4b exit check: “A seeded code-factory:ready label added by the App, not by Gordon’s account, is ignored, the workflow comments why, and foreman reports it (R40).”
Advances:    P4b exit check: “spec-writer turns an idea into an issue with acceptance criteria and without code-factory:ready; a failing checker verdict keeps the PR a draft and sends it back through the fix loop until the fix cap (R40).”
Advances:    P4b exit check: “Config validation fails when the builder and checker are from the same model family (seeded test, R41).”
Advances:    P4b exit check: “Token scopes show no agent holds push credentials and the code-factory cannot write to jarvis.”
Advances:    P4b exit check: “A forced timeout gets the issue the code-factory:failed label, and foreman says so.”
Advances:    P4b exit check: “The code-factory App adding admin-approved fails admin-gate.”
Depends on:  p4b-08, p4b-10, p4b-11, p4b-12, p4b-13, p4b-14
Touches:     Makefile, tools/verify/p4b
Hard limits: R35; R36; R40; R42. This PR touches tools/verify/ and waits for admin-approved. Gordon applying code-factory:ready is a Gordon-run step. D17. §2: make verify prints NEEDS GORDON for each [G] check and each Gordon-run step; the attached output includes his pasted result. The checklist reads live state and links evidence; it does not re-run CI checks (D17).
Done checks: make verify PHASE=p4b prints NEEDS GORDON for Gordon’s code-factory:ready step, and exits 0 with the draft PR URL, the ignored-label comment, the spec-writer issue, the fix-cap log, the family-seed failure, the token-scope output, the timeout label, and the code-factory App admin-approved failure. One PR.
Timebox:     4h
```

### Exit checks

- A toy code-factory:ready issue becomes a draft PR in jarvis-apps with a green CI check and a checker check run (pass or fail with notes) from a different model family.
- A seeded code-factory:ready label added by the App, not by Gordon’s account, is ignored, the workflow comments why, and foreman reports it (R40).
- The code-factory App adding admin-approved fails admin-gate.
- spec-writer turns an idea into an issue with acceptance criteria and without code-factory:ready; a failing checker verdict keeps the PR a draft and sends it back through the fix loop until the fix cap (R40).
- Config validation fails when the builder and checker are from the same model family (seeded test, R41).
- Token scopes show no agent holds push credentials and the code-factory cannot write to jarvis.
- A forced timeout gets the issue the code-factory:failed label, and foreman says so.

Done = attached make verify PHASE=p4b output
