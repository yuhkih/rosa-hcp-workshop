

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

作成したローカルイメージにタグを付けます。

```
podman tag mynginx:latest $IMAGE_SERVER/new-nginx/mynginx:latest
```

```
podman push mynginx:latest $IMAGE_SERVER/new-nginx/mynginx:latest
```
