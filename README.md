# IaC Confluent Cloud Resources Terraform Configuration

**Table of Contents**

<!-- toc -->
+ [Purpose](#purpose)
    + [AWS Secrets Manager]()
        - [`/confluent_cloud_resource/schema_registry_cluster/java_client`](#confluent_cloud_resourceschema_registry_clusterjava_client)
        - [`/confluent_cloud_resource/kafka_cluster/java_client`](#confluent_cloud_resourcekafka_clusterjava_client)
    + [AWS Systems Manager Parameter Store]()
        - [`/confluent_cloud_resource/consumer_kafka_client`](#confluent_cloud_resourceconsumer_kafka_client)
        - [`/confluent_cloud_resource/producer_kafka_client`](#confluent_cloud_resourceproducer_kafka_client)
+ [How to use this repo?](#how-to-use-this-repo)
    + [GitHub Setup](#github-setup)
        - [Terraform Cloud API token](#terraform-cloud-api-token)
        - [Confluent Cloud API](#confluent-cloud-api)
<!-- tocstop -->

## Purpose
The configuration leverages the [IaC Confluent Cloud Resource API Key Rotation Terraform module](https://github.com/j3-signalroom/iac-confluent_cloud_resource_api_key_rotation-tf_module) to handle the creation and rotation of each of the Confluent Cloud Resource API Key for each of the Confluent Cloud Resources:
- [Schema Registry Cluster](https://registry.terraform.io/providers/confluentinc/confluent/latest/docs/resources/confluent_schema_registry_cluster)
- [Kafka Cluster](https://registry.terraform.io/providers/confluentinc/confluent/latest/docs/resources/confluent_kafka_cluster)

Along with the Schema Registry Cluster REST endpoint, and Kafka Cluster's Bootstrap URI are stored in the [AWS Secrets Manager](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/secretsmanager_secret).

In addition, the [consumer](https://docs.confluent.io/platform/current/installation/configuration/consumer-configs.html) and [producer]() Kafka client configuration parameters are stored in the [AWS Systems Manager Parameter Store](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/ssm_parameter).

### AWS Secrets Manager
Here is the list of secrets generated by the Terraform configuration:

#### `/confluent_cloud_resource/schema_registry_cluster/java_client`
> Key|Description
> -|-
> `api_key`|The ID of the API Key.
> `api_secret`|The secret of the API Key.
> `schema.registry.url`|The HTTP endpoint of the Schema Registry cluster.

#### `/confluent_cloud_resource/kafka_cluster/java_client`
> Key|Description
> -|-
> `sasl.jaas.config`|Java Authentication and Authorization Service (JAAS) for SASL configuration.
> `bootstrap.servers`|The bootstrap endpoint used by Kafka clients to connect to the Kafka cluster.

### AWS Systems Manager Parameter Store
Here is the list of parameters generated by the Terraform configuration:

#### `/confluent_cloud_resource/consumer_kafka_client`
> Key|Description
> -|-
> `auto.commit.interval.ms`|The `auto.commit.interval.ms` property in Apache Kafka defines the frequency (in milliseconds) at which the Kafka consumer automatically commits offsets. This is relevant when `enable.auto.commit` is set to true, which allows Kafka to automatically commit the offsets periodically without requiring the application to do so explicitly.
> `auto.offset.reset`|Specifies the behavior of the consumer when there is no committed position (which occurs when the group is first initialized) or when an offset is out of range. You can choose either to reset the position to the `earliest` offset or the `latest` offset (the default).
> `basic.auth.credentials.source`|This property specifies the source of the credentials for basic authentication.
> `client.dns.lookup`|This property specifies how the client should resolve the DNS name of the Kafka brokers.
> `enable.auto.commit`|When set to true, the Kafka consumer automatically commits the offsets of messages it has processed at regular intervals, specified by the `auto.commit.interval.ms` property. If set to false, the application is responsible for committing offsets manually.
> `max.poll.interval.ms`|This property defines the maximum amount of time (in milliseconds) that can pass between consecutive calls to poll() on a consumer. If this interval is exceeded, the consumer will be considered dead, and its partitions will be reassigned to other consumers in the group.
> `request.timeout.ms`|This property sets the maximum amount of time the client will wait for a response from the Kafka broker. If the server does not respond within this time, the client will consider the request as failed and handle it accordingly.
> `sasl.mechanism`|This property specifies the SASL mechanism to be used for authentication.
> `security.protocol`|This property specifies the protocol used to communicate with Kafka brokers.
> `session.timeout.ms`|This property sets the timeout for detecting consumer failures when using Kafka's group management. If the consumer does not send a heartbeat to the broker within this period, it will be considered dead, and its partitions will be reassigned to other consumers in the group.

#### `/confluent_cloud_resource/producer_kafka_client`
> Key|Description
> -|-
> `sasl.mechanism`|This property specifies the SASL mechanism to be used for authentication.
> `security.protocol`|This property specifies the protocol used to communicate with Kafka brokers.
> `client.dns.lookup`|This property specifies how the client should resolve the DNS name of the Kafka brokers.
> `acks`|This property specifies the number of acknowledgments the producer requires the leader to have received before considering a request complete.

## How to use this repo?
In the [main.tf](main.tf) replace **`<TERRAFORM CLOUD ORGANIZATION NAME>`** in the `terraform.cloud` block with your [Terraform Cloud Organization Name](https://developer.hashicorp.com/terraform/cloud-docs/users-teams-organizations/organizations) and **`<TERRAFORM CLOUD ORGANIZATION's WORKSPACE NAME>`** in the `terraform.cloud.workspaces` block with your [Terraform Cloud Organization's Workspaces Name](https://developer.hashicorp.com/terraform/cloud-docs/workspaces).

### GitHub Setup
In order to run the Terraform configuration, the Terraform Cloud API token and Confluent Cloud API Key are required as GitHub Secret variables:

#### Terraform Cloud API token
From the [Tokens page](https://app.terraform.io/app/settings/tokens), create/update the API token and store it in the [AWS Secrets Manager](https://us-east-1.console.aws.amazon.com/secretsmanager/secret?name=%2Fsi-iac-confluent_cloud_kafka_api_key_rotation-tf%2Fconfluent&region=us-east-1).  Then add/update the `TF_API_TOKEN` secret on the [GitHub Action secrets and variables, secret tab](https://github.com/signalroom/si-iac-confluent_cloud_kafka_api_key_rotation-tf/settings/secrets/actions).

#### Confluent Cloud API
Confluent Cloud requires API keys to manage access and authentication to different parts of the service.  An API key consists of a key and a secret.  You can create and manage API keys by using the [Confluent Cloud CLI](https://docs.confluent.io/confluent-cli/current/overview.html).  Learn more about Confluent Cloud API Key access [here](https://docs.confluent.io/cloud/current/access-management/authenticate/api-keys/api-keys.html#ccloud-api-keys).

Using the Confluent CLI, execute the follow command to generate the Cloud API Key:
```
confluent api-key create --resource "cloud" 
```
Then, for instance, copy-and-paste the API Key and API Secret values to the respective, `CONFLUENT_CLOUD_API_KEY` and `CONFLUENT_CLOUD_API_SECRET` secrets, that need to be created/updated on the [J3 repository Actions secrets and variables page](https://github.com/j3-signalroom/j3-iac-confluent_cloud_resources_api_keys-tf/settings/secrets/actions).
