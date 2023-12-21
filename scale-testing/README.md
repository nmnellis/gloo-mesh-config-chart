


## We could possible remove the top-level route table if we could default to Table weight `sortMethod: TABLE_WEIGHT`

## is it more expensive to store the CRDs in a workload cluster vs the management plane cluster?

* Example Route Table provided by customer

```yaml
# The client workspace needs to create a root RouteTable and select the following sub-tables.
---
apiVersion: networking.gloo.solo.io/v2
kind: RouteTable
metadata:
  name: grpc-mesh-reference-service-infra-rt
  namespace: grpc-mesh-reference-service-gloo--infra
  labels:
    soa.solo.com/route-table: grpc-mesh-reference-service
spec:
  weight: 10
  http:
    # For a service with one zone "zone-a" & one pool "pool-a". This would expand to 4 combinations
    # {zone-a, pool-a}, {zone-a, default}, {default, pool-a}, {default, default}
    {{- range $zone, $pool := .Values.zoneAndPools }}
    - labels:
        route: default-5xx-retry
        service: grpc-mesh-reference-service
      matchers:
        - headers:
            - name: "X-Idempotency-Key"
            - name: ":authority"
              value: "infra.grpc-mesh-reference-service-gloo.service.mesh:8080"
            - name: soa-zone
              value: $zone
            - name: soa-pool
              value: $pool
          port: 8080
      forwardTo:
        destinations:
          - ref:
              name: grpc-mesh-reference-service
              namespace: grpc-mesh-reference-service-gloo--infra
            kind: VIRTUAL_DESTINATION
            port:
              number: 8080
            subset:
              soa.solo.com/isolated-env: infra
              soa.solo.com/zone: $zone
              soa.solo.com/pool: $pool
    - labels:
        route: default-route-with-retry
        service: grpc-mesh-reference-service
      matchers:
        - port: 8080
        - headers:
            - name: ":authority"
              value: "infra.grpc-mesh-reference-service-gloo.service.mesh:8080"
            - name: soa-zone
              value: $zone
            - name: soa-pool
              value: $pool
      forwardTo:
        destinations:
          - ref:
              name: grpc-mesh-reference-service
              namespace: grpc-mesh-reference-service-gloo--infra
            kind: VIRTUAL_DESTINATION
            port:
              number: 8080
            subset:
              soa.solo.com/isolated-env: infra
              soa.solo.com/zone: $zone
              soa.solo.com/pool: $pool
    - labels:
        route: default-5xx-retry
        service: grpc-mesh-reference-service
      matchers:
        - headers:
            - name: "X-Idempotency-Key"
            - name: ":authority"
              value: "infra.grpc-mesh-reference-service-gloo.service.mesh:8080"
            - name: soa-zone
              value: $zone
            - name: soa-pool
              value: $pool
          port: 8083
      forwardTo:
        destinations:
          - ref:
              name: grpc-mesh-reference-service
              namespace: grpc-mesh-reference-service-gloo--infra
            kind: VIRTUAL_DESTINATION
            port:
              number: 8083
            subset:
              soa.solo.com/isolated-env: infra
              soa.solo.com/zone: $zone
              soa.solo.com/pool: $pool
    - labels:
        route: default-route-with-retry
        service:
          grpc-mesh-reference-service
      matchers:
        - port: 8083
        - headers:
            - name: ":authority"
              value: "infra.grpc-mesh-reference-service-gloo.service.mesh:8080"
            - name: soa-zone
              value: $zone
            - name: soa-pool
              value: $pool
      forwardTo:
        destinations:
          - ref:
              name: grpc-mesh-reference-service
              namespace: grpc-mesh-reference-service-gloo--infra
            kind: VIRTUAL_DESTINATION
            port:
              number: 8083
            subset:
              soa.solo.com/isolated-env: infra
              soa.solo.com/zone: $zone
              soa.solo.com/pool: $pool
    {{- end }}
    # Default route for requests don't match with any zones & pools. Might not be necessary if we always have the
    # correct request headers and pod labels.
    - labels:
        route: default-5xx-retry
        service: grpc-mesh-reference-service
      matchers:
        - headers:
            - name: "X-Idempotency-Key"
            - name: ":authority"
              value: "infra.grpc-mesh-reference-service-gloo.service.mesh:8080"
          port: 8080
      forwardTo:
        destinations:
          - ref:
              name: grpc-mesh-reference-service
              namespace: grpc-mesh-reference-service-gloo--infra
            kind: VIRTUAL_DESTINATION
            port:
              number: 8080
            subset:
              soa.solo.com/isolated-env: infra
              soa-default: default
    - labels:
        route: default-route-with-retry
        service: grpc-mesh-reference-service
      matchers:
        - port: 8080
        - headers:
            - name: ":authority"
              value: "infra.grpc-mesh-reference-service-gloo.service.mesh:8080"
      forwardTo:
        destinations:
          - ref:
              name: grpc-mesh-reference-service
              namespace: grpc-mesh-reference-service-gloo--infra
            kind: VIRTUAL_DESTINATION
            port:
              number: 8080
            subset:
              soa.solo.com/isolated-env: infra
              soa-default: default
    - labels:
        route: default-5xx-retry
        service: grpc-mesh-reference-service
      matchers:
        - headers:
            - name: "X-Idempotency-Key"
            - name: ":authority"
              value: "infra.grpc-mesh-reference-service-gloo.service.mesh:8080"
          port: 8083
      forwardTo:
        destinations:
          - ref:
              name: grpc-mesh-reference-service
              namespace: grpc-mesh-reference-service-gloo--infra
            kind: VIRTUAL_DESTINATION
            port:
              number: 8083
            subset:
              soa.solo.com/isolated-env: infra
              soa-default: default
    - labels:
        route: default-route-with-retry
        service:
          grpc-mesh-reference-service
      matchers:
        - port: 8083
        - headers:
            - name: ":authority"
              value: "infra.grpc-mesh-reference-service-gloo.service.mesh:8080"
      forwardTo:
        destinations:
          - ref:
              name: grpc-mesh-reference-service
              namespace: grpc-mesh-reference-service-gloo--infra
            kind: VIRTUAL_DESTINATION
            port:
              number: 8083
            subset:
              soa.solo.com/isolated-env: infra
              soa-default: default

```