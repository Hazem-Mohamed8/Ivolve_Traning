# Lab 17: Control Pod-to-Pod Traffic via Network Policy

## Requirements

| Item         | Value                                                                             |
| ------------ | --------------------------------------------------------------------------------- |
| Name         | `ivolvenp` (Kubernetes names must be lowercase, so `ivolveNP` becomes `ivolvenp`) |
| Pod selector | MySQL pod (`app=mysql`)                                                           |
| Policy types | Ingress only                                                                      |
| Ingress rule | Allow only from application pods (`app=ivolveapp`)                                |
| Port         | TCP 3306                                                                          |

## Manifest

`ivolvenp.yaml`:

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: ivolvenp
  namespace: ivolve
spec:
  podSelector:
    matchLabels:
      app: mysql
  policyTypes:
    - Ingress
  ingress:
    - from:
        - podSelector:
            matchLabels:
              app: ivolveapp
      ports:
        - protocol: TCP
          port: 3306
```

## Steps

### 1. Check the pod labels

The policy selectors must match the real labels, so check them first:

```bash
kubectl get pods -n ivolve --show-labels
```

Output:

```
NAME                         READY   STATUS    LABELS
ivolveapp-7b7f87b579-g9r9s   1/1     Running   app=ivolveapp,pod-template-hash=7b7f87b579
ivolvedb-0                   1/1     Running   app=mysql,apps.kubernetes.io/pod-index=0,...
```

### 2. Apply the policy

```bash
kubectl apply -f ivolvenp.yaml
kubectl describe netpol ivolvenp -n ivolve
```

Expected `describe` output:

```
Name:         ivolvenp
Namespace:    ivolve
Spec:
  PodSelector:     app=mysql
  Allowing ingress traffic:
    To Port: 3306/TCP
    From:
      PodSelector: app=ivolveapp
  Not affecting egress traffic
  Policy Types: Ingress
```

### 3. Test: traffic from the application pod (should be allowed)

```bash
kubectl exec -n ivolve deploy/ivolveapp -c app -- sh -c 'nc -zv -w 3 ivolvedbsvc 3306'
```

Result:

```
ivolvedbsvc (192.168.235.145:3306) open
```

### 4. Test: traffic from an unlabeled pod (should be blocked)

The worker node has the taint `workload=ivolveApp:NoSchedule` and the label `project=ivolve`. The test pod therefore needs a matching node selector and toleration to be scheduled:

```bash
kubectl run test --rm -it --image=busybox -n ivolve \
  --overrides='{"spec":{"nodeSelector":{"project":"ivolve"},"tolerations":[{"key":"workload","operator":"Equal","value":"ivolveApp","effect":"NoSchedule"}]}}' \
  -- nc -zv -w 3 ivolvedbsvc 3306
```

Result:

```
nc: ivolvedbsvc (192.168.235.145:3306): Connection timed out
```
