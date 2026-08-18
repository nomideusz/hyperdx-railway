# Telemetry store: logs, traces, metrics, session replays. The bulk of the
# disk and RAM. Stock image — upstream's compose replaces config.xml wholesale,
# but that file is their local-dev one (10GB memory cap, passwordless default
# user); the stock image plus a password is the right shape for Railway and
# already ships <listen_host>::</listen_host> for the IPv6 private network.
FROM clickhouse/clickhouse-server:25.3

ENV CLICKHOUSE_DB=default \
    CLICKHOUSE_USER=hyperdx
