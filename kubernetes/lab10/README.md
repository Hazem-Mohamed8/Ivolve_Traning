# Lab 10: Node Isolation Using Taints and Labels in Kubernetes

```bash
kubectl get nodes
```

![Nodes](images/1.png)

## Steps

**1. Taint the worker node**

Key `workload`, value `ivolveApp`, effect `NoSchedule`. Pods without a matching toleration will not be scheduled on this node.

```bash
kubectl taint nodes worker1 workload=ivolveApp:NoSchedule
```

**2. Label the worker node**

```bash
kubectl label nodes worker1 project=ivolve
```

## Verification

Check the taint:

```bash
kubectl describe node worker1 | grep -i taint
```

![Taint verified](images/4.png)

Check the label:

```bash
kubectl get nodes --show-labels
```

![Label verified](images/2.png)

![Label verified](images/3.png)
