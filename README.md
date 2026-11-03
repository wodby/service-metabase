# Metabase service for Kubernetes on Wodby

Run Metabase as a reusable Kubernetes application service with Wodby.

This repository defines the Wodby service manifests and operational
configuration for Metabase.

- [Browse Wodby services](https://wodby.com/services)
- [Wodby service documentation](https://wodby.com/docs/2.0/services/)
- [Service manifest reference](https://wodby.com/docs/2.0/services/template/)

## Wodby stacks using this service

- [Metabase application stack](https://github.com/wodby/stack-metabase)

## Service overview

| Property | Manifest configuration |
| --- | --- |
| Service name | `metabase` |
| Type | Application service |
| Versions | `0.65` by default, `0.64` |
| Workloads | `main` (Deployment, primary) |
| Containers | `metabase` using `metabase/metabase` |
| Endpoints | `metabase`: HTTP 3000 (main) |
| Service links | PostgreSQL (`postgres`), required; SMTP server (`sendmail`) |
| Application build | Not buildable from application source |
| Helm | chart `oci://registry-1.docker.io/wodby/stateless`; version `0.4.0` |

## Use this service

Use this service through [Metabase application stack](https://github.com/wodby/stack-metabase), or reference
`metabase` from a custom Wodby stack.

A service is a reusable component and does not deploy by itself. The stack
defines its links, settings, versions, resources, and relationship to the rest
of the application.

## Maintain a custom version

1. Fork this repository.
2. Edit the service manifest and referenced files.
3. Import the repository as a [Git-backed service](https://wodby.com/docs/2.0/services/create/#create-a-git-backed-service).
4. Reference the service from a stack manifest.

Keep service, workload, container, endpoint, link, volume, config, and
derivative names stable unless dependent stacks and app-level overrides are
updated at the same time.

Validate the manifests with:

```bash
wodby service validate-manifest service.yml --org <org-id> --include GUIDANCE.md
```

See the [service manifest reference](https://wodby.com/docs/2.0/services/template/) and the [managed services index](https://github.com/wodby/services).

## First administrator

The first deployment creates the Metabase administrator from the service's
required Admin email and Admin password settings, so Metabase's setup page is
never open to the public. The pod reports ready only after the administrator
exists. A password Metabase refuses stops the container with Metabase's reason
in the log. Changing the settings later does not change an existing account.

## Upgrades

Metabase migrates its application database when a newer version starts, and
older versions can't use a migrated database. Deployments stop every replica
before the new version starts. Back up the PostgreSQL database before moving
to a newer version; moving back to an older version is not supported.
