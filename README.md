# Firelink Proxy

Ingress proxy for the Firelink project: a web GUI for [Bonfire][bonfire]. See the
[Firelink backend][firelink-backend] and [Firelink frontend][firelink-frontend] for the application
components this proxy sits in front of.

Firelink Proxy is a single OpenShift pod running two containers in a sidecar pattern. An OpenShift
OAuth Proxy handles authentication and TLS, then forwards requests to a Caddy reverse proxy that
routes `/api/*` to the backend service and everything else to the frontend service. This ensures
both the frontend and backend are behind OAuth without either needing to implement the OAuth flow
themselves.

## Prerequisites

- Access to an OpenShift cluster
- The `firelink-proxy` Secret provisioned in the target namespace (see `deploy/secret.yaml` for the
  required format)
- The [Firelink backend][firelink-backend] and [Firelink frontend][firelink-frontend] services
  deployed in the same namespace

## Deploying

Deploy with the OpenShift template:

```bash
oc apply -f deploy/secret.yaml -n $NS
oc process -f deploy/deploy.yaml \
  -p IMAGE="quay.io/rh_ee_addrew/firelink-proxy" \
  -p IMAGE_TAG="latest" \
  -p INGRESS_DOMAIN="apps.ocp4.example.com" \
  | oc apply -n $NS -f -
```

The template accepts these parameters:

| Parameter | Default | Description |
|---|---|---|
| `IMAGE` | `quay.io/rh_ee_addrew/firelink-proxy` | Container image |
| `IMAGE_TAG` | `latest` | Image tag |
| `BACKEND_SERVICE` | `firelink-backend` | Backend service name |
| `BACKEND_PORT` | `8000` | Backend service port |
| `FRONTEND_SERVICE` | `firelink-frontend` | Frontend service name |
| `FRONTEND_PORT` | `8000` | Frontend service port |
| `SECRET_NAME` | `firelink-proxy` | OAuth client secret name |
| `INGRESS_DOMAIN` | `apps.ocp4.example.com` | Domain for the Route hostname |

The deployed Route will be available at `firelink.${INGRESS_DOMAIN}`.

## Building

```bash
docker build -t firelink-proxy:latest .
```

The Dockerfile copies the Caddyfile into a UBI-based Caddy image. Caddy listens on port 8000 with
auto-HTTPS disabled.

To build and push to Quay using the local build script (requires `QUAY_USER`, `QUAY_TOKEN`,
`RH_REGISTRY_USER`, and `RH_REGISTRY_TOKEN` environment variables):

```bash
./build_deploy.sh
```

The script tags the image with the short git commit hash and pushes to
`quay.io/cloudservices/firelink-proxy`.

## Architecture

For details on the sidecar container architecture, OAuth Proxy configuration, Caddy routing rules,
and design decisions, see the [architecture documentation][architecture].

[bonfire]: https://github.com/RedHatInsights/bonfire
[firelink-backend]: https://github.com/RedHatInsights/firelink-backend
[firelink-frontend]: https://github.com/RedHatInsights/firelink-frontend
[architecture]: ./ARCHITECTURE.md
