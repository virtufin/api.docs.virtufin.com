# Virtufin API

📖 Documentation: [api.doc.virtufin.com](https://api.doc.virtufin.com)

High-performance gRPC API gateway with event streaming, service discovery, and Dapr integration.

## Purpose

Virtufin API provides a **unified gateway** for accessing various Dapr services through a single API surface. Instead of connecting directly to multiple Dapr sidecars, clients interact with the Virtufin API gateway, which:

- **Discovers** available services and their gRPC methods via gRPC reflection
- **Invokes** methods on any registered Dapr service dynamically
- **Streams** state changes, pub/sub events, and system events to subscribers
- **Transcodes** REST/JSON requests to gRPC and vice versa

### Requirements for Dapr Services

For a Dapr service to be accessible through Virtufin API:

1. **gRPC Exposure**: The service must expose its methods via gRPC (not REST-only)
2. **gRPC Reflection**: The service must have **gRPC reflection enabled** so that Virtufin can discover its method schemas at runtime
3. **Registration**: The service must register with Virtufin API (via configuration or dynamic discovery)
4. **Unary Methods Only**: Only unary gRPC methods should be exposed (no server streaming, no client streaming, no bidirectional streaming)
5. **Streaming Alternatives**: If streaming behavior is required, use either:
   - The API client to publish/subscribe to pub/sub events directly
   - Change state and subscribe to state changes via the `WatchStateChanges` subscription type

Without gRPC reflection enabled, Virtufin cannot determine the method signatures needed for JSON transcoding and dynamic invocation.

## Documentation

### Building Documentation Locally

```bash
# Install dependencies
pip install -r requirements_docs.txt

# Serve locally (http://localhost:8000)
mkdocs serve -f mkdocs_v0.yml

# Build for self-hosting
mkdocs build -f mkdocs_v0.yml -d site/

# Version management
mike deploy v0 -p                    # Deploy v0 version
mike set-default v0 -p                # Set v0 as default
mike deploy v1 -p                     # Deploy new version
mike list                             # List all versions
```

## Deployment

All deployment infrastructure is managed in dedicated repositories.

### Local (Native Dapr)

Run services natively with Dapr sidecars. No Docker containers for services. Requires Dapr CLI, .NET SDK, and Redis.

```bash
git clone https://git.haenerconsulting.com/virtufin/deploy-local.git
cd deploy-local
./scripts/deploy_backend.sh
```

See [deploy-local](https://git.haenerconsulting.com/virtufin/deploy-local) for full documentation.

### Docker Dev

Build and run from source via Docker Compose.

```bash
git clone https://git.haenerconsulting.com/virtufin/docker-compose.git
cd docker-compose
./scripts/deploy_backend.dev.sh
```

### Docker Prod

Run pre-built Harbor images via Docker Compose.

```bash
git clone https://git.haenerconsulting.com/virtufin/docker-compose.git
cd docker-compose
./scripts/deploy_backend.prod.sh
```

See [docker-compose](https://git.haenerconsulting.com/virtufin/docker-compose) for full documentation.

### Kubernetes

Deploy with the Virtufin Helm chart. Requires Dapr installed on the cluster.

```bash
helm repo add virtufin https://helm.haenerconsulting.com/virtufin/helm
helm install virtufin virtufin/virtufin \
  --namespace virtufin \
  --create-namespace \
  --set stateStore.host=<redis-host> \
  --set pubsub.host=<redis-host>
```

See [helm](https://helm.haenerconsulting.com/virtufin/helm) for all configurable values.

## Project Structure

```
virtufin-api/
├── src/
│   ├── Virtufin.Api/               # .NET API Gateway service
│   ├── Virtufin.Api.Client/        # .NET client NuGet package
│   ├── Virtufin.Api.Protos/        # Protobuf definitions (gateway, pubsub, config, state)
│   ├── python/virtufin/api/        # Python client library (PubSub, service discovery)
│   └── typescript/src/             # TypeScript client library (ApiClient, publishWithResult)
├── tests/
│   ├── python/                     # Python client tests
│   ├── typescript/                 # TypeScript client tests
│   └── Virtufin.Api.*.Tests/       # .NET integration and unit tests
├── docs/                           # MkDocs documentation (api.doc.virtufin.com)
├── scripts/                        # Helper scripts
├── deploy/                         # Deployment configs
├── versions.env                    # API_VERSION and LIBRARY_VERSION pin
└── AGENTS.md                       # Agent documentation

## API Endpoints

| Protocol | Port | Description |
|----------|------|-------------|
| HTTP | 5001 | REST API (JSON transcoding) |
| gRPC | 5002 | gRPC API |

## Features

- **gRPC Gateway**: Dynamic service discovery and invocation
- **Event Streaming**: Server-side streaming via `Subscribe` and `WatchSystemEvents`
- **State Management**: `SaveState` broadcasts events to all subscribers
- **Dapr Integration**: Pub/sub, state store, and service invocation
- **Client Libraries**: C# and Python with async support

## Development

See [docs/v0/development.md](docs/v0/development.md) for development setup.

## License

Proprietary - Virtufin