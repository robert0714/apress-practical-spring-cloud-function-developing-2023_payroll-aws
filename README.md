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
# Reference
* https://github.com/aws/serverless-java-container/wiki/Quick-start---Spring-Boot3
  * Orign source: 
    * https://github.com/aws/serverless-java-container/tree/main/samples/springboot3/pet-store-native
  * related articles 
    * https://aws.amazon.com/cn/blogs/china/re-platforming-java-applications-using-the-updated-aws-serverless-java-container/
    * https://catalog.workshops.aws/java-on-aws-lambda/en-US/02-accelerate/graal-plain-java
    * https://stackoverflow.com/questions/69906369/sam-cli-and-quarkus-var-task-bootstrap-no-such-file-or-directory
    * https://stackoverflow.com/questions/64749387/micronaut-graalvm-native-image-lambda-fails-with-an-error-error-fork-exec-va
* Serverless Java with Spring
  * Video: [Serverless Java with Spring by Maximilian Schellhorn & Dennis Kieselhorst @ Spring I/O 2024](https://youtu.be/AFIHug_HujI)
  * Slide: https://speakerdeck.com/deki/serverless-java-with-spring
    * Method 1: Handling via functions
      * [Tradition](https://docs.aws.amazon.com/zh_tw/lambda/latest/dg/java-handler.html#java-best-practices)
      * Handling via Spring Cloud Functions ( Spring Cloud AWS is designed for non-lambda , other aws services)
      * Multiple functions
    * Method 1: HTTP adapter
      * AWS Serverless Java: [serverless-java-container](https://github.com/aws/serverless-java-container)
    * Summary    
      |Method|Handling via functions|HTTP adapter|
      |------|--------------------------------|----------------------|
      |Approach|Directly deserialize JSON payloads into Java objects without involving HTTP request/response abstractions.|An adapter transforms events into HTTP request/response objects (e.g., Jakarta Servlet API).|
      |Usage|Process content using event-specific code.|Retains compatibility with existing HTTP-based frameworks.|
      |Strengths|1. Well-suited for non-HTTP use cases.<br/>2. Standardized for serverless or event-driven architectures.|Enables reuse of existing code and frameworks without modification|
      |Weaknesses|More challenging to adapt existing applications and frameworks that rely on HTTP constructs.|Introduces performance overhead due to the additional transformation layer.|
* [2022 Reduce Java cold starts by 10x with AWS Lambda](https://youtu.be/Y5b8_KToeDY?t=1163)
* Other implements
  * Spring Cloud AWS 3
    * https://github.com/awspring/spring-cloud-aws
    * Video
      * Spring I/O 2024: [Spring Cloud AWS 3 upgrade and customisation for over 100 teams at Ocado Technology by M. Telepchuk](https://youtu.be/-PgFoRGaa6s)
    * baeldung
       * [github](https://github.com/eugenp/tutorials/tree/master/spring-cloud-modules/spring-cloud-aws-v3) 
  * Spring Cloud Function
    * LocalStack: https://docs.localstack.cloud/user-guide/integrations/spring-cloud-function/ 
    * Recommended books
      * [Practical Spring Cloud Function: Developing Cloud-Native Functions for Multi-Cloud and Hybrid-Cloud Environments](https://link.springer.com/book/10.1007/978-1-4842-8913-6) ![cover](https://learning.oreilly.com/library/cover/9781484289136/250w/)
