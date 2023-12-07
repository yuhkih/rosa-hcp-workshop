
# ローカル Image Registry の公開


Image Registry をインターネットに公開します。

```
oc patch configs.imageregistry.operator.openshift.io/cluster --patch '{"spec":{"defaultRoute":true}}' --type=merge
```

# ローカル Image Registry へのログイン

Image Registry の ドメイン名を取得します。

```
export IMAGE_SERVER=`oc get route default-route -n openshift-image-registry --template='{{ .spec.host }}'`
```

Image Registry にログインします。

```
podman login -u `oc whoami` -p `oc whoami --show-token` ${IMAGE_SERVER}
```

# ローカル Image Registry への Image の Push

現在、ある image を確認します。

```
nginx $ podman images
REPOSITORY                   TAG         IMAGE ID      CREATED         SIZE
localhost/new-nginx  latest      d623ca329bc4  19 minutes ago  303 MB
nginx $ 
```

作成したローカルイメージにタグを付けます。

```
podman tag localhost/new-nginx:latest $IMAGE_SERVER/new-nginx/mynginx:latest
```

イメージを push します

```
podman push  $IMAGE_SERVER/new-nginx/mynginx:latest
```

# Push した Image を使用した コンテナの Deploy

ローカル Image Registry に Push したイメージを使用して Deploy します。

```
oc new-app --name new-nginx --image $IMAGE_SERVER/new-nginx/mynginx:latest
```

今度は Pod が起動している事を確認します。

```
oc get pods
```

**出力例:**

```
 $ oc get pods
NAME                            READY   STATUS    RESTARTS   AGE
new-nginx-599f687494-vk78j      1/1     Running   0          7s
$ 
```

サービスを公開します。

```
oc expose svc new-nginx
```

route を確認します。

```
oc get route
```

**出力例:**

```
$ oc get route
NAME           HOST/PORT                                                                   PATH   SERVICES       PORT       TERMINATION   WILDCARD
new-nginx      new-nginx-new-nginx.apps.rosa.my-hpc-cluster.pd80.p3.openshiftapps.com             new-nginx      8080-tcp                 None
$
```

curl でアクセスしてみます。

```
curl <Route の Host名>
```

**出力例:**
```
$ curl new-nginx-new-nginx.apps.rosa.my-hpc-cluster.pd80.p3.openshiftapps.com
Hello OpenShift World!
$
```


**出力例:**
