# ROSA 用の Network の作成

HCP ROSA は、ユーザーが既にもっているネットワークにデプロイする事が前提になります。
ここでは Terraform を使用して AWS　上に Network を作成します。




# ROSA HCP Cluster の 作成

```
hcp reate cluster aws \
 --name $CLUSTER_NAME \
 --node-pool-replicas=3 \
 --base-domain $BASE_DOMAIN \
 --pull-secret $PULL_SECRET \
 --aws-creds $AWS_CREDS \
 --region $REGION \
 --zones $ZONES \
 --namespace $NAMESPACE
```


# ROSA HCP Cluster へのアクセス確認
