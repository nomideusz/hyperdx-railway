# Metadata: users, dashboards, saved searches, alert rules. Small.
# Pinned to what upstream's compose pins — HyperDX has not been validated
# against Mongo 6/7 by upstream, and this is not the place to find out.
FROM mongo:5.0.32-focal

# --ipv6 is mandatory on Railway and easy to miss: `--bind_ip_all`, which the
# official entrypoint adds on its own, binds 0.0.0.0 only. Railway's private
# network is IPv6-only, so without this mongod listens on an address the app
# can never reach, and the only symptom is a connection timeout.
CMD ["mongod", "--ipv6"]
