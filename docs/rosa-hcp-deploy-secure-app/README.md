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

コンテナ環境は、仮想マシンと違い、一つのホストのカーネルを共有している環境であるため、その上で稼働するアプリの権限も適切に管理する必要があります。

# Nginx をセキュアに作り直す

ここでは標準の nginx のアプリをセキュアに作りなおすと同時に、Kubernetes 環境にそったカスタマイズをしてみます。

**Rule1:** ログやエラーは標準出力 / 標準入力に吐き出す

**Rule2:** nginx 等の固有ユーザー名は使用しない

**Rule3:** well-know port と呼ばれる 1024以下の TCPポートは使用しない

**Rule4:** Process ID 等の保存に /run 等の Linux のシステムディレクトリは使用しない

これらのルールは、例えば特定のアプリに PV(Persistent Volume) を持たせてログをそこに出力するようにする。等のように環境によって違う事もありますが、大体の Kubernetes 環境で共通のルールとして使用できます。
このルールに従っていれば、大半の Kubernetes 環境にコンテナをデプロイする事が可能です。

nginx の設定ファイルである `nginx.conf` を以下のように書き替えます。[1]～[4]

```nginx.conf
$ cat nginx.conf 
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

# Pod自身はHTTPSに対応してなくても Router が終端するので問題無い。
# Settings for a TLS enabled server.
#
#    server {
#        listen       443 ssl http2 default_server;
#        listen       [::]:443 ssl http2 default_server;
#        server_name  _;
#        root         /usr/share/nginx/html;
#
#        ssl_certificate "/etc/pki/nginx/server.crt";
#        ssl_certificate_key "/etc/pki/nginx/private/server.key";
#        ssl_session_cache shared:SSL:1m;
#        ssl_session_timeout  10m;
#        ssl_ciphers PROFILE=SYSTEM;
#        ssl_prefer_server_ciphers on;
#
#        # Load configuration files for the default server block.
#        include /etc/nginx/default.d/*.conf;
#
#        location / {
#        }
#
#        error_page 404 /404.html;
#            location = /40x.html {
#        }
#
#        error_page 500 502 503 504 /50x.html;
#            location = /50x.html {
#        }
#    }

}

$
```



