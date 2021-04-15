


## Setting up S3 source bucket

```
aws --profile kbasedev cloudformation validate-template --template-body file://s3_source.yml
# aws --profile kbasedev cloudformation delete-stack --stack-name kendra-s3-source
aws --profile kbasedev cloudformation create-stack --stack-name kendra-s3-source \
    --template-body file://s3_source.yml \
    --parameters ParameterKey=MyBucketName,ParameterValue=${KENDRA_SOURCE_S3_BUCKET} # get from .env file
```