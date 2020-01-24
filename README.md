# kpack
Play with kpack

## 1. install kpack

1. [install](https://github.com/pivotal/kpack/blob/master/docs/install.md)
   ```bash
   kubectl apply -f https://github.com/pivotal/kpack/releases/download/v0.0.6/release-0.0.6.yaml
   ```
1. watch
   ```bash
   kubectl get pods --namespace kpack -w
   ```
1. create ClusterBuilder:
   ```bash
   vi cluster-builder.yaml
   ```
   ```yaml
   apiVersion: build.pivotal.io/v1alpha1
   kind: ClusterBuilder
   metadata:
     name: default
   spec:
     image: cloudfoundry/cnb:bionic
   ```
   ```bash
   kubectl apply -f cluster-builder.yaml
   sleep 10s
   kubectl describe clusterbuilder default
   ```

## 2. install kpack logs

[download](https://github.com/pivotal/kpack/releases/download/v0.0.6/logs-v0.0.6-darwin.tgz), unpack and add to PATH:

```bash
wget -qO- https://github.com/pivotal/kpack/releases/download/v0.0.6/logs-v0.0.6-darwin.tgz | tar xvz -
chmod +x logs
export PATH="$PATH:$(pwd)"
logs -image $IMAGE_NAME
```

## 3. create docker registry secret

```bash
vi secret.yaml
```

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: tutorial-registry-credentials
  annotations:
    build.pivotal.io/docker: https://index.docker.io/v1/
type: kubernetes.io/basic-auth
stringData:
  username: daggerok
  password: ololo-trololo
```

```bash
kubectl apply -f secret.yaml
```

## 4. create service account

```bash
vi service-account.yaml
```

```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
 name: tutorial-service-account
secrets:
 - name: tutorial-registry-credentials
```

```bash
kubectl apply -f service-account.yaml
```

## 5. apply kpack image config

```bash
vi spring-petclinic.yaml
```

```yaml
apiVersion: build.pivotal.io/v1alpha1
kind: Image
metadata:
  name: tutorial-image
spec:
  tag: daggerok/kpack-playground
  serviceAccount: tutorial-service-account
  cacheSize: "1.5Gi"
  builder:
    name: default
    kind: ClusterBuilder
  source:
    git:
      url: https://github.com/spring-projects/spring-petclinic
      revision: 82cb521d636b282340378d80a6307a08e3d4a4c4
```

```bash
kubectl apply -f spring-petclinic.yaml

logs -image tutorial-image
#...
#[completion] Build successful

kubectl get images
docker run -p 8080:8080 index.docker.io/daggerok/kpack-playground@sha256:ba8a6089308f65bdde15897253a243094b50825cd1fdf2f76fb3e1cc2720bd42
```

open http://127.0.0.1:8080/
