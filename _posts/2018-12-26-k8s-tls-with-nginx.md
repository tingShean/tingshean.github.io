---
title: "[Kubernetes]TLS with nginx"
key: 20181226
layout: post
tags: Kubernetes k8s nginx ssl tls 
---
最近在弄SSL相關的東西，用K8S在設定上相對麻煩很多，當初在用Docker的時候真的是簡單到不行

進入主題，作法有很多不管是你想包進去還是用K8S內Certificate Rotation的功能或者是用Ingress的作法

隨便說真的就三種以上，不過今天要說的是從外部掛進去的方式來做

現在很多人會直接買第三方的SSL(Letsencrypt不在此篇內)，但是大多數的人會直接的包進nginx

~~所以SSL過期後就再包第二次，以此類推好像很簡單~~

回歸正題，接下來的作法不會要你一直包新的Docker image，只需要更換secret就可以了

<!--more-->
## Step 1 把TLS檔移到指定位置
`
$ mv secret.crt ${CERT_FILE}
$ mv secret.key ${KEY_FILE}
`

## Step 2 建立Secret
{% highlight shell %}
$ kubectl create secret <secret_name> ${CERT_NAME} --key ${KEY_FILE} --cert ${CERT_FILE} --namespace=${NAMESPACE}
{% endhighlight %}

## Step 3 nginx內加入
`
ssl_certificate /etc/nginx/ssl/<secret_name>.crt;
ssl_certificate_key /etc/nginx/ssl/<secret_name>.key;
`

## Step 4 K8s nginx yaml加上
{% highlight yaml %}
.spec.template.spec.volumes:
  - name: secret-volume
    secret:
      secretName: ${CERT_NAME}
.spec.template.spec.containers[].volumeMounts:
  - mountPath: /etc/nginx/ssl
    name: secret-volume
{% endhighlight %}

---

參考資料 

[k8s Secret](https://kubernetes.io/docs/concepts/configuration/secret/)
