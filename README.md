# VPC Route Server
This tutorial walks you through the process of setting up and configuring VPC Route Server to enable dynamic routing in your VPC. 
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
- Check region availability
- Update you aws cli if necessary
#### Create the route server
```
aws ec2 create-route-server --amazon-side-asn 65000 --region eu-west-1
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
aws ec2 describe-route-servers --region eu-west-1
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
aws ec2 associate-route-server --route-server-id rs-054f70d70eff70cc6 --vpc-id vpc-571faf2e --region eu-west-1
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
aws ec2 get-route-server-associations --route-server-id rs-054f70d70eff70cc6 --region eu-west-1
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
aws ec2 create-route-server-endpoint --route-server-id rs-054f70d70eff70cc6 --subnet-id subnet-7a0ae703 --region eu-west-1
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
aws ec2 describe-route-server-endpoints --region eu-west-1
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
aws ec2 enable-route-server-propagation --route-table-id rtb-1be73d63 --route-server-id rs-054f70d70eff70cc6 --region eu-west-1
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
aws ec2 get-route-server-propagations --route-server-id rs-054f70d70eff70cc6 --region eu-west-1
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
aws ec2 create-route-server-peer --route-server-endpoint-id rse-0b3aab44fa6f0bb15 --peer-address 10.0.2.3 --bgp-options PeerAsn=65001,PeerLivenessDetection=bfd --region eu-west-1
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
aws ec2 describe-route-server-peers --region eu-west-1
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

