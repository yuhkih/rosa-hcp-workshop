# 1.変数を確認する

クラスター名が変数にセットされているか確認します。

```
echo $CLUSTER_NAME
```

# 2.ROSA クラスターの削除

以下のコマンドでクラスターを削除します。

```
 rosa delete cluster -c $CLUSTER_NAME
```

ログの最後の出てくる以下の部分はメモしておきます。
```
        rosa delete operator-roles --prefix <IAM Role prefix>
        rosa delete oidc-provider --oidc-config-id <OIDC provider config ID>
```

**実行例:**

```
$ rosa delete cluster -c $CLUSTER_NAME
? Are you sure you want to delete cluster my-hpc-cluster? Yes
I: Cluster 'my-hpc-cluster' will start uninstalling now
I: Your cluster 'my-hpc-cluster' will be deleted but the following objects may remain
I: Operator IAM Roles: - arn:aws:iam::972708010773:role/my-hpc-cluster-n8m7-kube-system-capa-controller-manager
 - arn:aws:iam::972708010773:role/my-hpc-cluster-n8m7-kube-system-control-plane-operator
 - arn:aws:iam::972708010773:role/my-hpc-cluster-n8m7-kube-system-kms-provider
 - arn:aws:iam::972708010773:role/my-hpc-cluster-n8m7-openshift-image-registry-installer-cloud-cre
 - arn:aws:iam::972708010773:role/my-hpc-cluster-n8m7-openshift-ingress-operator-cloud-credentials
 - arn:aws:iam::972708010773:role/my-hpc-cluster-n8m7-openshift-cluster-csi-drivers-ebs-cloud-cred
 - arn:aws:iam::972708010773:role/my-hpc-cluster-n8m7-openshift-cloud-network-config-controller-cl
 - arn:aws:iam::972708010773:role/my-hpc-cluster-n8m7-kube-system-kube-controller-manager

I: OIDC Provider : https://rh-oidc.s3.us-east-1.amazonaws.com/267bh4stja59r9gc896alcbdfls2h6jp

I: Once the cluster is uninstalled use the following commands to remove the above aws resources.

        rosa delete operator-roles --prefix my-hpc-cluster-n8m7
        rosa delete oidc-provider --oidc-config-id 267bh4stja59r9gc896alcbdfls2h6jp
I: To watch your cluster uninstallation logs, run 'rosa logs uninstall -c my-hpc-cluster --watch'
$
```

クラスタの削除過程は、以下のコマンドで確認できます。

```
rosa logs uninstall -c $CLUSTER_NAME --watch
```

# 3.IAM Role と OIDC Provider の削除

クラスターの削除が完了したら、Operator 用の IAM Role と OIDC Provider を削除します。

Operator 用 IAM Role を削除します。これは、`rosa delete cluster` コマンドを実行した時にログの最後に出てきたコマンドです。

```
rosa delete operator-roles --prefix <IAM Role prefix>
```

OIDC Provider を削除します。これは、`rosa delete cluster` コマンドを実行した時にログの最後に出てきたコマンドです。

```
rosa delete oidc-provider --oidc-config-id <OIDC provider config ID>
```
