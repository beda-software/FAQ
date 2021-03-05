How to setup k8s autocomplete and other useful information – see official [kubectl Cheat Sheet](https://kubernetes.io/docs/reference/kubectl/cheatsheet/)

First of all, export ENV with the path to k8s config:
```Shell
export KUBECONFIG=~/.kube/juli
```

## Namespace

```Shell
kubectl get ns
```

Then you can work in particular namespace by adding `-n namespace-name` to every command or by specyfing current default namespace:
```Shell
kubectl get pod -n namespace-name
kubectl edit secret/app-secret -n namespace-name

# same result by setting default namespace
kubectl config set-context --current --namespace=namespace-name
kubectl get pod
kubectl edit secret/app-secret
```

Or you can get resources across all namespaces:
```Shell
kubectl get pods --all-namespaces
kubectl get services --all-namespaces
``` 

## Secrets

### Get secret
```Shell
kubectl get secret/some-secret -n namespace-name -o yaml
```

### Edit secret
```Shell
kubectl edit secret/some-secret -n namespace-name
```

> Sometimes you need to "restart" pod after changing a secret to use new version of secret.
> First option is to delete a pod. Find a pod(s): `kubectl get pod -n some-namespace`, delete it `kubectl delete pod-name -n some-namespace`.
> Second option:
```Shell
kubectl -n ... scale deploy DEPLOYNAME --replicas=0
kubectl -n ... scale deploy DEPLOYNAME --replicas=1
```


