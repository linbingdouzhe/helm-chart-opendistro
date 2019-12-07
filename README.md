# Opendistro for elasticsearch

## Useage

helm version: 3.0 release

> This chart base on Elasticsearch 7.4.0 helm chart  [helm_chart_elasticsearch](https://hub.helm.sh/charts/elastic/elasticsearch)

```shell
#Generate Key if you want demo installer 
# Root CA
openssl genrsa -out root-ca-key.pem 2048
openssl req -new -x509 -sha256 -key root-ca-key.pem -out root-ca.pem
# Admin cert
openssl genrsa -out admin-key-temp.pem 2048
openssl pkcs8 -inform PEM -outform PEM -in admin-key-temp.pem -topk8 -nocrypt -v1 PBE-SHA1-3DES -out admin-key.pem
openssl req -new -key admin-key.pem -out admin.csr
openssl x509 -req -in admin.csr -CA root-ca.pem -CAkey root-ca-key.pem -CAcreateserial -sha256 -out admin.pem
# Node cert
openssl genrsa -out node-key-temp.pem 2048
openssl pkcs8 -inform PEM -outform PEM -in node-key-temp.pem -topk8 -nocrypt -v1 PBE-SHA1-3DES -out node-key.pem
openssl req -new -key node-key.pem -out node.csr
openssl x509 -req -in node.csr -CA root-ca.pem -CAkey root-ca-key.pem -CAcreateserial -sha256 -out node.pem
# Cleanup
rm admin-key-temp.pem
rm admin.csr
rm node-key-temp.pem
rm node.csr


#append to these key to "35-es-bootstrap-secrets.yml"
vim 35-es-bootstrap-secrets.yml

kubectl create namespace es

kubectl apply -f 35-es-bootstrap-secrets.yml

helm upgrade --install --values elasticsearch/data.yaml opendistro-data elasticsearch -n es

helm upgrade --install --values elasticsearch/master.yaml opendistro-master elasticsearch -n es
```



### opendisto security setting

So far, opendistro elasticsearch Deployment compelete 

implement securityadmin.sh

```shell
kubectl exec -it "master-node-name" /bin/bash -n es

cd plugins/opendistro_security/tools/
chmod +x hash.sh
chmod +x securityadmin.sh

# init admin password
./hash.sh
[enter password what you want]
qwoheuqiwehoanupdanwd-q92j01doq # this is hash value 

vim ../internal_user.yaml
#copy hash value and replace user admin on this file 


./securityadmin.sh -cd ../securityconfig/ -icl -nhnv -cacert ../../../config/root-ca.pem -cert ../../../config/admin.pem -key ../../../config/admin-key.pem

#until you see SUCC , mean cluster set is done , check your cluster health 

```

 about key setting , can reference offical guide [GenerateKey](https://opendistro.github.io/for-elasticsearch-docs/docs/security-configuration/generate-certificates/)



so far , you can create your load balancer 

```shell
kubectl apply -f 35-es-service.yml
```