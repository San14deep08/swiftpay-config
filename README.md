# swiftpay-config

Centralised configuration for **SwiftPay**, served by `config-server` (Spring Cloud Config)
to the four client applications.

Main repository: https://github.com/San14deep08/swiftpay

## Layout

| File | Applies to |
|---|---|
| `application.yml` | every client — shared actuator, Eureka, Jackson and logging settings |
| `gateway-service.yml` | Service A — Transaction Gateway |
| `ledger-service.yml` | Service B — Ledger Service |
| `analytics-worker.yml` | Service C — Analytics Worker (bonus) |
| `api-gateway.yml` | Spring Cloud Gateway edge service |

Spring Cloud Config resolves a client's configuration by its `spring.application.name`, so
the filename must match it exactly. `application.yml` is merged underneath, with the
service-specific file taking precedence.

## This repository is public. No secrets.

Database passwords, tokens and any other credential are supplied as **environment
variables** by `docker-compose.yml` and the Kubernetes manifests — never committed here.
Every value in these files is either a non-sensitive default or an `${ENV_VAR:default}`
placeholder.

The repository is public deliberately: a reviewer cloning SwiftPay has no credential of
ours, and a config server that cannot authenticate turns into a failed `docker compose up`
with a cause that is hard to diagnose. Public config plus environment-variable secrets is
the combination that survives a clean clone.

## Changes take effect only after push

The config server reads from the GitHub **remote**, not from a local working copy. A change
is live only once it is committed *and* pushed. Clients read configuration at startup, so a
service must also be restarted to pick it up — there is no `/refresh` bus wired up, which
is a known limitation recorded in the main repository's README.

## Verifying the server is serving this repo

```bash
# shared config
curl http://localhost:8888/application/default

# a service's merged view — this is exactly what the client receives
curl http://localhost:8888/gateway-service/default
```

An empty `propertySources` array means the server started but is serving nothing: wrong
branch (`default-label`), wrong URI, or the file name does not match the client's
`spring.application.name`.
