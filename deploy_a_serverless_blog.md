- Install serverless

```
npm install -g serverless

```

- create user with policies and access key

```
aws iam create-user \
    --user-name serverless-user
```

```
aws iam attach-user-policy \
    --policy-arn arn:aws:iam::aws:policy/AdministratorAccess \
    --user-name serverless-user
```

```
aws iam create-access-key \
    --user-name serverless-user
```

- configure serverless

```
serverless config credentials \
    --provider aws \
    --key xxxxxxxxxxxxxx \
    --secret xxxxxxxxxxxxxx \
    --profile serverless-user
```

```
cat ~/.aws/credentials
```

- create project and set enviroment's node version

```
serverless create --template aws-nodejs --path serverless-blog
cd serverless-blog
echo v6.10 > .nvmrc
git init && git commit -am 'initial commit'
```

- deploy

```
serverless deploy -v --aws-profile serverless-user
```

- invoke

```
serverless invoke -f hello -l \
    --aws-profile serverless-user
```

- remove

```
serverless remove \
    --aws-profile serverless-user
```

- invoke from aws

```
aws lambda invoke \
--invocation-type RequestResponse \
--function-name <function-name> \
--log-type Tail \
outputfile.txt
```