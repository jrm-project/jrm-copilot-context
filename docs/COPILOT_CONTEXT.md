# JRM Copilot Context

## Purpose

This file is the operating contract for any Copilot session working on JRM.

Before proposing or making a JRM change, read `docs/JRM_AS_BUILT.md` and this file, then verify live state. The repository documentation is context, not a substitute for live verification where the requested work depends on current state.

## Start-of-session checks

On `jet` (`192.168.0.70`):

```bash
jrm-status
cd /srv/jrm-source
git status --short --branch
git rev-parse HEAD
git rev-parse origin/main
```

Normal clean baseline:

- `jrm-status` has no unexplained Unknown or Drift/failure results.
- Git working tree is clean unless intentionally continuing a documented feature branch.
- `main` and `origin/main` are synchronized before starting a new feature.

If the observed state differs from `docs/JRM_AS_BUILT.md`, stop and resolve/document the difference before making a consequential change.

## Verified infrastructure anchors

- `jet`: `192.168.0.70`, main website/application/Git/release/deployment host.
- `jrm-research`: `192.168.0.34`, Research/Agent/Ollama/SearXNG/Valkey host.
- Git repository on `jet`: `/srv/jrm-source`.
- Git remote: `git@github.com:jrm-project/jrm.git`.
- Immutable releases: `/opt/jrm-releases`.
- Shared runtime: `/opt/jrm-runtime/venv`.
- Production: `/opt/jrm`, `jrm.service`, port `8000`.
- Test 1: `/opt/jrm-test1`, `jrm-test1.service`, port `8101`.
- Test 2: `/opt/jrm-test2`, `jrm-test2.service`, port `8102`.

Do not rely on this summary when detail matters. Use `docs/JRM_AS_BUILT.md` and live configuration.

## Evidence-first rule

Never guess JRM technical details, including:

- paths;
- schemas;
- filenames;
- versions;
- hashes;
- ports;
- hostnames or IP addresses;
- service configuration;
- source metadata;
- DNS, Cloudflare, nginx, or SSL configuration;
- release identity;
- deployment behaviour.

Inspect actual source, configuration, Git state, release metadata, or command output before providing commands or implementation guidance.

## Change workflow

For application/admin changes:

1. Verify live health and Git state.
2. Inspect the relevant current source/configuration.
3. Create a dedicated feature branch.
4. Make a narrow change.
5. Perform syntax/static checks.
6. Inspect the exact Git diff.
7. Commit the feature branch.
8. Get the full commit SHA.
9. Build one immutable release from that exact SHA.
10. Deploy that release to Test 1.
11. Run `jrm-status` and perform functional/browser validation.
12. Promote the same release to Test 2.
13. Run `jrm-status` and validate again.
14. Fast-forward the tested feature into `main`.
15. Push and verify `main` equals `origin/main`.
16. Promote the same immutable release to Production through guarded `jrm-deploy`.
17. Run `jrm-status` immediately and functionally check Production.
18. Update `docs/JRM_AS_BUILT.md` when material deployed/infrastructure state changes.

Never rebuild between Test 1, Test 2, and Production.

## Production deployment rule

Never bypass `jrm-deploy` by manually switching `/opt/jrm`.

Production syntax:

```bash
JRM_PRODUCTION_CONFIRM=<release-id> jrm-deploy production <release-id>
```

The guarded Production path verifies release integrity, Git ancestry, clean Git state, persistent paths/runtime, service health, process/runtime state, and payload content, and has automatic rollback.

## Troubleshooting style

For potentially disruptive work:

- Proceed one verified step at a time.
- State the exact machine/session for every command.
- Prefer read-only inspection first.
- Give the next required action, then inspect its output before advancing.
- Preserve rollback before changing working systems.
- Stop on unexpected output.
- Do not assume operator error when a copied command fails.
- Prefer short commands and targeted output over huge dumps.
- Avoid large heredocs in chat because pasted heredocs have been mangled previously.

## Generated files and link rules

These rules are mandatory:

1. Create the complete file before presenting it.
2. Use the exact intended filename.
3. Any link must be properly named and suitable for its target.
4. For a generated file, visible link text must be exactly the intended filename, with no `Download`, `corrected`, instructional, or other prefix/suffix.
5. The local downloaded filename must exactly match the filename used in the supplied transfer command.
6. Choose an artifact format that actually downloads in Copilot.
7. If HTML opens in a viewer rather than downloading, package the exact content as a suitably named `.txt` transfer artifact, then copy it to the required `.html` destination on the server.
8. Do not require manual filename correction or inference.
9. Validate generated technical files before delivery where possible.
10. Provide SHA-256 for generated technical files.
11. After creating a file, provide the exact Windows PowerShell `scp` command.
12. Then provide the exact verification/run/install command for the relevant SSH console.
13. Verify the transferred hash before execution.

## Secret handling

Never place secret values in Git documentation or generated context files. This includes passwords, API keys, `SECRET_KEY`, SMTP password, private SSH keys, Cloudflare API tokens, and future database credentials.

Locations and variable names may be documented when operationally useful.

## Documentation rule

`docs/JRM_AS_BUILT.md` is the version-controlled technical source of truth for the verified as-built environment.

When infrastructure, deployment tooling, service layout, versions, or material environment state changes:

- update the as-built;
- validate the documentation diff;
- commit the documentation with, or immediately adjacent to, the change it describes;
- do not knowingly leave deployed state and documentation inconsistent.

`docs/JRM_ADMIN_GUIDE.md` is the operating manual for the administrator-facing JRM interface.

When a change modifies an Admin screen, field, button, workflow, validation rule, side effect, warning, or operating procedure:

- review `docs/JRM_ADMIN_GUIDE.md` in the same feature branch;
- update the relevant guide section before promotion to Production;
- verify the guide against the implemented templates and backend behaviour;
- do not document proposed functionality as though it is already implemented;
- publish the updated guide to the public Copilot context repository after the authoritative private documentation is committed.

## Current checkpoint summary

At the checkpoint that created this context:

```text
Git main/origin:    42a72450ff56950016f2e4a4bad2e7d608dfd8d2
Website release:    admin-navigation-376bb76
Application commit: 376bb767276a5131d5017d443d77f37256b932a1
Production:         admin-navigation-376bb76
Test 1:             admin-navigation-376bb76
Test 2:             admin-navigation-376bb76
jrm-status:          v0.3.0
jrm-deploy:          v0.3.0
Health:              22 OK / 0 Unknown / 0 Drift/failures
```

Always re-run the start-of-session checks instead of assuming this checkpoint is still current.
