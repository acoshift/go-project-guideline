# Kubernetes Checklist

- Label

```yaml
metadata:
  labels:
    app: appname
```

- Pod Resources

```yaml
resoucres:
  requests:
    cpu: 100m
    memory: 200Mi
```

- Health Check

```yaml
livenessProbe:
  httpGet:
    path: /healthz
    port: 18080
    scheme: HTTP
  initialDelaySeconds: 10
  periodSeconds: 10
  successThreshold: 1
  failureThreshold: 3
  timeoutSeconds: 5
readinessProbe:
  httpGet:
    path: /healthz
    port: 18080
    scheme: HTTP
  periodSeconds: 1
  successThreshold: 1
  failureThreshold: 3
  timeoutSeconds: 1
```

- Deployment Strategy

```yaml
strategy:
  type: RollingUpdate
  rollingUpdate:
    maxSurge: 1
    maxUnavailable: 0
```

- Deployment Replicas

```yaml
replicas: 2 # set to node - 1
```

- Revision History Limit

```yaml
revisionHistoryLimit: 3
```

- Pod Anti-Affinity

```yaml
affinity:
  podAntiAffinity:
    preferredDuringSchedulingIgnoredDuringExecution:
    - podAffinityTerm:
        labelSelector:
          matchExpressions:
          - key: app
            operator: In
            values:
            - appname
        topologyKey: kubernetes.io/hostname
      weight: 100
```

or

```yaml
affinity:
  podAntiAffinity:
    requiredDuringSchedulingIgnoredDuringExecution:
    - labelSelector:
        matchExpressions:
        - key: app
          operator: In
          values:
          - appname
      topologyKey: kubernetes.io/hostname
```

- Pre-stop hook (if app not graceful shutdown)

```yaml
lifecycle:
  preStop:
    exec:
      command:
      - sleep
      - "10"
```

- Use External Name service if connect to external database (not in k8s cluster)

```yaml
apiVersion: v1
kind: Service
metadata:
  name: postgres
spec:
  type: ExternalName
  externalName: postgres.cluster.yourdomain.com
```

- Add minimum pod for kube-dns-autoscaler config

`$ kubectl edit cm/kube-dns-autoscaler -n kube-system`

set min to 2

```yaml
data:
  linear: '{"coresPerReplica":256,"min":2,"nodesPerReplica":16}'
```
