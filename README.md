# Container Workshop Script

## Container Basics

1. Create a Quarkus Project

   ```bash
   cd /projects
   quarkus create
   cd code-with-quarkus
   ```

1. Add project to workspace

1. Explore Dockerfile.jvm

1. Build Project

   ```bash
   mvn package
   ```

1. Build container image

   ```bash
   podman build -t quarkus:test -f /src/main/docker/Dockerfile.jvm .
   ```

1. Run the container locally

   ```bash
   podman run -d --rm --name quarkus -p 8080:8080 quarkus:test
   ```

1. Tag the container for Nexus

   ```bash
   podman tag quarkus:test nexus.clg.lab:5001/clg/quarkus:new
   ```

1. Log into Nexus

   ```bash
   podman login nexus.clg.lab:5001 --tls-verify=false
   ```

1. Push the image to Nexus

   ```bash
   podman push nexus.clg.lab:5001/clg/quarkus:new --tls-verify=false
   ```

1. Create a Pod

oc new-project quarkus-app

```bash
cat << EOF | oc apply -f -
apiVersion: v1
kind: Pod
metadata:
 name: quarkus-demo
 namespace: quarkus-app
spec:
  containers:
  - name: quarkus-container
    image: nexus.clg.lab:5001/clg/quarkus:new
EOF
```

oc new-app nexus.clg.lab:5001/clg/quarkus:new -n quarkus-app