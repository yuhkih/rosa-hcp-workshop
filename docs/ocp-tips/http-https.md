# HTTP と HTTPS の両方を受け付ける Route を作る

デフォルトでは Route は HTTP か HTTPS のどちらかしかトラフィックを終端しません。
Route で HTTP/HTTPS の両方を終端するには、Route で `insecureEdgeTerminationPolicy: Allow` を設定します。

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
`Allow` がデフォルトとあるが、明らかにそのようには動いてない。実際の動きは、OCP4.14 時点で `Disable` と同じになる。
また `sent to the server on the insecure port` とあるが、`Route` から traffic 送られる(`sent`) わけではなく、`Route` での `Termination` の話しなので `received / accepted` の方が正しい気がする。
