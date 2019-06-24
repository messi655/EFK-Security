# Deploy Elastic search, Fluentbit and Kibana on K8S with Enable security.

## Deploy Elasticsearch by helm

Note: By default this will be create 3 ES nodes (https://github.com/elastic/helm-charts/blob/master/elasticsearch/README.md)

1. Add repo

```bash
helm repo add elastic https://helm.elastic.co
```

2. Deploy

- Create namespace with name is securees

- Deploy Elasticsearch

```bash
helm install --name elasticsearch --namespace securees elastic/elasticsearch --version 7.1.0 --set nodeSelector.worker-node=logging
```

3. Enable security

- Generate SSL certificates following the https://www.elastic.co/guide/en/elasticsearch/reference/6.7/configuring-tls.html#node-certificates

- Create Kubernetes secrets for authentication credentials and certificates

```bash
kubectl create secret generic elastic-credentials --namespace=securees --from-literal=password=changeme --from-literal=username=elastic
kubectl create secret generic elastic-certificates --namespace=securees --from-file=elastic-certificates.p12
```

- Download helm chart repository

```bash
git clone git@github.com:elastic/helm-charts.git
```

- Change to `helm-charts/elasticsearch/examples/security` replace `security.yml` file with default security.yml

```bash
cd helm-charts/elasticsearch/examples/security
helm upgrade --wait --timeout=600 --install --namespace securees --values ./security.yml elasticsearch ../../
```


- Attach into one of the containers

```bash
kubectl exec -ti $(kubectl get pods -l release=elasticsearch -o name -n=securees | awk -F'/' '{ print $NF }' | head -n 1) -n=securees bash
```


## Deploy Kibana

```bash
kubectl apply -f kibana-configMap.yaml
kubectl apply -f kibana-dep.yaml
kubectl apply -f kibana-ingress.yaml
```

## Deploy fluent bit

```bash
kubectl apply -f fluentbit-configmap.yaml
kubectl apply -f fluentbit-secure.yaml
kubectl apply -f fluentbit.yaml
```

## Check

- Open browse and access `http://kibana.yourdomain.com` and enter username/password (this is credential of elasticsearch that you did above)