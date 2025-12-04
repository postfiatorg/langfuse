# ClickHouse for Langfuse on Fly.io

This directory contains the ClickHouse configuration for deploying Langfuse on Fly.io.

## Files

- `Dockerfile` - Custom Docker image based on ClickHouse Alpine
- `config.xml` - ClickHouse server configuration

## Deployment Steps

### 1. Create the Volume (First Time Only)

```bash
fly volumes create clickhouse_data_iad --region iad --size 10 --app langfuse-ch
```

### 2. Set the Password

```bash
# Generate a secure password
CH_PASSWORD=$(openssl rand -base64 32)

# Set it as a secret
fly secrets set --app langfuse-ch CLICKHOUSE_PASSWORD="$CH_PASSWORD"

# Save it for later use in Langfuse API configuration
echo "ClickHouse Password: $CH_PASSWORD"
```

### 3. Deploy ClickHouse

```bash
# From the repository root
fly deploy --config fly-ch.toml
```

### 4. Verify Deployment

```bash
# Check the logs
fly logs --app langfuse-ch

# Check if it's running
fly status --app langfuse-ch

# Test the connection (from inside another Fly app or SSH)
curl http://langfuse-ch.internal:8123/ping
```

## Connection Details

- **HTTP Interface**: `http://langfuse-ch.internal:8123`
- **Native Interface**: `tcp://langfuse-ch.internal:9000`
- **User**: `default`
- **Password**: Set via `CLICKHOUSE_PASSWORD` secret
- **Database**: `default`

## Configuration for Langfuse API

Add these secrets to your Langfuse API app:

```bash
fly secrets set --app langfuse-api \
  CLICKHOUSE_URL="http://langfuse-ch.internal:8123" \
  CLICKHOUSE_USER="default" \
  CLICKHOUSE_PASSWORD="your-password-from-step-2" \
  CLICKHOUSE_CLUSTER_ENABLED="false"
```

## Troubleshooting

### Check if ClickHouse is responding

```bash
fly ssh console --app langfuse-ch
# Inside the container:
curl http://localhost:8123/ping
```

### View logs

```bash
fly logs --app langfuse-ch
```

### Restart the app

```bash
fly apps restart langfuse-ch
```

## Storage

ClickHouse data is stored in a persistent volume mounted at `/var/lib/clickhouse`.

To resize the volume:
```bash
fly volumes extend <volume-id> --size 20 --app langfuse-ch
```

To list volumes:
```bash
fly volumes list --app langfuse-ch
```

