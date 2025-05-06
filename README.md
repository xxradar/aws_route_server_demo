# VPC Route Server
This tutorial walks you through the process of setting up and configuring VPC Route Server to enable dynamic routing in your VPC. <br>
https://docs.aws.amazon.com/vpc/latest/userguide/route-server-tutorial.html
### Configure required IAM Role permissions 
#### Create a IAM policy
```
aws iam create-policy \
  --policy-name RouteServerPolicy \
  --policy-document file://route-server-policy.json
```
#### Create a role
```
aws iam create-role \
  --role-name RouteServerRole \
  --assume-role-policy-document file://trust-policy.json
```
#### Attach the policy to a Role
```
aws iam attach-role-policy \
  --role-name RouteServerRole \
  --policy-arn arn:aws:iam::974654858447:policy/RouteServerPolicy
```
### Route Server creation
#### Pre-requisites
Update you aws cli if necessary to the latest version.

#### Check region availability
```
export AWS_DEFAULT_REGION=eu-west-1
```
#### Create the route server
```
aws ec2 create-route-server --amazon-side-asn 65000
```
```
{
    "RouteServer": {
        "RouteServerId": "rs-054f70d70eff70cc6",
        "AmazonSideAsn": 65000,
        "State": "pending",
        "PersistRoutesState": "disabled",
        "SnsNotificationsEnabled": false
    }
}
```
**Note:** `State=pending`
#### Check the route server state
```
OUTPUT=$(aws ec2 describe-route-servers)
echo $OUTPUT
```
```
RSID=$(echo $OUTPUT | jq -r '.RouteServers[0].RouteServerId')
echo $RSID
```
```
{
    "RouteServers": [
        {
            "RouteServerId": "rs-054f70d70eff70cc6",
            "AmazonSideAsn": 65000,
            "State": "available",
            "Tags": [],
            "PersistRoutesState": "disabled",
            "SnsNotificationsEnabled": false
        }
    ]
}
```
**Note:** After a few seconds `State=available`

### Associate route server with a VPC
```
VPC="vpc-571faf2e"
```
```
aws ec2 associate-route-server --route-server-id $RSID --vpc-id $VPC
```
```
{
    "RouteServerAssociation": {
        "RouteServerId": "rs-054f70d70eff70cc6",
        "VpcId": "vpc-571faf2e",
        "State": "associating"
    }
}
```
**Note:** `State=associating`

```
aws ec2 get-route-server-associations --route-server-id $RSID
```
```
{
    "RouteServerAssociations": [
        {
            "RouteServerId": "rs-054f70d70eff70cc6",
            "VpcId": "vpc-571faf2e",
            "State": "associated"
        }
    ]
}
```
**Note:** After a few seconds `State=associated`
### Create route server endpoints
```
SUBNET="vpc-571faf2e"
```
```
aws ec2 create-route-server-endpoint --route-server-id $RSID --subnet-id $SUBNET
```
```
{
    "RouteServerEndpoint": {
        "RouteServerId": "rs-054f70d70eff70cc6",
        "RouteServerEndpointId": "rse-0b3aab44fa6f0bb15",
        "VpcId": "vpc-571faf2e",
        "SubnetId": "subnet-7a0ae703",
        "State": "pending"
    }
}
```
```
OUTPUT=$(aws ec2 describe-route-server-endpoints)
echo $OUTPUT
```
```
RSIDE=$(echo $OUTPUT | jq -r '.RouteServerEndpoints[0].RouteServerEndpointId')
echo $RSIDE
```
```
{
    "RouteServerEndpoints": [
        {
            "RouteServerId": "rs-054f70d70eff70cc6",
            "RouteServerEndpointId": "rse-0b3aab44fa6f0bb15",
            "VpcId": "vpc-571faf2e",
            "SubnetId": "subnet-7a0ae703",
            "EniId": "eni-0e9d73874dccdfea8",
            "EniAddress": "172.31.14.193",
            "State": "available",
            "Tags": []
        }
    ]
}
```
### Enable route server propagation 
```
RT="rtb-1be73d63"
```
```
aws ec2 enable-route-server-propagation --route-table-id $RT --route-server-id $RSID
```
```
{
    "RouteServerPropagation": {
        "RouteServerId": "rs-054f70d70eff70cc6",
        "RouteTableId": "rtb-1be73d63",
        "State": "pending"
    }
}
```
```
aws ec2 get-route-server-propagations --route-server-id $RSID 
```
```
{
    "RouteServerPropagations": [
        {
            "RouteServerId": "rs-054f70d70eff70cc6",
            "RouteTableId": "rtb-1be73d63",
            "State": "available"
        }
    ]
}
```
### Create route server peer
```
aws ec2 create-route-server-peer --route-server-endpoint-id $RSIDE --peer-address 10.0.2.3 --bgp-options PeerAsn=65001,PeerLivenessDetection=bfd
```
```
{
    "RouteServerPeer": {
        "RouteServerPeerId": "rsp-0e1c240ae33a4f642",
        "RouteServerEndpointId": "rse-0b3aab44fa6f0bb15",
        "RouteServerId": "rs-054f70d70eff70cc6",
        "VpcId": "vpc-571faf2e",
        "SubnetId": "subnet-7a0ae703",
        "State": "pending",
        "EndpointEniId": "eni-0e9d73874dccdfea8",
        "EndpointEniAddress": "172.31.14.193",
        "PeerAddress": "10.0.2.3",
        "BgpOptions": {
            "PeerAsn": 65001,
            "PeerLivenessDetection": "bfd"
        },
        "BgpStatus": {
            "Status": "down"
        },
        "BfdStatus": {
            "Status": "down"
        }
    }
}
```
```
aws ec2 describe-route-server-peers
```
```
{
    "RouteServerPeers": [
        {
            "RouteServerPeerId": "rsp-0e1c240ae33a4f642",
            "RouteServerEndpointId": "rse-0b3aab44fa6f0bb15",
            "RouteServerId": "rs-054f70d70eff70cc6",
            "VpcId": "vpc-571faf2e",
            "SubnetId": "subnet-7a0ae703",
            "State": "available",
            "EndpointEniId": "eni-0e9d73874dccdfea8",
            "EndpointEniAddress": "172.31.14.193",
            "PeerAddress": "10.0.2.3",
            "BgpOptions": {
                "PeerAsn": 65001,
                "PeerLivenessDetection": "bfd"
            },
            "BgpStatus": {
                "Status": "down"
            },
            "BfdStatus": {
                "Status": "down"
            },
            "Tags": []
        }
    ]
}
```
### Initiate BGP sessions from the devices
Setup FGT to iniiate a BGP session


### Cleanup
```
aws ec2 disable-route-server-propagation --route-table-id $RT --route-server-id $RSID
aws ec2 delete-route-server-peer --route-server-peer-id "rsp-0e1c240ae33a4f642"
aws ec2 delete-route-server-endpoint --route-server-endpoint-id $RSIDE
aws ec2 disassociate-route-server --route-server-id $RSID --vpc-id $VPC
aws ec2 delete-route-server --route-server-id $RSID
```
