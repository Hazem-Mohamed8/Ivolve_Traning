# Lab 11: Namespace Management and Resource Quota Enforcement

## Steps

**1. Create the namespace**

```bash
kubectl create namespace ivolve
```

**2. Create the ResourceQuota**

`quota.yaml`

```yaml
apiVersion: v1
kind: ResourceQuota
metadata:
  name: ivolvequota
  namespace: ivolve
spec:
  hard:
    pods: "3"
```

```bash
kubectl apply -f quota.yaml
```

## Verification

```bash
kubectl describe resourcequota ivolvequota -n ivolve
```

![Quota applied](images/2)

```bash
kubectl create deployment nginx-test --image=nginx --replicas=3 -n ivolve
```

![pod4 rejected](images/1)

```bash
kubectl describe resourcequota ivolvequota -n ivolve
```

![Quota full](images/3)
