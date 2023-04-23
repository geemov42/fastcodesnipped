
# links
https://k6.io/blog/running-distributed-tests-on-k8s/#deploying-our-test-script
https://opensource.com/article/19/2/deploy-influxdb-grafana-kubernetes
https://k6.io/blog/comparison-of-k6-test-result-visualizations/

https://codebeautify.org/yaml-beautifier
https://kubernetes.io/docs/reference/kubectl/cheatsheet/

## Install k6 operator

It is possible to install the operator from a wsl

```shell
git clone https://github.com/grafana/k6-operator && cd k6-operator
make deploy
```

## InfluxDB

```shell
kubectl create deployment influxdb --image=docker.io/influxdb:1.6.4
```

```shell
kubectl create secret generic influxdb-creds --from-literal=INFLUXDB_DATABASE=k6 --from-literal=INFLUXDB_USERNAME=root --from-literal=INFLUXDB_PASSWORD=root --from-literal=INFLUXDB_HOST=influxdb
kubectl edit deployment influxdb
```

```yaml
spec:
  containers:
  - name: influxdb
    envFrom:
    - secretRef:
        name: influxdb-creds
```

```shell
kubectl expose deployment influxdb --type=LoadBalancer --port=8086 --target-port=8086 --protocol=TCP
```

## Grafana

```shell
kubectl create deployment grafana --image=docker.io/grafana/grafana:5.3.2
```

```shell
kubectl create secret generic grafana-creds --from-literal=GF_SECURITY_ADMIN_USER=admin --from-literal=GF_SECURITY_ADMIN_PASSWORD=graphsRcool
kubectl edit deployment grafana
```

```yaml
spec:
  containers:
  - name: grafana
    envFrom:
    - secretRef:
        name: grafana-creds
```

influxdb-datasource.yml :

```yaml
# config file version
apiVersion: 1

# list of datasources to insert/update depending
# what's available in the database
datasources:
  # <string, required> name of the datasource. Required
- name: influxdb
  # <string, required> datasource type. Required
  type: influxdb
  # <string, required> access mode. proxy or direct (Server or Browser in the UI). Required
  access: proxy
  # <int> org id. will default to orgId 1 if not specified
  orgId: 1
  # <string> url
  url: http://host.docker.internal:8086
  # <string> database password, if used
  password: root
  # <string> database user, if used
  user: root
  # <string> database name, if used
  database: k6
  # version
  version: 1
  # <bool> allow users to edit datasources from the UI.
  editable: false
```

grafana-dashboard-provider.yml :

```yaml
apiVersion: 1

providers:
- name: 'default'
  orgId: 1
  folder: ''
  type: file
  disableDeletion: false
  updateIntervalSeconds: 10 #how often Grafana will scan for changed dashboards
  options:
    path: /var/lib/grafana/dashboards
```

```shell
kubectl create configmap grafana-config --from-file=influxdb-datasource.yml=influxdb-datasource.yml --from-file=grafana-dashboard-provider.yml=grafana-dashboard-provider.yml
kubectl edit deployment grafana
```

```yaml
spec:
  template:
    spec:
      volumes:
      - configMap:
          name: grafana-config
        name: grafana-config
```

```yaml
spec:
  template:
    spec:
      containers:
      - name: grafana
        volumeMounts:
        - mountPath: /etc/grafana/provisioning/datasources/influxdb-datasource.yml
          name: grafana-config
          readOnly: true
          subPath: influxdb-datasource.yml
        - mountPath: /etc/grafana/provisioning/dashboards/grafana-dashboard-provider.yml
          name: grafana-config
          readOnly: true
          subPath: grafana-dashboard-provider.yml
```

```shell
kubectl expose deployment grafana --type=LoadBalancer --port=80 --target-port=3000 --protocol=TCP
```

## Reboot pods to apply new config

```shell
kubectl rollout restart deployment influxdb
kubectl rollout restart deployment grafana
```

## Prepare tests

```shell
npm i -g @apideck/postman-to-k6
```

```shell
postman-to-k6 perftest.postman_collection.json -o pscript/script.js
```

## Run tests

### Create configMap

```shell
kubectl create configmap crocodile-stress-test --from-file script.js
```

For a postman with his libs :

```shell
k6 archive script.js
kubectl create configmap crocodile-stress-test --from-file archive.tar
```

### Run k6 test on kubernetes

```yaml
apiVersion: k6.io/v1alpha1
kind: K6
metadata:
  name: k6-sample
spec:
  parallelism: 4
  arguments: --out influxdb=http://host.docker.internal:8086/k6
  script:
    configMap:
      name: crocodile-stress-test
      file: archive.tar or script.js
```

```shell
kubectl create -f cr.yml
kubectl delete -f cr.yml
```

### Run k6 test in local

```shell
k6 run -o influxdb=http://localhost:8086 pscript\script.js
```
