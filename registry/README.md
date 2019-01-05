Deploy Registry
----------------
Have not yet been able to connect to the built-in minikube registry, therefore including this quick easy deployment of a standalone registry to connect to from minikube's host. **Make sure you disable minikube's internal registry first.**

Deploy registry:
```
kubectl create -f kube-registry.yaml
```

Test deployment:
```
minikube ssh && curl -v localhost:5000
```

Map the host port 5000 to minikube registry pod
```
kubectl port-forward --namespace kube-system \
$(kubectl get po -n kube-system | grep kube-registry-v0 | \
awk '{print $1;}') 5000:5000
```

Test connectivity from host
```
curl -v localhost:5000
```

License
-------

BSD

Author Information
------------------

An optional section for the role authors to include contact information, or a website (HTML is not allowed).
