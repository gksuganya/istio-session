# 8 - Encryption & Mutual TLS

Encrypting traffic between services with mutual TLS

## 8.0 Prerequisites

Reset to default state (Istio & book-info deployed):

```
kubectl delete -f ../reset/
```
```
kubectl apply -f ../setup/
```

## 8.1 Deploy orphan app
```
kubectl apply -f sleep.yaml
```
Using the [Istio simple sleep service](https://github.com/istio/istio/tree/master/samples/sleep) as an orphan app
> The [orphan app](sleep.yaml) is not managed through Istio

Check the orphan POD is up
```
kubectl get pods -n orphan
```

## 8.2 Access details API

Check default mesh policy:

```
kubectl describe meshpolicy default
```

Find the orphan app container:

```
kubectl get pod -n orphan
```
Can also use `docker container ls --filter name=k8s_sleep`

```
export id=$(kubectl get pods -n orphan --output=jsonpath={.items..metadata.name})
```
Can also use `$id=$(docker container ls --filter name=k8s_sleep --format '{{ .ID}}')`

Exec into the container:

```
kubectl -n orphan exec -it $id sh
```

Use the details API:

```
curl http://details.default.svc.cluster.local:9080/details/1
```

## 8.3 Secure the bookinfo services

> In a new terminal

Enforce mTLS for [all services in the default namespace](mutual-tls.yaml):

```
kubectl apply -f mutual-tls.yaml
```

> Back to the sleep container

Check the API again:

```
curl http://details.default.svc.cluster.local:9080/details/1
```

## 8.4 Check the product page is down as well

Check http://localhost/productpage

## 8.5 Apply mTLS for client

Add [default destination rule for mTLS](mutual-tls-istio-auth.yaml):

```
kubectl apply -f mutual-tls-istio-auth.yaml
```

> Back to the sleep container

Check the API again:

```
curl http://details.default.svc.cluster.local:9080/details/1
```

> However check http://localhost/productpage works

## 8.6 View the TLS certs

Connect the product page proxy:

```
docker container ls --filter name=istio-proxy_productpage

docker container exec -it $(docker container ls --filter name=istio-proxy_productpage --format '{{ .ID}}') sh
```

Access the details API:

```
curl http://details:9080/details/1

curl https://details:9080/details/1

curl -k https://details:9080/details/1
```

Check the certs generated by Istio:

```
ls /etc/certs

curl -k https://details:9080/details/1 --key /etc/certs/key.pem --cert /etc/certs/cert-chain.pem --cacert /etc/certs/root-cert.pem

cat /etc/certs/cert-chain.pem | openssl x509 -text -noout  | grep Validity -A 2

cat /etc/certs/cert-chain.pem | openssl x509 -text -noout  | grep 'Subject Alternative Name' -A 1
```

## 8.7 Clean up for the next demo

Reset to default state (Istio & book-info deployed):

```
kubectl delete -f ../reset/
```
```
kubectl apply -f ../setup/
```