# Cloudformation templates to spin up kendra

Execute the following in order and make sure to use the correct `--profile`.

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

```
aws --profile kbasedev cloudformation validate-template --template-body file://index.yml
aws --profile kbasedev cloudformation create-stack --stack-name kendra-index \
    --template-body file://index.yml \
    --parameters ParameterKey=KendraIndexRoleStack,ParameterValue=kendra-indexing-roles
# aws --profile kbasedev cloudformation delete-stack --stack-name kendra-index
```

## Creating S3 source roles

```
aws --profile kbasedev cloudformation validate-template --template-body file://roles_data_source.yml
aws --profile kbasedev cloudformation create-stack --stack-name kendra-data-source-roles \
    --template-body file://roles_data_source.yml \
    --capabilities "CAPABILITY_IAM" "CAPABILITY_NAMED_IAM" \
    --parameters ParameterKey=KendraIndexStack,ParameterValue=kendra-index ParameterKey=S3DataSource,ParameterValue=${KENDRA_SOURCE_S3_BUCKET} # get from .env file
# aws --profile kbasedev cloudformation delete-stack --stack-name kendra-data-source-roles
```

## Creating Kendra Data Source

```
aws --profile kbasedev cloudformation validate-template --template-body file://data_source.yml
aws --profile kbasedev cloudformation create-stack --stack-name kendra-s3-data-source \
    --template-body file://data_source.yml \
    --parameters ParameterKey=KendraIndexStack,ParameterValue=kendra-index ParameterKey=KendraDataSourceRoleStack,ParameterValue=kendra-data-source-roles ParameterKey=S3DataSource,ParameterValue=${KENDRA_SOURCE_S3_BUCKET} # get from .env file
# aws --profile kbasedev cloudformation delete-stack --stack-name kendra-s3-data-source
```

## Syncing Data Source

```
index_id=$(aws --profile kbasedev cloudformation list-exports --output text --query "Exports[?Name=='kendra-index-KendraIndexId'].Value") && echo ${index_id}
data_source_id=$(aws --profile kbasedev cloudformation list-exports --output text --query "Exports[?Name=='kendra-s3-data-source-KendraS3DataSourceId'].Value") && echo ${data_source_id}

aws kendra start-data-source-sync-job --index-id ${index_id} --id ${data_source_id}
```
