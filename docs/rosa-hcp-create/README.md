# ROSA 用の Network の作成

HCP ROSA は、ユーザーが既にもっているネットワークにデプロイする事が前提になります。
ここでは Terraform を使用して AWS　上に Network を作成します。

ここの手順は、ドキュメントの「<a href="https://docs.openshift.com/rosa/rosa_hcp/rosa-hcp-sts-creating-a-cluster-quickly.html#rosa-hcp-creating-vpc" target="_blank">Creating a Vritual Private Cloud for your ROSA with HCP clusters</a>
」をベースにしています。


```
git clone https://github.com/openshift-cs/terraform-vpc-example
```

```
cd terraform-vpc-example
```

```
terraform init
```

変数を準備しておきます。(日本 Region が使用可能になるのは 2024/1月頃なので US-EAST を使用）

```
export CLUSTER_NAME=my-hpc-cluster
export REGION=us-east-2
export AZ=["us-east-2a", "us-east-2b", "us-east-2c"]
```

Terraform の plan を作成します。

```
terraform plan -out rosa.tfplan -var region=$REGION -var cluster_name=$CLUSTER_NAME
```

Apply して Network を作成します。

```
terraform apply rosa.tfplan
```

AWS CLI を使用して作成された VPC と Subnet を確認します。

```
aws ec2 describe-subnets | jq -r '.Subnets[] | [ .CidrBlock, .SubnetId, .AvailabilityZone, .Tags[].Value ] | @csv'
```

```
aws ec2 describe-nat-gateways | jq -r .NatGateways[].NatGatewayId
```


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
