


## Setting up S3 source bucket

```
aws --profile kbasedev cloudformation validate-template --template-body file://s3_source.yml
aws --profile kbasedev cloudformation create-stack --stack-name kendra-s3-source \
    --template-body file://s3_source.yml \
    --parameters ParameterKey=MyBucketName,ParameterValue=${KENDRA_SOURCE_S3_BUCKET} # get from .env file

aws --profile kbasedev cloudformation create-change-set --stack-name kendra-s3-source \
    --change-set-name kendra-s3-source-cs-1 \
    --template-body file://s3_source.yml \
    --parameters ParameterKey=MyBucketName,ParameterValue=${KENDRA_SOURCE_S3_BUCKET} # get from .env file
aws cloudformation describe-change-set --change-set-name kendra-s3-source-cs-1 --stack-name kendra-s3-source
aws --profile kbasedev cloudformation update-stack --stack-name kendra-s3-source \
    --template-body file://s3_source.yml \
    --parameters ParameterKey=MyBucketName,ParameterValue=${KENDRA_SOURCE_S3_BUCKET} # get from .env file
# aws --profile kbasedev cloudformation delete-stack --stack-name kendra-s3-source
```

## Creating roles for Kendra Index

```
aws --profile kbasedev cloudformation validate-template --template-body file://roles_indexing.yml
aws --profile kbasedev cloudformation create-stack --stack-name kendra-indexing-roles \
    --template-body file://roles_indexing.yml \
    --capabilities "CAPABILITY_IAM" "CAPABILITY_NAMED_IAM" \
    --parameters ParameterKey=S3DataSource,ParameterValue=${KENDRA_SOURCE_S3_BUCKET} # get from .env file
# aws --profile kbasedev cloudformation delete-stack --stack-name kendra-indexing-roles
```

## Creating Kendra Index
