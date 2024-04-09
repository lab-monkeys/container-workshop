cat <<EOF >$HOME/netpol/allow-same-namespace.yml
---
kind: NetworkPolicy
apiVersion: networking.k8s.io/v1
metadata:
  name: allow-same-namespace
spec:
  podSelector: {}
  ingress:
  - from:
    - podSelector: {}
EOF

oc create -n cakephp-example -f $HOME/netpol/allow-same-namespace.yml

cat <<EOF >$HOME/netpol/allow-openshift-ingress.yml
---
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-openshift-ingress
spec:
  podSelector: {}
  ingress:
  - from:
    - namespaceSelector:
        matchLabels:
          network.openshift.io/policy-group: ingress
EOF

oc create -n cakephp-example -f $HOME/netpol/allow-openshift-ingress.yml

## Bug: https://bugzilla.redhat.com/show_bug.cgi?id=1768608
oc patch namespace default --type=merge -p '{"metadata": {"labels": {"network.openshift.io/policy-group": "ingress"}}}'

git clone https://github.com/redhat-gpte-devopsautomation/cakephp-ex.git

- apiVersion: networking.k8s.io/v1
  kind: NetworkPolicy
  metadata:
    name: allow-same-namespace
  spec:
    podSelector: {}
    ingress:
    - from:
      - podSelector: {}
- apiVersion: networking.k8s.io/v1
  kind: NetworkPolicy
  metadata:
    name: allow-from-openshift-ingress
  spec:
    podSelector: {}
    ingress:
    - from:
      - namespaceSelector:
          matchLabels:
            network.openshift.io/policy-group: ingress