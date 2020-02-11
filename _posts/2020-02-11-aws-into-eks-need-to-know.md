---
title: "[EKS]aws進入k8s領域前要注意的事"
key: 20200211
layout: article
tags: aws eks k8s eksctl
---
轉了跑道後還是天天踩坑天天填坑

從GCP跨到AWS，只是用的都是k8s，但為啥在AWS上使用有這麼困難？

在GCP上，帳號權限是admin後基本上是暢行無阻~~想幹啥就幹啥~~

換到AWS後，各種不行，搞到有點懷疑人生

但是關關難過關關過，既然是全雲端服務裡擁有最多服務的雲端公司 ~~公三小~~

沒道理一堆前輩、大神們待的公司都在用，然後我沒辦法用。

<!--more-->

後來翻了很多github上的issue討論，以及試了很多種方法

當你下指令是`eksctl create`相關的錯誤

然後錯誤是以下圖片這類的時候，應該有八七趴都是權限的問題

![eksctl image error 1](/assets/images/eksctl-role-error.png)

找到的相關解決法方就是讓你當前使用指令的帳號有各種相關權限

因為牽扯的相關權限蠻多的，所以我是建議管理一種權限就建立一個政策，最後再綁進你使用指令的帳號上就好

## 相關權限如下(直接建政策後用JSON編輯儲存)
{% highlight json %}
{
    "Version": "2012-10-17",
    "Statement": [
/// CloudFormation
        {
            "Sid": "eksCtlCloudFormation",
            "Effect": "Allow",
            "Action": "cloudformation:*",
            "Resource": "*"
        },
/// EKS
        {
            "Effect": "Allow",
            "Action": [
                "eks:*"
            ],
            "Resource": "*"
        },
/// IAM
        {
            "Sid": "VisualEditor0",
            "Effect": "Allow",
            "Action": [
                "iam:CreateInstanceProfile",
                "iam:DeleteInstanceProfile",
                "iam:GetRole",
                "iam:GetInstanceProfile",
                "iam:RemoveRoleFromInstanceProfile",
                "iam:CreateRole",
                "iam:DeleteRole",
                "iam:AttachRolePolicy",
                "iam:PutRolePolicy",
                "iam:ListInstanceProfiles",
                "iam:AddRoleToInstanceProfile",
                "iam:ListInstanceProfilesForRole",
                "iam:PassRole",
                "iam:DetachRolePolicy",
                "iam:DeleteRolePolicy",
                "iam:GetRolePolicy"
            ],
            "Resource": [
                "arn:aws:iam::142267666880:instance-profile/eksctl-*",
                "arn:aws:iam::142267666880:role/eksctl-*"
            ]
        },
/// Networking
        {
            "Sid": "EksInternetGateway",
            "Effect": "Allow",
            "Action": "ec2:DeleteInternetGateway",
            "Resource": "arn:aws:ec2:*:*:internet-gateway/*"
        },
        {
            "Sid": "EksNetworking",
            "Effect": "Allow",
            "Action": [
                "ec2:AuthorizeSecurityGroupIngress",
                "ec2:DeleteSubnet",
                "ec2:DeleteTags",
                "ec2:CreateNatGateway",
                "ec2:CreateVpc",
                "ec2:AttachInternetGateway",
                "ec2:DescribeVpcAttribute",
                "ec2:DeleteRouteTable",
                "ec2:AssociateRouteTable",
                "ec2:DescribeInternetGateways",
                "ec2:CreateRoute",
                "ec2:CreateInternetGateway",
                "ec2:RevokeSecurityGroupEgress",
                "ec2:CreateSecurityGroup",
                "ec2:ModifyVpcAttribute",
                "ec2:DeleteInternetGateway",
                "ec2:DescribeRouteTables",
                "ec2:ReleaseAddress",
                "ec2:AuthorizeSecurityGroupEgress",
                "ec2:DescribeTags",
                "ec2:CreateTags",
                "ec2:DeleteRoute",
                "ec2:CreateRouteTable",
                "ec2:DetachInternetGateway",
                "ec2:DescribeNatGateways",
                "ec2:DisassociateRouteTable",
                "ec2:AllocateAddress",
                "ec2:DescribeSecurityGroups",
                "ec2:RevokeSecurityGroupIngress",
                "ec2:DeleteSecurityGroup",
                "ec2:DeleteNatGateway",
                "ec2:DeleteVpc",
                "ec2:CreateSubnet",
                "ec2:DescribeSubnets"
            ],
            "Resource": "*"
        },
/// AutoScaling
        {
            "Sid": "VisualEditor0",
            "Effect": "Allow",
            "Action": [
                "autoscaling:CreateLaunchConfiguration",
                "autoscaling:DeleteLaunchConfiguration"
            ],
            "Resource": "arn:aws:autoscaling:*:*:launchConfiguration:*:launchConfigurationName/*"
        },
        {
            "Sid": "VisualEditor1",
            "Effect": "Allow",
            "Action": [
                "autoscaling:UpdateAutoScalingGroup",
                "autoscaling:DeleteAutoScalingGroup",
                "autoscaling:CreateAutoScalingGroup"
            ],
            "Resource": "arn:aws:autoscaling:*:*:autoScalingGroup:*:autoScalingGroupName/*"
        },
        {
            "Sid": "VisualEditor2",
            "Effect": "Allow",
            "Action": [
                "autoscaling:DescribeAutoScalingGroups",
                "autoscaling:DescribeLaunchConfigurations"
            ],
            "Resource": "*"
        }
/// ecr
        {
            "Effect": "Allow",
            "Action": [
                "ecr:GetAuthorizationToken",
                "ecr:BatchCheckLayerAvailability",
                "ecr:GetDownloadUrlForLayer",
                "ecr:BatchGetImage"
            ],
            "Resource": "*"
        }
    ]
}
{% endhighlight %}

最後在執行完create之後使用`kubectl get svc`就會看到
```
NAME         TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
kubernetes   ClusterIP   10.100.0.1   <none>        443/TCP   8m31s
```


相關出處
[GitHub](https://github.com/weaveworks/eksctl/issues/204)
