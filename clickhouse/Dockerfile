# Telemetry store: logs, traces, metrics, session replays. The bulk of the
# disk and RAM. Stock image — upstream's compose replaces config.xml wholesale,
# but that file is their local-dev one (10GB memory cap, passwordless default
# user); the stock image plus a password is the right shape for Railway and
# already ships <listen_host>::</listen_host> for the IPv6 private network.
FROM clickhouse/clickhouse-server:25.3

# Logs to the console instead of 10 x 1 GB files, and ClickHouse's own
# telemetry tables (trace_log, metric_log, ...) off: they write every few
# seconds and their merges keep idle memory climbing (~1 GB in 3 h).
COPY railway.xml /etc/clickhouse-server/config.d/railway.xml
# The stock file listens on :: and 0.0.0.0; on Railway :: is already
# dual-stack, so the second bind fails with an "Address already in use"
# warning on every boot. Overwritten, not deleted.
COPY listen.xml /etc/clickhouse-server/config.d/docker_related_config.xml

# Railway hosts run transparent huge pages "always": jemalloc frees 4K pages
# inside 2 MB huge pages that stay charged, so idle ClickHouse creeps ~25 MB/min.
ENV CLICKHOUSE_DB=default \
    CLICKHOUSE_USER=hyperdx \
    MALLOC_CONF=thp:never
