

Image Registry をインターネットに公開します。

```
oc patch configs.imageregistry.operator.openshift.io/cluster --patch '{"spec":{"defaultRoute":true}}' --type=merge
```

Image Registry の ドメイン名を取得します。


```
export IMAGE_SERVER=`oc get route default-route -n openshift-image-registry --template='{{ .spec.host }}'`
```

Image Registry にログインします。

```
podman login -u `oc whoami` -p `oc whoami --show-token` ${IMAGE_SERVER}
```

現在、ある image を確認します。

```
nginx $ podman images
REPOSITORY                   TAG         IMAGE ID      CREATED         SIZE
localhost/yuhkih/nginx-ubi8  latest      d623ca329bc4  19 minutes ago  303 MB
nginx $ 
```

作成したローカルイメージにタグを付けます。

```
podman tag localhost/yuhkih/nginx-ubi8:latest $IMAGE_SERVER/new-nginx/mynginx:latest
```

イメージを push します

```
podman push  $IMAGE_SERVER/new-nginx/mynginx:latest
```

# コンテナの Deploy

```
oc create deployment new-nginx --image $IMAGE_SERVER/new-nginx/mynginx:latest
```

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


