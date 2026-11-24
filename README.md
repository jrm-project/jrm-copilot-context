# JRM Copilot Context

Public technical context for the JRM project.

This repository provides Copilot and other web-accessible tools with authoritative technical context for JRM without exposing the private application source repository.

## Start here

For any JRM session, read these documents first:

1. [Copilot operating context](docs/COPILOT_CONTEXT.md)
2. [JRM Functional Design](docs/JRM_FUNCTIONAL_DESIGN.md)
3. [JRM as-built technical documentation](docs/JRM_AS_BUILT.md)
4. [JRM Administration Guide](docs/JRM_ADMIN_GUIDE.md)

## Important

The documentation records verified JRM architecture, infrastructure, deployment tooling, application structure, research/AI services, release state and operating procedures.

Documentation is context, not proof of current runtime state.

Before proposing or performing a consequential JRM change:

- verify the current live state;
- do not guess paths, schemas, filenames, versions, hashes, ports, configuration or deployment behaviour;
- prefer read-only inspection before changes;
- proceed one verified step at a time;
- stop if live state conflicts with the documented as-built environment.

## Security boundary

This repository is intentionally public.

It may contain non-secret technical information including internal IP addresses, hostnames, paths, ports, release identifiers, Git commit hashes and architecture details.

It must never contain secret values including passwords, API keys, tokens, private SSH keys, session secrets or credentials.

## Source repository

The JRM application source repository is private:

`jrm-project/jrm`

Do not assume access to the private repository merely because this public context repository is accessible.
