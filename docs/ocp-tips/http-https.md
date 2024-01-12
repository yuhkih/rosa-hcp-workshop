![image](https://github.com/yuhkih/rosa-hcp-workshop/assets/8530492/a12ab9af-578a-4968-8eae-94fc44afb42c)# HTTP と HTTPS の両方を受け付ける Route を作る

デフォルトでは `Route` は `HTTP` か `HTTPS` のどちらかしかトラフィックを終端しません。
`Route` で `HTTP`/`HTTPS` の両方を終端するには、`Route`内で `insecureEdgeTerminationPolicy: Allow` を設定します。

```
# Please edit the object below. Lines beginning with a '#' will be ignored,
# and an empty file will abort the edit. If an error occurs while saving this file will be
# reopened with the relevant failures.
#
apiVersion: route.openshift.io/v1
kind: Route
metadata:
  annotations:
    openshift.io/host.generated: "true"
<-- snip -->
spec:
  host: hello-openshift-test.apps.rosa-hpxrf.zpq2.p1.openshiftapps.com
  tls:
    insecureEdgeTerminationPolicy: Allow
    termination: edge
  to:
    kind: Service
    name: hello-openshift
<-- snip -->
```

# 実験


作成した Route は、1つ

```
$ oc get route
NAME              HOST/PORT                                                        PATH   SERVICES          PORT    TERMINATION   WILDCARD
hello-openshift   hello-openshift-test.apps.rosa-hpxrf.zpq2.p1.openshiftapps.com          hello-openshift   <all>   edge/Allow    None
$
```

`HTTP` と `HTTPS` のどちらでも上手く行く事を確認

```
$ curl https://hello-openshift-test.apps.rosa-hpxrf.zpq2.p1.openshiftapps.com
Hello OpenShift!
$ curl http://hello-openshift-test.apps.rosa-hpxrf.zpq2.p1.openshiftapps.com
Hello OpenShift!
$
```


# 補則情報

- `route` を `HTTP` と `HTTPS` でそれぞれ作るというアイディアは上手くいかない。`HOST`名が被るため、どちらか1つしか作成できない。
- `oc explain` では `insecureEdgeTerminationPolicy` は以下のように表示される。
```
$ oc explain route.spec.tls.insecureEdgeTerminationPolicy
GROUP:      route.openshift.io
KIND:       Route
VERSION:    v1

FIELD: insecureEdgeTerminationPolicy <string>

DESCRIPTION:
    insecureEdgeTerminationPolicy indicates the desired behavior for insecure
    connections to a route. While each router may make its own decisions on
    which ports to expose, this is normally port 80.
    
    * Allow - traffic is sent to the server on the insecure port (default) *
    Disable - no traffic is allowed on the insecure port. * Redirect - clients
    are redirected to the secure port.
$ 
```
> [!NOTE]
> `Allow` がデフォルトとあるが、実際の動きは、OCP4.14 時点で `Disable` と同じ動きになる。もともと HTTPS での接続を強制する流れが一般的になってきているので、気がつかずに `HTTP` での接続をしている事がないように、`Disable` がデフォルトになったのかもしれない。
> また `traffic is sent to the server on the insecure port` とあるが、`Route` から traffic 送られる(`sent`) わけではなく、`Route` の着信ポートの話しなので `received / accepted` の方がわかりやすい気がする。

- デフォルトの状態 (`insecureEdgeTerminationPolicy`指定なし）で、アクセスできないプロトコル側にアクセスすると以下のようなエラーになる。
```
$ curl http://hello-openshift-test.apps.rosa-hpxrf.zpq2.p1.openshiftapps.com 
<-- snip! -->

      <h1>Application is not available</h1>
      <p>The application is currently not serving requests at this endpoint. It may not have been started or is still starting.</p>

      <div class="alert alert-info">
        <p class="info">
          Possible reasons you are seeing this page:
        </p>
        <ul>
          <li>
            <strong>The host doesn't exist.</strong>
            Make sure the hostname was typed correctly and that a route matching this hostname exists.
          </li>
          <li>
            <strong>The host exists, but doesn't have a matching path.</strong>
            Check if the URL path was typed correctly and that the route was created using the desired path.
          </li>
          <li>
            <strong>Route and path matches, but all pods are down.</strong>
            Make sure that the resources exposed by this route (pods, services, deployment configs, etc) have at least one pod running.

<-- snip! -->
$
```
上記のページはブラウザで見ると以下のようになる。`Route` が返しているエラーページ。
![image](https://github.com/yuhkih/rosa-hcp-workshop/assets/8530492/27366099-b4e0-46c2-af42-62b8c5b5bed7)

この時の `HTTP Status Code` は `503 Service Unavailable` である。

```
$ curl http://hello-openshift-test.apps.rosa-hpxrf.zpq2.p1.openshiftapps.com  -s -w '%{http_code}\n' -o /dev/null
503
$ 
```
