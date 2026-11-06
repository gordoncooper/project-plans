# JARVIS AlphaCut — Plan

Derived only from jarvis-alphacut/framework.md. Defaults the framework leaves open are in jarvis-alphacut/decisions.md and cited here as D numbers. One PR per task. Live done checks (make verify ENV=, make drill ENV=, kubectl) are the operator’s (framework §2), who attaches the output to the PR; a coding agent’s are CI or offline checks.

## Stop and ask

- Before building p1-10, confirm with Gordon whether alert routing stays the default or changes.
- Before building p0b-13a or p2-08a, confirm with Gordon which exit check each advances. No current P0b or P2 exit check names the restic-password rotation rehearsal or the signed-tag deploy gate, and an exit check changes only by his decision (§11).

## Gordon-only actions

One list, from §8. Each action names the task that waits on it.

- Before P0: the repo owner merges every PR (D61); gordon5417-ai may open PRs. Task: p0a-01.
- Before P0: merge jarvis-alphacut/framework.md, then jarvis-alphacut/plan.md (§11). Task: p0a-01.
- Before P0: archive the previous system’s repos read-only. Task: p0a-01.
- Before P0: every token, key and webhook for the new build is new; none is reused. Task: p0a-01.
- P0: create the product repo under gordoncooper (D1, D60), before anything writes to jarvis. Task: p0a-01.
- P0: after p0a-06 merges, optionally set branch protection requiring the ci check (paid protection is not the deploy gate, D56). Task: p0a-05.
- P0: set the owner-name repo variable on jarvis (D44). Task: p0a-07.
- P0: invite gordon5417-ai (write) to jarvis. Task: p0a-03.
- P0: accept the invite signed in as gordon5417-ai. Task: p0a-03.
- P0: set up the bastion’s agent user. The Cursor login and the gordon5417-ai token (which Gordon creates signed in as gordon5417-ai and pastes onto the bastion) go to the user that runs the coding agents: the agent user while the separate worker user is deferred (framework §9). Task: p0a-03.
- P0: generate the age key (two copies of one key, the second offline; public key in .sops.yaml) and prove each copy with the pass/fail decrypt command (D50). Task: p0a-03.
- P0: put the age key, and later the off-host backup password and the signing-key fingerprint, on the keep-safe list (D50, D56). Task: p0a-03.
- P0: install Renovate. Task: p0a-15.
- P0: wipe the env-01 node (a separate spare compute node; never a node of the previous system). Task: p0a-11b.
- P0: fresh-install Ubuntu 26.04 Server (minimal install) with a login user that has sudo. Task: p0a-11b.
- P0: make the node’s name resolve on the router’s DNS, then run tools/prep-node from the controller (§8 step 3): SSH key and passwordless sudo for the Ansible user (ssh_user, D34), an optional hostname change, the node address recorded with a warning when it is not stable (a router DHCP reservation is recommended; prep-node does not write or rename netplan), optional SSH password login off, and a clock-sync check. Task: p0a-14.
- P0: run the first site.yml from the controller (an owner run, D17). Task: p0a-14.
- P0: seal the k3s server token with the env’s secrets; the value is never printed (D49). Task: p0a-14.
- P0: rotate any age key an agent user held during bootstrap (a one-off exception to D17), before real data. Task: p0a-14.
- P0: before any env holds real data, confirm both offline age-key copies and test one by decrypting a secret (§8 custody, D50). Task: p0b-12.
- P0: set up env-01’s temporary off-host restic target on the bastion with its own user and append-only access (D48). Task: p0b-09e.
- P0: create gordoncooper/jarvis-internal and rebuild env-01 from the instance repo, with Flux deploying both sources from main (D60, D68). Task: p0b-13.
- P0: run the restic-password rotation rehearsal on env-01; it needs the age key, so it is an owner run (D17). Task: p0b-13a.
- Deferred, not a P0b exit requirement: run the coding-agent worker as its own OS user on the bastion, with no SSH keys to nodes, no kubeconfig, no age key or sops config and no access to other users’ homes; it reads and writes only its own repo checkout and pushes through its GitHub token. Until then .cursor/cli.json is the coding agents’ only guard, and live checks stay the operator’s (§2). Trigger: before coding runs that are unattended or take untrusted input (framework §9). Task: none.
- P0: for each node that has a second disk, the hardware admin prepares and mounts it at the data path, and inventory sets data_disk: true (D42). Example commands are in docs/deploy.md. env-01 has none. Task: p0a-11b.
- P0: once App DNS and the CA exist, set router forwarding of the env’s DNS suffix and trust envs/<env>/ca.crt on the controller and tailnet test devices (framework §8 step 7). Task: p0b-08.
- P1 (observability component): create the send-only alerts bot with BotFather. Task: p1-09.
- P1 (observability component): send it /start once and record his chat ID. Task: p1-09.
- P2: push the first signed release tags, before the chat bot holds real data (D56, D68). Task: p2-08a.
- P2: signing key lost or compromised: rotate the trusted key in Flux (D56). Task: p2-08a.
- P2: create concierge’s Telegram token. Task: p2-15.
- P2: record Gordon’s user ID (the chat ID from P1) for concierge’s allow-list. Task: p2-15.
- P2: approve the ops deploy’s xAI device login. Task: p2-13.
- P2: during the provider-down and capped-login drills, send a Telegram message. Task: p2-18.
- P2: at the exit, create the admin-approved label. Task: p2-24.
- P2: if using branch protection, add admin-gate on the product repo (paid protection is not the deploy gate, D56). Task: p2-24.
- P3: publish the Google OAuth app as In production. Task: p3-00.
- P3: create the Google Cloud OAuth client and enable the Gmail and Calendar APIs. Task: p3-00.
- P3: create the no-repos public-read-only GitHub token. Task: p3-02.
- P3: run the Hermes add-on’s google-consent (make -C services/hermes google-consent ENV=<env>). Task: p3-00.
- P3: send himself one email with subject jarvis-p3-seed. Task: p3-01a.
- P4: approve the code-factory deploy’s xAI device login. Task: p4b-05.
- P4: create the Cursor user API key. Task: p4b-06a.
- P4: set Cursor billing to auto top-up off with a monthly limit (on the plan tier that includes Claude for code-factory:hard). Task: p4b-06a.
- P4: create the GitHub App by hand from its four-permission checklist (contents, pull requests, issues, checks; no manifest flow). Task: p4b-06b.
- P4a: merge the app’s name into the env’s apps list in the instance repo’s apps.yaml. Task: p4a-07.
- P4a: set its GHCR package to public on first publish. Task: p4a-04.
- P4b: apply code-factory:ready. Task: p4b-11.
- P4b: merge code-factory PRs. Task: p4b-11.
- Standing: own router-side access (Tailscale on the router advertises a route to the LAN; the router handles LAN and tailnet DNS; nothing for remote access is installed on the nodes). Task: p0b-11.
- Standing: verify the [G] exit checks (§2). Tasks: p0b-11, p1-13, p2-15, p2-25, p3-03, p3-08, p4a-07.
- Standing: the repo owner merges every PR (D61). Task: p0a-01.
- Standing: review PRs held by admin-gate and add admin-approved from his own account. Task: p2-24 (also any later protected PR, including p4a-05, p4a-07, p4b-01, p4b-15).
- Standing: once the gate lands, push signed release tags; coding agents never create or push tags (D56, D68). Task: p2-08a.
- Standing: merge Hermes bump PRs (R33). Task: p2-20.
- Standing: re-run site.yml after merging an infra/ PR. Task: p0a-13.
- Standing: reboot when a pending-reboot notice says so, using the drain-and-reboot runbook (D52). Task: p0a-11.
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
Advances:    P0a exit check: “One required check on main, ci, runs every ci/checks/ rule check (admin-gate joins at the P2 exit, R36), and a seeded bad PR fails each rule check as it lands (Examples: a hardcoded DNS suffix, a plaintext secret, a missing CONTRACT.md, a missing flux_git_repo_url, a deny list missing a whole deny class).”
Depends on:  none
Touches:     docs/CONTRACT.template.md, schema/CONTRACT.md
Hard limits: R3; R4; R23 (tiers and model names only under envs/). hermes.yaml, apps.yaml, their validators and build_agent are absent until the step that adds their consumer (D54). code_delivery, apps_git_repo_url, llm_provider_host and hermes.* are absent until that consumer step adds the key and its validation (D45). When set, code_delivery is github (§5). No rule ties envs to one GitHub owner (D55). The time zone is part of the env schema. env-01 runs on America/Los_Angeles, and the node and the Hermes schedules both use it. The schema includes ssh_user, the node SSH login, default ubuntu (D34). Gordon creates the product repo under gordoncooper before this PR writes to jarvis (D1, D60). D7. The Hermes schema CONTRACT is planned. Its file is services/hermes/core/schema/CONTRACT.md (p2-04).
Done checks: The file has Offers, Expects, Health, Config, and Proven by, naming checks without plan codes, and the schema includes the time zone and ssh_user. One PR.
Proven by:   Env schema validation in CI.
```

```text
ID:          p0a-02
Goal:        Write the Code delivery provider CONTRACT.md for the six capabilities, GitHub only, with proven by.
Why:         Services, the code-factory, and app hosting depend on this contract and not on GitHub by name (§5, R34).
Advances:    P0a exit check: “One required check on main, ci, runs every ci/checks/ rule check (admin-gate joins at the P2 exit, R36), and a seeded bad PR fails each rule check as it lands (Examples: a hardcoded DNS suffix, a plaintext secret, a missing CONTRACT.md, a missing flux_git_repo_url, a deny list missing a whole deny class).”
Depends on:  p0a-01
Touches:     docs/code-delivery/CONTRACT.md
Hard limits: R34; R36 (admin-gate is the PR-risk check; the deploy gate is Flux signed tags, D56, landing in p2-08a, D68). Gitea is retired for Jarvis (D60). Forgejo stays a §9 seam: this contract names it as the planned second provider and adds no Forgejo code. D7.
Done checks: CONTRACT.md lists repo hosting, CI, the issue queue, PR open and merge, the approval-label check, and the registry, and names the check that proves each, without a plan code. One PR.
Proven by:   Env schema validation and the provider checks (§5).
```

### Build

```text
ID:          p0a-03
Goal:        Create the jarvis repo skeleton, env-01 values, and the inventory (hostnames, no IPs).
Why:         Later tasks need one tree that matches §6, with env specifics only under envs/env-01 (R3).
Advances:    P0a exit check: “From a blank node, every Flux object reports Ready; a second run of bootstrap-secrets and site.yml changes nothing.”
Depends on:  p0a-01, p0a-02
Touches:     jarvis Makefile, docs/README.md, .sops.yaml, envs/env-01/, rulebooks/ left for p0a-17
Hard limits: R3; R10; R25 (inventory holds hostnames, never IPs; the age public key is the recipient Gordon provides). D1; D2; D45 (one .sops.yaml rule per env). Create only paths this checkpoint uses. Hermes deploy folders, app-hosting, and observability wait for their phases. docs/README.md does not link the private plan repo and does not copy the framework. Plan codes stay out of product docs and CONTRACTs; AGENTS.md and rulebooks/ may carry plan codes and plan-repo links (§6).
Done checks: cluster-vars.yaml sets dns_suffix: alpha.lan, flux_git_repo_url, time zone America/Los_Angeles, and the §7 retention. hermes.yaml, apps.yaml and build_agent are absent (D54). code_delivery, apps_git_repo_url, llm_provider_host and hermes.* are absent (D45). inventory/hosts.yml lists bench.lan (D2). envs/env-01/README.md states purpose, what deploys, and promotion. One PR.
```

```text
ID:          p0a-04
Goal:        Add make bootstrap-secrets and make seal: create-if-missing, second run changes nothing.
Why:         Git has to hold the §8 bootstrap secrets before Flux can decrypt one (R7, R25).
Advances:    P0a exit check: “From a blank node, every Flux object reports Ready; a second run of bootstrap-secrets and site.yml changes nothing.”
Depends on:  p0a-03
Touches:     tools/bootstrap-secrets, tools/seal, Makefile
Hard limits: R7; R25; R35. The script creates the internal CA (name-constrained to the env’s DNS suffix, D57) and, for each add-on the env enables, the secrets that add-on declares (Examples: observability’s Grafana admin password; each Hermes deploy’s API key, dashboard password hash and signing secret, D16). It hard-codes no add-on name and mints nothing for an add-on the env leaves off (D69). The core restic credential and the per-app credentials land in p0b-09d (D51). It never overwrites. The real age private key stays out of CI (D33). Two copies of one age key; a repo command test-decrypts one secret and prints only pass or fail (D50). D10: only a restic secret is referenced in-cluster at P0a. Both tools take an explicit env (D45). A .sops.yaml rule for an env that does not exist fails (D55).
Done checks: A unit test with a throwaway age key shows a second run leaves the files byte-identical. make bootstrap-secrets runs on the controller, by Gordon or the agent user since it needs only the .sops.yaml public recipient (D17), and its SOPS output is committed. One PR.
```

```text
ID:          p0a-06
Goal:        Add the required ci workflow, the gitleaks secret scan, a seeded plaintext secret, and a green jarvis-apps ci.
Why:         P0a’s required check is ci, and §4 names the pinned gitleaks CLI as the CI secret scan (R25, §8 phase 0 order).
Advances:    P0a exit check: “One required check on main, ci, runs every ci/checks/ rule check (admin-gate joins at the P2 exit, R36), and a seeded bad PR fails each rule check as it lands (Examples: a hardcoded DNS suffix, a plaintext secret, a missing CONTRACT.md, a missing flux_git_repo_url, a deny list missing a whole deny class).”
Depends on:  p0a-03
Touches:     jarvis/.github/workflows/ci.yml, ci/checks/secret-scan, jarvis-apps/.github/workflows/ci.yml
Hard limits: R25; R36 (admin-gate is not part of ci; it joins at the P2 exit). The secret scan is the pinned gitleaks CLI (§4). jarvis-apps ci runs on pull_request and holds no secrets (D35). Workflows that would hold secrets do not run on fork PRs. D1; D35.
Done checks: ci runs every script in ci/checks/. A seeded PR with a plaintext secret fails gitleaks. jarvis-apps ci reports success on an empty tree. One PR.
```

```text
ID:          p0a-05
Goal:        Implement the env schema validator and seed one PR that drops a required key.
Why:         Fix order (§11): the schema is the boundary; CI only runs it. This is the code that makes the easy values the legal values (R4).
Advances:    P0a exit check: “One required check on main, ci, runs every ci/checks/ rule check (admin-gate joins at the P2 exit, R36), and a seeded bad PR fails each rule check as it lands (Examples: a hardcoded DNS suffix, a plaintext secret, a missing CONTRACT.md, a missing flux_git_repo_url, a deny list missing a whole deny class).”
Depends on:  p0a-01, p0a-03, p0a-06
Touches:     schema/, ci/checks/env-schema
Hard limits: R3; R4; R10 (Python). D3; D34; D45; D54. Validates cluster-vars.yaml and inventory/hosts.yml for every env (§5). hermes.yaml, apps.yaml and build_agent are absent and are not validated here. Gordon may set branch protection requiring ci after p0a-06 merges; paid protection is not the deploy gate (D56). code_delivery, apps_git_repo_url, llm_provider_host and hermes.* are absent (D45). dns_suffix and flux_git_repo_url are required. The defaults for ssh_user, local_path_data and backup_path live only here. Core naming an env, a model, or a DNS suffix is p0a-07. The per-node inventory shape is p0a-11a. No cross-env owner-handle match (D55). The schema refuses the sample add-on (hello) in any env but the example env. An env-level rule makes every stage Kustomization substitute only from cluster-vars. The apps_git_repo_url drop (D45) is a two-repo change: remove the key from the env's values/cluster-vars.yaml in the instance repo first, then from the schema, env-00 and core-names in jarvis.
Done checks: pytest covers a valid env-01 with hermes.yaml, apps.yaml and build_agent absent, and ssh_user defaulting to ubuntu. A seeded PR that omits flux_git_repo_url fails ci. One PR.
```

```text
ID:          p0a-05a
Goal:        Add envs/env-00, the never-deployed 7-node example env that sets every schema key (D43).
Why:         Core is proven against a multi-node env, not only the one-node env-01 (R3).
Advances:    P0a exit check: “One required check on main, ci, runs every ci/checks/ rule check (admin-gate joins at the P2 exit, R36), and a seeded bad PR fails each rule check as it lands (Examples: a hardcoded DNS suffix, a plaintext secret, a missing CONTRACT.md, a missing flux_git_repo_url, a deny list missing a whole deny class).”
Depends on:  p0a-05
Touches:     envs/env-00/, schema/ (tests, CONTRACT.md)
Hard limits: R3; R4; R25. D43; D44. Invented values only; nothing copied from env-01. No secrets/ or flux/. This PR adds the env-00 section to the env schema CONTRACT.md (D45).
Done checks: ci validates env-00. A behavior test fails when the schema allows a key env-00 does not set. One PR.
```

```text
ID:          p0a-07
Goal:        Add the core-names check, scrub existing mentions, and seed failing PRs (D44).
Why:         Core stays env- and owner-agnostic only if CI rejects env names and identity values in it (R3). Model names join at p2-23.
Advances:    P0a exit check: “One required check on main, ci, runs every ci/checks/ rule check (admin-gate joins at the P2 exit, R36), and a seeded bad PR fails each rule check as it lands (Examples: a hardcoded DNS suffix, a plaintext secret, a missing CONTRACT.md, a missing flux_git_repo_url, a deny list missing a whole deny class).”
Depends on:  p0a-05a, p0a-06
Touches:     ci/checks/core-names, .github/workflows/ci.yml, README.md, docs/README.md, envs/env-01/secrets/README.md
Hard limits: R3; R28. D43; D44 (sources, scan paths, match rules, product rules). The check holds no values and no word lists. Contracts and docs use placeholders such as <env> and <dns_suffix>, and the words owner or operator. The env schema CONTRACT.md lists the identity values and the product rules.
Done checks: Seeded PRs fail ci: alpha.lan under platform/, and the owner handle in a docs/ URL. Behavior tests show ci fails with the owner-name variable unset, and passes a value inside a longer word and ubuntu:26.04 when ssh_user is ubuntu. Two non-example envs may use different owner handles (D55). ci passes on main after the scrub. One PR.
```

```text
ID:          p0a-08
Goal:        Add the CONTRACT.md check and seed a PR that drops one.
Why:         Fix order (§11): the folder layout is the boundary (R2); this script only reports a missing contract. D8 defines which folders count.
Advances:    P0a exit check: “One required check on main, ci, runs every ci/checks/ rule check (admin-gate joins at the P2 exit, R36), and a seeded bad PR fails each rule check as it lands (Examples: a hardcoded DNS suffix, a plaintext secret, a missing CONTRACT.md, a missing flux_git_repo_url, a deny list missing a whole deny class).”
Depends on:  p0a-06, p0a-01
Touches:     ci/checks/contract-md, ci/checks/product-ci
Hard limits: R2; D8. The seeded folder is a throwaway component root on the §6 list, removed when the PR closes.
Done checks: A seeded PR whose component root has no CONTRACT.md fails ci, and so does one where product code reads a path under ci/ (§6). Existing contract files pass. One PR.
```

```text
ID:          p0a-10
Goal:        Add the Cursor and Claude Code deny lists and the category-corpus check, so agents change the cluster only through Git.
Why:         Fix order (§11): the deny lists are the automated guard for R34. The rulebook comes after the checks (p0a-17) and does not repeat this list.
Advances:    P0a exit check: “One required check on main, ci, runs every ci/checks/ rule check (admin-gate joins at the P2 exit, R36), and a seeded bad PR fails each rule check as it lands (Examples: a hardcoded DNS suffix, a plaintext secret, a missing CONTRACT.md, a missing flux_git_repo_url, a deny list missing a whole deny class).”
Depends on:  p0a-06
Touches:     .cursor/cli.json, .claude/settings.json, ci/checks/cli-deny
Hard limits: R34. D58. Deny direct kubectl and flux writes, helm writes, ssh, secret decryption, and reading the age private key. Reads stay allowed. Both adapter lists are checked against one category corpus for parity. The guard files are protected and change only in an isolated PR the owner approves. The lists are a seatbelt that shell wrappers can bypass, and they are the only guard until the separate worker user lands (framework §9), so live checks stay the operator’s (§2). Adapters never pass --force or --yolo (p4b-04).
Done checks: ci/checks/cli-deny dry-runs both lists: kubectl apply, helm install, ssh, sops decrypt and an age-key read are blocked, and kubectl get is allowed. A seeded PR that removes a whole deny class from either list (Example: the age-key read) fails ci. One PR.
```

```text
ID:          p0a-11
Goal:        Add the Ansible base-os role: journald cap, chrony with NTS, the time zone, security-only unattended upgrades, and no automatic reboot.
Why:         The blank node has to match the §4 OS before k3s is installed (R1, R10).
Advances:    P0a exit check: “From a blank node, every Flux object reports Ready; a second run of bootstrap-secrets and site.yml changes nothing.”
Depends on:  p0a-03, p0a-05a
Touches:     infra/ansible/roles/base-os, infra/ansible/ansible.cfg, infra/ansible/site.yml
Hard limits: R1 (Ansible only in infra/); R10; §4 Ubuntu 26.04 minimal. D46 (journald SystemMaxUse 2G); D52; D62. The time zone comes from the env schema. Security-only unattended upgrades are on; automatic reboot is off; the owner is told when a reboot is pending; a drain-and-reboot runbook exists. GPU nodes use Ubuntu’s signed linux-modules-nvidia-* packages so kernel and driver stay matched. The package-list refresh stays on; our roles may still install packages. Outbound 123/udp and 4460/tcp stay open for chrony. Use Ansible validate: where a tool can check its own file. Ansible is the pinned PyPI ansible package, which bundles community.general. Collections come only from that package: ansible.cfg sets a repo-local collections_path, so stray ~/.ansible copies are ignored, and there is no Galaxy requirements file until a collection outside the package is needed. No Tailscale or VPN on the node (R28). The first play asserts amd64 hosts before any change (D46, D59).
Done checks: ansible-playbook --syntax-check site.yml passes against the env-00 inventory. Small tests parse the rendered files for the journald cap, the NTS pools, security-only unattended upgrades, and no automatic reboot. The live proof is p0a-14: chronyc -N authdata shows NTS, and no reboot is pending after a clean boot. One PR.
```

```text
ID:          p0a-11a
Goal:        Declare each node’s roles and optional data disk in inventory/hosts.yml, turn the roles into groups with the constructed config, and validate both (D42).
Why:         Roles decide where each play runs, and one block per node holds them (R3).
Advances:    P0a exit check: “From a blank node, every Flux object reports Ready; a second run of bootstrap-secrets and site.yml changes nothing.”
Depends on:  p0a-05a
Touches:     envs/env-01/inventory/hosts.yml, envs/env-00/inventory/hosts.yml, infra/inventory/roles.yml, schema/ (validator, CONTRACT.md)
Hard limits: R3; R4; R25. D42 (shape, schema rules, constructed config); D43; D59. One control plane, amd64 only; GPU hosts join as agents. Nodes without the workers role carry a NoSchedule taint; platform pods and tools tolerate it; agents restart one at a time. infra/inventory/roles.yml is the core constructed inventory (a folder: the constructed config plus an empty data_disk group), passed as a second -i and never copied into an env. This PR adds the node roles and labels section to the env schema CONTRACT.md (D45).
Done checks: Behavior tests show the schema rejects an unknown role, an env without exactly one control_plane or exactly one backup host, a second control_plane, an unknown host key, and a data_disk value other than true. In ci, ansible-inventory --graph on env-00 shows each role and data_disk as a group, with no leading underscore. One PR.
```

```text
ID:          p0a-11b
Goal:        Add the node-disk role that checks a declared data disk, and the site.yml fact asserts (D42).
Why:         k3s, etcd snapshots, volumes and backups write under the data paths, so a declared data disk has to be a real mount before k3s starts (R31).
Advances:    P0a exit check: “From a blank node, every Flux object reports Ready; a second run of bootstrap-secrets and site.yml changes nothing.”
Advances:    P0b exit check: “Restore drill: a deleted volume is restored by make restore from its rest-server backup and its checksum matches; an etcd snapshot exists for same-node rollback and comes back from its restic copy; verify reports the off-host copy’s freshness and that no app credential opens a core repository (R31, D48, D51).”
Depends on:  p0a-11, p0a-11a
Touches:     infra/ansible/roles/node-disk, infra/ansible/site.yml
Hard limits: R1; R31. D3; D42 (asserts, disk check, boot safety). Ansible checks that the data path exists, is a real mount when data_disk is true, and is empty or already holds Jarvis data, and refuses otherwise. It does not format, wipe, label or mount the disk. The hardware admin prepares and mounts the disk; example commands are in docs/deploy.md. A node with no second disk leaves data_disk out. Shell only where no module exists.
Done checks: A small test shows the role refuses a declared data disk whose path is missing, not a mount, or neither empty nor already Jarvis data, and that it changes nothing on a host that leaves data_disk out. env-01 has no data disk. One PR.
```

```text
ID:          p0a-12
Goal:        Add the k3s role: config.yaml, fixed CIDRs, secrets encryption, reserves, data paths, etcd snapshots, node labels, marker file, and agents joining the server.
Why:         Phase 0 installs k3s from config.yaml because Ubuntu sudo-rs drops INSTALL_K3S_* (§8, R1).
Advances:    P0a exit check: “From a blank node, every Flux object reports Ready; a second run of bootstrap-secrets and site.yml changes nothing.”
Advances:    P0b exit check: “Restore drill: a deleted volume is restored by make restore from its rest-server backup and its checksum matches; an etcd snapshot exists for same-node rollback and comes back from its restic copy; verify reports the off-host copy’s freshness and that no app credential opens a core repository (R31, D48, D51).”
Depends on:  p0a-11b
Touches:     infra/ansible/roles/k3s, envs/env-01/values/cluster-vars.yaml
Hard limits: R1; §7 (kube and system reserves about 1 GB each, eviction thresholds); R28; R31; R33. D2; D3; D4; D5; D6; D34; D42; D46; D49; D59. The k3s version and binary checksum are pinned in cluster-vars.yaml and change only by PR; Renovate does not manage them. The server installs on control_plane, and every other host joins as an agent (D42 Join): server https://<control_plane inventory_hostname>:6443, its own node-name, the agent token, no cluster-init and no node-label. The server token is sealed with the env’s secrets; agents join with a separate agent token (D49). Neither token is printed. Sealing an existing env’s token is an owner step. A task on control_plane reconciles the role labels and servicelb’s enablelb label on every run with kubernetes.core (python3-kubernetes from apt). The role creates local_path_data, backup_path and etcd-snapshots at 0700 (node-disk does not). It adds a k3s.service drop-in with RequiresMountsFor= those paths so k3s waits for the mount (D42). On a server/agent switch it disables the opposite k3s unit. site.yml refuses a host that already runs k3s unless /etc/jarvis/env holds this env’s name; the first run writes that marker. The dead-node rebuild path is written; the single-node rehearsal is a planned outage on the owner’s go and never touches the data disk (D49).
Done checks: A small test of the rendered config.yaml shows --cluster-init, secrets encryption, the CIDRs, the reserves, the snapshot directory, node-name, no node-label, and no INSTALL_K3S_* variable. The rendered agent config shows the server URL, its own node-name and no cluster-init or node-label. A host-pattern check selects 6 agent hosts on env-00 and none on env-01, where the agent play exits 0. A localhost play with a fake systemctl shows site.yml refuses a host that runs k3s without the marker. The live proof is p0a-14. One PR.
```

```text
ID:          p0a-13
Goal:        Add the Flux Operator Ansible role: one-time apply, sops-age, and the root GitRepository; FluxInstance sync is dropped.
Why:         The age key and the first Flux apply are the bootstrap bridge (R1, R25). After that, Git is the source (R34). The FluxInstance sync block has no verify field, so the env’s root source is a self-managing GitRepository (D56).
Advances:    P0a exit check: “From a blank node, every Flux object reports Ready; a second run of bootstrap-secrets and site.yml changes nothing.”
Depends on:  p0a-12, p0a-04
Touches:     infra/ansible/roles/flux-operator, infra/ansible/roles/kubeconfig, infra/ansible/site.yml, platform/flux-operator/ (chart pin)
Hard limits: R1; R25. D11; D17; D34; D42; D53; D56. The private age key is applied once as flux-system/sops-age, which Flux never reconciles, with no_log on every task that handles it. Only flux-system decrypts. Flux runs with multi-tenancy lockdown; every Kustomization and HelmRelease names its service account. The root source is a self-managing GitRepository that Ansible applies (FluxInstance sync dropped). Signature verification of both sources lands in p2-08a (D68), not in p0b-13. site.yml needs the age key only while flux-system/sops-age is absent, so the first run on a fresh node and an age-key swap are owner runs; later runs may be the agent user’s (D17). kubeconfig is its own small role that writes only the running user’s ~/.kube/jarvis-<env>.yaml, mode 0600, by slurp and copy, check-mode safe, with no backup, and never writes ~/.kube/config. Bastion kubectl uses that file. Tags are set at role or play level (§11). The operator chart version has one pin, in platform/flux-operator/, which this role reads and Renovate bumps.
Done checks: site.yml runs in §8 order: base-os on all, node-disk on data_disk, the k3s server on control_plane, the k3s agent on all:!control_plane, then kubeconfig and flux-operator on control_plane. site.yml --tags kubeconfig runs only the kubeconfig role. The live proof is p0a-14. One PR.
```

```text
ID:          p0a-14
Goal:        Add the Flux entry point: four Kustomizations with dependsOn, the named core source, the operator HelmRelease, and one decrypted Secret.
Why:         Git has to drive a blank node, with platform before configs before services before apps (R1, R7). Services land as per-service Flux Kustomizations from the named core source (D53).
Advances:    P0a exit check: “From a blank node, every Flux object reports Ready; a second run of bootstrap-secrets and site.yml changes nothing.”
Advances:    P0a exit check: “A SOPS-encrypted secret decrypts in-cluster.”
Depends on:  p0a-13, p0a-02, p0a-08
Touches:     envs/env-01/flux/, platform/flux-operator/, platform/backups/ namespace only, tools/lib/render_cluster_vars_configmap.py, the stale-render test in ci/tests/test_flux_entry.py, ci/checks/flux-substitute
Hard limits: R1; R4; R25; R34. D9; D10; D11; D45 (lowercase variables, JSON lists, strict substitution, $${} escapes, explicit env); D53. Resources that use their own ${…} syntax set substitute: disabled. Every ${var} a Flux Kustomization substitutes is a key in that env’s cluster-vars; $${} is a literal escape. One script renders the cluster-vars ConfigMap from an allow-list of cluster keys in values/cluster-vars.yaml; host-only keys stay in Ansible and are not published. This step decides which keys are published and which stay host-only, and lists them in the env schema CONTRACT (the cluster-vars to postBuild section). tools/prep-node validates the env before touching a host and re-runs the render after editing cluster-vars. make site runs the schema before Ansible. Every Kustomization and HelmRelease names its service account. Tenant secrets live under envs/<env>/secrets/<namespace>/; a tenant namespace never gets the age key.
Done checks: CI runs kustomize build and a schema check on every path the Flux Kustomizations name. A seeded stale cluster-vars render fails ci. A seeded ${var} a Flux Kustomization substitutes that is not a key in that env’s cluster-vars fails ci, and a $${} escape passes. A seeded Kustomization or HelmRelease with no service account fails ci. Gordon’s first site.yml on the fresh node (an owner run, D17), after tools/prep-node and make bootstrap-secrets, passes the D42 asserts; runs no node-disk, since env-01 has no data_disk and keeps its data on lv-data on the OS disk (D42); seals the k3s server token (D49); and leaves each node with exactly its role labels and the enablelb label on exactly the edge set. After it, every Flux object reports Ready and a restic Secret exists in platform-backups, its value unprinted. A second site.yml and a second bootstrap-secrets, run by Gordon or the agent user (D17), change nothing. One PR.
```

```text
ID:          p0a-15
Goal:        Add Renovate on jarvis and jarvis-apps, opening update PRs right away; auto-merge stays off.
Why:         P0a is done only after Renovate has opened a PR (R33).
Advances:    P0a exit check: “Renovate is installed and has opened at least one PR (its onboarding PR counts).”
Depends on:  p0a-14
Touches:     renovate.json in both repos
Hard limits: R33. D5; D11; D61. Auto-merge stays off; the owner merges every PR. Renovate config has no release-age or stability hold and opens update PRs right away; whether to wait a few days before merging a fresh release is a line in the operations docs, not config. Majors and Hermes digests are human-reviewed (p2-20). The image digest check lands with the first third-party image (p0b-07). Gordon installs the hosted Renovate GitHub App; no local Renovate tooling.
Done checks: Renovate has opened its onboarding PR or a dependency PR. The config has no release-age hold, automerge is off, and the k3s exclusion is set. One PR.
```

```text
ID:          p0a-16
Goal:        Run the rollback drill: change a throwaway ConfigMap through Git, revert the commit, and show Flux restored the prior value.
Why:         Rollback is revert-the-commit (R37). P0a has to prove it once.
Advances:    P0a exit check: “Rollback drill: reverting a commit restores the prior state.”
Depends on:  p0a-14
Touches:     a throwaway ConfigMap under the platform-configs path
Hard limits: R34; R37. The drill uses a ConfigMap, not a Secret, and needs only git revert and kubectl reads. The ConfigMap is removed afterward. Data restore is P0b (R31).
Done checks: The attached log shows the new value, the revert, and the prior value back on the cluster, and the ConfigMap is gone afterward. A few tiny PRs (baseline, change, revert, remove).
```

```text
ID:          p0a-17
Goal:        Write the coding-agent rulebook, a lean AGENTS.md, and the PR template.
Why:         Fix order (§11): prose only for what the checks and the deny list cannot enforce (R39). This task comes after p0a-06 through p0a-10.
Advances:    P0a exit check: “One required check on main, ci, runs every ci/checks/ rule check (admin-gate joins at the P2 exit, R36), and a seeded bad PR fails each rule check as it lands (Examples: a hardcoded DNS suffix, a plaintext secret, a missing CONTRACT.md, a missing flux_git_repo_url, a deny list missing a whole deny class).”
Depends on:  p0a-06, p0a-07, p0a-08, p0a-10
Touches:     rulebooks/coding-agents.md, AGENTS.md, .github/pull_request_template.md
Hard limits: R35; R39. AGENTS.md points at the rulebook and does not copy it or the framework. The rulebook states the §11 lean standard, code design and review rules in a few lines. The PR template is the brief template plus an evidence line and a short ready-to-merge list, including that each new or changed rule check fails on a seeded bad value. Branch names for coding agents are agent/<topic> (§6).
Done checks: A reviewer can point at one rule that exists only in the rulebook because it cannot be a check. The template contains the §11 fields. One PR.
```

```text
ID:          p0a-18
Goal:        Add make verify ENV= with the P0a standing checks. The owner checklist stays in the plan, outside the script.
Why:         Done for the checkpoint is attached evidence that Gordon reviews (R35, §2).
Advances:    P0a exit check: “From a blank node, every Flux object reports Ready; a second run of bootstrap-secrets and site.yml changes nothing.”
Advances:    P0a exit check: “A SOPS-encrypted secret decrypts in-cluster.”
Advances:    P0a exit check: “One required check on main, ci, runs every ci/checks/ rule check (admin-gate joins at the P2 exit, R36), and a seeded bad PR fails each rule check as it lands (Examples: a hardcoded DNS suffix, a plaintext secret, a missing CONTRACT.md, a missing flux_git_repo_url, a deny list missing a whole deny class).”
Advances:    P0a exit check: “Renovate is installed and has opened at least one PR (its onboarding PR counts).”
Advances:    P0a exit check: “Rollback drill: reverting a commit restores the prior state.”
Depends on:  p0a-04, p0a-05, p0a-14, p0a-15, p0a-16, p0a-17
Touches:     Makefile, tools/verify/
Hard limits: R35; R37. D17; D42. The script checks: kubectl wait Ready on every Flux Kustomization, GitRepository and HelmRelease, failing when none are found; a restic Secret, its value unprinted; each node’s role labels equal to its node_roles; the enablelb label on exactly the edge set. It does not list owner items, and it exits non-zero if anything is unchecked. infra/ansible/README.md has Recovery steps for an address change (k3s --cluster-reset), a broken root GitRepository or operator (re-apply it from the reverted checkout), a lost age key (copy flux-system/sops-age back before deleting anything) and an unsynced clock; the checklist’s rollback line points at them. Owner checklist, pasted by Gordon outside the script: the second site.yml and second make bootstrap-secrets outputs, whoever ran them, the seeded PR for each rule check, the Renovate PR, and the p0a-16 rollback log.
Done checks: CI passes. The operator attaches make verify ENV=<env> output: exit 0 on the live cluster, and non-zero against an inventory copy that gives a node a role it lacks. The rollback drill is attached as its own run. Gordon pastes the owner checklist outside the script. One PR.
```

### Exit checks

- From a blank node, every Flux object reports Ready; a second run of bootstrap-secrets and site.yml changes nothing.
- A SOPS-encrypted secret decrypts in-cluster.
- One required check on main, ci, runs every ci/checks/ rule check (admin-gate joins at the P2 exit, R36), and a seeded bad PR fails each rule check as it lands (Examples: a hardcoded DNS suffix, a plaintext secret, a missing CONTRACT.md, a missing flux_git_repo_url, a deny list missing a whole deny class).
- Renovate is installed and has opened at least one PR (its onboarding PR counts).
- Rollback drill: reverting a commit restores the prior state.

Done = attached make verify ENV=<env> output, plus this phase’s drills and evals as their own runs

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
Hard limits: R5; R28. D7. The contract states App DNS answers the name, the wildcard covers it, and, when the observability component is enabled, the component adds its own Gatus check, marked planned until Gatus is installed. Gatus trusts the public root and resolves the zone through cluster DNS. Core’s own health is make verify, not Gatus (D69).
Done checks: CONTRACT.md has Offers, Expects, Health, Config, and Proven by, naming checks without plan codes. One PR.
Proven by:   The R5 manifest lint and the reachability checks (§5).
```

```text
ID:          p0b-02
Goal:        Write the Backups CONTRACT.md with proven by.
Why:         Every PVC uses one shared backup contract before VolSync is installed (§5, R31).
Advances:    P0b exit check: “Restore drill: a deleted volume is restored by make restore from its rest-server backup and its checksum matches; an etcd snapshot exists for same-node rollback and comes back from its restic copy; verify reports the off-host copy’s freshness and that no app credential opens a core repository (R31, D48, D51).”
Depends on:  p0a-18
Touches:     platform/backups/CONTRACT.md
Hard limits: R31. D7; D48; D51; D67. Target path, off-host target, retention, and weekly prune come from cluster-vars.yaml. Every PVC is enrolled by the backup policy, and restore is make restore ENV=<env> APP=<app> and make restore-all ENV=<env> (D47). Operations holds the restore, CA-rotation, restic-rotation and age-key recovery runbooks. The opt-out is the unprefixed annotation backup: none on a PVC or on a volumeClaimTemplate’s metadata, allowed under services/observability/victoria-metrics/, services/observability/victoria-logs/, and the rebuildable-cache class (model weights, build caches). The policy creates one ReplicationSource per claim, including each StatefulSet ordinal; none is written in Git. Core namespaces share one restic credential; each app-<name> namespace gets its own (D51, p0b-09d). The off-host copy is p0b-09e.
Done checks: CONTRACT.md has Offers, Expects, Health, Config, and Proven by, naming checks without plan codes. One PR.
Proven by:   The R31 CI check and the restore drills (§5).
```

### Build

```text
ID:          p0b-03
Goal:        Install Traefik’s HelmChartConfig for plain Ingress, limiting the default TLSStore to the cert-manager namespace.
Why:         P0b reachability is HTTPS through Traefik Ingress (OQ8, R28).
Advances:    P0b exit check: “A test app answers over HTTPS on <svc>.<DNS suffix> (env-01: <svc>.alpha.lan) with a cert from the internal CA, from LAN and from tailnet [G], through Traefik Ingress.”
Depends on:  p0b-01
Touches:     platform/traefik/
Hard limits: R1; R4; R5; R9; R30 waits for P4a. D30; D32; D42 (80/443 land on the edge set through servicelb). kube-system receives only this HelmChartConfig. No Gateway API. platform-controllers points at one kustomization.yaml; later controllers join its resources list rather than add Flux paths. The placeholder ConfigMap on this path is removed (D9).
Done checks: The HelmChartConfig is applied and Traefik is Ready, probes and memory limits are set, and no Gateway API object exists. One PR.
```

```text
ID:          p0b-04
Goal:        Install cert-manager, the internal CA, and the wildcard certificate. Commit the public root. Move the controller list to platform/kustomization.yaml, and tie Traefik’s config to the k3s pin.
Why:         The test app’s cert has to come from the internal CA, and rebuilds have to reuse the root key (R28).
Advances:    P0b exit check: “A test app answers over HTTPS on <svc>.<DNS suffix> (env-01: <svc>.alpha.lan) with a cert from the internal CA, from LAN and from tailnet [G], through Traefik Ingress.”
Depends on:  p0b-03
Touches:     platform/cert-manager/controller, platform/cert-manager/config, platform/kustomization.yaml, platform/flux-operator/kustomization.yaml, envs/<env>/flux/kustomizations/platform-controllers.yaml, ci/tests/test_traefik.py, ci/tests/test_backup_enrollment.py, ci/tests/test_prune_guards.py, tools/bootstrap-secrets, envs/<env>/ca.crt, platform/traefik/CONTRACT.md, renovate.json
Hard limits: R1; R5; R25; R28; R33. D5. The root key is a SOPS secret from make bootstrap-secrets, run on the controller by Gordon or the agent user (D17): the script generates the key and encrypts it to the .sops.yaml recipient in one step. The goal is to keep the plaintext key out of the repo, logs and anything persistent: bootstrap pipes it straight into sops where that is small, and otherwise uses 0600 temp files it deletes right away. The public root is plain text for Gatus and for devices: bootstrap-secrets also writes it to envs/<env>/ca.crt and never writes the key there. It is idempotent and never rewrites an existing CA or root file. With a new CA it writes the root as it generates the CA, which needs only the public recipient, so the agent user can run it. When the CA secret already exists and the root file does not, it decrypts only tls.crt in memory; that needs the age key, so that one run is an owner run (D17). The CA lifetime is 10 years. A new env’s CA is name-constrained to its DNS suffix (D57); an existing CA gains the constraint at its next planned rotation. Gordon keeps the calendar reminder (§8 custody). The CA sets its extensions explicitly through its own openssl config, not the host’s defaults: basicConstraints critical, CA:true, pathlen:0; keyUsage critical, keyCertSign and cRLSign; and a subject key identifier, so strict X.509 clients (openssl -verify_return_error, Python ssl) accept it. Tools refuse a path that would replace the CA. cert-manager does not reissue on a CA change, so a CA rotation deletes each wildcard’s TLS Secret (the apps wildcard’s too, when app hosting is on) to force the reissue. cert-manager’s namespace, HelmRepository and HelmRelease carry Flux’s prune: disabled (R1), and ci/tests/test_prune_guards.py lists every guarded platform object; each later platform controller adds its objects there. D30. The controller index moves out of platform/flux-operator/kustomization.yaml into platform/kustomization.yaml, so the flux-operator folder keeps only the operator; the platform-controllers Flux Kustomization path becomes ./platform, and cert-manager joins the list as one line. ci/tests/test_traefik.py follows the move to platform/. The bundled Traefik chart version is set only by the k3s pin, so a k3s bump can silently break the HelmChartConfig’s assumptions. The Traefik CONTRACT.md records the chart version and the k3s version it was checked against, and a ci test fails when an env’s k3s_version differs from that k3s version, so a k3s bump PR re-checks platform/traefik and updates the line. Renovate does not manage k3s (D5), so its disabled k3s rule only names that re-check in its description; no prBodyNotes, which a disabled rule never shows.
Done checks: A Certificate for *.<DNS suffix> (env-01: *.alpha.lan) is Ready and the cert chains to the committed root, which openssl verify accepts with -x509_strict. The default TLSStore sits next to that Certificate in the platform-cert-manager namespace, so kube-system keeps only the HelmChartConfig. After merge, every Kustomization is Ready, and the rendered objects for the Flux Operator, Traefik and VolSync are unchanged apart from cert-manager. A seeded k3s_version bump without a Traefik CONTRACT.md update fails ci. The k3s Renovate rule’s description names the re-check. The live checks (Certificate Ready, every Kustomization Ready) are a post-merge drill step, not something the coding agent proves. One PR.
```

```text
ID:          p0b-05
Goal:        Install App DNS (CoreDNS with k8s_gateway) and the coredns-custom forward.
Why:         Names under alpha.lan have to resolve from the cluster and, once Gordon forwards the zone, from the LAN (R28, §4).
Advances:    P0b exit check: “A test app answers over HTTPS on <svc>.<DNS suffix> (env-01: <svc>.alpha.lan) with a cert from the internal CA, from LAN and from tailnet [G], through Traefik Ingress.”
Depends on:  p0b-04
Touches:     platform/app-dns/, kube-system coredns-custom, infra/ansible/roles/base-os (edge host resolver), schema/ (app_dns_cluster_ip), ci/tests/test_prune_guards.py, renovate.json
Hard limits: R1; R5; R28; R33. D30; D42 (port 53 on the edge set). This CoreDNS is separate from k3s cluster DNS. LoadBalancer Services are only Traefik and this DNS (the lint is p0b-06). No VPN agent on the node. The App DNS Service has a fixed ClusterIP, app_dns_cluster_ip in cluster-vars, because CoreDNS forward takes an address: the schema requires a host address inside service_cidr that is not the network, broadcast, first (Kubernetes API) or tenth (cluster DNS) address. Cluster DNS forwards the zone to that address through coredns-custom, so pods and Gatus resolve it. App DNS’s namespace, HelmRepository, HelmRelease and the coredns-custom ConfigMap carry prune: disabled and join ci/tests/test_prune_guards.py. Renovate tracks the OCI chart pin with the docker datasource. Host resolver: ServiceLB publishes port 53 as a hostPort with no hostIP, and the CNI portmap DNATs locally generated traffic too, so on an edge host lookups to the systemd-resolved stub at 127.0.0.53 land on App DNS, which refuses names outside its zone, and apt, image pulls and NTS break (k3s-io/k3s#1640). So base-os, on edge hosts only, turns the stub off with a resolved drop-in (DNSStubListener=no) and links /etc/resolv.conf to resolved’s upstream list, /run/systemd/resolve/resolv.conf, under a resolved tag. A host leaving edge keeps the stub off. Rejected: binding port 53 to one address (ServiceLB has no hostIP field), MetalLB or another exposer (breaks the LoadBalancer constraint), iptables exceptions (k3s rewrites its chains), and forwarding everything from App DNS upstream (host DNS would depend on a pod, and App DNS would become an open resolver). Order: on an existing cluster, run site.yml --tags resolved on the edge hosts before App DNS first applies, and again whenever a run adds an edge host; a blank-node build gets it from base-os before k3s.
Done checks: CI passes, including schema tests that reject a bad app_dns_cluster_ip. Post-merge operator checks: a second site.yml --tags resolved run reports changed=0; an edge host resolves outside names; a throwaway test Ingress host resolves to an edge node address through App DNS, through cluster DNS from a pod, and from the LAN. testapp is not required yet. One PR.
```

```text
ID:          p0b-06
Goal:        Add the manifest lint for probes, resources, and stdout, and the LoadBalancer allow-list, each with a seeded PR.
Why:         Fix order (§11): components already set the fields (p0b-03 through p0b-05); these checks only catch a later miss (R5, R28).
Advances:    P0b exit check: “A test app answers over HTTPS on <svc>.<DNS suffix> (env-01: <svc>.alpha.lan) with a cert from the internal CA, from LAN and from tailnet [G], through Traefik Ingress.”
Depends on:  p0b-03, p0b-05
Touches:     ci/checks/manifest-lint, ci/checks/loadbalancer
Hard limits: R5 (memory request and limit, CPU request, no CPU limit on long-running containers, probes, stdout logs, and a Gatus check once the endpoint exists and the observability component is enabled, D69). R28. D30: short-lived chart hooks, Jobs, init containers and test pods keep chart defaults.
Done checks: A seeded manifest missing a memory limit fails. A seeded LoadBalancer besides Traefik and App DNS fails. Current platform manifests pass. One PR, shared with p0b-07 (same area).
```

```text
ID:          p0b-07
Goal:        Add the image digest pin check and seed an unpinned image.
Why:         Images are pinned by digest so Renovate is the upgrade path (R33).
Advances:    P0b exit check: “A test app answers over HTTPS on <svc>.<DNS suffix> (env-01: <svc>.alpha.lan) with a cert from the internal CA, from LAN and from tailnet [G], through Traefik Ingress.”
Depends on:  p0b-06
Touches:     ci/checks/digest-pin
Hard limits: R33. The k3s host binary stays a cluster-vars pin (D5), not an image digest. The digest-pin check scans images the repo writes, including HelmChartConfig valuesContent. Images k3s bundles that the repo does not write stay pinned by k3s_version.
Done checks: A seeded image: tag with no digest fails ci, including one in HelmChartConfig valuesContent. cert-manager and App DNS images are digest-pinned. Ships in the p0b-06 PR.
```

```text
ID:          p0b-08
Goal:        Deploy the test app and its Ingress, and prove HTTPS from the LAN.
Why:         P0b’s reachability check needs one named HTTPS service on the internal CA (R28).
Advances:    P0b exit check: “A test app answers over HTTPS on <svc>.<DNS suffix> (env-01: <svc>.alpha.lan) with a cert from the internal CA, from LAN and from tailnet [G], through Traefik Ingress.”
Depends on:  p0b-04, p0b-05, p0b-06, p0b-07
Touches:     platform/testapp/, envs/<env>/flux/platform-configs/kustomization.yaml, renovate.json
Hard limits: R5; R28. D12. Hostname testapp.alpha.lan. The app is a platform fixture, not an app-<name> hosted app (that isolation is P4a, R30). It is the platform canary: the P0b check, Gatus, the P1 log proof and the P1 alert drill use it. It never depends on services or apps, and nothing depends on it. It stays on the platform-configs Flux stage, which has no wait or healthChecks, so a crashloop blocks nothing; only a failed apply would. Pod seccomp RuntimeDefault; read-only root filesystem with only /tmp writable. Renovate tracks the image with a custom regex manager scoped to platform/testapp/ (the kubernetes manager has no default file patterns).
Done checks: Gordon does the router forwarding and root-cert trust (framework §8 step 7) before the LAN curl. curl from the LAN to https://testapp.alpha.lan returns the known file and the cert chains to the internal CA. The tailnet half is an owner checklist item (D17). One PR.
```

```text
ID:          p0b-09
Goal:        Add the R31 PVC backup-entry check, run the VolSync backup-and-restore test (if it fails, install the single restic CronJob), then apply retention and the weekly prune.
Why:         OQ7 is a Phase 0 test with one fallback and no further searching (R31, §4).
Advances:    P0b exit check: “Restore drill: a deleted volume is restored by make restore from its rest-server backup and its checksum matches; an etcd snapshot exists for same-node rollback and comes back from its restic copy; verify reports the off-host copy’s freshness and that no app credential opens a core repository (R31, D48, D51).”
Depends on:  p0b-02, p0a-14
Touches:     ci/checks/pvc-backup, platform/backups/controller, platform/backups
Hard limits: R9; R31. No local S3 server. No volume snapshot controller (VolSync gets only the VolumeSnapshot CRD). D3; D15; D30. Prune is weekly through pruneIntervalDays: 7 (D15). The ~10-minute test ran early, right after p0b-03, as its own small PR and passed, so VolSync is the choice; the second PR writes that choice into the backup CONTRACT.md config section and adds the R31 check, retention and prune. The R31 check is small: every PVC and StatefulSet volumeClaimTemplate has a ReplicationSource or the p0b-02 backup: none annotation inside the allow-list. It does not see PVCs a HelmRelease renders; p0b-09c replaces it. Outcome: VolSync stays; the follow-up is the rest-server, the etcd-snapshot copy and the enrollment policy (p0b-09a, p0b-09b, p0b-09c), not another tool search.
Done checks: A seeded PR with a PVC that has neither a backup entry nor backup: none fails ci. The log shows the VolSync test result and either a working VolSync ReplicationSource or the one CronJob, writing to backup_path. Retention in cluster-vars is 7 daily / 4 weekly / 3 monthly (§7). Two PRs.
```

```text
ID:          p0b-09a
Goal:        Serve every core restic repository from one restic rest-server, one repository per PVC.
Why:         One server owns the core backup path, so a new volume needs no storage of its own, and NFS later changes one volume (R31, D47).
Advances:    P0b exit check: “Restore drill: a deleted volume is restored by make restore from its rest-server backup and its checksum matches; an etcd snapshot exists for same-node rollback and comes back from its restic copy; verify reports the off-host copy’s freshness and that no app credential opens a core repository (R31, D48, D51).”
Depends on:  p0b-09
Touches:     platform/backups/rest-server.yaml, platform/backups/controller (VolSync TZ), platform/backups/CONTRACT.md, schema/CONTRACT.md, tools/bootstrap-secrets
Hard limits: R5; R9; R25; R31. D3; D15; D42; D47; D51. One Deployment and one ClusterIP Service named restic on port 8000 in platform-backups, no Ingress, pinned to role.jarvis/backup=true, storing in backup_path/restic. Each repository is rest:http://restic.platform-backups.svc.cluster.local:8000/ns-<namespace>/pvc-<pvc> (on disk backup_path/restic/ns-<namespace>/pvc-<pvc>): the slash keeps two names from colliding, and the prefixes keep every segment off rest-server’s reserved names (data, keys, locks, snapshots, config). This server holds core’s one login. Hosted apps use the optional restic-apps server and their own credentials (p0b-09d). No per-volume repo PV or PVC. The VolSync pod’s TZ is time_zone (D15). platform-backups drops privileged-movers once no mover there mounts a hostPath. etcd snapshots stay in backup_path/etcd-snapshots.
Done checks: The rest-server is Ready on the backup node. A scratch volume, applied through Flux like the p0b-09 test and removed afterward, backs up through it and restores into an empty PVC with a matching checksum; the log is in the PR. One PR.
```

```text
ID:          p0b-09d
Goal:        Give core one restic credential for all core backups, and each hosted app namespace with data its own restic credential and private repository.
Why:         Core is one trusted zone; hosted apps must not read or harm core or the chat bot’s data (D51, R31).
Advances:    P0b exit check: “Restore drill: a deleted volume is restored by make restore from its rest-server backup and its checksum matches; an etcd snapshot exists for same-node rollback and comes back from its restic copy; verify reports the off-host copy’s freshness and that no app credential opens a core repository (R31, D48, D51).”
Depends on:  p0b-09a
Touches:     platform/backups/, tools/bootstrap-secrets, envs/<env>/secrets/<namespace>/, platform/backups/CONTRACT.md
Hard limits: R25; R31. D47; D51. Core namespaces and the host-etcd-snapshots repository use the one core restic password and storage login. Each app-<name> namespace with data gets its own restic credential and private repository, which cannot open core’s or another app’s. Apps use a second server, restic-apps, with --private-repos; it is core, not an add-on, and off by default (one line per env turns it on). Core’s check and off-host copy may hold app passwords. App movers run privileged for now. An app login may forget and prune its own repositories for now; append-only servers with core-run pruning come later. Secrets live under envs/<env>/secrets/<namespace>/; only flux-system decrypts (D53). The core credential lands before the off-host copy (p0b-09e); an app’s credential lands before that app stores data.
Done checks: A scratch PVC in an app-<name> namespace cannot open a core repository. One PR.
```

```text
ID:          p0b-09b
Goal:        Copy the k3s etcd snapshots into their own restic repository on the rest-server.
Why:         The etcd snapshots hold the cluster’s state; a restic copy adds retention, dedup and the same later NFS path and off-host copy as the volumes (R31, D47, D48, p0b-09e).
Advances:    P0b exit check: “Restore drill: a deleted volume is restored by make restore from its rest-server backup and its checksum matches; an etcd snapshot exists for same-node rollback and comes back from its restic copy; verify reports the off-host copy’s freshness and that no app credential opens a core repository (R31, D48, D51).”
Depends on:  p0b-09d, p0a-12
Touches:     platform/backups/etcd-snapshots.yaml, platform/backups/CONTRACT.md
Hard limits: R5; R9; R31; R33. D6; D15; D17; D42; D47. k3s keeps writing its snapshots to backup_path/etcd-snapshots; the k3s config does not change. One CronJob in platform-backups, pinned to role.jarvis/control_plane=true where the snapshots are written, mounts that directory read-only and runs restic backup into a host-scope repository, initializing it if missing. The repository is host-etcd-snapshots, one segment that is not a rest-server reserved name. It uses the host-scope credential (D51) and the cluster-vars retention (forget and prune on each run; the repository is small). Its timeZone is time_zone, and it runs shortly after each k3s snapshot (D6), one run at a time. Restore is owner-run (D17) and written in the backups CONTRACT.md: restic restore of the chosen snapshot file onto the server node, then k3s’s cluster reset with --cluster-reset-restore-path. While the rest-server’s data sits on the same disk, this adds retention and dedup, not disk-failure protection; the off-host copy is that path (D48, p0b-09e).
Done checks: A Job created from the CronJob completes, and restic lists a snapshot in that repository holding the newest k3s snapshot file. One PR.
```

```text
ID:          p0b-09c
Goal:        Install Kyverno and the backup policy that enrolls every PVC, and change the R31 check to match.
Why:         A new app has to be backed up without wiring by hand (R31, D47).
Advances:    P0b exit check: “Restore drill: a deleted volume is restored by make restore from its rest-server backup and its checksum matches; an etcd snapshot exists for same-node rollback and comes back from its restic copy; verify reports the off-host copy’s freshness and that no app credential opens a core repository (R31, D48, D51).”
Depends on:  p0b-09d
Touches:     platform/kyverno/, platform/flux-operator/kustomization.yaml (controller list, until p0b-04 moves it), platform/backups/enrollment.yaml, schema/env.py and schema/CONTRACT.md (backup_schedule), platform/backups/CONTRACT.md, ci/checks/pvc-backup
Hard limits: R5; R9; R31. D15; D30; D47. The Kyverno 1.19 chart, classic ClusterPolicy only, reports and cleanup controllers off. The project tests it only up to Kubernetes 1.35, though it builds against the 1.36 libraries, so the first gate is the admission controller Ready; if it is not, stop and report (the rest-server stands). The policy sits on the platform-configs path, after the CRDs, with failurePolicy Ignore, generateExisting and synchronize. It enrolls a PVC only once it is bound (spec.volumeName set): an unbound WaitForFirstConsumer PVC has no volume to back up, and VolSync starts the mover as soon as the source exists, so enrolling earlier races two restic inits on the new repository. For each PVC not annotated backup: none and not labeled app.kubernetes.io/created-by: volsync (VolSync’s own cache and destination claims), it generates in the PVC’s namespace a Secret with that PVC’s repository and that namespace’s credentials (p0b-09d, D51) and a ReplicationSource: Direct, the backup schedule (backup_schedule) and retention from cluster-vars, pruneIntervalDays 7, and a small cacheCapacity. It never sets trigger.manual: VolSync ignores the schedule while a manual trigger is set, so every PVC would back up once and stop (on-demand backup is p2-20). A mutate-existing rule sets privileged-movers on that namespace; it lands asynchronously, so a first sync may fail once and pass on retry. Kyverno’s controllers get aggregated ClusterRoles for Secrets and ReplicationSources. StatefulSet claims need no per-ordinal object. ci/checks/pvc-backup now checks that the policy is present, that backup: none appears only under the p0b-02 allow-list, and that no ReplicationSource is written in Git (it would fight synchronize). It also fails when any file under services/ or apps/ hand-writes a VolSync ReplicationSource or ReplicationDestination, or a restic repository URL: the policy owns those, so they are never copied per service. The p0b-10 restore commands create their ReplicationDestinations at run time, not in Git, so they are unaffected. The policy acts on live PVCs, so PVCs a HelmRelease renders are covered too.
Done checks: Kyverno admission is Ready. A scratch PVC gets its Secret and ReplicationSource with no per-PVC manifest, and the source completes a sync; a PVC annotated backup: none gets neither. Seeded PRs fail ci: backup: none outside the allow-list, a hand-written ReplicationSource, a services/ file with a hand-written ReplicationSource, and the policy removed. One PR.
```

```text
ID:          p0b-09e
Goal:        Add the nightly restic copy of every repository to the env’s off-host target, and have verify report the copy’s freshness.
Why:         Same-disk backups cover mistakes, not disk loss (D48, R31). This lands before the first real data volume (p2-09).
Advances:    P0b exit check: “Restore drill: a deleted volume is restored by make restore from its rest-server backup and its checksum matches; an etcd snapshot exists for same-node rollback and comes back from its restic copy; verify reports the off-host copy’s freshness and that no app credential opens a core repository (R31, D48, D51).”
Depends on:  p0b-09d, p0b-09b, p0b-09c
Touches:     platform/backups/, schema/ (off-host target), tools/verify/, platform/backups/CONTRACT.md
Hard limits: R31. D48; D51. A nightly restic copy of every repository (each volume and the etcd copy) to a per-env off-host target, any restic backend. Core repositories are copied with the core credential; an app’s repositories are copied with that app’s own credential. The target is env config; core stays bastion-agnostic. env-01’s target is, for now, a temporary test target on the bastion that Operator sets up with its own user and append-only access (add, never delete). The bastion copy covers a dead disk, not loss of the site. The off-host password is on the keep-safe list. A scheduled restic check reads part of every repository; verify reports the copy’s freshness and the check’s freshness. A scheduled job younger than one interval is a warning (D17).
Done checks: verify reports the off-host copy’s freshness. A Job using an app’s credential cannot open a core off-host repository. One PR.
```

```text
ID:          p0b-10
Goal:        Add make restore and make restore-all, run the checksum restore drill with them, and restore an etcd snapshot file from restic.
Why:         P0b has to prove a deleted volume comes back with the same checksum through the command a rebuild uses, that an etcd snapshot exists for same-node rollback and comes back from its restic copy, and that the off-host copy and the core and app backup credentials stand (R31, D48, D51).
Advances:    P0b exit check: “Restore drill: a deleted volume is restored by make restore from its rest-server backup and its checksum matches; an etcd snapshot exists for same-node rollback and comes back from its restic copy; verify reports the off-host copy’s freshness and that no app credential opens a core repository (R31, D48, D51).”
Depends on:  p0b-09b, p0b-09c, p0b-09e, p0a-12
Touches:     Makefile, tools/restore, platform/backups/enrollment.yaml (pause), platform/kyverno/rbac.yaml (hold read), infra/ansible/roles/flux-operator/ (rebuild hold), platform/backups/CONTRACT.md
Hard limits: R31. D47. The etcd half restores the newest snapshot file from its restic repository (p0b-09b) into a scratch pod and compares its checksum with the file on the backup path. It never runs the cluster reset on env-01, which would restart the only server; that step stays the owner’s written procedure. make restore ENV=<env> APP=<app> suspends the app’s Flux Kustomization, scales its workloads to 0, and for each backed-up PVC applies one ReplicationDestination into the app’s own PVC (Direct, trigger.manual, restoreAsOf, enableFileDeletion), waits for lastManualSync, deletes it, then resumes. make restore-all ENV=<env> runs it for every app with a backed-up PVC. restoreAsOf is a time before the loss, by default just before the PVC’s creation time, with an override: after a rebuild the new source backs up the fresh, empty app first, and the latest snapshot would restore that. No backup runs on a PVC while it is being restored. make restore ENV=<env> APP=<app> fails when a PVC has no eligible snapshot. make restore-all ENV=<env> fails on any selected PVC with no eligible snapshot unless ACCEPT_EMPTY names that PVC. This holds with or without a rebuild hold, including for a PVC that is not in the hold. It never silently skips a PVC. ACCEPT_EMPTY and RETIRE are only valid with make restore-all ENV=<env>. ACCEPT_EMPTY names a selected PVC that is accepted empty. RETIRE names a held repository that has no PVC. Rebuild hold: when site.yml’s flux-operator role finds no Flux root source and restic repositories already under backup_path/restic, it writes ConfigMap kube-system/backup-rebuild-hold listing them; the pvc-backup policy keeps every generated ReplicationSource paused while the hold exists (a missing hold is not paused). make restore-all ENV=<env> lifts the hold only when every repository in the hold is restored, accepted empty, or retired by name. One make restore ENV=<env> never lifts it. A run that leaves any listed repository unaccounted for keeps the hold. Reason: fresh empty volumes must not back up and push the good snapshots out. Re-running site.yml on a live cluster never sets the hold. Generated sources also set retain.within: 7d, which only delays that loss (keep-within counts from the newest snapshot), so it does not replace the hold. Kyverno generate rules can’t be edited in place, so the policy carries Flux’s force annotation and a change recreates every generated Secret and ReplicationSource; a policy change must not merge during a restore. No dataSourceRef (D47). make drill ENV=<env> runs the restore drill in the repo: a scratch volume, checksum before and after. It does not parse a transcript.
Done checks: CI passes. The operator attaches make drill ENV=<env>. The log shows the original checksum, the delete, the make restore ENV=<env> run, and the matching checksum, plus an etcd snapshot filename on the backup path and the same file restored from restic with a matching checksum. make restore ENV=<env> of a PVC with no eligible snapshot fails. A PVC with no snapshot and no ACCEPT_EMPTY makes make restore-all ENV=<env> fail, with or without a rebuild hold, including when that PVC is not in the hold. make restore-all ENV=<env> lifts the hold only when every listed repository is restored, accepted empty (ACCEPT_EMPTY names that PVC), or retired by name (RETIRE names a held repository with no PVC). A run that leaves any listed repository unaccounted for keeps the hold, and either flag on make restore fails (ci/tests/test_restore.py). The drill volume is removed afterward. One PR.
```

```text
ID:          p0b-11
Goal:        Extend make verify ENV= with the P0b standing checks and attach that run. The restore drill and the [G] tailnet line are separate evidence.
Why:         Done for the checkpoint is the attached verify output (§2, R35).
Advances:    P0b exit check: “A test app answers over HTTPS on <svc>.<DNS suffix> (env-01: <svc>.alpha.lan) with a cert from the internal CA, from LAN and from tailnet [G], through Traefik Ingress.”
Advances:    P0b exit check: “Restore drill: a deleted volume is restored by make restore from its rest-server backup and its checksum matches; an etcd snapshot exists for same-node rollback and comes back from its restic copy; verify reports the off-host copy’s freshness and that no app credential opens a core repository (R31, D48, D51).”
Depends on:  p0b-08, p0b-10, p0b-07
Touches:     Makefile, tools/verify/
Hard limits: R28; R35. D17; D48; D51. Router forwarding of alpha.lan and the trusted root cert are Gordon’s steps on p0b-08. Nothing for Tailscale is installed on the node. make verify ENV=<env> is per component, runs only what the env enables, and reports PASS, WARN or FAIL (a scheduled job younger than one interval is a warning). It does not read a drill transcript and does not re-run the drill. make drill ENV=<env> (p0b-10) is attached as its own run. Its TLS checks trust the committed envs/<env>/ca.crt explicitly (curl --cacert, openssl -CAfile), not the controller’s system trust store. Standing checks include the off-host copy’s freshness and that no app credential opens a core repository.
Done checks: CI passes. The operator attaches make verify ENV=<env> output, exit 0 with the LAN curl, the cert issuer, the off-host copy’s freshness, and the app-credential check, and attaches make drill ENV=<env> as its own run. Owner checklist, pasted outside the script: Gordon’s tailnet [G] paste. One PR.
```

```text
ID:          p0b-12
Goal:        Close P0b’s loose ends: finish the prune guards, make the version tests Renovate-proof, confirm etcd snapshot rotation, and explain any missing privileged-movers opt-in.
Why:         Guards and backups have to hold on every object they protect before real data lands (R1, R31).
Advances:    P0b exit check: “Restore drill: a deleted volume is restored by make restore from its rest-server backup and its checksum matches; an etcd snapshot exists for same-node rollback and comes back from its restic copy; verify reports the off-host copy’s freshness and that no app credential opens a core repository (R31, D48, D51).”
Depends on:  p0b-11
Touches:     platform/flux-operator/, platform/kyverno/rbac.yaml, envs/<env>/secrets/<namespace>/, ci/tests/test_prune_guards.py, ci/tests/test_backup_enrollment.py, ci/tests/test_cert_manager.py, ci/tests/test_app_dns.py, platform/backups/enrollment.yaml
Hard limits: R1; R25; R31. D17; D50. Add prune: disabled to the flux-operator HelmRelease and its OCIRepository, the Kyverno ClusterRoles and each namespace restic Secret, and list them in ci/tests/test_prune_guards.py. Tests check that a chart or image is pinned (an image by digest), not which version it is, so a Renovate bump passes CI (the Kyverno image tag regex, the cert-manager and k8s-gateway chart versions). The Secret’s annotation sits in its SOPS file’s plain metadata, but the file’s MAC needs the age key, so that edit is an owner run. If an enrolled PVC’s namespace lacks volsync.backube/privileged-movers, fix the policy’s mutate rule, not the namespace. Both age-key copies are proven by the pass/fail decrypt command (D50). The separate worker user is deferred (framework §9) and is not checked here.
Done checks: The guard test lists the four new objects and passes. None of those three tests asserts a literal version. Post-merge operator checks: the live objects show the annotation, k3s holds no more etcd snapshot files than its retention after a few snapshot cycles, and every enrolled namespace has the opt-in. One PR.
```

```text
ID:          p0b-13
Goal:        Move env-01 to gordoncooper/jarvis-internal and rebuild env-01 fresh from the instance repo.
Why:         Instance data must live in the private instance repo before the chat-bot phase writes it (D60). The move and the rebuild are one large change; the signed-tag gate is its own later step (D68).
Advances:    P0b exit check: “Restore drill: a deleted volume is restored by make restore from its rest-server backup and its checksum matches; an etcd snapshot exists for same-node rollback and comes back from its restic copy; verify reports the off-host copy’s freshness and that no app credential opens a core repository (R31, D48, D51).”
Depends on:  p0b-12
Touches:     infra/ansible/roles/flux-operator, the env-01 instance repo
Hard limits: R1; R25; R34. D49; D51; D57; D60; D68. Flux reads the instance root with one read-only deploy key (D51) and core with no credential, both at main, with no signature check; the gate is p2-08a. env-01 is rebuilt fresh from the instance repo, not migrated in place. The rebuild is the D49 rehearsal: a k3s uninstall and reinstall with the OS and /var/lib/jarvis kept. The new cluster starts from an empty etcd datastore and restores no snapshot, and Git rebuilds everything. The CA is minted fresh with name constraints (D57). The agent token is rotated to its own value. The sealed server token is kept in the short, passphrase-only form: a K10<ca-hash>::server:<passphrase> token carries the old CA’s hash and fails on a fresh CA. The passphrase is unchanged, so older etcd snapshots stay restorable. The restic password, the rest-server login and the off-host credentials move unchanged. bench is test-only; its data disk holds only test data and is never wiped without the owner’s explicit say-so. After this step, later-phase instance writes (p2-07, p2-10, P4a apps.yaml) go to the instance repo.
Done checks: Both GitRepositories are Ready at main. env-01 is Ready from the instance repo after the rebuild, started from an empty etcd datastore; that rebuild is the dead-node rehearsal (owner’s go, OS and /var/lib/jarvis kept, D49). The sealed server token is the short form, and the fresh CA carries the name constraint. One PR.
```

```text
ID:          p0b-13a
Goal:        Rehearse the restic-password rotation runbook once on the rebuilt env-01.
Why:         The rebuild moves the restic password unchanged (D60), so rotation is not rehearsed there; the runbook must work before real data depends on it (R31, D50, D57).
Advances:    none fits: no P0b exit check names this rehearsal (see Stop and ask).
Depends on:  p0b-13
Touches:     the restic-rotation runbook, the env-01 instance repo secrets/<namespace>/
Hard limits: R25; R31. D17; D48; D50; D51; D57. The rotation needs the age key, so it is an owner run. Tools refuse paths that would strand backups. The value is never printed. The data path is never wiped.
Done checks: The runbook has been run once on env-01. Afterwards make verify ENV=env-01 passes, the off-host copy’s freshness reports current, and a volume restore from a backup made before the rotation matches its checksum. One PR, plus the owner run.
```

### Exit checks

- A test app answers over HTTPS on <svc>.<DNS suffix> (env-01: <svc>.alpha.lan) with a cert from the internal CA, from LAN and from tailnet [G], through Traefik Ingress.
- Restore drill: a deleted volume is restored by make restore from its rest-server backup and its checksum matches; an etcd snapshot exists for same-node rollback and comes back from its restic copy; verify reports the off-host copy’s freshness and that no app credential opens a core repository (R31, D48, D51).

Done = attached make verify ENV=<env> output, plus this phase’s drills and evals as their own runs

## P1

Goal: Gordon sees the system and gets alerts without Hermes.

P1 builds the observability component (D69). Its tasks apply to an env that enables it; core’s own health is make verify.

### Contracts

```text
ID:          p1-01
Goal:        Write the Login and Alert path sections of the observability CONTRACT.md with proven by.
Why:         Grafana’s built-in login and the send-only alert path are one contract, written before either is built (R8, R27, §5).
Advances:    P1 exit check: “Grafana shows node, pod and disk metrics, and logs for every namespace.”
Advances:    P1 exit check: “Killing a test endpoint sends a Telegram alert within 5 minutes; the disk, node-memory and clock-not-synchronised alert rules fire in a test with thresholds lowered.”
Advances:    P1 exit check: “A forced backup failure (VolSync or the etcd-snapshot CronJob) sends a Telegram alert.”
Depends on:  p0b-13
Touches:     services/observability/CONTRACT.md
Hard limits: R8; R18 (logins work offline: no external IdP); R27. D7; D16. The Hermes dashboard half is added in p2-05. Multi-user OIDC stays the §9 seam. The alert list is disk full, node memory above 80%, node not ready, clock not synchronised, backup failures, and bot cron failures. Bot cron alerts are delivered in P3 (p3-07); the contract names them now. Egress to api.telegram.org is required. The api.x.ai probe is added in P2 (R18). Before observability lands, move the core `backup: none` entries for observability volumes out of core and into an add-on contract field, so core names no add-on.
Done checks: The Login section and the Alert path section each have Offers, Expects, Health, Config, and Proven by, naming checks without plan codes. One PR.
Proven by:   The R27 CI check and the dashboard-login check (§5). Grafana anonymous-off is one half. The kill-endpoint alert, the lowered-threshold alerts, and the backup-failure alert.
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
```

```text
ID:          p1-08
Goal:        Deploy Gatus with a check for every exposed endpoint, the public CA root, and the login-free status page.
Why:         The status page is the NOC surface that loads without a login on the LAN and the tailnet (R5, R27).
Advances:    P1 exit check: “Gatus checks every exposed endpoint; its status page (env-01: gatus.alpha.lan) loads without login on LAN and on tailnet [G].”
Depends on:  p1-07, p0b-08
Touches:     services/observability/gatus/, platform/traefik/CONTRACT.md
Hard limits: R5; R8; R27; R28. D12; D13; D30. Gatus resolves alpha.lan through the coredns-custom forward and trusts the committed public root (§5). Checks include testapp, grafana, and gatus, plus TCP/53 for App DNS. The exposure contract’s Gatus check is no longer marked planned.
Done checks: https://gatus.alpha.lan loads with no login from the LAN and lists those checks. The tailnet load is an owner checklist item (D17). One PR.
```

```text
ID:          p1-09
Goal:        Point Gatus and Grafana at the send-only Telegram alerts bot.
Why:         Alert delivery has to work while Hermes is absent (R8).
Advances:    P1 exit check: “Killing a test endpoint sends a Telegram alert within 5 minutes; the disk, node-memory and clock-not-synchronised alert rules fire in a test with thresholds lowered.”
Depends on:  p1-01, p1-08
Touches:     services/observability/ network policy, Gatus provider, Grafana contact point, SOPS secret
Hard limits: R8. D39 (egress; api.telegram.org listed as documentation, from observability only). The bot token and chat ID are the SOPS values Gordon records after /start. The alerts bot has no gateway and no allow-list (R21).
Done checks: A Grafana test notification returns success from the Telegram API. The token value is not printed. One PR.
```

```text
ID:          p1-10
Goal:        Add the disk, node-memory, node-not-ready, clock, and backup-failure alert rules, and fire them with the thresholds lowered.
Why:         The P1 check requires those rules to fire in a test, at the §7 thresholds in normal operation (R8).
Advances:    P1 exit check: “Killing a test endpoint sends a Telegram alert within 5 minutes; the disk, node-memory and clock-not-synchronised alert rules fire in a test with thresholds lowered.”
Advances:    P1 exit check: “A forced backup failure (VolSync or the etcd-snapshot CronJob) sends a Telegram alert.”
Depends on:  p1-09
Touches:     services/observability/grafana/alerts/
Hard limits: R8; §7 (disk at 80% full, node memory at 80%). D14. The lowered-threshold test is reverted at the end of the run. Backup failure covers the mechanism p0b-09 kept and the etcd-snapshot CronJob (p0b-09b). It is a warning, not a page: it goes through the same alerts bot, once per failure with a long repeat interval, so a missed nightly run informs rather than demands action; the next good run clears it. Every alert goes to the alerts bot’s Telegram chat.
Done checks: The verify log shows each rule firing under the lowered threshold and the committed rules back at 80%. One PR.
```

```text
ID:          p1-11
Goal:        Kill the test endpoint and force a backup failure; both send a Telegram alert.
Why:         These are the two live alert proofs that do not depend on Hermes (R8).
Advances:    P1 exit check: “Killing a test endpoint sends a Telegram alert within 5 minutes; the disk, node-memory and clock-not-synchronised alert rules fire in a test with thresholds lowered.”
Advances:    P1 exit check: “A forced backup failure (VolSync or the etcd-snapshot CronJob) sends a Telegram alert.”
Depends on:  p1-10
Touches:     platform/testapp scale
Hard limits: R8. D12; D13. Restore the test app and the backup path when the check finishes. The drill reads send success and does not print the bot token. It is attached as its own run.
Done checks: The log shows the testapp alert arriving within 5 minutes and the backup-failure alert arriving. testapp is Ready again afterward. One PR.
```

```text
ID:          p1-12
Goal:        Add the R27 check that Grafana anonymous access is off.
Why:         Fix order (§11): Grafana is already configured (p1-07); CI keeps a later edit from turning anonymous access on (R27).
Advances:    P1 exit check: “Grafana shows node, pod and disk metrics, and logs for every namespace.”
Depends on:  p1-07
Touches:     ci/checks/ui-auth
Hard limits: R27. The Hermes dashboard half of this check is added in p2-23, when that dashboard exists.
Done checks: A seeded Grafana manifest with anonymous access on fails ci. The live manifest passes. One PR.
```

```text
ID:          p1-13
Goal:        Extend make verify ENV= with the P1 standing checks, and attach the alert drills as their own runs.
Why:         Done for the checkpoint is the attached verify output (§2, R35).
Advances:    P1 exit check: “Grafana shows node, pod and disk metrics, and logs for every namespace.”
Advances:    P1 exit check: “Gatus checks every exposed endpoint; its status page (env-01: gatus.alpha.lan) loads without login on LAN and on tailnet [G].”
Advances:    P1 exit check: “Killing a test endpoint sends a Telegram alert within 5 minutes; the disk, node-memory and clock-not-synchronised alert rules fire in a test with thresholds lowered.”
Advances:    P1 exit check: “A forced backup failure (VolSync or the etcd-snapshot CronJob) sends a Telegram alert.”
Depends on:  p1-11, p1-12, p1-08
Touches:     services/observability/verify.py (the observability verify component)
Hard limits: R8; R35. D17; D69 (the component runs only when the env enables observability). The tailnet Gatus load is the [G] line.
Done checks: make verify ENV=<env> exits 0 with dashboard evidence and the Gatus page. The alert proofs are attached as their own drills. Owner checklist, pasted outside the script: Gordon’s tailnet [G] paste. One PR.
```

### Exit checks

- Grafana shows node, pod and disk metrics, and logs for every namespace.
- Gatus checks every exposed endpoint; its status page (env-01: gatus.alpha.lan) loads without login on LAN and on tailnet [G].
- Killing a test endpoint sends a Telegram alert within 5 minutes; the disk, node-memory and clock-not-synchronised alert rules fire in a test with thresholds lowered.
- A forced backup failure (VolSync or the etcd-snapshot CronJob) sends a Telegram alert.

Done = attached make verify ENV=<env> output, plus this phase’s drills and evals as their own runs

## P2

Goal: Gordon chats with the ops bots through the dashboard and Telegram, with memory and evals.

### Contracts

```text
ID:          p2-01
Goal:        Write the Model provider, Assistant API, and Skill contract sections of the chat-bot slot CONTRACT.md with proven by, and the Hermes core CONTRACT.md that says how Hermes meets them.
Why:         Tiers, the assistant API, and the skill rules are one slot contract, written before any bot is deployed, so core and other add-ons depend on the slot and never on Hermes by name (R15, R17, R18, R23, §5, D69).
Advances:    P2 exit check: “With the primary model blocked, chat answers from the fallback model; with the provider blocked, the dashboard and Telegram show an error within 30 s and Gatus flags it (R18).”
Advances:    P2 exit check: “A scripted prompt through the assistant API (/api/<profile>/v1) to each of the three bots gets a non-error answer; a Telegram message from Gordon’s ID to concierge gets a reply [G], and one from another ID gets none.”
Advances:    P2 exit check: “Memory: a fact is recalled after a pod restart in a new session; “forget that” is staged, and approving it removes the fact; a memory export produces a file. Memory and skill writes (R12) wait for approval; a test approves one and rejects one.”
Advances:    P2 exit check: “Weather and distance questions get correct answers through the geo MCP server (weather: Open-Meteo; distance: Nominatim plus the OSRM demo, typical drive time) with attribution; the geo tool’s evals in concierge pass.”
Advances:    P3 exit check: “An eval asserts no bot can send, delete, apply Gmail labels, write to calendar or reach Drive.”
Depends on:  p1-13
Touches:     services/slots/chat-bot/CONTRACT.md, services/hermes/core/CONTRACT.md
Hard limits: R13; R15; R16; R17; R18; R23; R27. D7; D66; D69. The three sections live in services/slots/chat-bot/CONTRACT.md (hosting, storage, backups, secrets and URL/ingress are the slot’s other sections); services/hermes/core/CONTRACT.md holds only how Hermes meets the slot and no model-provider, assistant-API or skill-contract text. The profile schema and the deploy contracts stay with Hermes. Hermes’s model login is a swappable provider (subscription login, a local model, or an API key); env-01’s first login is xai-oauth. A local LLM or a gateway is another base_url and is not built (§9). Paths are / for the dashboard and /api/<profile>/v1 per profile. /v1/audio/* is reserved and this task adds no audio handler; the GPU runtime and HTTPS path land now so a first voice version over HTTPS fits later (D62). Memory export produces a file. MCP servers run inside the deploy pod, pinned, no sidecars. Write tools run only behind a confirm step. This plan’s writes are inbox drafts and spec-writer’s issues. The geo section records the Nominatim policy decision: at most 1 request per second, cache, User-Agent with Gordon’s contact email, attribution, home taken from SOPS and never looked up.
Done checks: The three sections each have Offers, Expects, Health, Config, and Proven by, naming checks without plan codes, including the Nominatim decision. One PR.
Proven by:   The fallback check, Gatus provider reachability, and the R23 CI grep (§5). The scripted-prompt check and the memory-export check (§5). The R16 pin and tag check, the R13 toolset policy check, and the no-write eval (§5).
```

```text
ID:          p2-04
Goal:        Write the Profile schema CONTRACT.md, its own file, with proven by.
Why:         A bot is a profile.yaml, validated before the three profiles are added (R19, §5).
Advances:    P2 exit check: “A scripted prompt through the assistant API (/api/<profile>/v1) to each of the three bots gets a non-error answer; a Telegram message from Gordon’s ID to concierge gets a reply [G], and one from another ID gets none.”
Depends on:  p2-01, p0a-01
Touches:     services/hermes/core/schema/CONTRACT.md, schema/CONTRACT.md
Hard limits: R19; R21; R23. Fields: job, toolsets with reasons and read/write tags, cron, evals, telegram true/false with allow-list required when true. Names that collide with status, cron, memory, or skills are rejected. Tiers come from the env bot tier map, not from the profile. This file is the Hermes schema CONTRACT; schema/CONTRACT.md no longer marks it planned.
Done checks: The profile schema CONTRACT.md has Offers, Expects, Health, Config, and Proven by, naming checks without plan codes. One PR.
Proven by:   CI validation of profile.yaml (§5) and the three bots loading.
```

```text
ID:          p2-05
Goal:        Add the Hermes dashboard half of the Login contract, with proven by.
Why:         The ops dashboard requires its own login, and that login works offline (R18, R27).
Advances:    P2 exit check: “The Hermes ops dashboard (hermes-ops.<DNS suffix>) shows up in Gatus (R5) and requires its login (R27).”
Depends on:  p2-01, p1-01
Touches:     services/observability/CONTRACT.md, services/hermes/core/CONTRACT.md (how Hermes meets the slot’s URL/ingress and login terms)
Hard limits: R18; R27. D7; D16; D69. Username admin, password hash and signing secret from bootstrap-secrets. The assistant API uses the deploy API key, separate from the dashboard login.
Done checks: The Login contract names both UIs and the Hermes proven-by line. One PR.
Proven by:   The R27 CI check and the dashboard-login check (§5).
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
```

```text
ID:          p2-07
Goal:        Re-add hermes.yaml in the instance repo, add the render script, extend the stale-render test in ci/tests/test_flux_entry.py to it, and seed a stale render.
Why:         Hermes is the consumer that re-adds hermes.yaml (D54). One checked-in script renders profiles, and a hand-edited rendered/ must fail (R4).
Advances:    P2 exit check: “admin-gate is the PR-risk check (R36); branch protection is optional and is not the deploy gate (D56): seeded PRs that change a protected path or remove a Namespace fail it without the label and when admin-approved comes from gordon5417-ai or a fixture Bot/App actor, and pass once Gordon’s own account adds it after the latest commit; a seeded PR fails the render check (a stale Hermes render, R4).”
Advances:    P0a exit check: “One required check on main, ci, runs every ci/checks/ rule check (admin-gate joins at the P2 exit, R36), and a seeded bad PR fails each rule check as it lands (Examples: a hardcoded DNS suffix, a plaintext secret, a missing CONTRACT.md, a missing flux_git_repo_url, a deny list missing a whole deny class).”
Depends on:  p2-06b, p0a-05, p0b-13
Touches:     services/hermes/tools/render-profiles, services/hermes/Makefile, the env-01 instance repo (values/hermes.yaml, rendered/), the stale-render test in ci/tests/test_flux_entry.py (adds the Hermes render)
Hard limits: R4. D54; D60; D69 (the render script belongs to the Hermes add-on, not to core tools/ or the root Makefile). hermes.yaml is re-added in the instance repo. rendered/ is generated and never hand-edited. The stale-render seed lands with this render. This step adds validation of the hermes.* keys it renders (D45).
Done checks: A fresh render matches rendered/ in the instance repo. A seeded PR that edits rendered/ without rerendering fails ci. One PR.
```

```text
ID:          p2-08
Goal:        Validate profile.yaml in CI against the profile schema.
Why:         Fix order (§11): the schema is the boundary (p2-04); CI rejects a profile that breaks it (R19).
Advances:    P2 exit check: “A scripted prompt through the assistant API (/api/<profile>/v1) to each of the three bots gets a non-error answer; a Telegram message from Gordon’s ID to concierge gets a reply [G], and one from another ID gets none.”
Depends on:  p2-04, p0a-06
Touches:     services/hermes/core/schema/, ci/checks/profile-schema
Hard limits: R19; R21. A telegram: true profile without an allow-list fails. A bot name of memory, skills, status, or cron fails.
Done checks: pytest covers one valid profile and one colliding name. A seeded bad profile fails ci. One PR.
```

```text
ID:          p2-08a
Goal:        Turn on Flux signature checks on owner-signed tags (mode Tag) for the root sync and the named core source.
Why:         Nothing may deploy unsigned once the chat bot holds Gordon’s real data; the env-01 move deployed from main (R25, R36, D56, D68).
Advances:    none fits: no P2 exit check names the deploy gate (see Stop and ask).
Depends on:  p0b-13
Touches:     infra/ansible/roles/flux-operator, the env-01 instance repo, Flux GitRepository verify
Hard limits: R25; R34; R36. D56; D68. The whole gate (both sources, mode Tag, trusting only Gordon’s keys) lands in this step, after the foundation build and before the first task that stores Gordon’s real data (p2-09). Coding agents never create or push tags. The signing key’s public fingerprint and where the key lives go on the keep-safe list (D56).
Done checks: Flux verifies both GitRepositories. A commit on main without a signed tag does not deploy. One PR, plus the owner’s first signed tags.
```

```text
ID:          p2-09
Goal:        Deploy the ops overlay: its own volume, secrets, read-only ServiceAccount, network policy, Ingress, and dashboard login.
Why:         Gordon reaches the three bots through the ops dashboard, which Gatus watches and which requires a login (R11, R13, R27).
Advances:    P2 exit check: “A scripted prompt through the assistant API (/api/<profile>/v1) to each of the three bots gets a non-error answer; a Telegram message from Gordon’s ID to concierge gets a reply [G], and one from another ID gets none.”
Advances:    P2 exit check: “The Hermes ops dashboard (hermes-ops.<DNS suffix>) shows up in Gatus (R5) and requires its login (R27).”
Depends on:  p2-06b, p2-05, p1-08, p0b-09e, p0b-13, p2-08a
Touches:     services/hermes/deploy/ops/, the env-01 instance repo secrets/<namespace>/
Hard limits: R5; R11; R13; R27; R28. D30; D39 (egress; hosts in the ops deploy’s CONTRACT.md as documentation); D48; D51; D60. Namespace hermes-ops. No shared volume or secret with the other deploy. ServiceAccount: get, list, watch, no Secrets. The pod sets TZ from cluster-vars. The API key, password hash, and signing secret are the bootstrap Secrets in the instance repo, applied now that the namespace exists (D10). Its volume is the first real data PVC with a backup (R31); the core backup credential and the off-host copy already exist. The backup policy enrolls it, so the deploy carries no ReplicationSource, restic Secret or mover annotation (D47). A live SQLite copy can restore torn, so a consistent copy is on the volume before the nightly backup (Example: a dump written just before the backup hour). Hostname hermes-ops.alpha.lan. Before Hermes holds real data, run the add-on disable test live on a scratch env as an owner run: enable an add-on, disable it, confirm its Kustomizations are gone and its namespace, tenant account and secrets stay until deleted by hand.
Done checks: The pod is Ready, https://hermes-ops.alpha.lan asks for login, and Gatus lists the endpoint. One PR.
```

```text
ID:          p2-10
Goal:        Add concierge, inbox, and homelab from the profile template, with generic SOUL.md files.
Why:         The starter bots are code, and only concierge faces Telegram (R19, R21).
Advances:    P2 exit check: “A scripted prompt through the assistant API (/api/<profile>/v1) to each of the three bots gets a non-error answer; a Telegram message from Gordon’s ID to concierge gets a reply [G], and one from another ID gets none.”
Depends on:  p2-07, p2-08, p2-09
Touches:     services/hermes/deploy/ops/profiles/{concierge,inbox,homelab}/, the env-01 instance repo values/hermes.yaml
Hard limits: R13; R15; R19; R21. D60. No placeholder bots. SOUL.md has no personal facts. inbox has the fewest tools at this checkpoint (no Gmail until P3). homelab proposes fixes only as chat. Brief crons are not listed yet, so they are absent rather than running. Built-in reminders stay available for the concierge reminder seed. Bot list writes go to the instance repo.
Done checks: Three profile.yaml files validate. The instance hermes.yaml lists the three bots. A rendered diff is clean. One PR.
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
```

```text
ID:          p2-12
Goal:        Pin kubernetes-mcp-server read-only and pin mcp-victoriametrics for homelab metric reads.
Why:         homelab has to read the cluster, Flux, and metrics, and has to answer the memory question from the appendix. §4 names mcp-victoriametrics for those reads (R13, R16).
Advances:    P2 exit check: “A scripted prompt through the assistant API (/api/<profile>/v1) to each of the three bots gets a non-error answer; a Telegram message from Gordon’s ID to concierge gets a reply [G], and one from another ID gets none.”
Depends on:  p2-10, p1-05
Touches:     ops deploy volume, ops profile toolsets, the pinned kubernetes-mcp-server and mcp-victoriametrics binaries
Hard limits: R13; R15; R16. D29; D69 (the metric reads apply when the env enables observability). The Kubernetes MCP is read-only and cannot read Secrets. mcp-victoriametrics is the official VictoriaMetrics MCP server, read-only, a pinned release binary on the deploy volume, and it queries VictoriaMetrics only. homelab’s written reason for these tools is in profile.yaml. No terminal tool. No in-repo metrics server.
Done checks: The deploy volume holds the pinned mcp-victoriametrics binary, and a fixture query returns the workload with the highest memory. The deployed memory prompt is p2-17. One PR.
```

```text
ID:          p2-13
Goal:        Set the env-01 tier map, the fallback model, and the ops xai-oauth device login.
Why:         Chat has to survive a blocked primary model on the same subscription login (R18, R23).
Advances:    P2 exit check: “With the primary model blocked, chat answers from the fallback model; with the provider blocked, the dashboard and Telegram show an error within 30 s and Gatus flags it (R18).”
Depends on:  p2-09, p2-07
Touches:     the env-01 instance repo values/hermes.yaml, ops volume
Hard limits: R18; R23. D66. Mid: concierge, inbox reply drafting, homelab chat. Cheap: cron, inbox triage, homelab checks. Frontier: on demand only. Hermes’s model login is a swappable provider; env-01’s first login is xai-oauth. The device login is Gordon’s approval of the code the pod prints. The token stays on the volume, never in Git or CI. This step adds validation of the hermes.* tier and model keys it sets (D45).
Done checks: The instance hermes.yaml maps each tier to a primary and a fallback on a swappable provider login. Gordon has approved the device login and a short chat answers. One PR.
```

```text
ID:          p2-14
Goal:        Route /api/<profile>/v1 to each bot and run the scripted non-error prompt.
Why:         The assistant API is the only supported way to talk to a bot (R17).
Advances:    P2 exit check: “A scripted prompt through the assistant API (/api/<profile>/v1) to each of the three bots gets a non-error answer; a Telegram message from Gordon’s ID to concierge gets a reply [G], and one from another ID gets none.”
Depends on:  p2-10, p2-13
Touches:     services/hermes/deploy/ops/ routing, services/hermes/verify.py
Hard limits: R17; D69. The API requires the deploy API key. The three profile names are concierge, inbox, and homelab. The check is part of the Hermes verify component in services/hermes/verify.py, which make verify ENV= runs only when the env enables Hermes; core tools/verify/ gains no Hermes check.
Done checks: A scripted prompt to each of the three URLs returns a non-error answer. One PR.
```

```text
ID:          p2-15
Goal:        Turn on concierge’s Telegram gateway with the allow-list, and prove another ID gets no reply.
Why:         Concierge is the only Telegram-facing bot, and the gateway accepts only Gordon’s ID (R21).
Advances:    P2 exit check: “A scripted prompt through the assistant API (/api/<profile>/v1) to each of the three bots gets a non-error answer; a Telegram message from Gordon’s ID to concierge gets a reply [G], and one from another ID gets none.”
Depends on:  p2-14
Touches:     concierge profile.yaml, SOPS telegram token
Hard limits: R21. The token and Gordon’s user ID are the SOPS values he records (the chat ID from P1). inbox and homelab stay telegram: false. The alerts bot is unchanged and has no gateway.
Done checks: A gateway-handler fixture for an ID outside the allow-list produces no reply (D17). Gordon’s own reply is an owner checklist item (D17). One PR.
```

```text
ID:          p2-16
Goal:        Prove memory recall after restart, staged forget, export, and one approval plus one rejection.
Why:         Learned state lives on the volume, and writes wait for approval (R12, R17).
Advances:    P2 exit check: “Memory: a fact is recalled after a pod restart in a new session; “forget that” is staged, and approving it removes the fact; a memory export produces a file. Memory and skill writes (R12) wait for approval; a test approves one and rejects one.”
Depends on:  p2-09, p2-14
Touches:     ops memory volume
Hard limits: R12; R15; R17. The test approves one memory write and rejects one. Skill writes use the same approval switch; the test covers one skill-write rejection.
Done checks: The log shows the fact surviving a pod restart in a new session, the forget staying staged until approval, the fact gone after approval, and an export file on disk. The memory drill is attached as its own run. One PR.
```

```text
ID:          p2-17
Goal:        Add the pytest eval harness and at least three evals per ops bot, graded by the cheap tier.
Why:         Evals run against the deployed bots through the assistant API, on that deploy’s login (R38).
Advances:    P2 exit check: “Each bot’s evals (≥3) pass against the deployed bots (the eval run attached beside make verify ENV=<env>, R38); usage is readable in Hermes analytics and the grok.com dashboard [G]; with the subscription login revoked or capped (a 401 test fixture stands in; the real login stays signed in), the dashboard and Telegram show an error within ~30 s and the eval run fails (R18).”
Depends on:  p2-11, p2-12, p2-14
Touches:     services/hermes/core/testing/, profiles/*/evals/
Hard limits: R38. The grader uses the ops assistant API and the deploy API key. No separate grader key and no key in GitHub Actions. Cover the appendix P2 seeds: weather, distance, reminder, memory and forget, and the three homelab read prompts. Inbox’s three evals stay inside P2 scope (no Gmail yet): a non-error reply, a refusal to send mail, a refusal to label mail. About 10 evals on the ops deploy.
Done checks: The eval run is attached on its own and the evals pass. One PR.
```

```text
ID:          p2-18
Goal:        Run the fallback, provider-down, and capped-login drills, and add the api.x.ai Gatus check.
Why:         A dead provider has to show an error within about 30 seconds instead of hanging (R18).
Advances:    P2 exit check: “Each bot’s evals (≥3) pass against the deployed bots (the eval run attached beside make verify ENV=<env>, R38); usage is readable in Hermes analytics and the grok.com dashboard [G]; with the subscription login revoked or capped (a 401 test fixture stands in; the real login stays signed in), the dashboard and Telegram show an error within ~30 s and the eval run fails (R18).”
Advances:    P2 exit check: “With the primary model blocked, chat answers from the fallback model; with the provider blocked, the dashboard and Telegram show an error within 30 s and Gatus flags it (R18).”
Depends on:  p2-13, p2-15, p2-17, p1-08
Touches:     Gatus provider check (observability component), services/hermes/verify.py, llm_provider_host in the env schema, env-01 and env-00
Hard limits: R8; R18. D19; D20; D39; D45 (llm_provider_host, read by Hermes and the Gatus provider probe, so one override trips both). The capped-login drill is a 401 fixture. The real login stays signed in. The drills restore xai-oauth before the task ends. Hosts in the contract are documentation. This step adds validation of llm_provider_host (D45). The drills are their own runs.
Done checks: The log shows a fallback-model answer, a dashboard error within 30 seconds when the provider host is overridden, the same dashboard error within about 30 seconds for the 401 fixture, and the eval run failing during that window and passing after restore. Owner checklist: Gordon’s paste from a Telegram message he sends during the drill (D17). One PR.
```

```text
ID:          p2-19
Goal:        Record usage from Hermes analytics and the grok.com confirmation line.
Why:         Subscription use has to be visible per bot and on the subscription dashboard (R23).
Advances:    P2 exit check: “Each bot’s evals (≥3) pass against the deployed bots (the eval run attached beside make verify ENV=<env>, R38); usage is readable in Hermes analytics and the grok.com dashboard [G]; with the subscription login revoked or capped (a 401 test fixture stands in; the real login stays signed in), the dashboard and Telegram show an error within ~30 s and the eval run fails (R18).”
Depends on:  p2-14, p2-17
Touches:     services/hermes/verify.py
Hard limits: R23. D18; D69 (the Hermes verify component). No usage data is copied into Git.
Done checks: The verify log quotes a non-empty Hermes analytics usage field. The grok.com dashboard line is a [G] owner checklist item (D17). One PR.
```

```text
ID:          p2-20
Goal:        Make a Hermes bump wait on a digest-keyed VolSync backup, keep auto-merge off, and run evals after deploy.
Why:         Hermes upgrades break, and the pre-upgrade backup has to finish before the new digest rolls (R33, R38).
Advances:    P2 exit check: “A test Hermes bump PR shows auto-merge is off, its deploy waited for the digest-keyed pre-upgrade backup, and the evals then ran against the deployed bot (R33).”
Depends on:  p2-17, p0b-09
Touches:     services/hermes/deploy/ops/ Flux dependency, services/hermes/tools/backup, services/hermes/Makefile, renovate.json
Hard limits: R31; R33; R38. D69 (the backup command belongs to the Hermes add-on and meets the chat-bot slot’s backup term, not core tools/ or the root Makefile). The ops Kustomization depends on a VolSync backup triggered by the new digest. make -C services/hermes backup ENV=<env> quiesces or dumps SQLite first, runs one on-demand backup of the PVC and waits for it to finish, and leaves no manual trigger set on the generated source, because VolSync ignores the schedule while one is set; nothing is added to Git (D47). Renovate does not auto-merge the Hermes digest. Gordon merges Hermes bump PRs. The code-factory deploy does not exist yet; this task covers the ops pin. The code-factory pin repeats the same pattern in p4b-05.
Done checks: A test bump PR shows auto-merge off, the backup Job complete for that digest before the rollout, and the evals passing against the deployed bot. One PR.
```

```text
ID:          p2-21
Goal:        After a pod restart, prove Git config was re-applied, and restore the Hermes volume from backup.
Why:         Runtime edits must not survive a restart, and the Hermes volume needs its own restore drill (R12, R31).
Advances:    P2 exit check: “After a pod restart, Git config is re-applied; restoring the Hermes volume from backup works.”
Depends on:  p2-06b, p2-16, p0b-10
Touches:     ops volume, services/hermes/verify.py
Hard limits: R12; R31. D47; D69 (the Hermes verify component). The restore is make restore ENV=<env> APP=hermes-ops (p0b-10), attached as its own run. ACCEPT_EMPTY and RETIRE are only valid with make restore-all, and this run does not lift the rebuild hold (D47). If the restored token is stale, Gordon redoes the device login (§8 custody) before chat is called again.
Done checks: The log shows a planted runtime edit gone after restart, and the restored volume’s checksum matching the backup. One PR.
```

```text
ID:          p2-22
Goal:        Add CI guards for digest pins, unshared volumes and secrets, the toolset policy, and MCP pin and read/write tags.
Why:         Fix order (§11): the core defaults already set these boundaries; the checks stop a later PR from slipping (R11, R13, R16).
Advances:    P2 exit check: “A scripted prompt through the assistant API (/api/<profile>/v1) to each of the three bots gets a non-error answer; a Telegram message from Gordon’s ID to concierge gets a reply [G], and one from another ID gets none.”
Depends on:  p2-08, p2-09, p2-10
Touches:     ci/checks/
Hard limits: R11; R13; R16. Each new script gets one seeded failing PR.
Done checks: Three seeded PRs fail ci (shared secret, a terminal tool with no written reason, an unpinned MCP). The live ops manifests pass. One PR.
```

```text
ID:          p2-23
Goal:        Add CI guards for the Telegram allow-list, model names outside envs/, and Hermes dashboard auth.
Why:         Fix order (§11): profiles and the dashboard already require these; CI keeps a later edit from dropping them (R21, R23, R27).
Advances:    P2 exit check: “The Hermes ops dashboard (hermes-ops.<DNS suffix>) shows up in Gatus (R5) and requires its login (R27).”
Depends on:  p2-10, p2-15, p1-12
Touches:     ci/checks/ui-auth, ci/checks/allow-list, ci/checks/core-names (model-name half)
Hard limits: R21; R23; R27. A telegram: true profile with an empty allow-list fails. A model name in core fails: core-names reads the names from the env tier and lane maps, which hold names from p2-13. Grafana anonymous-off stays in the same ui-auth script from p1-12.
Done checks: Three seeded PRs fail ci, and the live ops dashboard manifest has auth configured. One PR.
```

```text
ID:          p2-24
Goal:        Add admin-gate, seed the protected-path and label-actor PRs, and have Gordon turn the check on if using branch protection.
Why:         admin-gate stays the PR-risk check on the product repo and jarvis-apps; the deploy gate is Flux signed tags, landing in p2-08a (R36, D56, D61, D68). Fix order (§11): the check is the automated PR-risk gate; the rulebook does not replace it.
Advances:    P2 exit check: “admin-gate is the PR-risk check (R36); branch protection is optional and is not the deploy gate (D56): seeded PRs that change a protected path or remove a Namespace fail it without the label and when admin-approved comes from gordon5417-ai or a fixture Bot/App actor, and pass once Gordon’s own account adds it after the latest commit; a seeded PR fails the render check (a stale Hermes render, R4).”
Depends on:  p2-07, p2-22, p2-23
Touches:     .github/workflows/admin-gate.yml, ci/checks/admin-gate, jarvis-apps/.github/workflows/ci.yml, docs/code-delivery/CONTRACT.md
Hard limits: R36. D35; D38; D44; D56; D61. The repo owner merges every PR. An agent never adds admin-approved, even under the owner’s account. Paid branch protection is optional, not the deploy gate. No owner or repo is hardcoded: the approver is the repository owner from the event (github.repository_owner), and each caller passes its R36 path list. pull_request_target on opened, synchronize, reopened, labeled, unlabeled. The job reads PR metadata only and checks out no PR code: the workflow sparse-checks out only ci/checks/admin-gate at the pinned jarvis SHA and runs it on the event. A reusable workflow cannot see its own SHA (github.sha and workflow_sha are the caller’s), so that SHA is a workflow input the caller pins to the same SHA as its uses: line, never the PR head. It removes admin-approved on every new push. A Namespace removal fails closed. The jarvis-apps admin-gate job runs only on pull_request_target and calls admin-gate.yml at a pinned jarvis SHA, written literally because uses: takes no expressions; jarvis-apps is not scanned by core-names. It is its own required check when branch protection is on. p4a-05 adds the namespaced-only check to the pull_request ci job. Image publish stays in build.yml. Instance-repo paths (envs/*/secrets/, envs/*/values/apps.yaml) live in the private instance and are owner-merged.
Done checks: Seeded PRs fail without the label, fail when gordon5417-ai adds admin-approved, fail when the fixture actor is a Bot or an App, and pass after Gordon’s own account adds the label on the latest commit. A seeded removal of a PVC or a StorageClass fails without the label (R36). If branch protection is on, admin-gate is its own check, separate from ci (D56). The jarvis-apps admin-gate job checks out no PR code. One PR.
```

```text
ID:          p2-25
Goal:        Extend make verify ENV= with the P2 standing checks, and attach drills and evals as their own runs.
Why:         Done for the checkpoint is the attached verify output (§2, R35). P4 starts only after this exit passes.
Advances:    P2 exit check: “A scripted prompt through the assistant API (/api/<profile>/v1) to each of the three bots gets a non-error answer; a Telegram message from Gordon’s ID to concierge gets a reply [G], and one from another ID gets none.”
Advances:    P2 exit check: “Weather and distance questions get correct answers through the geo MCP server (weather: Open-Meteo; distance: Nominatim plus the OSRM demo, typical drive time) with attribution; the geo tool’s evals in concierge pass.”
Advances:    P2 exit check: “Memory: a fact is recalled after a pod restart in a new session; “forget that” is staged, and approving it removes the fact; a memory export produces a file. Memory and skill writes (R12) wait for approval; a test approves one and rejects one.”
Advances:    P2 exit check: “Each bot’s evals (≥3) pass against the deployed bots (the eval run attached beside make verify ENV=<env>, R38); usage is readable in Hermes analytics and the grok.com dashboard [G]; with the subscription login revoked or capped (a 401 test fixture stands in; the real login stays signed in), the dashboard and Telegram show an error within ~30 s and the eval run fails (R18).”
Advances:    P2 exit check: “With the primary model blocked, chat answers from the fallback model; with the provider blocked, the dashboard and Telegram show an error within 30 s and Gatus flags it (R18).”
Advances:    P2 exit check: “A test Hermes bump PR shows auto-merge is off, its deploy waited for the digest-keyed pre-upgrade backup, and the evals then ran against the deployed bot (R33).”
Advances:    P2 exit check: “The Hermes ops dashboard (hermes-ops.<DNS suffix>) shows up in Gatus (R5) and requires its login (R27).”
Advances:    P2 exit check: “After a pod restart, Git config is re-applied; restoring the Hermes volume from backup works.”
Advances:    P2 exit check: “admin-gate is the PR-risk check (R36); branch protection is optional and is not the deploy gate (D56): seeded PRs that change a protected path or remove a Namespace fail it without the label and when admin-approved comes from gordon5417-ai or a fixture Bot/App actor, and pass once Gordon’s own account adds it after the latest commit; a seeded PR fails the render check (a stale Hermes render, R4).”
Depends on:  p2-11, p2-15, p2-16, p2-17, p2-18, p2-19, p2-20, p2-21, p2-24
Touches:     services/hermes/verify.py
Hard limits: R33; R35; R36; R38. D17; D61; D69. The standing checks go in the Hermes verify component, which make verify ENV= runs only when the env enables Hermes. This PR waits for admin-approved. make verify reads standing state. The admin-gate seeds are seeded PRs, attached beside it, never inside the script. The [G] lines are concierge’s Telegram reply and the grok.com dashboard. Renovate auto-merge stays off; the owner merges every PR, including minor and patch bumps.
Done checks: make verify ENV=<env> exits 0 with the standing checks this phase adds (scripted prompts, dashboard login). The admin-gate seeds are seeded PRs attached beside it. Drills and evals (geo evals, memory, eval summary, degrade, bump, restore) are attached as their own runs. Owner checklist, pasted outside the script: Gordon’s Telegram [G] paste and the grok.com dashboard [G] paste. One PR.
```

### Exit checks

- A scripted prompt through the assistant API (/api/<profile>/v1) to each of the three bots gets a non-error answer; a Telegram message from Gordon’s ID to concierge gets a reply [G], and one from another ID gets none.
- Weather and distance questions get correct answers through the geo MCP server (weather: Open-Meteo; distance: Nominatim plus the OSRM demo, typical drive time) with attribution; the geo tool’s evals in concierge pass.
- Memory: a fact is recalled after a pod restart in a new session; “forget that” is staged, and approving it removes the fact; a memory export produces a file. Memory and skill writes (R12) wait for approval; a test approves one and rejects one.
- Each bot’s evals (≥3) pass against the deployed bots (the eval run attached beside make verify ENV=<env>, R38); usage is readable in Hermes analytics and the grok.com dashboard [G]; with the subscription login revoked or capped (a 401 test fixture stands in; the real login stays signed in), the dashboard and Telegram show an error within ~30 s and the eval run fails (R18).
- With the primary model blocked, chat answers from the fallback model; with the provider blocked, the dashboard and Telegram show an error within 30 s and Gatus flags it (R18).
- A test Hermes bump PR shows auto-merge is off, its deploy waited for the digest-keyed pre-upgrade backup, and the evals then ran against the deployed bot (R33).
- The Hermes ops dashboard (hermes-ops.<DNS suffix>) shows up in Gatus (R5) and requires its login (R27).
- After a pod restart, Git config is re-applied; restoring the Hermes volume from backup works.
- admin-gate is the PR-risk check (R36); branch protection is optional and is not the deploy gate (D56): seeded PRs that change a protected path or remove a Namespace fail it without the label and when admin-approved comes from gordon5417-ai or a fixture Bot/App actor, and pass once Gordon’s own account adds it after the latest commit; a seeded PR fails the render check (a stale Hermes render, R4).

Done = attached make verify ENV=<env> output, plus this phase’s drills and evals as their own runs

## P3

Goal: scheduled briefs and read skills deliver the daily butler, with no writes beyond inbox drafts.

### Contracts

No new §5 contract. The skill contract remains the one written in P2.

### Build

```text
ID:          p3-00
Goal:        Add google-consent to the Hermes add-on (make -C services/hermes google-consent ENV=<env>), the OAuth loopback flow on the controller.
Why:         The Gmail and Calendar token has to be sealed before a bot reads mail (R15, R25). The command belongs to the Hermes add-on, not to core (D69).
Advances:    P3 exit check: “The morning brief arrives at its scheduled time with calendar, email and weather (Open-Meteo, attributed; “weather unavailable” if it is down), and reaches Telegram [G].”
Depends on:  p0a-04, p2-25
Touches:     services/hermes/tools/google-consent, services/hermes/Makefile
Hard limits: R15; R25. D40; D69. Loopback flow on the controller. Scopes are gmail.readonly, gmail.compose, and calendar.readonly only. The token is written through make seal into SOPS, then into WORKSPACE_MCP_CREDENTIALS_DIR/<email>.json with USER_GOOGLE_EMAIL set. That credentials file stays writable so the server can refresh it.
Done checks: A fixture loopback run seals a token whose scopes are only those three. One PR.
```

```text
ID:          p3-01a
Goal:        Add concierge’s Gmail and Calendar read through workspace-mcp.
Why:         The daily butler reads mail and calendar (R15, §5 skill contract).
Advances:    P3 exit check: “The morning brief arrives at its scheduled time with calendar, email and weather (Open-Meteo, attributed; “weather unavailable” if it is down), and reaches Telegram [G].”
Depends on:  p3-00, p2-25
Touches:     ops profiles, deploy volume uv cache
Hard limits: R13; R15; R16. D22; D39 (egress; Google hosts in the ops CONTRACT.md as documentation); D40. workspace-mcp==2.1.1 via uvx, uv cache on the deploy volume, single-user, stdio. Concierge runs `--single-user --read-only --tools gmail calendar`. No extra tool allowlist.
Done checks: It runs on a token with only gmail.readonly, gmail.compose, and calendar.readonly, and registers no send tool. A read-only call returns one inbox thread and one calendar event after Gordon’s consent step. One PR.
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
```

```text
ID:          p3-03
Goal:        Enable the morning brief, evening wrap, and weekly review crons on concierge.
Why:         Listed crons start enabled; the PR is the approval (R19).
Advances:    P3 exit check: “The morning brief arrives at its scheduled time with calendar, email and weather (Open-Meteo, attributed; “weather unavailable” if it is down), and reaches Telegram [G].”
Advances:    P3 exit check: “The evening wrap arrives at its scheduled time with the day’s unread inbox threads, ranked by importance in chat only, and tomorrow’s calendar, and reaches Telegram [G].”
Advances:    P3 exit check: “The weekly review arrives at its scheduled time with the past week’s calendar and email highlights and the coming week’s calendar, and reaches Telegram [G].”
Depends on:  p3-01a
Touches:     the env-01 instance repo values/hermes.yaml, concierge profile
Hard limits: R15; R19. D21; D22. Cron times are America/Los_Angeles, the env schema time zone, the same value the node uses. The brief includes calendar, email, and Open-Meteo weather with attribution, or the words “weather unavailable” when Open-Meteo is down. The wrap includes the day’s unread inbox threads, ranked by importance in chat only, and tomorrow’s calendar. The weekly review includes the past week’s highlights and the coming week’s calendar. Telegram delivery is through concierge.
Done checks: A manual run of each cron produces the three bodies in the dashboard session. The cron expressions in hermes.yaml match D21, and the next run time is in America/Los_Angeles. Telegram arrival is an owner checklist item (D17). One PR.
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
```

```text
ID:          p3-06
Goal:        Ask inbox to draft a reply to the seeded email, and show the Sent folder unchanged.
Why:         Drafts are the one Gmail write in this plan, and they must not send (R15).
Advances:    P3 exit check: “Asked in chat to draft a reply to a seeded email, inbox creates one Gmail draft in that thread and nothing is sent (the Sent folder is unchanged).”
Depends on:  p3-01b, p3-05
Touches:     inbox draft tool
Hard limits: R15. D24. The seed subject is jarvis-p3-seed. The draft is created in that thread. No send tool is called.
Done checks: The log shows one new draft in the seed thread and a Sent-folder id list that matches the pre-check list. The draft drill is attached as its own run. One PR.
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
```

```text
ID:          p3-08
Goal:        Extend make verify ENV= with the P3 standing checks, and attach drills and evals as their own runs.
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
Touches:     services/hermes/verify.py
Hard limits: R35; R36. D17; D69. The standing checks go in the Hermes verify component. This PR waits for admin-approved. The [G] lines are the morning brief, the evening wrap, and the weekly review on Telegram.
Done checks: make verify ENV=<env> exits 0 with the standing checks this phase adds. The brief runs, the stuck-update finding, the evals, the draft proof and the cron alert are attached as their own runs. Owner checklist, pasted outside the script: Gordon’s three Telegram [G] pastes. One PR.
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

Done = attached make verify ENV=<env> output, plus this phase’s drills and evals as their own runs

## P4a

Goal: with app hosting on, one name in the env’s apps list deploys an outside app, isolated from JARVIS, from that app’s one app file.

### Contracts

```text
ID:          p4a-01
Goal:        Write the App hosting CONTRACT.md with proven by.
Why:         One name in the env’s apps list is the whole enable path once app hosting is on, and the contract comes before the wiring (§5, R30, D70).
Advances:    P4a exit check: “With app hosting on, after Gordon adds the app’s name to the env’s apps list, the outside app deploys to its own namespace, is reachable at <name>.apps.<DNS suffix> on LAN and on tailnet [G], shows up in Gatus (R5), and a network-policy test shows it cannot reach core services.”
Depends on:  p2-25
Touches:     platform/app-hosting/CONTRACT.md
Hard limits: R4; R30. D7; D63; D70. The contract covers the app file (a field table: required and optional fields, defaults, per-env differences, and the risky items that need an env exception line), namespace, quota, default-deny, service account, per-namespace GitRepository, impersonated Kustomization, the address <name>.apps.<DNS suffix>, and the Gatus ConfigMap. Defaults: 512Mi, 0.5 CPU, and a closed network: no web address, no LAN port, no egress beyond DNS and no caller (D70). Hosted apps are stateless by default; stateful apps are allowed, with backup enrollment, once that app has its own backup credential (D51); a volume may be marked not backed up. App hosting is optional and off by default; an env turns it on with one line in its repo, and with it off the env gets none of the app pieces. apps.yaml is re-added in the instance repo and lists names only (D54, D60).
Done checks: CONTRACT.md has Offers, Expects, Health, Config, and Proven by, naming checks without plan codes. One PR.
Proven by:   The reachability, Gatus, and network-policy checks, and the seeded cluster-scoped PR (R30).
```

### Build

```text
ID:          p4a-02
Goal:        Add the app-hosting wiring, fed by the env’s apps list and each app’s app file, with app hosting still off on env-01.
Why:         Fix order (§11): one name in the env’s apps list is the easy path; the cluster refuses what the app file and the env’s exception lines do not allow, and CI rejects the dangerous objects later (R30, R4, D70).
Advances:    P4a exit check: “With app hosting on, after Gordon adds the app’s name to the env’s apps list, the outside app deploys to its own namespace, is reachable at <name>.apps.<DNS suffix> on LAN and on tailnet [G], shows up in Gatus (R5), and a network-policy test shows it cannot reach core services.”
Depends on:  p4a-01, p0a-05
Touches:     platform/app-hosting/, env schema apps.yaml rules
Hard limits: R1; R4; R30. D30 for the default size: 512Mi and 0.5 CPU, unless the app file sets its own (§7). D54; D60; D63; D70. apps.yaml is re-added in the instance repo and holds app names and exception lines only: no settings per app, and no per-env on/off in the app file. A missing or empty list means no apps. Each app’s GitRepository URL comes from apps_git_repo_url (D45). This step adds the key and its validation. No cross-namespace source refs. Flux may use an optional pull secret (D56). No pull secret for public GHCR. The env schema accepts app hosting on or off: the switch is one kustomize components entry for core/platform/app-hosting in envs/<env>/flux/platform-configs/kustomization.yaml, off by default, and an env that lists apps with it off fails the schema check. A render script writes one app-<name>-wiring Kustomization per listed app into envs/<env>/flux/generated/apps.yaml. Exception lines go under exceptions: in the same apps.yaml. The cluster, not CI, enforces the app file: an app with no valid app file fails closed, the object matches the file, and a risky item (privileged mode, host access, a custom address outside .apps, a port open to the LAN (lan-port), egress to the LAN (lan-all, or lan-target <name> <cidr> <port>/<proto>)) passes only with an ask in the app file and a matching exception line in the env’s repo. The uptime opt-out (no-uptime-check) is an env exception line with no ask. LAN egress grants need the env’s LAN ranges, lan_cidrs in values/cluster-vars.yaml; a lan-target range sits inside them. Hosted-app internet egress is any port to the internet, never the env’s LAN or cluster ranges (D70). An app cannot take a core or add-on name, and no add-on may be named apps. Stateful apps are allowed only with backup enrollment.
Done checks: With app hosting off, or with apps.yaml empty or missing, no app namespace exists. A fixture render with one listed name produces the namespace, quota, default-deny policy, service account, GitRepository, and impersonated Kustomization. A listed name with no valid app file deploys nothing. A risky item without both the ask and the exception line is refused. One PR.
```

```text
ID:          p4a-03
Goal:        When the observability component is enabled, have the app wiring write the app’s Gatus endpoint ConfigMap, and load it from a Gatus sidecar.
Why:         A hosted app has to show up in Gatus without a hand-edited Gatus file (R5, §5). With observability off, app hosting writes no Gatus object (D69).
Advances:    P4a exit check: “With app hosting on, after Gordon adds the app’s name to the env’s apps list, the outside app deploys to its own namespace, is reachable at <name>.apps.<DNS suffix> on LAN and on tailnet [G], shows up in Gatus (R5), and a network-policy test shows it cannot reach core services.”
Depends on:  p4a-02, p1-08
Touches:     platform/app-hosting/, services/observability/gatus/
Hard limits: R5. The ConfigMap is labeled in the observability namespace. The sidecar is the same pattern Grafana uses for dashboards. The check probes the app’s address (<name>.apps.<DNS suffix>) from outside, like a user, at the path it reads from the app file’s network.web.healthPath; if it is missing the path is /, and any answer except a server error (5xx) is healthy. It is always on for hosted apps with network.web on an env with the observability component; an app opts out only with the env’s exception line <app>: no-uptime-check, which it reads from the app’s grants ConfigMap (no-uptime-check: "true"); the line needs no ask in the app file (D70). Kubernetes probes stay the app’s own in its deploy/ manifests, separate from this check.
Done checks: A fixture listed app produces a Gatus check for <name>.apps.alpha.lan at its health path, or / when it sets none. A fixture app with the env line <app>: no-uptime-check produces none. One PR.
```

```text
ID:          p4a-04
Goal:        Add demo/ (its app file, Dockerfile and deploy/) and the jarvis-apps GHCR build.
Why:         P4a needs one real app whose image CI publishes as a public GHCR package (§4, R30).
Advances:    P4a exit check: “With app hosting on, after Gordon adds the app’s name to the env’s apps list, the outside app deploys to its own namespace, is reachable at <name>.apps.<DNS suffix> on LAN and on tailnet [G], shows up in Gatus (R5), and a network-policy test shows it cannot reach core services.”
Depends on:  p4a-02
Touches:     jarvis-apps/demo/, jarvis-apps/.github/workflows/build.yml, platform/app-hosting/CONTRACT.md
Hard limits: R5; R10 (Python); R33 (digest). D25; D35; D70. demo/app.yaml is the app’s one app file: its name matches the folder, and it defines everything about the app, including per-env differences, in the keys of the app-hosting contract only (the network section: web with its port and optional healthPath, ports, egress and callers; volumes, resources, risky-item asks and per-env overrides); it holds no image or source. demo/app.yaml sets network.web.port to 8080 and network.web.healthPath to /health (D25). The image and the Kubernetes probes are the app’s own, in demo/deploy/. An app with no valid app file fails closed. deploy/ holds namespaced objects only. The image is a public package; Gordon sets the package public on first publish. build.yml is no longer marked planned in the app-hosting contract.
Done checks: build.yml publishes the image. The deploy manifests pin a digest. /health answers in a local container run. One PR.
```

```text
ID:          p4a-05
Goal:        Reject cluster-scoped objects in jarvis-apps, allow an enrolled stateful PVC, and seed one cluster-scoped PR.
Why:         Fix order (§11): the cluster’s admission rules already limit what an app is; CI is the backstop (R30).
Advances:    P4a exit check: “A seeded jarvis-apps PR with a cluster-scoped object fails CI (R30).”
Depends on:  p4a-04, p2-24
Touches:     jarvis-apps/.github/checks/namespaced-only.sh, jarvis-apps/.github/workflows/ci.yml
Hard limits: R30; R36. D31; D35; D63. The pull_request ci job runs the namespaced-only check and holds no secrets. Cluster-scoped objects fail. A PVC is allowed only for an enrolled stateful app (D51). The admin-gate job added in p2-24 stays on pull_request_target, checks out no PR code, and remains its own required check when branch protection is on. Image publish stays in build.yml on push to main (§6). .github/ on jarvis-apps is protected, so this PR waits for admin-approved.
Done checks: A seeded PR that adds a ClusterRole fails ci. A seeded unenrolled PVC fails ci. An enrolled stateful fixture passes. demo/ passes. One PR.
```

```text
ID:          p4a-06
Goal:        Write jarvis-apps/AGENTS.md as the app rulebook.
Why:         Fix order (§11): the namespaced-only check already enforces the boundary; this file states only what that check cannot (R39).
Advances:    P4a exit check: “With app hosting on, after Gordon adds the app’s name to the env’s apps list, the outside app deploys to its own namespace, is reachable at <name>.apps.<DNS suffix> on LAN and on tailnet [G], shows up in Gatus (R5), and a network-policy test shows it cannot reach core services.”
Depends on:  p4a-05
Touches:     jarvis-apps/AGENTS.md
Hard limits: R30; R39. D63. The builder sees only this repo. The file says apps are stateless by default, namespaced, and free of JARVIS secrets; stateful apps need backup enrollment; each app has one app file, <name>/app.yaml, that matches its folder and asks for any risky item (the env must also grant it, D70). It does not copy the jarvis rulebook. This path is protected (R36).
Done checks: AGENTS.md is the only rule file in the repo and names that the image and Kubernetes probes are the app’s own in deploy/, that the app file’s optional health path (default /; any answer but 5xx is healthy) feeds the uptime check, and the namespace rule. One PR.
```

```text
ID:          p4a-07
Goal:        Extend make verify ENV= with the P4a standing checks (the app Ready, the LAN curl, and, when the observability component is enabled, the Gatus row). The network-policy test, the seeded cluster-scoped failure, Gordon’s apps list change and the tailnet check stay outside the script.
Why:         Done for the checkpoint is the attached verify output (§2). The name in the env’s apps list is Gordon’s merge (R36).
Advances:    P4a exit check: “With app hosting on, after Gordon adds the app’s name to the env’s apps list, the outside app deploys to its own namespace, is reachable at <name>.apps.<DNS suffix> on LAN and on tailnet [G], shows up in Gatus (R5), and a network-policy test shows it cannot reach core services.”
Advances:    P4a exit check: “A seeded jarvis-apps PR with a cluster-scoped object fails CI (R30).”
Depends on:  p4a-03, p4a-04, p4a-05, p4a-06
Touches:     the env-01 instance repo values/apps.yaml, tools/verify/
Hard limits: R28; R30; R35; R36. D12 stays the platform fixture; the hosted app is app-demo. D17; D60. make verify reads standing state only. The network-policy test shows the app pod cannot reach Grafana, the ops API, DNS-to-core service ClusterIPs, or the node IP on 443 (Traefik). That test and the seeded cluster-scoped PR are their own runs, attached beside make verify, never inside the script. Tailnet reachability is the [G] check. Gordon’s merge of the app’s name into the env’s apps list in the instance repo is a Gordon-run step. The apps checks load only for an env that lists an app.
Done checks: make verify ENV=<env> exits 0 with the app Ready in namespace app-demo, the LAN curl, and, when observability is enabled, the Gatus row. The denied-egress log, including the node IP on 443, is its own run attached beside it. The seeded cluster-scoped failure is a seeded PR attached beside it. Owner checklist, pasted outside the script: Gordon’s tailnet [G] paste and his merged apps list change. One PR.
```

### Exit checks

- With app hosting on, after Gordon adds the app’s name to the env’s apps list, the outside app deploys to its own namespace, is reachable at <name>.apps.<DNS suffix> on LAN and on tailnet [G], shows up in Gatus (R5), and a network-policy test shows it cannot reach core services.
- A seeded jarvis-apps PR with a cluster-scoped object fails CI (R30).

Done = attached make verify ENV=<env> output, plus this phase’s drills and evals as their own runs

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
Done checks: CONTRACT.md has Offers, Expects, Health, Config, and Proven by, naming checks without plan codes. One PR.
Proven by:   Env schema validation and the lane checks (§5).
```

```text
ID:          p4b-02
Goal:        Write the Sandbox CONTRACT.md with proven by.
Why:         The lane’s inputs and outputs are the seam gVisor will reuse later, and gVisor itself is not built (§5, §9, R42).
Advances:    P4b exit check: “A toy code-factory:ready issue becomes a draft PR in jarvis-apps with a green CI check and a checker check run (pass or fail with notes) from a different model family.”
Advances:    P4b exit check: “A forced timeout gets the issue the code-factory:failed label, and foreman says so.”
Depends on:  p4b-01
Touches:     services/hermes/deploy/code-factory/sandbox/CONTRACT.md
Hard limits: R42. D64. Inputs: issue, repo, timeout. Outputs: status, patch, verdict, logs. The contract names GitHub-hosted runners for the initial setup and in-cluster lanes as the product path. No RuntimeClass and no gVisor code in this step.
Done checks: CONTRACT.md has Offers, Expects, Health, Config, and Proven by, naming checks without plan codes. One PR.
Proven by:   The toy-issue, timeout, and fix-loop checks.
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
Done checks: CONTRACT.md has Offers, Expects, Health, Config, and Proven by, naming checks without plan codes. One PR.
Proven by:   The lane checks (§5).
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
Hard limits: R23; R41; R42. D28; D54; D65. Lane logins are an adapter list (a Cursor key, a Claude subscription token); each adopter brings their own; parallel unattended lanes are capped; token expiry is on the keep-safe list. Builder is Composer 2.5, checker is Grok 4.7 in ask mode, code-factory:hard builder is Claude. The checker family differs from the builder family. Each stub exits with the text not implemented. admin-gate’s protected paths stay the R36 list; .grok/ and .opencode/ join that list only when that adapter is enabled (§9). This step re-adds the adapter list and its validation (D45, D54).
Done checks: cursor.sh dry run prints the pinned path and the role map. Both stubs exit non-zero with the text not implemented. The live lane map records different families. One PR.
```

```text
ID:          p4b-05
Goal:        Deploy the code-factory Hermes overlay with foreman and spec-writer, and no ServiceAccount token.
Why:         Foreman reports and spec-writer files issues. Neither one codes, and the pod holds no lane credentials (R13, R40).
Advances:    P4b exit check: “A seeded code-factory:ready label added by the App, not by Gordon’s account, is ignored, the workflow comments why, and foreman reports it (R40).”
Advances:    P4b exit check: “spec-writer turns an idea into an issue with acceptance criteria and without code-factory:ready; a failing checker verdict keeps the PR a draft and sends it back through the fix loop until the fix cap (R40).”
Depends on:  p4b-03, p2-20
Touches:     services/hermes/deploy/code-factory/, profiles/foreman, profiles/spec-writer
Hard limits: R5; R11; R12; R13; R15; R19; R33. D30; D39 (egress; api.x.ai and api.github.com in the CONTRACT.md as documentation). Its own volume, secrets, and digest pin. Pre-upgrade backup matches the ops pattern. No ServiceAccount token. YOLO stays off on this pod; YOLO is only inside the throwaway lane (R13, R42). Gordon approves this deploy’s xAI device login. Evals for these bots are p4b-09a and p4b-09b.
Done checks: The pod is Ready, the two profiles validate, and the manifest has no ServiceAccount token and no Cursor key. One PR.
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
```

```text
ID:          p4b-07
Goal:        Enforce the label actor, the one-run queue, the 30-minute timeout, and the fix cap of 2 runs.
Why:         A ready label counts only from Gordon’s account. A run that times out or hits the fix cap gets code-factory:failed, and foreman reports it (R40).
Advances:    P4b exit check: “A seeded code-factory:ready label added by the App, not by Gordon’s account, is ignored, the workflow comments why, and foreman reports it (R40).”
Advances:    P4b exit check: “A forced timeout gets the issue the code-factory:failed label, and foreman says so.”
Depends on:  p4b-06b
Touches:     code-factory workflow
Hard limits: R40; R42. D26; D27; D44 (the ready label counts only from the repository owner). One workflow invocation does builder, then checker, then on fail one builder fix with the checker’s notes, then checker, then publish draft. A second fail or the 30-minute job timeout gets code-factory:failed, and foreman reports it. Any other actor is ignored with a comment. Concurrency is a queue, one lane at a time.
Done checks: A unit fixture of the actor check accepts the repository owner’s login and rejects an App login. The workflow file shows that one invocation, the 30-minute job timeout, and code-factory:failed on a second fail. One PR.
```

```text
ID:          p4b-08
Goal:        Fail config validation when the builder and checker families match, and seed that config.
Why:         The checker has to be a different model family from the builder (R41).
Advances:    P4b exit check: “Config validation fails when the builder and checker are from the same model family (seeded test, R41).”
Depends on:  p4b-04, p0a-05
Touches:     ci/checks/lane-families, hermes.yaml
Hard limits: R23; R41. D28. The live env-01 map stays composer versus grok. The seed is a fixture file, not the live env file.
Done checks: The seeded same-family file fails ci. env-01’s lane map passes. One PR.
```

```text
ID:          p4b-09a
Goal:        Teach spec-writer to file an issue only after a chat confirm, with at least three evals.
Why:         spec-writer never adds code-factory:ready (R15, R40).
Advances:    P4b exit check: “spec-writer turns an idea into an issue with acceptance criteria and without code-factory:ready; a failing checker verdict keeps the PR a draft and sends it back through the fix loop until the fix cap (R40).”
Depends on:  p4b-05, p4b-07
Touches:     profiles/spec-writer
Hard limits: R15; R19; R40. D41; D45 (the issue repo comes from apps_git_repo_url; this step fails if that key is unset). The issue has acceptance criteria and does not carry code-factory:ready. spec-writer runs github-mcp-server `stdio --toolsets issues` with code-factory App auth, behind the confirm step. No extra tool allowlist. Evals cover the appendix line for “turn this idea into a code-factory issue” and at least two more. This task holds spec-writer’s evals from the deploy task.
Done checks: A scripted confirm files one issue without the ready label, and a second run with the confirm denied files nothing. spec-writer’s evals pass. One PR.
```

```text
ID:          p4b-09b
Goal:        Teach foreman to report finished, failed, and ignored runs, with at least three evals.
Why:         Foreman watches and reports, and never codes (R40).
Advances:    P4b exit check: “A seeded code-factory:ready label added by the App, not by Gordon’s account, is ignored, the workflow comments why, and foreman reports it (R40).”
Depends on:  p4b-05, p4b-06b, p4b-07
Touches:     profiles/foreman
Hard limits: R19; R40. D37; D45 (foreman reads the repo named by apps_git_repo_url; this step fails if that key is unset). Foreman’s cron skips the model call when nothing changed. Evals cover “what did the code-factory finish overnight?” and at least two more. Together with p4b-09a the deploy has about 10 evals.
Done checks: Foreman’s session lists an ignored fixture run. Foreman’s evals pass. One PR.
```

```text
ID:          p4b-10
Goal:        Show the lane token scopes and show the code-factory cannot write to jarvis.
Why:         The lane holds no push credential, and the App is installed on jarvis-apps only (R42, R40).
Advances:    P4b exit check: “Token scopes show no agent holds push credentials and the code-factory cannot write to jarvis.”
Advances:    P2 exit check: “admin-gate is the PR-risk check (R36); branch protection is optional and is not the deploy gate (D56): seeded PRs that change a protected path or remove a Namespace fail it without the label and when admin-approved comes from gordon5417-ai or a fixture Bot/App actor, and pass once Gordon’s own account adds it after the latest commit; a seeded PR fails the render check (a stale Hermes render, R4).”
Advances:    P4b exit check: “The code-factory App adding admin-approved fails admin-gate.”
Depends on:  p4b-06b
Touches:     ci/checks/token-scopes, both workflows, jarvis-apps/.github/workflows/seed-app.yml
Hard limits: R36; R40; R42. The check fails if the lane job’s permissions include write, or if the App token secret is passed into the lane job. seed-app.yml is one workflow_dispatch workflow, admin-gate protected. It uses the App secret to add code-factory:ready, add admin-approved, and try a jarvis write. It checks out no PR code.
Done checks: make verify ENV=<env> prints the lane permission set, a read of the workflow. The rejected write to jarvis is the seed-app.yml dispatch, attached beside make verify, never inside the script. Dispatching seed-app.yml makes the code-factory App adding admin-approved fail admin-gate, and the jarvis write fails. One PR.
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
```

```text
ID:          p4b-12
Goal:        Seed a code-factory:ready label added by the App, and show it ignored, commented, and reported.
Why:         A ready label from anyone but Gordon’s account must not start a lane (R40).
Advances:    P4b exit check: “A seeded code-factory:ready label added by the App, not by Gordon’s account, is ignored, the workflow comments why, and foreman reports it (R40).”
Depends on:  p4b-07, p4b-09b, p4b-10
Touches:     jarvis-apps issue, foreman session, jarvis-apps/.github/workflows/seed-app.yml
Hard limits: R40. seed-app.yml is the same workflow_dispatch workflow as p4b-10. It uses the App secret to add code-factory:ready, add admin-approved, and try a jarvis write. This task dispatches the ready label. The workflow comments why it was ignored. No lane branch is pushed. That dispatch, including the rejected write to jarvis, is its own run, attached beside make verify ENV=, never inside the script.
Done checks: Dispatching seed-app.yml adds the label as the App, and the jarvis write fails. The issue has the workflow comment, no code-factory branch, and a foreman report of the ignored run. make verify does not perform the write. One PR.
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
```

```text
ID:          p4b-15
Goal:        Extend make verify ENV= with the P4b standing checks, and attach drills as their own runs.
Why:         Done for the checkpoint is the attached verify output (§2, R35).
Advances:    P4b exit check: “A toy code-factory:ready issue becomes a draft PR in jarvis-apps with a green CI check and a checker check run (pass or fail with notes) from a different model family.”
Advances:    P4b exit check: “A seeded code-factory:ready label added by the App, not by Gordon’s account, is ignored, the workflow comments why, and foreman reports it (R40).”
Advances:    P4b exit check: “spec-writer turns an idea into an issue with acceptance criteria and without code-factory:ready; a failing checker verdict keeps the PR a draft and sends it back through the fix loop until the fix cap (R40).”
Advances:    P4b exit check: “Config validation fails when the builder and checker are from the same model family (seeded test, R41).”
Advances:    P4b exit check: “Token scopes show no agent holds push credentials and the code-factory cannot write to jarvis.”
Advances:    P4b exit check: “A forced timeout gets the issue the code-factory:failed label, and foreman says so.”
Advances:    P4b exit check: “The code-factory App adding admin-approved fails admin-gate.”
Depends on:  p4b-08, p4b-10, p4b-11, p4b-12, p4b-13, p4b-14
Touches:     the code-factory add-on’s verify component, if it has one: services/<name>/verify.py (the add-on’s shape is decided at P4b)
Hard limits: R35; R36; R40; R42. D17; D69. If the code-factory add-on has a verify component, it is services/<name>/verify.py and make verify ENV= runs it only when the env enables the add-on. services/ is an R36 protected path, so this PR waits for admin-approved. make verify reads standing state only. The family-seed failure is a seeded PR, attached beside it, never inside the script. Gordon applying code-factory:ready is a Gordon-run step.
Done checks: make verify ENV=<env> exits 0 with the standing checks this phase adds (the draft PR exists, and the token-scope output). The family-seed failure is a seeded PR attached beside it. The lane drills, including the rejected write to jarvis, are attached as their own runs. Owner checklist, pasted outside the script: Gordon applies code-factory:ready. One PR.
```

### Exit checks

- A toy code-factory:ready issue becomes a draft PR in jarvis-apps with a green CI check and a checker check run (pass or fail with notes) from a different model family.
- A seeded code-factory:ready label added by the App, not by Gordon’s account, is ignored, the workflow comments why, and foreman reports it (R40).
- The code-factory App adding admin-approved fails admin-gate.
- spec-writer turns an idea into an issue with acceptance criteria and without code-factory:ready; a failing checker verdict keeps the PR a draft and sends it back through the fix loop until the fix cap (R40).
- Config validation fails when the builder and checker are from the same model family (seeded test, R41).
- Token scopes show no agent holds push credentials and the code-factory cannot write to jarvis.
- A forced timeout gets the issue the code-factory:failed label, and foreman says so.

Done = attached make verify ENV=<env> output, plus this phase’s drills and evals as their own runs
