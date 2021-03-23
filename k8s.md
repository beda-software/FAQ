How to setup k8s autocomplete and other useful information – see official [kubectl Cheat Sheet](https://kubernetes.io/docs/reference/kubectl/cheatsheet/)

First of all, export ENV with the path to k8s config:
```Shell
export KUBECONFIG=~/.kube/your-project-config
```

## Namespace

Get all namespaces
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

## Investigate problems

Find needed deploy and pod and describe them
```
kubectl get pod
kubectl describe pod/PODNAME
kubectl get deploy
kubectl describe deploy/DEPLOYNAME
```

Get logs
```
kubectl logs PODNAME
kubectl logs pod/PODNAME
kubectl logs deploy/DEPLOYNAME
```

## Connect to pod container in interactive mode / execute command
```
kubectl exec -it -n NAMESPACE deploy/DEPLOYNAME bash
```

## Show metrics
```
kubectl top nodes
kubectl top pod -n NAMESPACE
kubectl top pod POD_NAME --containers
```

## Create (run) job from existing one
```
kubectl create job --from=cronjob/JOB_NAME -n NAMESPACE manual-backup-`date +%s`
```

Wait for job completion (helpful for bash scripts)
```
kubectl wait --for=condition=complete job/YOUR_JOB_NAME --timeout=5m -n NAMESPACE
```

## Get all pods hosted on a particular node
```
kubectl get pods --all-namespaces -o wide --field-selector spec.nodeName=NODE_NAME
```

## Add basic auth
Create password
```
htpasswd -c auth USERNAME
```
Fill password. Then:
```
kubectl create secret generic basic-auth --from-file=auth -n NAMESPACE
```

Then find a deploy/statefulset, edit it (`kubectl edit deploy/DEPLOYNAME ...`) and add to `annotations`:
```
nginx.ingress.kubernetes.io/auth-type: basic
nginx.ingress.kubernetes.io/auth-secret: basic-auth
nginx.ingress.kubernetes.io/auth-realm: 'Authentication Required'
```


