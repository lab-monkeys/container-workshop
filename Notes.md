# Notes

NEXUS_CERT=$(openssl s_client -showcerts -connect ${LOCAL_REGISTRY} </dev/null 2>/dev/null|openssl x509 -outform PEM)

oc create configmap nexus-registry -n openshift-config --from-literal=${LOCAL_REGISTRY//:/..}=${NEXUS_CERT}

oc patch image.config.openshift.io/cluster --type=merge --patch '{"spec":{"additionalTrustedCA":{"name":"nexus-registry"}}}'                                         

NEXUS_CERT=$(openssl s_client -showcerts -connect ${LOCAL_REGISTRY} </dev/null 2>/dev/null|openssl x509 -outform PEM | while read line; do echo "    $line"; done)

cat << EOF | oc apply -n openshift-config -f -                                     
apiVersion: v1
kind: ConfigMap
metadata:
  name: lab-ca
data:
  ca-bundle.crt: |
    # Nexus Cert
${NEXUS_CERT}

EOF


oc patch proxy cluster --type=merge --patch '{"spec":{"trustedCA":{"name":"lab-ca"}}}'


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