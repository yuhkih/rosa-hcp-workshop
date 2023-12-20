# 1.AWS CLI / Git の準備

ここも書く。

## 1.1 AWS CLI の install

[こちらの AWS のページ](https://docs.aws.amazon.com/ja_jp/cli/latest/userguide/getting-started-install.html)を参考にして AWS CLI をインストールします。


aws configure を使用して、`AWS Access Key ID` や `AWS Secret Access Key` の値を構成します。`AWS Access Key ID` や `AWS Secret Access Key` は、AWS Console から取得できます。[こちら](https://docs.aws.amazon.com/ja_jp/IAM/latest/UserGuide/id_credentials_access-keys.html#Using_CreateAccessKey) を参考にしてください。

```
aws configure
```

以下のコマンドを実行して正しく構成されているか確認します。

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

# 3 ROSA 作成用の token の取得

ROSA cluster を作成するためには、Red Hat が提供する token が必要です。以下のコマンドを実行します。

```
rosa login
```

以下のように聞かれます。

```
$ rosa login
To login to your Red Hat account, get an offline access token at https://console.redhat.com/openshift/token/rosa
? Copy the token and paste it here: 
```

表示されたリンク [https://console.redhat.com/openshift/token/rosa](https://console.redhat.com/openshift/token/rosa) にログインして、token を取得します。Red Hat Portal の ID (無料) が必要になるので、作って無い場合は、作成してからこのリンクに再びアクセスします。

以下の画面が表示されるので、Token をコピーしてプロンプトに貼り付けます。
![image](https://github.com/yuhkih/rosa-hcp-workshop/assets/8530492/a4f34b68-c230-4b55-b3b2-7e602d76a62c)

```
$ rosa login
To login to your Red Hat account, get an offline access token at https://console.redhat.com/openshift/token/rosa
? Copy the token and paste it here: ******************************************************************************************************************************************************************************************************

I: Logged in as 'yuhkih' on 'https://api.openshift.com'
$
```

以上で token の準備は完了です。


# 4.ROSA HCP の有効化

## 4.1 ROSA HCP の有効化
以下のリンクをクリックして AWS の ROSA 設定画面に飛びます。
[https://console.aws.amazon.com/rosa/home#/get-started](https://console.aws.amazon.com/rosa/home#/get-started)

[Enable ROSA with HCP] のボタンをクリックします。
![image](https://github.com/yuhkih/rosa-hcp-workshop/assets/8530492/084e3a5c-e76f-460a-9164-b55dfe06a4cb)

有効化されるまで、暫く待ちます。数分かかるはずです。
![image](https://github.com/yuhkih/rosa-hcp-workshop/assets/8530492/26af4b3a-d7c6-4ebe-951f-fac2eac548c5)

有効化が完了すると以下のような表示になっているはずです。
![image](https://github.com/yuhkih/rosa-hcp-workshop/assets/8530492/15d51fb2-22bf-454b-aa95-5cfacacc678b)

## 4.2 Service Quota の確認

もし Service Quota が足りない場合はチケットを上げて確認します。最終的に以下の状態になっていれば大丈夫です。
![image](https://github.com/yuhkih/rosa-hcp-workshop/assets/8530492/41b134f4-8cbc-48bc-b023-104082d3550b)

## 4.3 ELB サービスにリンクされたロールの作成

過去に ELB をデプロイした事があれば `AWSServiceRoleForElasticLoadBalancing` というIAM Role が作成されており、以下のような表示になっているはずです。

![image](https://github.com/yuhkih/rosa-hcp-workshop/assets/8530492/35975e14-6847-4b9a-b36e-b52295f0891d)

もし作成されていない場合は、以下のコマンドで作成します。

```
aws iam create-service-linked-role --aws-service-name "elasticloadbalancing.amazonaws.com"
```
## 4.4 Red Hat Customer Portal の情報とリンクする

画面の一番下に移動して「Continue Red Hat」をクリックします。
![image](https://github.com/yuhkih/rosa-hcp-workshop/assets/8530492/87d5a503-7a0a-4a51-9c08-0aa7ad2dc026)

Red Hat の ポータルサイトにログインします。(Red Hat アカウントが無い場合ば作成してから、再度[こちら](https://console.redhat.com/connect/aws) にアクセスします。Red Hat アカウントは無料で作成できます）
![image](https://github.com/yuhkih/rosa-hcp-workshop/assets/8530492/565e0b8d-eada-4d52-a1c3-16c58bae93fa)

日本語を選んで「Connect accounts] をクリックします。
![image](https://github.com/yuhkih/rosa-hcp-workshop/assets/8530492/2148e419-4753-421b-a572-bcefc2660df3)

以上で HCP の有効化は完了です。
