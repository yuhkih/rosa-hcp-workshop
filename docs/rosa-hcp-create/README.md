# AWS CLI の準備

ここも書く。

```
aws configuire
```

# OpenShift / ROSA の CLI の準備

1.`rosa` コマンドと `oc` コマンドをダウンロードして展開します。

```
curl -LO https://mirror.openshift.com/pub/openshift-v4/clients/rosa/latest/rosa-linux.tar.gz
tar -zxf rosa-linux.tar.gz 
sudo mv ./rosa /usr/local/bin/
rosa download oc
tar -xzf openshift-client-linux.tar.gz 
sudo mv ./oc /usr/local/bin
sudo mv ./kubectl /usr/local/bin
```

2.インストールされたコマンドのバージョンを確認します。

```
$ rosa version
1.2.31
I: Your ROSA CLI is up to date.
$ oc version
Client Version: 4.14.2
Kustomize Version: v5.0.1
Unable to connect to the server: dial tcp: lookup api.my-hpc-cluster.rc4b.p3.openshiftapps.com on 172.28.240.1:53: no such host
$
```

# ROSA 用の AWS Network の作成

HCP ROSA は、ユーザーが既にもっているネットワークにデプロイする事が前提になります。
ここでは Terraform を使用して AWS　上に Network を作成します。

ここの手順は、ドキュメントの「<a href="https://docs.openshift.com/rosa/rosa_hcp/rosa-hcp-sts-creating-a-cluster-quickly.html#rosa-hcp-creating-vpc" target="_blank">Creating a Vritual Private Cloud for your ROSA with HCP clusters</a>
」をベースにしています。

## terraform を使用した VPC, Subnet,  NAT Gateway 等の必要な Network リソースの作成

```
git clone https://github.com/openshift-cs/terraform-vpc-example
```

```
cd terraform-vpc-example
```

```
terraform init
```

変数を準備しておきます。(日本 Region が使用可能になるのは 2024/1月頃なので US-EAST を使用）

```
export CLUSTER_NAME=my-hpc-cluster
export REGION=us-east-2
```

Terraform の plan を作成します。以下は Multi AZ 環境の Network を作成します。

```
terraform plan -out rosa.tfplan -var region=$REGION -var cluster_name=$CLUSTER_NAME -var single_az_only=false
```

Apply して Network を作成します。

```
terraform apply rosa.tfplan
```

## 作成された Subnet と NAT Gateway の確認

terraform で apply した時のログにも出ていますが、ここでは AWS CLI の練習も兼ねて、AWS CLI を使用して作成された VPC と Subnet 等を確認します。

VPC をリストします。
```
aws ec2 describe-vpcs | jq -r '.Vpcs[] | [.CidrBlock, .VpcId, .State] | @csv'
```

Subnet をリストします。
```
aws ec2 describe-subnets | jq -r '.Subnets[] | [ .CidrBlock, .SubnetId, .AvailabilityZone, .Tags[].Value ] | @csv'
```

NAT Gateway をリストします。
```
aws ec2 describe-nat-gateways | jq -r '.NatGateways[] | [.NatGatewayId, .State] | @csv'
```

変数 SUBNET_IDS を作成します。

```
export SUBNET_IDS=<上ででてきた Private と Public Subnet の ID 合計６つをカンマ区切りで並べます>
```

上記の実際の例

```
export SUBNET_IDS=subnet-0f0b7ebc07df35c69,subnet-084bb65941bee3d24,subnet-0fdeb4dc0c5415267,subnet-08617cb6e925ef45e,subnet-087957efbdd593739,subnet-09f859213ce4af732
```
# ROSA HCP の有効化

以下のリンクをクリックして AWS の ROSA 設定画面に飛びます。
[https://console.aws.amazon.com/rosa/home#/get-started](https://console.aws.amazon.com/rosa/home#/get-started)

[Enable ROSA with HCP] のボタンをクリックします。
![image](https://github.com/yuhkih/rosa-hcp-workshop/assets/8530492/f5a08db1-9fac-47de-9d88-4bde4dd6e7a6)

有効化されるまで、暫く待ちます。
![image](https://github.com/yuhkih/rosa-hcp-workshop/assets/8530492/26af4b3a-d7c6-4ebe-951f-fac2eac548c5)

HCPが有効化されたら次の画面に進み「Connect accounts] をクリックします。
![image](https://github.com/yuhkih/rosa-hcp-workshop/assets/8530492/596c9a5e-9874-45a9-9183-d46e1de3d13d)

以上で HCP の有効化は完了です。

# ROSA HCP Cluster の 作成

必要な IAM Role を作成します。

```
rosa create account-roles --hosted-cp
```

Cluster の作成を開始します。いろいろ聞かれますが、デフォルトで大丈夫です。

```
rosa create cluster --cluster-name=$CLUSTER_NAME --sts --hosted-cp  --region=us-east-2  --subnet-ids=$SUBNET_IDS
```

Cluster の作成を開始した後に Operator Role を作成します

```
rosa create operator-roles --cluster $CLUSTER_NAME
```

OIDC Provider を作成します

```
rosa create oidc-provider --cluster  $CLUSTER_NAME
```

ROSA のクラスターができるまで以下のコマンドでモニターします。大体 10分ほどかかるはずです。

```
rosa logs install -c my-hpc-cluster --watch
```



# ROSA HCP Cluster へのアクセス確認

インストールが完了したら管理者ユーザーを作成します。
ログインコマンド (`oc login`) パスワード付きで標準出力に表示されます。これはコマンドが終了してから、数分待つ必要があります。

```
$ rosa create admin --cluster=$CLUSTER_NAME
I: Admin account has been added to cluster 'my-hpc-cluster'.
I: Please securely store this generated password. If you lose this password you can delete and recreate the cluster admin user.
I: To login, run the following command:

   oc login https://api.my-hpc-cluster.rc4b.p3.openshiftapps.com:443 --username cluster-admin --password abc123-XYZZH-1dNpZ-DBVjg

I: It may take several minutes for this access to become active.
$
```

数分待ってから、上の出力で現れたログインコマンド(`oc login`) を実行します。

```
$  oc login https://api.my-hpc-cluster.rc4b.p3.openshiftapps.com:443 --username cluster-admin --password abc123-XYZZH-1dNpZ-DBVjg
Login successful.

You have access to 77 projects, the list has been suppressed. You can list all projects with 'oc projects'

Using project "default".
$ 
```

`oc get nodes` コマンドで compute node ができたか確認します。Worker node が 3本表示されるはずです。

```
$ oc get nodes
NAME                                       STATUS   ROLES    AGE     VERSION
ip-10-0-0-72.us-east-2.compute.internal    Ready    worker   9m44s   v1.27.6+b49f9d1
ip-10-0-1-195.us-east-2.compute.internal   Ready    worker   70s     v1.27.6+b49f9d1
ip-10-0-2-242.us-east-2.compute.internal   Ready    worker   9m28s   v1.27.6+b49f9d1
$
```

# 構成を探って見る

`rosa list machinepool` コマンドで、AZ毎に `machinepool` が出来ている事を確認します。`machinepool`単位で Node 数を増やす事ができます。

```
$ rosa list machinepool -c $CLUSTER_NAME
ID         AUTOSCALING  REPLICAS  INSTANCE TYPE  LABELS    TAINTS    AVAILABILITY ZONE  SUBNET                    VERSION  AUTOREPAIR  
workers-0  No           1/1       m5.xlarge                          us-east-2b         subnet-084bb65941bee3d24  4.14.3   Yes         
workers-1  No           1/1       m5.xlarge                          us-east-2a         subnet-0f0b7ebc07df35c69  4.14.3   Yes         
workers-2  No           1/1       m5.xlarge                          us-east-2c         subnet-0fdeb4dc0c5415267  4.14.3   Yes         
$ 
```

`rosa list ingress` コマンドで Cluster と一緒に作成された ingress を確認してみます。default の Load Balancer には NLB が使われているはずです。`LB-TYPE` を確認します。
この ingress 経由で、HTTP/HTTPS アプリケーションが公開されます。

```
$ rosa list ingress -c $CLUSTER_NAME
ID    APPLICATION ROUTER                                          PRIVATE  DEFAULT  ROUTE SELECTORS  LB-TYPE  EXCLUDED NAMESPACE  WILDCARD POLICY  NAMESPACE OWNERSHIP  HOSTNAME  TLS SECRET REF
m3x6  https://apps.rosa.my-hpc-cluster.rc4b.p3.openshiftapps.com  no       yes                       nlb                                                                          
$
```
