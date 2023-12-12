# 1.AWS CLI / Git の準備

ここも書く。

## 1.1 AWS CLI の install

[こちらの AWS のページ](https://docs.aws.amazon.com/ja_jp/cli/latest/userguide/getting-started-install.html)を参考にして AWS CLI をインストールします。

```
aws configure
```


```
aws sts get-caller-identity
```

**出力例:**

```
$ aws sts get-caller-identity
{
    "UserId": "ABCD1234BG6WLHYHMKFHK",
    "Account": "123407415212",
    "Arn": "arn:aws:iam::366607415212:user/yhanada@redhat.com-4w685"
}
```

## 1.2 Git CLI の install

[こちらのページ](https://git-scm.com/book/ja/v2/%E4%BD%BF%E3%81%84%E5%A7%8B%E3%82%81%E3%82%8B-Git%E3%81%AE%E3%82%A4%E3%83%B3%E3%82%B9%E3%83%88%E3%83%BC%E3%83%AB)を参照に git コマンドをインストールします。

```
git version
```

**出力例:**

```
$ git version
git version 2.17.1
$
```

# 2.OpenShift / ROSA の CLI の準備

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

`oc` コマンドのバージョンを確認します。`oc` コマンドは `kubectl` を拡張した OpenShift 独自のコマンドです。`kubectl` コマンドとほぼ同じ使い方ができます。

```
$ oc version
Client Version: 4.14.2
Kustomize Version: v5.0.1
Unable to connect to the server: dial tcp: lookup api.my-hpc-cluster.rc4b.p3.openshiftapps.com on 172.28.240.1:53: no such host
$
```

`rosa` コマンドのバージョンを確認します。`rosa` コマンドは、主に `oc` コマンドで取り扱う OpenShift のレイヤーより下のレイヤーを取り扱うコマンドです。AWSインフラにアクセスしてクラスターの作成/削除を行ったり、Kubernetes でカバーされていない AWS とのインフラ周りに関連する作業を行う時に使用します。

```
$ rosa version
1.2.31
I: Your ROSA CLI is up to date.
```

# 3.ROSA を install する AWS Network の作成

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

作成された AWS のサブネットIDを変数にセットしておきます。カンマ区切りで6つのサブネットIDが変数にセットされます。

```
export SUBNET_IDS=$(terraform output -raw cluster-subnets-string)
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

# 4.ROSA HCP の有効化

以下のリンクをクリックして AWS の ROSA 設定画面に飛びます。
[https://console.aws.amazon.com/rosa/home#/get-started](https://console.aws.amazon.com/rosa/home#/get-started)

[Enable ROSA with HCP] のボタンをクリックします。
![image](https://github.com/yuhkih/rosa-hcp-workshop/assets/8530492/084e3a5c-e76f-460a-9164-b55dfe06a4cb)

有効化されるまで、暫く待ちます。数分かかるはずです。
![image](https://github.com/yuhkih/rosa-hcp-workshop/assets/8530492/26af4b3a-d7c6-4ebe-951f-fac2eac548c5)

有効化が完了すると以下のような表示になっているはずです。
![image](https://github.com/yuhkih/rosa-hcp-workshop/assets/8530492/15d51fb2-22bf-454b-aa95-5cfacacc678b)

画面の一番下に移動して「Continue Red Hat」をクリックします。
![image](https://github.com/yuhkih/rosa-hcp-workshop/assets/8530492/87d5a503-7a0a-4a51-9c08-0aa7ad2dc026)

Red Hat の ポータルサイトにログインします。(Red Hat アカウントが無い場合ば作成してから、再度[こちら](https://console.redhat.com/connect/aws) にアクセスします。Red Hat アカウントは無料で作成できます）
![image](https://github.com/yuhkih/rosa-hcp-workshop/assets/8530492/565e0b8d-eada-4d52-a1c3-16c58bae93fa)

日本語を選んで「Connect accounts] をクリックします。
![image](https://github.com/yuhkih/rosa-hcp-workshop/assets/8530492/2148e419-4753-421b-a572-bcefc2660df3)

以上で HCP の有効化は完了です。

# 5.ROSA HCP Cluster の 作成

必要な IAM Role を作成します。いろいろ聞かれますが、デフォルトで大丈夫です。

```
rosa create account-roles --hosted-cp
```

必要な変数が全てセットされているか再確認します。もしセットされてない場合は、以前の手順に戻ってセットして下さい。

```
echo $CLUSTER_NAME
echo $REGION
echo $SUBNET_IDS
```

Cluster の作成を開始します。いろいろ聞かれますが、デフォルトで大丈夫です。

```
rosa create cluster --cluster-name=$CLUSTER_NAME --sts --hosted-cp  --region=$REGION --subnet-ids=$SUBNET_IDS
```

Cluster の作成を開始した後に Operator Role を作成します。いろいろ聞かれますが、デフォルトで大丈夫です。

```
rosa create operator-roles --cluster $CLUSTER_NAME
```

OIDC Provider を作成します。いろいろ聞かれますが、デフォルトで大丈夫です。

```
rosa create oidc-provider --cluster  $CLUSTER_NAME
```

ROSA のクラスターができるまで以下のコマンドでモニターします。大体 10分ほどかかるはずです。

```
rosa logs install -c my-hpc-cluster --watch
```



# 6.ROSA HCP Cluster へのアクセス確認

インストールが完了したら管理者ユーザーを作成します。
ログインコマンド (`oc login`) パスワード付きで標準出力に表示されます。これはコマンドが終了してから、数分待つ必要があります。

```
rosa create admin --cluster=$CLUSTER_NAME
```

**実行例:**

```
$ rosa create admin --cluster=$CLUSTER_NAME
I: Admin account has been added to cluster 'my-hpc-cluster'.
I: Please securely store this generated password. If you lose this password you can delete and recreate the cluster admin user.
I: To login, run the following command:

   oc login https://api.my-hpc-cluster.rc4b.p3.openshiftapps.com:443 --username cluster-admin --password abc123-XYZZH-1dNpZ-DBVjg

I: It may take several minutes for this access to become active.
$
```

数分待ってから、上の出力で現れた api server の URL、password を使ってログインコマンド(`oc login`) を実行します。
(準備ができるまで 401 Unauthorized が出ます) 

```
oc login <API Server> --username cluster-admin --password <password>
```

**実行例:**

```
$  oc login https://api.my-hpc-cluster.rc4b.p3.openshiftapps.com:443 --username cluster-admin --password abc123-XYZZH-1dNpZ-DBVjg
Login successful.

You have access to 77 projects, the list has been suppressed. You can list all projects with 'oc projects'

Using project "default".
$ 
```

`oc get nodes` コマンドで compute node ができたか確認します。Worker node が 3本表示されるはずです。
(まれに node の作成に時間がかかる場合があります。何も node が表示されない場合は、さらに10分程度待ってく見てください)

```
oc get nodes
```

**実行例:**

```
$ oc get nodes
NAME                                       STATUS   ROLES    AGE     VERSION
ip-10-0-0-72.us-east-2.compute.internal    Ready    worker   9m44s   v1.27.6+b49f9d1
ip-10-0-1-195.us-east-2.compute.internal   Ready    worker   70s     v1.27.6+b49f9d1
ip-10-0-2-242.us-east-2.compute.internal   Ready    worker   9m28s   v1.27.6+b49f9d1
$
```

# 7.構成を探って見る

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
