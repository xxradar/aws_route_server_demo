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
#### Attach the policy to a Role a
```
aws iam attach-role-policy \
  --role-name MyRoleName \
  --policy-arn arn:aws:iam::974654858447:policy/RouteServerPolicy
```


