# Ingest: OTLP over HTTP (4318) and gRPC (4317) into ClickHouse. Pulls its
# pipeline config from the app over OpAMP, so there is no config file here.
FROM hyperdx/hyperdx-otel-collector:2.35.0

# Legacy schema is what the app's DEFAULT_SOURCES below point at.
ENV HYPERDX_OTEL_EXPORTER_CREATE_LEGACY_SCHEMA=true \
    HYPERDX_OTEL_EXPORTER_CLICKHOUSE_DATABASE=default
