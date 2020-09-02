你好！
很冒昧用这样的方式来和你沟通，如有打扰请忽略我的提交哈。我是光年实验室（gnlab.com）的HR，在招Golang开发工程师，我们是一个技术型团队，技术氛围非常好。全职和兼职都可以，不过最好是全职，工作地点杭州。
我们公司是做流量增长的，Golang负责开发SAAS平台的应用，我们做的很多应用是全新的，工作非常有挑战也很有意思，是国内很多大厂的顾问。
如果有兴趣的话加我微信：13515810775  ，也可以访问 https://gnlab.com/，联系客服转发给HR。
### localstack-lambda-sqs-in-k8s
Example application to play around with SQS, S3 bucket and AWS Lambda in Go.
 Deployed in [Kind](https://kind.sigs.k8s.io/docs/user/quick-start/) k8s cluster. 

Uses [LocalStack](https://github.com/localstack/localstack) for emulating AWS cloud stack, 
[awslocal](https://github.com/localstack/awscli-local) as a wrapper for aws cli
 and [aws-sdk-go](github.com/aws/aws-sdk-go) for calling AWS from services in Go.

#### Build:
Sync submodule after cloning repository:
```
git submodule update --init -r

```
Build:
```
make
```

#### Create Kind cluster and configure Ingress:
```
./setup.sh init
```
Each Lambda function will run in a separate Docker container and docker socket needs to be mounted into the 
localstack container, so config for Kind has it mounted too

#### Install
```
./setup.sh install
```
It will run 2 services: `publisher` and `analyzer` and expose them locally (by Ingress). Publisher can send messages to SQS 
that invokes AWS Lambda (using event source mapping). Lambda function notifies `analyzer` service by adding file to S3 bucket, 
sending SQS message and direct call.
Analyzer keeps tracks of how many times it was invoked and by what localstack service, so that it can be checked later. <br>
Localstack is provisioned in init container for `analyzer` pod. It contains script for creating SQS, S3,
Lambda and event source mapping.

#### Call services:
- Check currently running pods and wait until analyzer is up:
```
watch kubectl get all -n test-localstack
```

- Open http://localhost/publish/msg in browser.<br>
This request calls `publisher` service that will send message to sqs (`worker_sqs` queue). 
Sqs message invokes Lambda and Lambda calls `analyzer` service by 3 ways:
  * call analyzer directly
  * send message to `result_sqs` queue. Analyzer polls this queue
  * put file with result to s3 bucket. It triggers sending message to `s3_sqs` queue using event source mapping.
  . Analyzer polls `s3_sqs` queue too.


- Get statistics, returns how many times analyzer was called: http://localhost/publish/stats <br>
It can take few seconds to finish work and show updated statistics. Each Lambda call should update it.
Analyzer keeps track separately for direct calls and received messages from `result_sqs` and 
`s3_sqs` queues.

Call analyzer manually (will update direct call statistic):
http://localhost/analyze/msg

Get statistics directly from analyzer:
http://localhost/analyze/stats
 
 #### Clean up:
 
 `./setup.sh delete` - remove deployments 
 
 `./setup.sh clean` - delete kind cluster




