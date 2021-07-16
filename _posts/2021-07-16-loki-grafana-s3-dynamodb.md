---
title: "[Monitor]loki use S3 and DynamoDB"
key: 20210716
layout: article
tags: loki s3 dynamodb OpenIDConnect AssumeRoleWithWebIdentity
---
### 前言

最近在點監控這邊的技能樹，多多少少也遇到一些問題，雖然說這部份討論真的不多，但是設定上還算簡單，照著網路上的設定大致上沒什麼問題，不過因為我把`loki`放在`k8s`，所以大致上都是`k8s`本身的問題

<!--more-->

### 遇到的問題

> No OpenIDConnect provider found in your account

這個問題應該是我用的是`EKS`的關係，所以找了一些資料後找到`AWS`提供的說明文，以下擷錄其中的一段話`此功能可讓您向支援的身分供應商驗證 AWS API 呼叫，並接收有效的 OIDC JSON Web 字符 (JWT)。(註1)`

所以依照網頁上的流程照著做發現我還真的沒有這個東西，建立完之後問題並沒有完全解決~~，是的抱持著「解決主要敵人後，次要敵人就會出來」的想法就不會有太大的衝擊~~，接下來遇到的也是權限上的問題。


### 繼續遇到問題

> Not authorized to perform sts:AssumeRoleWithWebIdentity\n\tstatus code: 403

這部份是需要加在你`K8s`使用的`serviceAccount`對應的帳號裡，像我是在`IAM`的`role`裡創了個`LokiAccount`，而且在`K8s`的`serviceAccount`也叫`LokiAccount`，這樣方便我不會搞混哪邊要用哪個，接著以下的`code`就是加在該`role`的`TrustRelationship`裡

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "Federated": "arn:aws:iam::<aws_account>:oidc-provider/oidc.eks.<region>.amazonaws.com/id/<region-provider>"
      },
      "Action": "sts:AssumeRoleWithWebIdentity",
      "Condition": {
        "StringEquals": {
          "oidc.eks.<region>.amazonaws.com/id/<region-provider>:sub": "system:serviceaccount:<k8s_namespace>:LokiAccount"
        }
      }
    }
  ]
}

```


### 結語

這次卡關的時間沒有想像中的久，然後`loki`使用上也算順利，部署設定也比想像中簡單很多，雖然也是用`helm`，不過官方頁面給的設定有點少所以我就不附連結了，我是直接參考`GihubGist`有人貼出他的`helm`設定檔，不過就像我說的遇到的問題不是`helm`，而是在`K8s`裡，所以算是對`K8s`的了解又更往上了一層。


### 參考鏈結

[註1](https://docs.aws.amazon.com/zh_tw/eks/latest/userguide/iam-roles-for-service-accounts-technical-overview.html)

[建立IAM OIDC](https://docs.aws.amazon.com/zh_tw/eks/latest/userguide/enable-iam-roles-for-service-accounts.html)

[sts:AssumeRoleWithWebIdentity status code: 403](https://github.com/grafana/loki/issues/3259)

[loki_values.yaml](https://gist.github.com/Eraac/4731ee1f5043e1ed3d15f02240b0d983)

[loki storage](https://grafana.com/docs/loki/latest/storage/)

[Setup Loki Storage to AWS S3 and DynamoDB in Kubernetes with Kiam](https://medium.com/codex/setup-loki-storage-to-aws-s3-and-dynamodb-in-kubernetes-with-kiam-4c223a0b7995)
