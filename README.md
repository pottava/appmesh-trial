# AWS App Mesh トライアル

## AWS で楽をするサービスメッシュ入門

[スライド](https://speakerdeck.com/pottava/appmesh-trial)

## 構成

- [構成図](https://github.com/pottava/appmesh-trial/blob/master/architecture.png)

## 1. プロビジョニング

```sh
$ aws configure
$ aws cloudformation validate-template --template-body "file://cfn.yaml"
{
    "Parameters": [
        {
            "ParameterKey": "ServicesDomain",
            "DefaultValue": "default.svc.cluster.local",
            "NoEcho": false,
            "Description": "DNS namespace used by services"
        }
    ],
    "Description": "Fargate minimum architecture",
    "Capabilities": [
        "CAPABILITY_IAM"
    ],
    "CapabilitiesReason": "The following resource(s) require capabilities: [AWS::IAM::Role]"
}
$ stack_name=appmesh-trial
$ aws cloudformation deploy \
    --stack-name "${stack_name}" --template-file cfn.yaml \
    --parameter-overrides ServicesDomain="default.svc.cluster.local" \
    --capabilities CAPABILITY_IAM

Waiting for changeset to be created..
Waiting for stack create/update to complete
Successfully created/updated stack - appmesh-trial
```

## 2. 動作確認

```sh
$ public_endpoint=$( aws cloudformation describe-stacks --stack-name "${stack_name}" \
    --query 'Stacks[0].Outputs[?OutputKey==`PublicEndpoint`].OutputValue' \
    --output text )
$ curl -sXGET "${public_endpoint}" | jq . > direct.json
$ curl --basic -u 'fargate:Pa$$w0rd' -sXGET "${public_endpoint}:8080" | jq . > proxied.json
$ open "${public_endpoint}"
# Open AWS Console
$ open "https://ap-northeast-1.console.aws.amazon.com/appmesh/meshes/${stack_name}-mesh"
$ open "https://ap-northeast-1.console.aws.amazon.com/cloudmap/home"
$ open "https://ap-northeast-1.console.aws.amazon.com/ecs/home"
$ open "https://ap-northeast-1.console.aws.amazon.com/xray/home"
```

## 3. お片づけ

```sh
$ aws cloudformation delete-stack --stack-name "${stack_name}"
$ aws cloudformation wait stack-delete-complete --stack-name "${stack_name}"
$ echo $?
0
```
