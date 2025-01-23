# localstack-persist

[LocalStack](https://github.com/localstack/localstack) Community Edition with support for persisted resources.

[![Docker pulls](https://img.shields.io/docker/pulls/gresau/localstack-persist?logo=docker)](https://hub.docker.com/r/gresau/localstack-persist)
[![CI Build](https://github.com/GREsau/localstack-persist/actions/workflows/test.yml/badge.svg)](https://github.com/GREsau/localstack-persist/actions/workflows/test.yml)

## Overview

As of LocalStack 1.0, [persistence](https://docs.localstack.cloud/references/persistence-mechanism/) is a pro-only feature, so is unavailable when using Community Edition. localstack-persist adds out-of-the-box persistence, which is saved whenever a resource is modified, and automatically restored on container startup.

## Usage

localstack-persist is distributed as a docker image, made to be a drop-in replacement for the official [LocalStack Community Edition docker image](https://hub.docker.com/r/localstack/localstack). For example, to use it with docker-compose, you could use a `docker-compose.yml` file like:

```yaml
version: "3.8"
services:
  localstack:
    image: gresau/localstack-persist:4 # instead of localstack/localstack:4
    ports:
      - "4566:4566"
    volumes:
      - "./my-localstack-data:/persisted-data"
```

This will use the latest available image with semver-major version 4 - if you prefer, you can pin to a specific version e.g. `4.0.2`. For other available tags, see the list on [Docker Hub](https://hub.docker.com/r/gresau/localstack-persist/tags) or the [GitHub releases](https://github.com/GREsau/localstack-persist/releases). The Major.Minor version of a localstack-persist image's tag will track the version of LocalStack that the image is based on - e.g. `gresau/localstack-persist:2.2.X` will always be based on `localstack/localstack:2.2.Y` (where X and Y may be different numbers). You can also use the `latest` image, which is built daily from the `main` branch, and based on `localstack/localstack:latest` (the nightly LocalStack image), but please be aware that image may not be stable.

Persisted data is saved inside the container at `/persisted-data`, so you'll typically want to mount a volume at that path - the example compose file above will keep persisted data in the `my-localstack-data` on the host.

## Configuration

By default, all services will persist their resources to disk. To disable persistence for a particular service, set the container's `PERSIST_[SERVICE]` environment variable to 0 (e.g. `PERSIST_CLOUDWATCH=0`). Or to enable persistence for only specific services, set `PERSIST_DEFAULT=0` and `PERSIST_[SERVICE]=1`. For example, to enable persistence for only DynamoDB and S3, you could use the `docker-compose.yml` file:

```yaml
    ...
    image: gresau/localstack-persist
    ports:
      - "4566:4566"
    volumes:
      - "./my-localstack-data:/persisted-data"
    environment:
      - PERSIST_DEFAULT=0
      - PERSIST_DYNAMODB=1
      - PERSIST_S3=1
```

You can still set any of [LocalStack's configuration options](https://docs.localstack.cloud/references/configuration/) in the usual way - however, you do NOT need to set `PERSISTENCE=1`, as that just controls LocalStack's built-in persistence which does not function in Community Edition.

## Supported Services

localstack-persist uses largely the same hooks as the official persistence mechanism, so all (non-pro) services supported by official persistence should work with localstack-persist - [see the list here](https://docs.localstack.cloud/references/persistence-mechanism/#supported--tested).

The following services have basic save/restore functionality verified by automated tests:

- ACM
- DynamoDB
- Elasticsearch
- IAM
- Lambda
- SQS
- S3

## License

localstack-persist is released under the [Apache License 2.0](LICENSE). LocalStack is used under the [Apache License 2.0](https://github.com/localstack/localstack/blob/master/LICENSE.txt).

# S3 Bucket Setup

## AWS Commands

## Start

```sh
docker-compose up -d
```

## Stop

```sh
docker-compose stop
```

## logs

```sh
docker-compose logs -f
```

## Get localstack bash shell

```sh
docker exec -it localstack_s3 bash
```

## To set configuration

```sh
aws configure

# This will ask you to set following things:
# - AWS Access Key ID [****]:
# - AWS Secret Access Key [****]:
# - Default region name [us-west-1]: us-east-1
# - Default output format [None]:
```

## S3 Bucket

### Create s3 bucket

```sh
aws --endpoint-url=http://localhost:4566 s3 mb s3://storage-name
# Note: 'storage-name' is bucket-name
```

### Attach an ACL to the bucket so it is readable

```sh
aws --endpoint-url=http://localhost:4566 s3api put-bucket-acl --bucket storage-name --acl public-read
```

### List s3 bucket

```sh
aws s3 ls --endpoint-url http://localhost:4566

aws --endpoint-url=http://localhost:4566 s3 ls s3://storage-name

# OR using awslocal
awslocal s3 ls

awslocal s3 ls storage-name
```

### Attach an ACL to the bucket so it is private

```sh
aws --endpoint-url=http://localhost:4566 s3api put-bucket-acl --bucket private-bucket-name --acl private
```

### Delete an S3 bucket

```sh
aws --endpoint-url=http://localhost:4566 s3 rb s3://storage-name --force
# Note: 'storage-name' is the name of your bucket
```

### Remove directory/file from s3 bucket

```sh
# To remove dir:
aws --endpoint-url=http://localhost:4566 s3 rm s3://storage-name/53779c28-3dd7-4e7d-bc2c-95aab8c1a67e/temp/ --recursive 

# To remove file:
aws --endpoint-url=http://localhost:4566 s3 rm s3://storage-name/53779c28-3dd7-4e7d-bc2c-95aab8c1a67e/temp.txt
```

## AWS queue

### Create queue in localstack

```sh
aws --endpoint-url=http://localhost:4576 sqs create-queue --queue-name <queue-name>

# example
aws --endpoint-url=http://localhost:4576 sqs create-queue --queue-name notifyqueue
```

### List queue

```sh
aws --endpoint-url=http://localhost:4576 sqs list-queues
```

### Create SNS topic

```sh
aws --endpoint-url=http://localhost:4575 sns create-topic --name my-topic-name

# example
aws --endpoint-url=http://localhost:4575 sns create-topic --name notifyTopic
#  output: "TopicArn": "arn:aws:sns:us-east-1:000000000000:notifyTopic"
```

### Subscribing to a Topic

```sh
aws --endpoint-url=http://localhost:4575 sns subscribe --topic-arn arn:aws:sns:us-east-1:000000000000:notifyTopic --protocol sqs --notification-endpoint http://localhost:4576/queue/notifyqueue

# output: "SubscriptionArn": "arn:aws:sns:us-east-1:000000000000:notifyTopic:ccb720ca-18cb-4e71-bd9e-bd385c226d32"
```

### Get messages

```sh
aws --endpoint-url=http://localhost:4576 sqs receive-message --queue-url http://localhost:4576/queue/notifyqueue
```

## Generate pre-signed url from terminal

```sh
aws s3 presign s3://storage-name/presentation.ppt --endpoint-url https://s3.wasabisys.com
```

## ***s3 - setup for minio and localstack:***

-----------------------------------

```dotenv
AWS_S3_ACCESS_KEY_ID=minio
AWS_S3_SECRET_ACCESS_KEY=minio123
AWS_S3_DEFAULT_REGION=us-east-1
AWS_S3_MAZE_BUCKET=maze
AWS_S3_URL=http://localhost:9001/maze
AWS_S3_ENDPOINT=http://localhost:9001
```

```dotenv
AWS_S3_ACCESS_KEY_ID=foo
AWS_S3_SECRET_ACCESS_KEY=bar
AWS_S3_DEFAULT_REGION=us-east-1
AWS_S3_MAZE_BUCKET=storage-name
AWS_S3_URL=http://localhost:4566/storage-name
AWS_S3_ENDPOINT=http://localhost:4566
```

## Docker build command

```sh
# Build command
docker-compose -f docker-compose.schedule.yml build

# Up command
docker-compose -f docker-compose.schedule.yml up

# Get bash shell command
docker exec -it mazetec-back_backend-schedule_1 bash

# Stop command
docker-compose -f docker-compose.schedule.yml stop
```
