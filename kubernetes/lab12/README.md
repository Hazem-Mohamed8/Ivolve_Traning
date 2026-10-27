# Lab 12: Managing Configuration and Sensitive Data with ConfigMaps and Secrets

## Steps

**1. Encode the Secret values**

The `-n` flag matters: without it, `echo` adds a newline that ends up inside the password.

```bash
echo -n 'root123' | base64     # cm9vdDEyMw==
echo -n 'user123' | base64     # dXNlcjEyMw==
```

**2. Create the ConfigMap**

`configmap.yaml`

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: ivolvecm
  namespace: ivolve
data:
  DB_HOST: ivolvedbsvc
  DB_USER: ivolveuser
```

```bash
kubectl apply -f configmap.yaml
```

**3. Create the Secret**

`secret.yaml`

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: ivolvesecret
  namespace: ivolve
type: Opaque
data:
  MYSQL_ROOT_PASSWORD: cm9vdDEyMw==
  DB_PASSWORD: dXNlcjEyMw==
```

```bash
kubectl apply -f secret.yaml
```

## Verification

```bash
kubectl get configmap,secret -n ivolve
```

```
NAME                         DATA   AGE
configmap/ivolvecm           2      15s
configmap/kube-root-ca.crt   1      3h38m

NAME                  TYPE     DATA   AGE
secret/ivolvesecret   Opaque   2      8s
```

Check the ConfigMap values:

```bash
kubectl describe configmap ivolvecm -n ivolve
```

```
Data
====
DB_HOST:
----
ivolvedbsvc

DB_USER:
----
ivolveuser
```

Decode a Secret value to confirm the encoding:

```bash
kubectl get secret ivolvesecret -n ivolve -o jsonpath='{.data.DB_PASSWORD}' | base64 -d
```

```
user123
```

