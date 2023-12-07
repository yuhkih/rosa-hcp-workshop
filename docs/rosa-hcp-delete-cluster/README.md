# クラスターを削除する

クラスター名が変数にセットされているか確認します。

```
echo $CLUSTER_NAME
```

以下のコマンドでクラスターを削除します。

```
 rosa delete cluster -c $CLUSTER_NAME
```

クラスタの削除過程は、以下のコマンドで確認できます。

```
rosa logs uninstall -c $CLUSTER_NAME --watch
```

クラスターの削除が完了したら、Operator 用の IAM Role と OIDC Provider を削除します。

Operator 用 IAM Role を削除します。

```
 rosa delete operator-roles --prefix my-hpc-cluster-n8m7
```

OIDC Provider を削除します。

```
 rosa delete oidc-provider --oidc-config-id 267bh4stja59r9gc896alcbdfls2h6jp
```
