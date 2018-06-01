## Ingress-controller搭建

#### 创建nginx-ingress-controller

```
创建RBAC：
kubectl create -f nginx-ingress-rbac.yaml
创建configmap：
kubectl create -f nginx-ingress-configmap.yaml
创建nginx-ingress-controller danmonset:
kubectl create -f nginx-ingress-controller-daemonset.yaml
创建默认404界面：
kubectl create -f nginx-ingress-default-backend-deployment.yaml
kubectl create -f nginx-ingress-default-backend-service.yaml
```

#### 创建测试ingress(无证书的)

```
kubectl create -f ingress-test.yaml
```



#### 生成证书

```
# 生成 CA 自签证书
mkdir cert && cd cert
openssl genrsa -out ca-key.pem 2048
openssl req -x509 -new -nodes -key ca-key.pem -days 10000 -out ca.pem -subj "/CN=kube-ca"

# 编辑 openssl 配置
cp /etc/pki/tls/openssl.cnf .
vim openssl.cnf

# 主要修改如下
[req]
req_extensions = v3_req # 这行默认注释关着的 把注释删掉
# 下面配置是新增的
[ v3_req ]
basicConstraints = CA:FALSE
keyUsage = nonRepudiation, digitalSignature, keyEncipherment
subjectAltName = @alt_names
[alt_names]
DNS.1 = my.com

# 生成证书
openssl genrsa -out ingress-key.pem 2048
openssl req -new -key ingress-key.pem -out ingress.csr -subj "/CN=kube-ingress" -config openssl.cnf
openssl x509 -req -in ingress.csr -CA ca.pem -CAkey ca-key.pem -CAcreateserial -out ingress.pem -days 365 -extensions v3_req -extfile openssl.cnf
```

#### 创建 secret

```
创建和ingress同一namespace下的secret
kubectl create secret tls ingress-secret --key cert/ingress-key.pem --cert cert/ingress.pem -n monitoring
```

#### 创建测试ingress(带证书的)

```
kubectl create -f ingress-test-tls.yaml
```



#### 参考文档

https://mritd.me/2017/03/04/how-to-use-nginx-ingress/

https://github.com/kubernetes/ingress-nginx

https://jimmysong.io/kubernetes-handbook/practice/traefik-ingress-installation.html

https://blog.opskumu.com/k8s-ingress.html