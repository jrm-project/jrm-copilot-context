# JRM Functional Design

**Purpose:** Defines what JRM is intended to do, why the capability exists, the expected user and operational behaviour, and the acceptance criteria that govern implementation.

**Authoritative repository:** `jrm-project/jrm`

> This document describes functional intent. Proposed behaviour must remain clearly identified as proposed until implemented and verified. Technical paths, commands, services, schemas, versions and deployment details must not be invented to complete a design. Existing implementation must be inspected before technical integration details are treated as established.

## 1. Role of this document

JRM maintains four complementary operational documents:

| Document | Question answered |
| --- | --- |
| `docs/JRM_FUNCTIONAL_DESIGN.md` | What is JRM intended to do and why? |
| `docs/JRM_AS_BUILT.md` | How is JRM actually implemented and deployed? |
| `docs/JRM_ADMIN_GUIDE.md` | How is JRM operated? |
| `docs/COPILOT_CONTEXT.md` | How must Copilot safely work on JRM? |

The Functional Design is not a substitute for the As-Built. The design can describe proposed functionality. The As-Built records verified implemented state.

---

## 2. Documentation lifecycle

Documentation is part of implementation and the JRM definition of done. It is not a separate task to be completed after development.

### 2.1 Functional design before implementation

Before implementing a substantial new feature or material behavioural change:

1. Define the functional intent in this document.
2. State why the capability exists.
3. Define expected user workflows and outcomes.
4. Define safety and security boundaries.
5. Define failure behaviour and recovery expectations.
6. Define audit and confirmation requirements where actions are consequential.
7. Define explicit out-of-scope items.
8. Define acceptance criteria.
9. Mark unimplemented behaviour as **Proposed**.

The design must not present guessed technical details as established fact.

### 2.2 Inspect the existing system before finalising technical design

After functional intent is agreed and before implementation begins, inspect the existing implementation and live environment where the feature depends on them.

Inspection should establish the relevant real interfaces, including as applicable:

- current source structure;
- paths and filenames;
- command syntax;
- command output and exit behaviour;
- service definitions;
- ports and bindings;
- permissions and privilege requirements;
- database/schema dependencies;
- release structure;
- deployment safeguards;
- rollback behaviour;
- persistent state;
- status and health interfaces;
- existing security boundaries.

Technical design is then filled in from this evidence.

Where a fact remains unverified, retain it as proposed or identify it as a design question. Do not silently promote an assumption into the design.

### 2.3 Implement on a dedicated feature branch

A substantial feature should be implemented on a dedicated feature branch.

The branch should contain the complete change required to represent the feature accurately, including the applicable combination of:

- application code;
- infrastructure/deployment code;
- configuration or service definitions;
- migrations;
- functional design updates;
- As-Built updates;
- Admin Guide updates;
- Copilot Context changes where the engineering operating rules themselves change.

### 2.4 Documentation impact rules

#### Functional behaviour changes

Review and update `docs/JRM_FUNCTIONAL_DESIGN.md` when implementation modifies the agreed functional behaviour, scope, safety model, workflow or acceptance criteria.

#### Infrastructure or technical architecture changes

Review and update `docs/JRM_AS_BUILT.md` when changes affect verified technical state, including:

- services;
- hosts;
- ports;
- paths;
- persistence;
- databases;
- runtime dependencies;
- deployment tooling;
- release mechanisms;
- monitoring/health behaviour;
- security boundaries;
- rollback mechanisms;
- material versions or environment state.

#### Administrator-visible changes

Review `docs/JRM_ADMIN_GUIDE.md` whenever a change modifies an administrator-facing:

- page;
- field;
- button;
- status;
- validation rule;
- workflow;
- warning;
- side effect;
- confirmation;
- operating procedure;
- recovery/troubleshooting process.

Where the administrator-visible behaviour changed, update the guide in the same feature branch.

Where review confirms that no guide change is required, that conclusion may be explicitly recorded by the change/release process rather than forcing a meaningless documentation edit.

#### Copilot operating rules

Update `docs/COPILOT_CONTEXT.md` only where the rules for safely developing, deploying, documenting or operating JRM change.

### 2.5 Build documentation while building the feature

Administrator-facing functionality is not considered complete merely because the UI works.

For each material administrator-facing control, the Admin Guide should document as applicable:

- **Purpose**: why the function exists.
- **When to use it**: the operational condition it addresses.
- **Preconditions**: what must already be true.
- **Controls**: what the administrator sees.
- **Operation**: the user steps.
- **Backend behaviour**: the verified operation invoked underneath the UI.
- **Validation**: checks before and after execution.
- **Side effects**: what is changed.
- **Non-effects**: what deliberately remains untouched.
- **Failure behaviour**: what happens when execution fails.
- **Rollback/recovery**: automatic or manual recovery behaviour.
- **Audit trail**: what is recorded and where.
- **Troubleshooting**: the first evidence an administrator should inspect.

Documentation should evolve with the implementation rather than be reconstructed afterwards.

### 2.6 Validate code and documentation together

Before a feature leaves its branch, validate both implementation and relevant documentation.

Validation should include as applicable:

- syntax/static checks;
- tests;
- credential/secret scanning;
- whitespace/diff checks;
- exact Git diff review;
- documentation impact review;
- confirmation that proposed functionality has not been described as deployed functionality;
- confirmation that As-Built claims have evidence;
- confirmation that Admin Guide behaviour matches the implementation.

### 2.7 Promotion workflow

The intended lifecycle is:

```text
Functional intent
      |
Existing-state inspection
      |
Technical design
      |
Feature branch
      |
Implementation + documentation
      |
Validation
      |
Test 1
      |
Functional validation
      |
Test 2
      |
Functional validation
      |
Main
      |
Production
      |
Verify deployed state
      |
Final As-Built reconciliation
      |
Publish operational documentation
```

Application releases must continue to follow the established immutable-release rule. The release tested in the test environments is the release promoted onward; do not rebuild between environments.

### 2.8 Documentation promotion gates

JRM release/operations tooling should ultimately enforce documentation impact checks rather than relying solely on memory.

Examples:

- A change beneath an administrator UI path with no Admin Guide review should raise a documentation gate.
- An infrastructure/service change with no As-Built review should raise a documentation gate.
- A functional-scope change with no Functional Design review should raise a documentation gate.

The gate should support two valid outcomes:

1. applicable documentation was updated; or
2. documentation was explicitly reviewed and recorded as not requiring a content change.

The tooling must not manufacture factual documentation to satisfy a gate.

### 2.9 Production is not the final documentation step

After Production deployment:

1. Verify live health and functional behaviour.
2. Reconcile material deployed state with `docs/JRM_AS_BUILT.md`.
3. Correct documentation if verified Production reality differs from pre-deployment expectations.
4. Commit authoritative private documentation first.
5. Run public-document credential/safety validation.
6. Synchronise approved operational documentation to `jrm-project/jrm-copilot-context`.
7. Publish the public context repository.

The authoritative flow is one direction:

```text
Private source of truth
jrm-project/jrm
      |
      | validated operational documentation
      v
Public body of knowledge
jrm-project/jrm-copilot-context
```

Do not edit the public copy and copy changes backwards into the private source of truth.

---

## 3. JRM definition of done

A JRM change is complete only when the requirements applicable to that change have been satisfied.

### Implementation

- Code/configuration is committed to the authoritative private repository.
- Implementation matches the agreed functional intent.
- Relevant validation/tests pass.

### Functional design

- Functional intent accurately describes the feature.
- Proposed and implemented behaviour are clearly distinguished.
- Material scope or behavioural changes are reflected in this document.

### Administrator operation

- Administrator-visible behaviour has been reviewed against `docs/JRM_ADMIN_GUIDE.md`.
- Changed administrator behaviour is documented.
- Consequential actions clearly document side effects, confirmations and recovery behaviour.

### As-Built

- Material infrastructure/technical changes are represented in `docs/JRM_AS_BUILT.md`.
- As-Built claims are supported by verified implementation/live evidence.

### Release and deployment

- Where an application release is required, the same immutable release passes Test 1 and Test 2 before Production.
- Production deployment uses the guarded deployment path.
- Post-deployment status and functional checks pass.

### Publication

- Private operational documentation describes the verified state.
- Public operational documentation is safety/credential checked.
- Approved documents are synchronised from private source to the public context repository.

---

## 4. Functional design template for new JRM capabilities

Use this structure when a substantial new capability is introduced.

```markdown
## <Capability name>

**Status:** Proposed | In implementation | Implemented and verified

### Purpose

### Problem being solved

### Functional objectives

### User / administrator workflows

### Inputs and outputs

### Safety and security boundaries

### Permissions / access model

### Confirmation requirements

### Audit requirements

### Failure behaviour

### Rollback / recovery expectations

### Monitoring and status requirements

### Documentation impacts

### Out of scope

### Acceptance criteria

### Verified technical integration

Fill this section only after existing-state inspection. Record verified interfaces, dependencies and constraints. Do not guess technical implementation details.
```

---

## 5. JRM Operations Console (`jrm-ops`)

**Status:** Proposed, with existing-state inspection completed for the current status, release-build, deployment, privilege, deployment-history, Research-status and public-documentation interfaces. Implementation has not started.

### Purpose

Provide an internal-only web operations console that makes the established JRM application, infrastructure, release and operational-documentation procedures observable and safely operable through a controlled UI.

The console is intended to reduce command-line friction without replacing or weakening existing safeguards.

### Functional objectives

`jrm-ops` should provide:

- a consolidated JRM operational status dashboard;
- status refresh on an approximately five-minute schedule;
- an explicit manual status refresh control;
- controlled UI actions for existing approved JRM operational commands;
- immutable release visibility and environment targeting;
- Test 1, Test 2 and Production promotion workflows;
- documentation-state visibility;
- controlled publication of operational documentation;
- activity/audit history for operations initiated from the console;
- clear separation between read-only/status functions and consequential actions.

### Status dashboard

The dashboard is intended to display at least:

- source/Git status;
- current source commit;
- Production release/state;
- Test 1 release/state;
- Test 2 release/state;
- Research/AI service health represented by the existing status model;
- operational-documentation synchronisation state;
- last status refresh time;
- next scheduled refresh time;
- overall health/drift indication.

Status should refresh automatically approximately every five minutes. A completed operational action should trigger an immediate status refresh rather than waiting for the next scheduled interval.

### Release operations

The intended controls include:

- validate source/state;
- build an immutable release;
- deploy the selected release to Test 1;
- deploy the same tested release to Test 2;
- promote the same tested release to Production;
- invoke supported rollback behaviour.

The UI must expose the exact release identity being acted on.

Production promotion must be visibly distinct from test deployment and require an explicit confirmation appropriate to a consequential Production action.

### Documentation operations

The console should make the documentation lifecycle visible rather than hiding it behind release actions.

Intended functions include:

- show whether Functional Design review is required/current;
- show whether As-Built review is required/current;
- show whether Admin Guide review is required/current;
- compare authoritative private operational documents with public copies;
- run documentation validation and credential/secret checks;
- publish approved operational documents to the public context repository;
- display public-document publication state.

The console must not fabricate documentation merely to satisfy a release gate.

### Documentation gates

The console should be able to flag likely documentation impact from changed source paths and require either:

- the corresponding documentation change; or
- an explicit recorded review stating that no content update is required.

At minimum, the design should support gates for:

- functional-scope changes;
- infrastructure/deployment/service changes;
- administrator-visible changes.

### Diagnostics

The UI should provide access to the existing approved JRM diagnostic/status capability, not implement an unrelated parallel health model.

An administrator should be able to:

- view current consolidated status;
- refresh status immediately;
- see which component caused an unhealthy/drift result;
- see the outcome of the most recent operational command.

### Activity history

Operations initiated through the console should produce an audit/activity record containing as applicable:

- timestamp;
- operator identity available to the application;
- requested operation;
- target environment;
- source commit;
- release identity;
- result;
- relevant error/status summary.

### Security model

`jrm-ops` is intended to be internal-only.

The console must not become a generic shell exposed through a web UI.

The implementation should:

- expose only explicitly approved operations;
- validate action parameters against constrained expected values;
- avoid arbitrary command construction/execution;
- use the least privilege necessary for each operation;
- preserve existing deployment confirmations and safeguards;
- provide stronger confirmation for Production/destructive operations;
- avoid exposing secrets in status output or activity history;
- preserve a clear boundary between status/read operations and change operations.

Existing service and privilege arrangements have now been inspected. The final `jrm-ops` service identity, internal bind address/port, systemd hardening, privilege-escalation mechanism and application-state locations remain proposed implementation decisions and must not be treated as deployed facts.

### Failure behaviour

A failed operation should:

- be reported as failed rather than inferred successful from partial output;
- retain the command/result evidence needed for troubleshooting;
- not automatically advance to the next promotion stage;
- preserve existing rollback behaviour where the wrapped JRM operation provides it;
- trigger an immediate status refresh after failure where safe;
- leave subsequent consequential actions gated until state is understood.

### Out of scope for initial implementation

Unless subsequently added to the design after inspection, v1 should not:

- execute arbitrary administrator-supplied shell commands;
- replace existing release/deployment safeguards;
- fabricate documentation;
- bypass Test 1/Test 2 promotion rules;
- automatically promote to Production merely because automated checks passed;
- provide broad control of the Research host beyond specifically designed JRM operations.

### Acceptance criteria

Before `jrm-ops` v1 can be considered implemented:

- existing JRM status/build/deploy interfaces have been inspected and documented;
- the console uses verified existing operational interfaces rather than recreating them without cause;
- status refresh works on the agreed schedule and on demand;
- each exposed consequential action has explicit parameter validation and confirmation behaviour;
- Production deployment preserves the guarded Production deployment path;
- operations are auditable;
- relevant failure and rollback behaviour is visible/documented;
- documentation gates operate without automatically inventing documentation;
- `docs/JRM_AS_BUILT.md` documents the verified deployed `jrm-ops` architecture;
- `docs/JRM_ADMIN_GUIDE.md` documents every exposed operator control and workflow;
- public operational documentation is synchronised after authoritative private documentation is committed.

### Verified technical integration

Existing-state inspection has been completed for the interfaces required to define the initial `jrm-ops` technical integration. The facts below are verified from the current repository-controlled tools and live `jet` configuration. Items explicitly labelled **Proposed** remain design decisions and are not yet implemented.

#### Existing operational tool model

The installed operational commands are:

- `/usr/local/bin/jrm-status`
- `/usr/local/bin/jrm-build-release`
- `/usr/local/bin/jrm-deploy`

Their authoritative repository sources are:

- `tools/jrm-status`
- `tools/jrm-build-release`
- `tools/jrm-deploy`

Inspection verified that each installed command is byte-for-byte identical to its repository-controlled source. `tools/jrm-research-status` is also repository-controlled and supports the restricted Research-host status path.

`jrm-ops` must wrap these established operational tools rather than reproduce their release, health or deployment logic inside the web application.

#### Status integration

`jrm-status` is currently version `0.3.0`. It checks:

- source repository, branch, exact tag, commit and origin;
- working-tree cleanliness;
- synchronization with the locally known upstream;
- Production, Test 1 and Test 2 release identity, release metadata, checksum integrity and service state;
- Research and Agent source hashes;
- Research, Agent and Ollama service state;
- Research and Agent HTTP checks;
- expected Ollama version/model;
- expected SearXNG and Valkey image digests.

Its overall exit contract is:

- `0`: no warning or failure checks;
- `1`: one or more unknown/warning checks and no failures;
- `2`: one or more drift/failure checks.

Research status is collected from `192.168.0.34` using the dedicated restricted key `/root/.ssh/jrm-research-status`, rather than duplicating remote inspection logic locally.

**Proposed structured interface:** extend the repository-controlled `jrm-status` with a backwards-compatible `--json` output mode. Invocation without `--json` must retain the existing human-readable output and exit-code semantics. `jrm-ops` should consume the structured mode rather than scrape formatted shell output. The JSON contract should expose both the overall result and the individual source, environment and Research checks needed by the dashboard.

`jrm-ops` should cache the most recent completed status result and refresh it approximately every five minutes. A manual refresh and completion of any operational action should request a fresh status immediately. Overlapping status runs should be prevented or coalesced.

#### Immutable release build integration

`jrm-build-release` is currently version `0.1.0` and has the verified interface:

```text
jrm-build-release <release-id> <full-commit-sha>
```

The builder already enforces the core immutable-release rules:

- release IDs are syntax constrained and cannot overwrite an existing release;
- the commit must be an exact 40-character lowercase SHA available in Git;
- `/srv/jrm-source` must have a clean working tree;
- payload content is built from `git archive` of the requested commit, not from mutable working-tree files;
- required application, frontend, migration, requirements, version and migration-checksum paths must exist in that commit;
- migration checksums are validated;
- Production, Test 1 and Test 2 payloads are built into the same immutable release;
- `RELEASE-METADATA` records release ID, commit, tag, source version, build time, builder version and requirements hash;
- `SHA256SUMS` covers the completed release payload and is validated before publication;
- release directories/files are made read-only before the final atomic move to `/opt/jrm-releases/<release-id>`.

The builder uses fail-fast shell behaviour and already distinguishes usage/input/repository/release/build failures with exit codes in the `64` to `70` range.

`jrm-ops` should pass only a validated release ID and exact full commit SHA to this tool. The web application must not create shell command strings from arbitrary user input.

**Proposed structured interface:** preserve the current CLI while adding a stable machine-consumable result mode or wrapper contract that exposes release ID, commit, final path, file count, success/failure category and diagnostic message without requiring the UI to scrape presentation text.

#### Guarded deployment integration

`jrm-deploy` is currently version `0.3.0` and has the verified interface:

```text
jrm-deploy <test1|test2|production> <release-id>
```

Production additionally requires:

```text
JRM_PRODUCTION_CONFIRM=<release-id>
```

The verified environment mapping is:

| Environment | Link | Service | Port | Environment file | Persistent state |
| --- | --- | --- | --- | --- | --- |
| Test 1 | `/opt/jrm-test1` | `jrm-test1.service` | 8101 | `/etc/jrm/jrm-test1.env` | `/var/lib/jrm-test1` |
| Test 2 | `/opt/jrm-test2` | `jrm-test2.service` | 8102 | `/etc/jrm/jrm-test2.env` | `/var/lib/jrm-test2` |
| Production | `/opt/jrm` | `jrm.service` | 8000 | `/etc/jrm/jrm.env` | `/var/lib/jrm` |

The deployer requires root and already protects deployment with `/run/lock/jrm-deploy.lock`. It verifies, before switching:

- repository availability;
- target payload, release metadata and checksum manifest;
- environment file;
- persistent `site.db` and uploads directory;
- shared runtime executables;
- existing environment symlink and valid previous target;
- release checksums;
- release metadata identity and exact Git commit;
- requirements metadata hash;
- exact package equality between the shared runtime and release requirements.

Production additionally requires the release commit to be contained in both local `main` and `origin/main` and requires a clean source working tree.

Deployment stops the environment service, atomically switches the environment symlink, and restarts the service. Post-switch verification requires:

- the service to be active;
- `/health` to return HTTP 200;
- `/` to return HTTP 200;
- Gunicorn to be running from `/opt/jrm-runtime/venv` on the expected environment port;
- the environment symlink and key deployed files to match the selected immutable release.

If post-switch service/health/runtime/content verification fails, the existing deployer automatically attempts rollback to the previous target and verifies recovery. `jrm-ops` must preserve this behaviour rather than provide a separate symlink-switch path.

The current deploy exit contract distinguishes usage/confirmation errors, precheck failures, switch failures, successful rollback, failed rollback and deployment-lock contention. The UI must surface these as distinct operation outcomes rather than flattening them into a generic failure.

#### Existing deployment audit data

`jrm-deploy` records tab-separated deployment history at `/var/lib/jrm-deploy/history.tsv` and also logs through syslog. The history directory/file are root controlled (`0750` directory, `0640` file at inspection time).

Existing history includes successful Test 1, Test 2 and Production deployments, precheck failures and a verified automatic rollback event. The fields already recorded are sufficient to seed the Deployment Activity view: timestamp, environment, release ID, release commit, previous target, target, result and invoking user.

`jrm-ops` should supplement, not replace, this deployment record with its own broader activity log for actions such as status refresh, validation, release build and operational-document publication.

#### Service and privilege boundary

The current JRM application services run as `jrm:jrm` with a nologin account. The three website services are systemd services and do not run as root. No JRM-specific sudoers rules existed at inspection time. Root ownership currently protects the source repository, release store, shared runtime and deployment history.

`jrm-deploy` genuinely requires privileged operations because it controls systemd services, environment symlinks, deployment locks and root-owned history. Therefore the `jrm-ops` web process must **not** run as root merely to invoke deployment actions.

**Proposed privilege model:**

- create a dedicated non-login service identity for `jrm-ops`, separate from the existing `jrm` website identity;
- run the `jrm-ops` web service under that dedicated identity;
- grant no generic shell or unrestricted `sudo`;
- expose privileged operations only through specifically approved command entry points with constrained argument validation;
- retain `jrm-deploy`'s own validation, Production confirmation, lock and rollback behaviour behind the privilege boundary;
- grant read access only to the status/history/source/release metadata required by the UI;
- ensure operation logs do not expose environment-file secrets or authentication material.

The exact service username, systemd hardening options, bind address/port, and sudo/polkit mechanism remain implementation decisions. They must be selected and then documented in `docs/JRM_AS_BUILT.md` from the implemented configuration.

#### Enrichment operational state

Existing systemd units include `jrm-enrichment-worker.service` and `jrm-enrichment-worker.timer`. At inspection time the timer was disabled. The current Admin application already contains explicit enable/disable and run-one controls for automatic enrichment. `jrm-ops` may surface status for these components, but v1 must not create a second independent enrichment control path without an explicit design decision and corresponding Admin Guide update.

#### Operational-document publication integration

The authoritative operational documents in `/srv/jrm-source/docs` are:

- `COPILOT_CONTEXT.md`;
- `JRM_FUNCTIONAL_DESIGN.md`;
- `JRM_AS_BUILT.md`;
- `JRM_ADMIN_GUIDE.md`.

The public body-of-knowledge working copy is `/srv/jrm-copilot-context`, tracking `git@github.com:jrm-project/jrm-copilot-context.git`. It uses a repository-specific SSH command configured locally with the dedicated key `/root/.ssh/jrm-copilot-context` and `IdentitiesOnly=yes`.

At inspection time all four private/public operational-document pairs had identical SHA-256 hashes and both repositories were clean.

The existing publication procedure is private-source-first:

1. commit authoritative documentation in `jrm-project/jrm`;
2. copy approved operational documents into `/srv/jrm-copilot-context/docs`;
3. run credential/secret and diff/whitespace validation;
4. commit and push `jrm-project/jrm-copilot-context`;
5. allow the public GitHub Pages publication to rebuild.

`jrm-ops` should automate this established direction of travel. It must compare private/public SHA-256 values and show per-document synchronisation state before publication. It must never treat the public repository as authoritative or copy documentation backwards.

Because the existing public-repository SSH credential is stored under `/root/.ssh`, the final non-root publication mechanism must be deliberately redesigned rather than granting the web process access to root's private key. The approved mechanism and credential location must be established during implementation and recorded in the As-Built.

#### Activity and concurrency model

**Proposed:** `jrm-ops` should serialize consequential operations so that a release build/deploy/publication action cannot race another conflicting operation. Existing `jrm-deploy` locking remains authoritative for deployments; the console should additionally maintain an application-level operation state for UI gating and auditability.

Each operation record should include:

- start and completion timestamp;
- operator identity available to `jrm-ops`;
- operation type;
- target environment where applicable;
- source commit;
- release ID where applicable;
- invoked approved backend operation;
- exit/result category;
- concise diagnostic summary.

Raw output may be retained for troubleshooting subject to size limits and secret filtering, but must not be used as the sole structured state model.

#### UI/backend interface requirements

The UI must consume stable structured state rather than parse terminal-oriented presentation text. The preferred design is to evolve repository-controlled operational tools with backwards-compatible structured modes where needed. Existing human CLI behaviour must remain usable for direct administration and troubleshooting.

The initial structured contracts should provide, at minimum:

- consolidated status/check data from `jrm-status`;
- immutable release build result data;
- deployment operation/result data while preserving `jrm-deploy` exit/rollback semantics;
- per-document private/public hash and Git-state data;
- deployment/activity history required by the Operations dashboard.

#### Remaining implementation decisions

The inspection intentionally does not establish the following, so these remain proposed decisions rather than facts:

- `jrm-ops` service username/group;
- internal bind address and port;
- authentication/session model for the internal console;
- exact sudo/polkit/helper design used to cross the root privilege boundary;
- exact storage/schema for `jrm-ops` activity records and cached status;
- exact JSON schemas and versioning for structured tool output;
- exact non-root Git credential mechanism for publishing `jrm-copilot-context`;
- whether rollback is initially exposed as an explicit button in addition to the existing automatic rollback path;
- whether enrichment timer/run-one controls belong in `jrm-ops` v1 or remain only in the existing Admin interface.

These decisions must be resolved in design/implementation before the associated capability is exposed.
