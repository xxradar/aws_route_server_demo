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
**Note:** State=pending
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
**Note:** After a few seconds State=available

