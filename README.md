## AWS Lambda
* Refernece
https://docs.spring.io/spring-cloud-function/reference/adapters/aws-intro.html

### Configure Information
Handler: `org.springframework.cloud.function.adapter.aws.FunctionInvoker::handleRequest`
```json
{
   "name": "banu",
   "employeeIdentifier":"1",
   "email": "banu@yahoo.com",
   "salary":"1000"
}
```
* Using lambda cli test
```bash
$ aws lambda invoke  \
  --invocation-type RequestResponse  \
  --function-name employeeConsumer-spring-cloud-function \
  --cli-binary-format raw-in-base64-out \
  --payload  '{ "name": "banu", "employeeIdentifier":"1", "email": "banu@yahoo.com","salary":"1000"}' \
    outputfile.txt
{
    "StatusCode": 200,
    "ExecutedVersion": "$LATEST"
}
```
And then see the content of file `outputfile.txt`  is `ok` .

## Deployment
In a shell, navigate to the sample's folder and use the SAM CLI to build a deployable package

```bash
$ sam deploy --s3-bucket $CF_BUCKET --stack-name employee-consumer --capabilities CAPABILITY_IAM
```

# TEARING DOWN RESOURCES
When you run `sam deploy`, it creates or updates a CloudFormation `stack`—a set of resources that has a name, which you’ve seen already with the `--stack-name` parameter of `sam deploy`.

When you want to clean up your AWS account after trying an example, the simplest method is to find the corresponding CloudFormation stack in the AWS Web Console (in the CloudFormation section) and delete the stack using the **Delete** button.

Alternatively, you can tear down the stack from the command line. For example, to tear down the **alt-pet-store** stack, run the following:
```bash
$ PIPELINE_BUCKET="$(aws cloudformation describe-stack-resource --stack-name employee-consumer --logical-resource-id PipelineStartBucket --query 'StackResourceDetail.PhysicalResourceId' --output text)" 
$ aws s3 rm s3://${PIPELINE_BUCKET} --recursive
$ aws cloudformation delete-stack --stack-name employee-consumer
```

# SAM LOCAL
## Ready
```bash
vagrant up
vagrant ssh
sudo apt install -y wget unzip


curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
unzip awscliv2.zip
sudo ./aws/install
aws --version

wget https://github.com/aws/aws-sam-cli/releases/latest/download/aws-sam-cli-linux-x86_64.zip
unzip aws-sam-cli-linux-x86_64.zip -d sam-installation
sudo ./sam-installation/install
sam --version
```
## Test in sam Local
```bash
sam local api-test
```
another terminal
```bash
vagrant@sam:~$ curl --location 'http://127.0.0.1:3000/employeeConsumer' \
--header 'Content-Type: application/json' \
--data '{
   "name": "banu",
   "employeeIdentifier":"1",
   "email": "banu@yahoo.com",
   "salary":"1000"
}'
```

In Postman, choose POST, the http://localhost:3000/employeeConsumer URL, and the JSON format input.
```json
{
   "name": "banu",
   "employeeIdentifier":"1",
   "email": "banu@yahoo.com",
   "salary":"1000"
}
```
You'll find the result not good. 
* event: 
  https://stackoverflow.com/questions/73325829/spring-cloud-function-with-aws-api-gateway-and-aws-lambda
* sulutions ?  
  * [`org.springframework.cloud.function.adapter.aws.SpringBootApiGatewayRequestHandler`](./https://docs.spring.io/spring-cloud-function/docs/current/reference/html/aws.html#_http_and_api_gateway)
     * Maybee replace `Handler: org.springframework.cloud.function.adapter.aws.FunctionInvoker::handleRequest` in `template.yml` by `Handler: org.springframework.cloud.function.adapter.aws.SpringBootApiGatewayRequestHandler::handleRequest`
       * But `SpringBootApiGatewayRequestHandler` does not exist !!
  * You maybe need another solution. https://github.com/aws/serverless-java-container/wiki/Quick-start---Spring-Boot3
