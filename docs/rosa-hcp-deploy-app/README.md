

# 簡単なサンプルコンテナをデプロイしてみる

hello-openshift という Web Server のコンテナをデプロイしてみます。アクセスすると、メッセージを返すシンプルなコンテナです。

`hello-openshift` という `project` (OpenShift で使われる `namespace` の拡張概念) を作成します

```
oc new-project hello-openshift
```

`hello-openshift` イメージを使った `deployment` を作成します。名前はコンテナ名と同じ　`hello-openshift` にします。


```
oc create deployment hello-openshift --image=quay.io/openshift/origin-hello-openshift
```

`deployment` が作成されたか確認します。

```
oc get deployment
```

`clusterip` の `service` を作成します。 hello-openshift コンテナが使用している 8080 を公開します。名前はコンテナ名と同じ　`hello-openshift` にします。
コンテナイメージがどのポートを使用しているかをコマンド等で突き止める事もできますが、基本的に事前に知っている必要がある事に注意してください。

```
oc create service clusterip hello-openshift --tcp=8080:8080
```

OpenShift での `ingress` の同等の概念である `route` を作成します。HTTP アプリケーションの場合、`route` は、`service` を公開することで作成されます。

```
oc expose service hello-openshift
```

`route` が作成されたか確認します。

```
oc get route
```

以下のような出力になるはずです。環境によって URL は違います。`route` の名前は、公開した `service` と同じ名前になっているはずです。

```
$ oc get route
NAME              HOST/PORT                                                                  PATH   SERVICES          PORT        TERMINATION   WILDCARD
hello-openshift   hello-openshift-test2.apps.rosa.my-hpc-cluster.rc4b.p3.openshiftapps.com          hello-openshift   8080-8080                 None
$
```

作成された `route` にアクセスしてみます。

```
$ curl hello-openshift-test2.apps.rosa.my-hpc-cluster.rc4b.p3.openshiftapps.com
Hello OpenShift!
$
```

# 作成したアプリのレプリカ数を増やしてみる

可用性を保つために `Pod` の replica 数を3つに増やしてみます。

```
oc scale deployment hello-openshift --replicas=3
```

Pod が ３つになっている事を以下のコマンドで確認します。

```
oc get pods
```

３つに増やしても引き続きアプリケーションにアクセスできることを curl コマンドで確認します。(アクセス先 URL は `oc get route` で表示される URL です)

```
curl hello-openshift-test2.apps.rosa.my-hpc-cluster.rc4b.p3.openshiftapps.com
```

# デプロイしたアプリケーションを削除する

作成した `deployment`、`service`、`route` は、`project` を削除することで全て消す事ができます。
以下のコマンドで実験で使用したアプリケーションを削除します。

```
oc delete project hello-openshift
```

# oc new-app を使用して簡単なサンプルコンテナをデプロイしてみる

前回は kubernetes の標準コマンドを使用してコンテナをデプロイしましたが、今度は同じ事を OpenShift の独自コマンドを使用して行ってみます。

新しい `project` を作成します。

```
oc new-project hello-openshift2
```

以下のコマンドで `hello-openshift` コンテナを使った `deployment` と `service` を一気に作成します。この `oc new-app` は OpenShift の独自コマンドです。
container のイメージが公開している port の情報を持っている場合は、`service` まで作成してくれます。

```
oc new-app hello-openshift --image quay.io/openshift/origin-hello-openshift
```

`route` を作成します。

```
oc expose service hello-oppenshift
```

作成された `route` を確認します。

```
oc get route
```

`curl` コマンドでアクセスして確認します。

```
$ curl hello-openshift-hello-openshift.apps.rosa.my-hpc-cluster.rc4b.p3.openshiftapps.com
Hello OpenShift!
$
```

ちょっとだけですが、OpenShift の独自コマンドを使うことで手数を減らす事ができるようになっています。




