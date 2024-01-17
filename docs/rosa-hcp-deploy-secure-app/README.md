# 標準の Nginx アプリがデプロイできない事を確認する

新しい Project を作ります。自動的に新しい Project に移動します。

```
oc new-project standard-nginx
```

docker hub にある、nginx の[公式イメージ](https://hub.docker.com/_/nginx)を Deployment を使用して Deploy します。

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

コンテナ環境は、仮想マシンと違い、一つのホストのカーネルを共有している環境であるため、その上で稼働するアプリの権限も適切に管理する必要があります。

# Nginx をセキュアに作り直す

DevSpace にログインします。

OpenShift で起動できるように、あらかじめ non root 化や、Kubernetes 環境用にカスタマイズされた nginx.conf ファイルと、イメージビルド用の Dockerfile をダウンロードして変更点を観察してみます。

```
git clone https://github.com/yuhkih/nginx-for-openshift.git 
```

ここでは標準の nginx のアプリをセキュアに作りなおすと同時に、Kubernetes 環境にそったカスタマイズをしてみます。

**Rule1:** ログやエラーはローカル・ファイルではなく、標準出力 / 標準入力に吐き出す (これはセキュリティというより Kubenretes 上のコンテナの一般的な"有るべし")

**Rule2:** non-root ユーザーで起動できるように、nginx 等の固有ユーザー名は使用しない (Dockerfile 内の USER 指定がある場合いは、消す必要まではないが、書いてあっても OpenShiftでは無視される。ランダムな Userが割り当てられる）

**Rule3:** non-root ユーザーで起動できるように、well-know port と呼ばれる 1024以下の TCPポートは使用しない

**Rule4:** non-root ユーザーで起動できるように、Process ID 等の保存に /run 等の Linux のシステムディレクトリは使用しない

このルールに従っていれば、大半の Kubernetes 環境にコンテナをデプロイする事が可能です。

nginx の設定ファイルである `nginx.conf` を以下のように書き替えます。[1]～[4]

```nginx.conf
# For more information on configuration, see:
#   * Official English Documentation: http://nginx.org/en/docs/
#   * Official Russian Documentation: http://nginx.org/ru/docs/

# user nginx;  # [1] 特定の User 名を使用しないようにコメントアウトします
worker_processes auto;
# error_log /var/log/nginx/error.log;  # [2] エラーログは標準エラーに出力するように書き直します。
error_log /dev/stderr ;
# pid /run/nginx.pid;　 # [3] Proccess ID の保存は /run を使わずに /tmp に変更します。
pid /tmp/nginx.pid;

# Load dynamic modules. See /usr/share/doc/nginx/README.dynamic.
include /usr/share/nginx/modules/*.conf;

events {
    worker_connections 1024;
}

http {
    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';

    # access_log  /var/log/nginx/access.log  main;
    access_log  /dev/stdout main;

    sendfile            on;
    tcp_nopush          on;
    tcp_nodelay         on;
    keepalive_timeout   65;
    types_hash_max_size 2048;

    include             /etc/nginx/mime.types;
    default_type        application/octet-stream;

    # Load modular configuration files from the /etc/nginx/conf.d directory.
    # See http://nginx.org/en/docs/ngx_core_module.html#include
    # for more information.
    include /etc/nginx/conf.d/*.conf;

    server {
        # listen       80 default_server;
        # listen       [::]:80 default_server;   # [4] 1024以下の Well-known ポートは使用しない。ここでは8080に変更します。
        listen 0.0.0.0:8080;
        server_name  _;
        root         /usr/share/nginx/html;

        # Load configuration files for the default server block.
        include /etc/nginx/default.d/*.conf;

        location / {
        }

        error_page 404 /404.html;
            location = /40x.html {
        }

        error_page 500 502 503 504 /50x.html;
            location = /50x.html {
        }
    }
}

$
```

index.html

```
Hello OpenShift World !
```


Dockerfile

```
FROM redhat/ubi8
RUN yum install -y nginx
COPY index.html /usr/share/nginx/html/index.html
COPY nginx.conf /etc/nginx/nginx.conf
EXPOSE 8080
CMD ["-g","daemon off;"]
ENTRYPOINT ["nginx"]
```

>[!TIP]
>このサンプルでは必要ありませんが、root group (GID=0) に所属するユーザーが OpenShift によって自動的に割り当てられるので、アプリの実行に必要なディレクトリに対して Dockefile 内で以下をおこなっておくのがベストプラクティスとされています。
>```
>RUN chgrp -R 0 /some/directory && \
>   chmod -R g=u /some/directory
>```
>こうする事で、`/some/directory` に、root group(GID=0) のユーザーにアプリの実行に十分な権限を与える事ができます。なお root group というのは、名前とは裏腹にただの一般グループで特別な権限を持つグループではありません。

これらのファイルを使用して Image をビルドします。

```
podman build . -t new-nginx
```

ビルドした Image を確認します。

```
podman images
```

**出力例:**

```
$ podman images
REPOSITORY                                                     TAG         IMAGE ID      CREATED         SIZE
localhost/new-nginx                                           latest      3c732cd2eabb  21 seconds ago  303 MB
$ 
```





