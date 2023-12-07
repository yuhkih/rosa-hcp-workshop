# 標準の Nginx アプリがデプロイできない事を確認する

新しい Project を作ります。自動的に新しい Project に移動します。

```
oc new-project standard-nginx
```

docker hub にある、nginx のコンテナを Deployment を使用して Deploy します。

```
 oc create deployment standard-nginx --image nginx
```

pod が `CrashLooBackOff` になっている事を確認します。

```
oc get pods
```

**実行例:**

```
$ oc get pods
NAME                              READY   STATUS             RESTARTS      AGE
standard-nginx-768459d6bc-wldlr   0/1     CrashLoopBackOff   2 (30s ago)   53s
$ 
```

Pod のログを確認してみます。

```
oc logs <Pod 名>
```

**実行例:**

```
$ oc logs standard-nginx-768459d6bc-wldlr
/docker-entrypoint.sh: /docker-entrypoint.d/ is not empty, will attempt to perform configuration
/docker-entrypoint.sh: Looking for shell scripts in /docker-entrypoint.d/
/docker-entrypoint.sh: Launching /docker-entrypoint.d/10-listen-on-ipv6-by-default.sh
10-listen-on-ipv6-by-default.sh: info: can not modify /etc/nginx/conf.d/default.conf (read-only file system?)
/docker-entrypoint.sh: Sourcing /docker-entrypoint.d/15-local-resolvers.envsh
/docker-entrypoint.sh: Launching /docker-entrypoint.d/20-envsubst-on-templates.sh
/docker-entrypoint.sh: Launching /docker-entrypoint.d/30-tune-worker-processes.sh
/docker-entrypoint.sh: Configuration complete; ready for start up
2023/12/07 04:23:32 [warn] 1#1: the "user" directive makes sense only if the master process runs with super-user privileges, ignored in /etc/nginx/nginx.conf:2
nginx: [warn] the "user" directive makes sense only if the master process runs with super-user privileges, ignored in /etc/nginx/nginx.conf:2
2023/12/07 04:23:32 [emerg] 1#1: mkdir() "/var/cache/nginx/client_temp" failed (13: Permission denied)
nginx: [emerg] mkdir() "/var/cache/nginx/client_temp" failed (13: Permission denied)
$ 
```

幾つか権限のエラーのようなものが出ている事を確認します。
これは OpenShift がデフォルトで、一般的には不必要な強い権限をもったアプリケーションの実行を防ぐ設定になっているため発生します。


# 標準のアプリをセキュアに作り直す
