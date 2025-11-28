FROM clickhouse/clickhouse-server:24-alpine

# Create config directory
RUN mkdir -p /etc/clickhouse-server/config.d

# Copy custom configuration (relative to repository root since that's the build context)
COPY clickhouse/config.xml /etc/clickhouse-server/config.d/config.xml

# Expose ports
EXPOSE 8123 9000

# Use the default entrypoint from the base image

