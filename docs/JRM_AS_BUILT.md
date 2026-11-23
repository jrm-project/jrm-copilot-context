# JRM As-Built

**Status:** Verified operational baseline
**Checkpoint:** 24 November 2026, Auckland
**Repository host:** GitHub
**Repository:** `git@github.com:jrm-project/jrm.git`
**Authoritative local repository:** `/srv/jrm-source` on `jet`

> This document records only state verified from JRM command output, source inspection, deployed release metadata, or explicit configuration checks. Do not infer unrecorded paths, secrets, domains, versions, ports, schemas, or deployment behaviour. Re-verify live state before making changes.

## 1. Purpose and operating principle

JRM is operated as a Git-backed application with isolated Production, Test 1, and Test 2 website environments, immutable release artifacts, guarded deployment tooling, rollback capability, and a separate Research/Agent host.

The deployment principle is:

```text
feature branch
    -> exact Git commit
    -> immutable release
    -> Test 1
    -> Test 2
    -> merge/push to main
    -> guarded Production promotion of the SAME release
```

Never rebuild an application release between Test 1, Test 2, and Production.

## 2. Infrastructure inventory

### 2.1 `jet`

| Item | Verified value |
|---|---|
| Hostname | `jet` |
| IP address | `192.168.0.70` |
| Role | Main JRM website/application host; Git working repository; immutable release store; Production/Test deployment host |
| Source repository | `/srv/jrm-source` |
| Release root | `/opt/jrm-releases` |
| Shared Python runtime | `/opt/jrm-runtime/venv` |
| Deployment history | `/var/lib/jrm-deploy/history.tsv` |
| Deployment lock | `/run/lock/jrm-deploy.lock` |

### 2.2 `jrm-research`

| Item | Verified value |
|---|---|
| Hostname | `jrm-research` |
| IP address | `192.168.0.34` |
| Role | Research service, local Agent, Ollama, SearXNG, Valkey |
| Status collector | `v0.1.0` |
| Access used by status | restricted SSH collector from `jet` using `/root/.ssh/jrm-research-status` |

The Research/Agent host does not yet use the same immutable-release awareness as the website environments. `jrm-status` currently compares Research and Agent code with current source.

## 3. Website environments

| Environment | Link | systemd service | Port | Environment file | Persistent state |
|---|---|---|---:|---|---|
| Production | `/opt/jrm` | `jrm.service` | `8000` | `/etc/jrm/jrm.env` | `/var/lib/jrm` |
| Test 1 | `/opt/jrm-test1` | `jrm-test1.service` | `8101` | `/etc/jrm/jrm-test1.env` | `/var/lib/jrm-test1` |
| Test 2 | `/opt/jrm-test2` | `jrm-test2.service` | `8102` | `/etc/jrm/jrm-test2.env` | `/var/lib/jrm-test2` |

### Production persistent paths

Verified from `/etc/jrm/jrm.env`:

```text
JRM_DATABASE=/var/lib/jrm/site.db
JRM_UPLOAD_DIR=/var/lib/jrm/uploads
```

Verified Production service characteristics:

```text
Service:          jrm.service
User:             jrm
Group:            jrm
WorkingDirectory: /opt/jrm
Runtime:          /opt/jrm-runtime/venv/bin/gunicorn
Bind:             0.0.0.0:8000
Environment file: /etc/jrm/jrm.env
```

Do not store secret values from `/etc/jrm/*.env` in Git. Verified Production variable names include `SECRET_KEY`, mail configuration, Gemini configuration, `JRM_DATABASE`, and `JRM_UPLOAD_DIR`, but their secret values are intentionally omitted here.

## 4. Public/site naming

Verified site information from prior configuration work:

- Primary JRM public domain: `jrmbrisbane.com.au`.
- Production, Test 1, and Test 2 have separate application instances and separate persistent data so testing does not overwrite Production data.
- Cloudflare, SSL, nginx, and reverse-proxy configuration had been completed and tested for the website/test setup before the release-workflow phase.

Exact Test 1/Test 2 public DNS hostnames, nginx filenames, Cloudflare record names, and certificate paths are not recorded here because they were not re-verified during this checkpoint. **Do not reconstruct them from memory. Inspect live nginx/Cloudflare configuration before changing them.**

## 5. Git state at this checkpoint

```text
Repository:    /srv/jrm-source
Remote:        git@github.com:jrm-project/jrm.git
Branch:        main
main:          42a72450ff56950016f2e4a4bad2e7d608dfd8d2
origin/main:   42a72450ff56950016f2e4a4bad2e7d608dfd8d2
Working tree:  clean
```

Recent relevant commits:

```text
42a7245 Add guarded Production deployment
376bb76 Add active state to admin navigation
986b004 Make jrm-status release-aware
4789b59 Extend guarded deployment tool to Test 2
```

Known feature branches retained at checkpoint:

```text
feature/admin-navigation
feature/jrm-status-v0.3
```

## 6. Current immutable application release

All three website environments currently run the same immutable application release:

```text
Release ID:     admin-navigation-376bb76
Release commit: 376bb767276a5131d5017d443d77f37256b932a1
Built at:       2026-11-23T21:18:25+00:00
Release root:   /opt/jrm-releases/admin-navigation-376bb76
```

Pointers at final verification:

```text
Production -> /opt/jrm-releases/admin-navigation-376bb76/production
Test 1     -> /opt/jrm-releases/admin-navigation-376bb76/test1
Test 2     -> /opt/jrm-releases/admin-navigation-376bb76/test2
```

Release contents include separate `production`, `test1`, and `test2` payloads, migration files, `requirements.txt`, `RELEASE-METADATA`, and `SHA256SUMS`.

`RELEASE-METADATA` records at least:

```text
release_id
commit
tag
source_version
built_at
builder_version
requirements_sha256
```

`SHA256SUMS` is used as the immutable release integrity manifest.

## 7. Retained releases and rollback material

Verified retained releases include:

```text
/opt/jrm-releases/g3-runtime-da0aeb4
/opt/jrm-releases/admin-navigation-376bb76
/opt/jrm-releases/rollback-test-health503-780924c
```

Previous Production/Test baseline release:

```text
Release: g3-runtime-da0aeb4
Commit:  da0aeb4f0ac8211829db9f94d17501971cb8d23b
```

Do not delete retained releases until release-retention and cleanup rules are separately designed and verified.

## 8. Release builder

Tracked tool:

```text
/srv/jrm-source/tools/jrm-build-release
```

Verified syntax:

```text
jrm-build-release <release-id> <full-commit-sha>
```

Example of the verified application release build:

```text
tools/jrm-build-release admin-navigation-376bb76 376bb767276a5131d5017d443d77f37256b932a1
```

That build completed successfully with `204` files and checksum validation.

## 9. `jrm-status` v0.3.0

Tracked source:

```text
/srv/jrm-source/tools/jrm-status
```

Installed tool:

```text
/usr/local/bin/jrm-status
```

Verified installed/tracked SHA-256 at installation:

```text
bfb7a76ddd7de1836db758b96bf09a4525befb93567ccc9fc5dbc58bee70a11b
```

Previous installed version backup:

```text
/usr/local/bin/jrm-status-v0.2.0-backup
```

### Behaviour

`jrm-status` v0.3 separates two concepts:

- **SOURCE:** current Git development state.
- **DEPLOYMENT:** release identity and integrity of each deployed environment.

For website deployments it:

1. Resolves the environment symlink.
2. Locates the immutable release root.
3. Reads `RELEASE-METADATA`.
4. Reports release ID, commit, tag, and build time.
5. Validates `SHA256SUMS`.
6. Verifies the associated systemd service.

It does **not** treat current `main` as the deployed website baseline. This prevents false drift while development advances beyond the deployed application release.

### Final verified status

```text
Tool version:      0.3.0
Source branch:     main
Source commit:     42a72450ff56950016f2e4a4bad2e7d608dfd8d2
Working tree:      clean
Remote sync:       matches origin/main
OK checks:         22
Unknown checks:    0
Drift/failures:    0
```

## 10. `jrm-deploy` v0.3.0

Tracked source:

```text
/srv/jrm-source/tools/jrm-deploy
```

Installed tool:

```text
/usr/local/bin/jrm-deploy
```

Verified installed/tracked SHA-256 at installation:

```text
f08b18c1b4285d0c673759d37210485e4d197e873d487b832e97d0be3c9a35ab
```

Previous installed version backup:

```text
/usr/local/bin/jrm-deploy-v0.2.0-backup
```

Supported targets:

```text
test1
test2
production
```

Syntax:

```text
jrm-deploy <test1|test2|production> <release-id>
```

Production requires an explicit matching confirmation value:

```text
JRM_PRODUCTION_CONFIRM=<release-id> jrm-deploy production <release-id>
```

### Guarded deployment checks

The tool includes:

- root-only execution;
- release-ID validation;
- a global deployment lock;
- release payload existence checks;
- environment-file checks;
- persistent database/upload checks;
- shared-runtime checks;
- existing deployment symlink checks;
- full release `SHA256SUMS` validation;
- metadata release-ID validation;
- release commit validation in Git;
- `requirements.txt` metadata validation;
- runtime-package comparison against release requirements;
- atomic symlink switching;
- systemd stop/start;
- HTTP `/health` and root-page validation;
- Gunicorn shared-runtime validation;
- environment-specific payload verification;
- deployment history logging;
- automatic rollback to the previous pointer if the post-switch verification fails.

Production additionally requires:

- `JRM_PRODUCTION_CONFIRM` exactly equal to the requested release ID;
- release commit must be an ancestor of local `main`;
- release commit must be an ancestor of `origin/main`;
- Git working tree must be clean.

The ancestry rule permits deployment-tool/documentation commits to exist after a tested application commit while preventing an unmerged application commit from being promoted to Production.

## 11. Deployment history and tested promotion path

The `admin-navigation-376bb76` release completed the full workflow:

1. Feature developed on `feature/admin-navigation`.
2. Commit `376bb767276a5131d5017d443d77f37256b932a1` created.
3. Immutable `admin-navigation-376bb76` release built.
4. Release checksums passed.
5. Same release deployed to Test 1.
6. Test 1 browser testing passed.
7. Same immutable release deployed to Test 2.
8. Test 2 browser testing passed.
9. Application commit fast-forwarded into `main` and pushed.
10. Guarded Production support was added to `jrm-deploy`, committed, pushed, and installed.
11. Same `admin-navigation-376bb76` release deployed to Production through the guarded path.
12. Final `jrm-status` returned `22 OK / 0 Unknown / 0 Drift`.

## 12. Admin application

Shared admin shell:

```text
app/templates/admin/base.html
```

Dashboard:

```text
app/templates/admin.html
```

All inspected functional admin templates inherit the shared base template.

Admin assets:

```text
app/static/css/admin.css
app/static/js/admin.js
```

### Current admin areas

Verified route/source inventory includes:

- Dashboard
- Bookings
- Booking detail
- Booking notes
- Booking status
- Booking preparation/research actions
- Parts management
- Motorcycle catalogue
- Manufacturers
- Variants
- Specifications
- Fitments
- Catalogue import/promotion
- Research submissions
- Research agents
- Enrichment
- Automatic enrichment controls
- Enrichment results/review/retry/cancel
- Website content administration
- Gallery
- Email administration/test
- Security/password management

### Shared navigation

The admin sidebar is persistent across the shared templates. The tested `admin-navigation-376bb76` release added route-aware active-state highlighting for:

- Dashboard
- Bookings and booking child pages
- Parts
- Catalogue and catalogue child pages
- Research
- Enrichment and enrichment child pages
- Website Content
- Gallery
- Email
- Security

Responsive behaviour already exists in `admin.css`/`admin.js`:

- fixed sidebar on desktop;
- mobile sidebar below `800px`;
- menu toggle;
- sidebar scrim;
- shared top bar;
- logout control;
- `data-confirm` form confirmation handling.

## 13. Application source inventory

Verified application Python modules include:

```text
app/app.py
app/booking_admin.py
app/bookings.py
app/catalogue.py
app/database.py
app/email_admin.py
app/enrichment.py
app/enrichment_control.py
app/enrichment_results.py
app/enrichment_worker.py
app/enrichment_worker_v2.py
app/free_catalogue.py
app/notifications.py
app/preparation.py
app/research_admin.py
app/research_api.py
```

Verified migrations in current releases:

```text
001_booking_catalogue.sql
002_booking_public_ui.sql
003_booking_admin_parts.sql
004_catalogue_fitment.sql
005_ai_research_agents.sql
006_catalogue_assisted_prep.sql
007_email_notifications.sql
008_free_catalogue_staging.sql
009_automated_enrichment_queue.sql
010_1a_enrichment_batch_foundation.sql
010_1b_provider_parser.sql
010_1c_admin_automation.sql
010_free_agent_worker.sql
011_au_variant_staging.sql
012_2_3_7c_deterministic_reconciliation.sql
```

Do not infer database schema solely from migration names. Inspect the actual migrations and live/test database schema when schema detail is required.

## 14. Research/AI stack

Final verified status from `jrm-status`:

```text
Host:              jrm-research
IP:                192.168.0.34
Collector:         v0.1.0
Research code:     matches source
Agent code:        matches source
Research service:  active
Agent service:     active
Ollama service:    active
Research HTTP:     expected 404 at root
Agent HTTP:        200
Ollama:            0.40.13
Ollama model:      qwen3:14b
Ollama model ID:   bdbd181c33f2
SearXNG:           sha256:892ed5710603e38f9dc201e6a0b2f3f14dbdb8108edcadac30e55b165c979fb7
Valkey:            sha256:2602ace2be6569bc2bbb06f873776516ab9089821525be34f4305c67eb9db02a
```

Version expectations used by `jrm-status` are read from:

```text
docs/G2B_COMPONENT_VERSIONS.txt
```

The Research/Agent release process is a known future design area. Do not assume it mirrors the `/opt/jrm-releases` website model.

## 15. Secrets and credential boundaries

Do **not** add any of the following to this repository or this document:

- passwords;
- `SECRET_KEY` value;
- SMTP password;
- Gemini API key;
- SSH private keys;
- Cloudflare API tokens;
- database credentials if introduced later;
- session secrets or access tokens.

This document may record secret *locations* and environment-variable *names* where operationally useful, but not values.

## 16. Standard change workflow

Before any change:

```bash
jrm-status
cd /srv/jrm-source
git status --short --branch
git rev-parse HEAD
git rev-parse origin/main
```

Expected starting condition is a healthy status, clean tree, and synchronized `main`/`origin/main`, unless the work is intentionally continuing an already documented feature branch.

For application/admin changes:

1. Inspect live/current source and configuration.
2. Create a dedicated feature branch.
3. Make the narrowest required change.
4. Perform syntax/static checks.
5. Inspect the exact diff.
6. Commit the feature branch.
7. Obtain the full commit SHA.
8. Build an immutable release from that exact SHA.
9. Deploy the same release to Test 1.
10. Run `jrm-status`.
11. Perform browser/functional validation.
12. Promote the same release to Test 2.
13. Run `jrm-status` and functional validation again.
14. Fast-forward the tested feature branch into `main`.
15. Push `main` and verify it matches `origin/main`.
16. Use guarded Production deployment for the same tested release.
17. Run `jrm-status` immediately.
18. Perform a Production functional check.

Stop if any relevant check is `UNKNOWN`, `DRIFT`, failed, inconsistent, or unexpectedly different from the documented design.

## 17. Working rules for Copilot and operators

### Evidence-first rule

Never guess:

- technical paths;
- filenames;
- database schemas;
- versions;
- hashes;
- ports;
- hostnames/IPs;
- source metadata;
- systemd configuration;
- nginx/Cloudflare configuration;
- release/deployment behaviour.

Verify the actual source/configuration/output first.

### One-step troubleshooting

For potentially disruptive work:

- give the next action only;
- identify the exact machine/session;
- state the expected result where practical;
- inspect output before advancing;
- preserve rollback;
- do not assume operator error when copied commands fail;
- avoid oversized pasted command blocks.

### Generated files and links

Mandatory rules:

1. Create the complete file before presenting it.
2. Use the exact intended filename.
3. **Any link provided must be properly named, descriptive, and suitable for the target.**
4. For generated file downloads, visible link text must be **exactly the intended filename**, without prefixes such as `Download`, `corrected`, or explanatory text.
5. The downloaded filename must exactly match the filename used in the supplied transfer command.
6. Choose a transfer format that actually downloads in the Copilot interface.
7. If an `.html` artifact will open in a viewer instead of downloading, package the exact HTML as a suitably named `.txt` transfer artifact, then copy it to the required `.html` path on the server.
8. Do not require manual filename repair or inference.
9. Validate generated scripts/files before delivery where validation is possible.
10. Provide SHA-256 for generated technical files.
11. After file creation, provide the exact **Windows PowerShell `scp` command first**.
12. Then provide the exact verification/run/install command for the relevant SSH console.
13. Verify the transferred hash before execution.
14. Avoid large heredocs in chat because pasted content has previously been mangled.

## 18. Documentation maintenance rule

This Markdown file is intended to become the version-controlled JRM as-built source of truth.

For every material infrastructure or deployment change:

1. Make and verify the technical change.
2. Update `docs/JRM_AS_BUILT.md` in the same body of work.
3. Update any separate infrastructure changelog if maintained.
4. Commit documentation with, or immediately adjacent to, the change it documents.
5. Never silently allow documented state to diverge from deployed state.

At the start of a future JRM session, Copilot should read the repository's current JRM context/as-built documentation and then verify live state with `jrm-status` before acting.

## 19. Known documentation gaps requiring re-verification before use

The following details are intentionally **not guessed** and should be added when next verified:

- exact Test 1 public DNS hostname;
- exact Test 2 public DNS hostname;
- complete nginx vhost filenames and content;
- Cloudflare record names/settings;
- SSL certificate file paths/issuer/renewal mechanism;
- host OS/version for `jet` and `jrm-research`;
- VM/hypervisor resource allocations;
- exact Research/Agent service unit definitions and internal ports;
- backup schedules/retention for application databases and uploads;
- disaster-recovery restore procedure;
- immutable-release cleanup/retention policy;
- exact Test 1/Test 2 environment-file data paths if not already documented elsewhere;
- Research/Agent immutable-release/promotion design.

Do not fill these gaps from memory. Retrieve live evidence when a gap becomes relevant.

## 20. Current checkpoint

At the final verified checkpoint:

```text
Git main/origin:      synchronized at 42a72450ff56950016f2e4a4bad2e7d608dfd8d2
Working tree:         clean
Production release:   admin-navigation-376bb76
Test 1 release:       admin-navigation-376bb76
Test 2 release:       admin-navigation-376bb76
Application commit:   376bb767276a5131d5017d443d77f37256b932a1
jrm-status:            v0.3.0
jrm-deploy:            v0.3.0
Status:                22 OK / 0 Unknown / 0 Drift
Research stack:        healthy
```

The release/status/deployment foundation is complete and healthy. The next normal activity should be a concrete JRM application/admin feature or defect, using the established promotion workflow.
