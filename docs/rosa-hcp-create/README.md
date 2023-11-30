# ROSA 用の Network の作成

HCP ROSA は、ユーザーが既にもっているネットワークにデプロイする事が前提になります。
ここでは Terraform を使用して AWS　上に Network を作成します。

ここの手順は、ドキュメントの「<a href="https://docs.openshift.com/rosa/rosa_hcp/rosa-hcp-sts-creating-a-cluster-quickly.html#rosa-hcp-creating-vpc" target="_blank">Creating a Vritual Private Cloud for your ROSA with HCP clusters</a>
」をベースにしています。

## terraform を使用した Subnet と NAT Gateway の作成

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
```

Terraform の plan を作成します。以下は Multi AZ 環境の Network を作成します。

```
terraform plan -out rosa.tfplan -var region=$REGION -var cluster_name=$CLUSTER_NAME -var single_az_only=false
```

Apply して Network を作成します。

```
terraform apply rosa.tfplan
```

以下のような出力が出るはずです。

```
<省略>
module.vpc.aws_route.private_nat_gateway[2]: Creation complete after 1s [id=r-rtb-0450c880ed2f1c9131080289494]
module.vpc.aws_route.private_nat_gateway[1]: Creation complete after 1s [id=r-rtb-0e8a172ae7dceff351080289494]

Apply complete! Resources: 31 added, 0 changed, 0 destroyed.

Outputs:

cluster-private-subnets = [
  "subnet-0b99d3132ac4385a0",
  "subnet-0b004648512872083",
  "subnet-0ea945bfaedc67bc3",
]
cluster-public-subnets = [
  "subnet-0205e882a6038c687",
  "subnet-0838709de6d36ea43",
  "subnet-0e968b1965e53084b",
]
cluster-subnets-string = "subnet-0205e882a6038c687,subnet-0838709de6d36ea43,subnet-0e968b1965e53084b,subnet-0b99d3132ac4385a0,subnet-0b004648512872083,subnet-0ea945bfaedc67bc3"
$
````

## 作成された Subnet と NAT Gateway の確認

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
