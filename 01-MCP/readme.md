# 01-MCP: Front MCP Server with Agent Gateway

## Prerequisites

- kagent installed with kmcp enabled (see `00-Install/readme.md`)
- Agent Gateway OSS v1.1.0 installed (see `00-Install/readme.md`)
- MCP server deployed to kagent

## Agent Gateway Config

### 1. Gateway

```bash
kubectl apply -f - <<'EOF'
apiVersion: gateway.networking.k8s.io/v1
kind: Gateway
metadata:
  name: agentgateway-proxy
  namespace: agentgateway-system
spec:
  gatewayClassName: agentgateway
  listeners:
    - name: http
      protocol: HTTP
      port: 80
      allowedRoutes:
        namespaces:
          from: All
EOF
```

### 2. MCP Backend

Routes to the kagent-managed MCP server via cluster DNS.

```bash
kubectl apply -f - <<'EOF'
apiVersion: agentgateway.dev/v1alpha1
kind: AgentgatewayBackend
metadata:
  name: ops-server
  namespace: agentgateway-system
spec:
  mcp:
    targets:
      - name: ops-server
        static:
          host: sebastianmaniak-ops-server-4e726047.kagent.svc.cluster.local
          port: 3000
          protocol: SSE
EOF
```

### 3. HTTPRoute

```bash
kubectl apply -f - <<'EOF'
apiVersion: gateway.networking.k8s.io/v1
kind: HTTPRoute
metadata:
  name: ops-server
  namespace: agentgateway-system
spec:
  parentRefs:
    - name: agentgateway-proxy
      namespace: agentgateway-system
  rules:
    - matches:
        - path:
            type: PathPrefix
            value: /mcp
      backendRefs:
        - name: ops-server
          namespace: agentgateway-system
          group: agentgateway.dev
          kind: AgentgatewayBackend
EOF
```

## Verify

```bash
# Check resources
kubectl get gateway,agentgatewaybackend,httproute -n agentgateway-system

# Port-forward and test
kubectl port-forward deployment/agentgateway-proxy -n agentgateway-system 8080:80 &
curl localhost:8080/mcp/sse
```
