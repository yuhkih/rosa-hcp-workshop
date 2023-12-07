
作成されたクラスターをリストします。

```
rosa list clusters
```

**出力例:**

```
$ rosa list clusters
ID                                NAME            STATE  TOPOLOGY
27vfb6mckkl9ndle98vvcgchc2nlc54m  my-hpc-cluster  ready  Hosted CP
$ 
```



```
rosa list machinepools -c $CLUSTER_NAME
```

**出力例:**

```
$ rosa list machinepools -c $CLUSTER_NAME
ID         AUTOSCALING  REPLICAS  INSTANCE TYPE  LABELS    TAINTS    AVAILABILITY ZONE  SUBNET                    VERSION  AUTOREPAIR  
workers-0  No           1/1       m5.xlarge                          us-east-2c         subnet-00e5abea4e74f6852  4.14.4   Yes         
workers-1  No           1/1       m5.xlarge                          us-east-2b         subnet-047bd69e1ae06c742  4.14.4   Yes         
workers-2  No           1/1       m5.xlarge                          us-east-2a         subnet-0c52bfe790deb6429  4.14.4   Yes         
$ 
```

```
rosa list ingress -c $CLUSTER_NAME
```

**出力例:**

```
$ rosa list ingress -c $CLUSTER_NAME
ID    APPLICATION ROUTER                                          PRIVATE  DEFAULT  ROUTE SELECTORS  LB-TYPE  EXCLUDED NAMESPACE  WILDCARD POLICY  NAMESPACE OWNERSHIP  HOSTNAME  TLS SECRET REF
c7n0  https://apps.rosa.my-hpc-cluster.pd80.p3.openshiftapps.com  no       yes                       nlb                                                                          
$ 
```


```
rosa list regions 
```

**出力例:**

```
$ rosa list regions
ID                NAME                        MULTI-AZ SUPPORT    HOSTED-CP SUPPORT
ap-northeast-1    Asia Pacific, Tokyo         true                false
ap-northeast-2    Asia Pacific, Seoul         true                false
ap-northeast-3    Asia Pacific (Osaka)        true                false
ap-south-1        Asia Pacific, Mumbai        true                false
ap-southeast-1    Asia Pacific, Singapore     true                false
ap-southeast-2    Asia Pacific, Sydney        true                false
ca-central-1      Canada, Central             true                false
eu-central-1      EU, Frankfurt               true                true
eu-north-1        EU, Stockholm               true                false
eu-west-1         EU, Ireland                 true                true
eu-west-2         EU, London                  true                false
eu-west-3         EU, Paris                   true                false
sa-east-1         South America, São Paulo    true                false
us-east-1         US East, N. Virginia        true                true
us-east-2         US East, Ohio               true                true
us-west-1         US West, N. California      false               false
us-west-2         US West, Oregon             true                true
$ 
```



```
 rosa list instance-types
```

**出力例:**

```
$ rosa list instance-types
ID                 CATEGORY               CPU_CORES  MEMORY  
dl1.24xlarge       accelerated_computing  96         768.0 GiB
g4dn.12xlarge      accelerated_computing  48         192.0 GiB
g4dn.16xlarge      accelerated_computing  64         256.0 GiB
g4dn.2xlarge       accelerated_computing  8          32.0 GiB
g4dn.4xlarge       accelerated_computing  16         64.0 GiB
g4dn.8xlarge       accelerated_computing  32         128.0 GiB

<省略>

m5d.4xlarge        storage_optimized      16         64.0 GiB
m5d.8xlarge        storage_optimized      32         128.0 GiB
m5d.xlarge         storage_optimized      4          16.0 GiB
$ 
```
