# Deploy and Host HyperDX on Railway

[![Deploy on Railway](https://railway.com/button.svg)](https://railway.com/new/template/hyperdx?utm_medium=integration&utm_source=button&utm_campaign=hyperdx)

[HyperDX](https://hyperdx.io/) — the open-source core of ClickHouse's ClickStack — unifies logs, traces, metrics, session replay, and errors over ClickHouse. It correlates a browser session with the exact backend trace and the logs around it, so a bug report turns into a root cause without hopping between four tools.

**Read this before deploying: ingest stays closed until you create your account.** The collector pulls its pipeline over OpAMP from the app, and that pipeline is keyed to your team's ingestion API key — which does not exist until the first user registers. Until then the collector runs but never opens its OTLP ports, and telemetry sent to it is refused. Open the app's domain and register first; ingest comes up within about half a minute.

## About Hosting HyperDX

This template runs four services so each can be sized on its own. ClickHouse holds the telemetry and is the one that grows — it gets a volume and it is where your storage bill lives. MongoDB holds metadata: users, dashboards, saved searches, and alert rules, and stays small. The collector is the ingest edge, taking OTLP over HTTP and gRPC and writing to ClickHouse. The app serves the UI, the API, and the OpAMP server that hands the collector its config — three listeners in one container, only one of which needs a domain.

Two services get public domains here, which is unusual and deliberate: the app's for people, and the collector's for telemetry. Keeping them apart means your ingest endpoint is not the same hostname as your dashboard, and either can be swapped for a custom domain without touching the other.

## Common Use Cases

- Session replay tied to backend traces — watch what the user did, then jump to the request that failed.
- A Datadog or Sentry replacement for a small team, on infrastructure you already pay Railway for.
- A ClickHouse-native log store you can query with SQL directly when the UI's search is not enough.
- An OTLP endpoint for apps running anywhere, not only the ones inside this Railway project.

## Dependencies for HyperDX Hosting

- A ClickHouse service with a volume (created by this template) for telemetry.
- A MongoDB service with a volume (created by this template) for metadata.
- A Railway domain on the `app` service for the UI, and one on `otel-collector` for OTLP ingest.

### Deployment Dependencies

- [HyperDX documentation](https://www.hyperdx.io/docs)
- [HyperDX source](https://github.com/hyperdxio/hyperdx)
- [Template source on GitHub](https://github.com/nomideusz/hyperdx-railway)
- [ClickStack docs at ClickHouse](https://clickhouse.com/docs/use-cases/observability/clickstack)

### Implementation Details

**Four services from one repo, each a thin Dockerfile over an official image: `hyperdx/hyperdx:2.35.0`, `hyperdx/hyperdx-otel-collector:2.35.0`, stock ClickHouse, and stock MongoDB.**

1. Deploy, then open the `app` service's domain and register. The first account created becomes the owner and creates the team; there is no seeded admin.
2. The bundled ClickHouse is pre-wired as a connection and the four sources (Logs, Traces, Metrics, Sessions) are pre-defined, so there is nothing to configure after signup.
3. Send telemetry to the `otel-collector` service's domain, with your ingestion API key (Team Settings) in the `authorization` header. For an OpenTelemetry SDK: set `OTEL_EXPORTER_OTLP_ENDPOINT` to `https://<collector-domain>`, `OTEL_EXPORTER_OTLP_PROTOCOL` to `http/protobuf`, and `OTEL_EXPORTER_OTLP_HEADERS` to `authorization=<your key>`.
4. For apps inside this same Railway project, point them at `http://${{otel-collector.RAILWAY_PRIVATE_DOMAIN}}:4318` instead and keep the telemetry off the public internet.

**Things worth knowing before you deploy:**

- **ClickHouse decides your bill.** Session replay in particular is heavy. Set a TTL on the `otel_*` tables and sample your traces before pointing production at this.
- **Unauthenticated ingest is rejected.** The collector answers `401` without a valid ingestion key, so its public domain is safe to hand out to your own services.
- **MongoDB runs with `--ipv6`.** Railway's private network is IPv6-only and `--bind_ip_all` alone binds `0.0.0.0`, so without that flag the app cannot reach the database at all. Don't remove it.
- **The collector's `PORT` pins the routed port.** Its image exposes five ports, and `PORT=4318` is what tells Railway which one the domain belongs to.
- **Give ClickHouse room.** It is the memory-hungry service here; if queries get killed or ingest stalls, that is the service to scale.

## Why Deploy HyperDX on Railway?

Railway is a singular platform to deploy your infrastructure stack. Railway will host your infrastructure so you don't have to deal with configuration, while allowing you to vertically and horizontally scale it.

By deploying HyperDX on Railway, you are one step closer to supporting a complete full-stack application with minimal burden. Host your servers, databases, AI agents, and more on Railway.
