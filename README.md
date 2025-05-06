### Configure required IAM Role permissions 
```
aws iam create-policy \
  --policy-name RouteServerPolicy \
  --policy-document file://route-server-policy.json
```
```
aws iam attach-role-policy \
  --role-name MyRoleName \
  --policy-arn arn:aws:iam::974654858447:policy/RouteServerPolicy
```


