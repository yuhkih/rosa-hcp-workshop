# 1. Network の SubnetId を確認する

ここでは、何らかの方法で AWS上に ROSA インストール用の Subnet を作成した状態だとします。

1-1. AWS の Subnet id を取得します。

```
aws ec2 describe-subnets | jq -r '.Subnets[] | [ .CidrBlock, .SubnetId, .AvailabilityZone, .Tags[].Value ] | @csv'
```

* コマンド実行例: *

```
$ aws ec2 describe-subnets | jq -r '.Subnets[] | [ .CidrBlock, .SubnetId, .AvailabilityZone, .Tags[].Value ] | @csv'
"10.0.128.0/24","subnet-07098183112673e5e","us-east-2a","true","1","ROSA","my-hpc-cluster-vpc-public-use2-az1","my-hpc-cluster"
"10.0.0.0/24","subnet-06cf09e21d4ab1e8f","us-east-2a","ROSA","1","true","my-hpc-cluster","my-hpc-cluster-vpc-private-use2-az1"
$
```

この例では ROSA Public Cluster 用に `subnet-07098183112673e5e` (Pulbic Subnet) と `subnet-06cf09e21d4ab1e8f` (Private Sunbet) を作成しています。


# 2. ManagedOpenShift-Installer-Role IAM Role の ARN を確認する。 

2-1. STS インストールを前提としています。インストール用の ROSA の IAM Role を作成します。(既に作成してある場合は必要ありません）

```
rosa create account-roles
```

* コマンド実行例: *

```
$ rosa create account-roles
I: Logged in as 'yuhkih' on 'https://api.openshift.com'
I: Validating AWS credentials...
I: AWS credentials are valid!
I: Validating AWS quota...
I: AWS quota ok. If cluster installation fails, validate actual AWS resource usage against https://docs.openshift.com/rosa/rosa_getting_started/rosa-required-aws-service-quotas.html
I: Verifying whether OpenShift command-line tool is available...
I: Current OpenShift Client Version: 4.14.2
I: Creating account roles
? Role prefix: ManagedOpenShift
? Permissions boundary ARN (optional): 
? Path (optional): 
? Role creation mode: auto
? Create Classic account roles: Yes
? Create Hosted CP account roles: No
I: Creating classic account roles using 'arn:aws:iam::864046375925:user/yhanada@redhat.com-x9bhx-admin'
I: Created role 'ManagedOpenShift-Installer-Role' with ARN 'arn:aws:iam::864046375925:role/ManagedOpenShift-Installer-Role'
I: Created role 'ManagedOpenShift-ControlPlane-Role' with ARN 'arn:aws:iam::864046375925:role/ManagedOpenShift-ControlPlane-Role'
I: Created role 'ManagedOpenShift-Worker-Role' with ARN 'arn:aws:iam::864046375925:role/ManagedOpenShift-Worker-Role'
I: Created role 'ManagedOpenShift-Support-Role' with ARN 'arn:aws:iam::864046375925:role/ManagedOpenShift-Support-Role'
I: To create an OIDC Config, run the following command:
        rosa create oidc-config
$ 
```

2-2. 作成された ROLE を確認します。

```
rosa list account-roles
```


* コマンド実行例: *

```
$ rosa list account-roles
I: Fetching account roles
ROLE NAME                           ROLE TYPE      ROLE ARN                                                           OPENSHIFT VERSION  AWS Managed
ManagedOpenShift-ControlPlane-Role  Control plane  arn:aws:iam::864046375925:role/ManagedOpenShift-ControlPlane-Role  4.14               No
ManagedOpenShift-Installer-Role     Installer      arn:aws:iam::864046375925:role/ManagedOpenShift-Installer-Role     4.14               No
ManagedOpenShift-Support-Role       Support        arn:aws:iam::864046375925:role/ManagedOpenShift-Support-Role       4.14               No
ManagedOpenShift-Worker-Role        Worker         arn:aws:iam::864046375925:role/ManagedOpenShift-Worker-Role        4.14               No
$ 
```

# 3. 作成した ROSA 用の Network を Debug する

3-1. ネットワークの検証を行います。検証したい subnet id と、`ManagedOpenShift-Installer-Role` IAM Role の arn が必要になります。

```
rosa verify network --watch --region us-east-2 --subnet-ids subnet-07098183112673e5e,subnet-06cf09e21d4ab1e8f  --role-arn arn:aws:iam::864046375925:role/ManagedOpenShift-Installer-Role
```

* コマンド実行例: *

```
$ rosa verify network --watch --region us-east-2 --subnet-ids subnet-07098183112673e5e,subnet-06cf09e21d4ab1e8f  --role-arn arn:aws:iam::864046375925:role/ManagedOpenShift-Installer-Role
I: Verifying the following subnet IDs are configured correctly: [subnet-07098183112673e5e subnet-06cf09e21d4ab1e8f]
I: subnet-07098183112673e5e: passed
I: subnet-06cf09e21d4ab1e8f: passed
$
```

