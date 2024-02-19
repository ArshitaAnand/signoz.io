---
id: collecting-ecs-sidecar-infra
title: Collecting Data from ECS using Sidecar
description: View metrics and logs for your ECS infrastructure
hide_table_of_contents: true
---

import Tabs from '@theme/Tabs';
import TabItem from '@theme/TabItem';
import SCTemplateIntro from '../shared/ecs-infra/sidecar/template-intro.md'
import SCConfigIntro from '../shared/ecs-infra/sidecar/config-intro.md'
import SCConfigCloud from '../shared/ecs-infra/sidecar/config-cloud.md'
import SCConfigOss from '../shared/ecs-infra/sidecar/config-oss.md'

## Overview

This documentation will show you how to collect data from your ECS infrastructure
using sidecar. The sidecar method involves deploying an additional container (the "sidecar") 
that runs in each application container of your ECS cluster. A sidecar container, is an auxiliary container 
that offers supporting features like logging and monitoring to the main application container.
The sidecar container will collect metrics and forward any received OTLP data to SigNoz.

<br></br>

Select the type of SigNoz instance you are running: **SigNoz Cloud** or **Self-Hosted**.

<Tabs>
<TabItem value="cloud" label="SigNoz Cloud" default>

### Prerequisites

- An ECS cluster running with at least one task definition
- ECS cluster can be either of the launch types: **Fargate**, **EC2** or **External**
- [SigNoz Cloud account](https://signoz.io/teams/)

<br></br>

## Setting up Sidecar Container

<br></br>

### Step 1: Create SigNoz OtelCollector Config

<figure data-zoomable align='center'>
    <img src="/img/docs/ecs-docs/ecs-otelcol-sidecar-config-ssm.webp" alt="otelcol-sidecar parameter in AWS Parameter Store"/>
    <figcaption><i>OTel Collector config in AWS Parameter Store</i></figcaption>
</figure>

<br></br>

- Navigate to AWS Parameter Store and create a new parameter named `/ecs/signoz/otelcol-sidecar.yaml`.


- Download the [otelcol-sidecar YAML](https://github.com/SigNoz/benchmark/blob/main/ecs/otelcol-sidecar.yaml) configuration file:

    ```bash
    wget https://github.com/SigNoz/benchmark/raw/main/ecs/otelcol-sidecar.yaml
    ```

- Update `{region}` and `SIGNOZ_INGESTION_KEY` values in your YAML configuration file and copy the updated content of the `otelcol-sidecar.yaml` file and paste it in the value field of the `/ecs/signoz/otelcol-sidecar.yaml` parameter that you created.

You will be able to get `{region}` and `SIGNOZ_INGESTION_KEY` values in your [SigNoz Cloud account](https://signoz.io/teams/) under **Settings --> Ingestion Settings**.

<figure data-zoomable align='center'>
    <img src="/img/docs/common/ingestion-key-details.webp" alt="Ingestion key details"/>
    <figcaption><i>Ingestion details in SigNoz dashboard</i></figcaption>
</figure>

<br></br>

:::info

After successful set up, feel free to remove `logging` exporter if it gets too noisy. To do so, simply remove the `logging` exporter from the `exporters` list in the following pipelines: `traces`, `metrics`, `metrics/aws`, and `logs` from the `otelcol-sidecar.yaml` file.

:::

<br></br>

### Step 2: Create Sidecar Collector Container

This step involves integrating the SigNoz collector into your ECS task definitions as a sidecar container. 
The sidecar collector container will run alongside your application container(s) within the same ECS task 
and will collect ECS container metrics and send them to SigNoz Cloud.
It also acts as a gateway to send any telemetry data from your application container to SigNoz Cloud.

</TabItem>
<TabItem value="self-host" label="Self-Host">


### Prerequisites

- An ECS cluster running with at least one task definition
- ECS cluster can be either of the launch types: **Fargate**, **EC2** or **External**
- Running SigNoz Self-Host instance - [Install SigNoz](/docs/install)

<br></br>

## Setting up Sidecar Container

<br></br>

### Step 1: Create SigNoz OtelCollector Config

<figure data-zoomable align='center'>
    <img src="/img/docs/ecs-docs/ecs-otelcol-sidecar-config-ssm.webp" alt="otelcol-sidecar parameter in AWS Parameter Store"/>
    <figcaption><i>OTel Collector config in AWS Parameter Store</i></figcaption>
</figure>

<br></br>

- Navigate to AWS Parameter Store and create a new parameter named `/ecs/signoz/otelcol-sidecar.yaml`.

- Download the [otelcol-sidecar-oss YAML](https://github.com/SigNoz/benchmark/blob/main/ecs/otelcol-sidecar-oss.yaml) configuration file:

    ```bash
    wget https://github.com/SigNoz/benchmark/raw/main/ecs/otelcol-sidecar-oss.yaml
    ```

- Update `<IP of machine hosting SigNoz>` with the proper values in the YAML configuration file and copy the updated content of the `otelcol-sidecar-oss.yaml` file and paste it in the value field of the `/ecs/signoz/otelcol-sidecar.yaml`` parameter that you created.

** Note: ** Replace `<IP of machine hosting SigNoz>` with the address of [SigNoz OTelCollector](https://signoz.io/docs/install/troubleshooting/#signoz-otel-collector-address-grid).

:::info

After successful set up, feel free to remove `logging` exporter if it gets too noisy. To do so, simply remove the `logging` exporter from the `exporters` list in the following pipelines: `traces`, `metrics`, `metrics/aws`, and `logs` from the `otelcol-sidecar.yaml` file.

:::

<br></br>

### Step 2: Create Sidecar Collector Container

This step involves integrating the SigNoz collector into your ECS task definitions as a sidecar container. 
The sidecar collector container will run alongside your application container(s) within the same ECS task 
and will collect ECS container metrics and send them to SigNoz Self-Host.
It also acts as a gateway to send any telemetry data from your application container to SigNoz Self-Host.

</TabItem>
</Tabs>

### Update task definition of your application

In your ECS [task definition](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/task_definitions.html), include a new container definition specifically for the sidecar container. This container will operate alongside your main application container(s) within the same task definition. The JSON configuration for that will look like this:

```json
{
    ...
    "containerDefinitions": [
        ...,
        {
            "name": "signoz-collector",
            "image": "signoz/signoz-otel-collector:0.88.7",
            "user": "root",
            "command": [
                "--config=env:SIGNOZ_CONFIG_CONTENT"
            ],
            "secrets": [
                {
                "name": "SIGNOZ_CONFIG_CONTENT",
                "valueFrom": "/ecs/signoz/otelcol-sidecar.yaml"
                }
            ],
            "memory": 1024,
            "cpu": 512,
            "essential": true,
            "portMappings": [
                {
                    "protocol": "tcp",
                    "containerPort": 4317
                },
                {
                    "protocol": "tcp",
                    "containerPort": 4318
                }
            ],
            "healthCheck": {
                "command": [
                    "CMD-SHELL",
                    "wget -qO- http://localhost:13133/ || exit 1"
                ],
                "interval": 5,
                "timeout": 6,
                "retries": 5,
                "startPeriod": 1
            },
            "logConfiguration": {
                "logDriver": "awslogs",
                "options": {
                "awslogs-group": "/ecs/signoz-otel-EC2-sidcar",
                "awslogs-region": "<aws-region>",
                "awslogs-stream-prefix": "ecs",
                "awslogs-create-group": "True"
                }
            }
        }
    ]
...
}
```

The sidecar container will run the SigNoz collector and includes essential components like the **name** ("signoz-collector"), **image** (e.g., "signoz/signoz-otel-collector:0.88.7"), and **command** for execution, alongside **secrets** for secure configuration. It also specifies **memory** and **CPU** resources, **port mappings** for network communication, a **health check** to ensure operational integrity, and **log configuration** for output management. These elements collectively enable the container to efficiently collect and forward telemetry data to SigNoz.

### Update ECS Task Execution Role

The ECS Task Execution Role is an AWS IAM role that grants permissions to the ECS agent to make AWS API calls on behalf of the user. When integrating SigNoz as a sidecar container, the task execution role needs additional permissions to access the AWS Systems Manager Parameter Store, where the SigNoz configuration is stored.

To modify the ECS Task Execution Role, follow these steps:


1. **Identify the Role:** Identify the IAM role used by your [ECS tasks for execution](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/task_execution_IAM_role.html). This role is specified when creating an ECS service or can be found in the ECS task definition. It's often named something like **ecsTaskExecutionRole**.

2. **Edit the Role:** Navigate to the IAM console in the AWS Management Console, find the role by name, and open its details page.

3. **Attach Policies or Add inline Policy:**

    There are two ways to grant access to the Parameter store: 

    - **Attach AWS Managed Policies:** If the role doesn't already have the `AmazonSSMReadOnlyAccess` policy attached, you can attach this managed policy to grant read-only access to the Systems Manager Parameter Store. This access is necessary for the task to retrieve the SigNoz configuration.

    - **Add Inline Policy:** Alternatively, for more granular control, you can create an inline policy that specifically grants access to only the necessary resources in the Parameter Store. The JSON for the inline policy will be:

    ```json
    {
        "Version": "2012-10-17",
        "Statement": [
            {
                "Action": [
                    "ssm:GetParameter"
                ],
                "Resource": [
                    "arn:aws:ssm:<aws-region>:<aws-account-id>:parameter/ecs/signoz/otelcol-sidecar.yaml"
                ],
                "Effect": "Allow"
            }
        ]
    }
    ```

### Update ECS Task Role

The ECS Task Role is distinct from the ECS Task Execution Role. While the execution role grants the ECS agent permission to manage the lifecycle of containers (like pulling images and storing logs), the task role is assumed by the task itself, including your application and any sidecar containers. This role provides the permissions necessary for the application to make AWS API calls directly.

To update the ECS Task Role, follow these steps:

1. **Identify the Role:** Determine the IAM role your ECS tasks are currently using to interact with AWS services. This role is specified in the ECS task definition under the "taskRoleArn" field.

2. **Edit the Role**: Go to the IAM section of the AWS Management Console, locate the role by its name, and open its configuration.

3. **Attach Policies or Add Inline Policy:**

    There are two ways to grant access to the Parameter store: 

    - **Attach AWS Managed Policies:** If the role doesn't already have the `AmazonSSMReadOnlyAccess` policy attached, you can attach this managed policy to grant read-only access to the Systems Manager Parameter Store. 

    - **Add Inline Policy for Granular Access:** For tighter security, you might opt to create an inline policy that specifies exactly which resources the tasks can access and what actions they can perform on those resources. This is particularly important for accessing specific resources like the Parameter Store parameters used by the SigNoz sidecar. The JSON for the inline policy will be:

    ```json
    {
        "Version": "2012-10-17",
        "Statement": [
            {
                "Action": [
                    "ssm:GetParameter"
                ],
                "Resource": [
                    "arn:aws:ssm:<aws-region>:<aws-account-id>:parameter/ecs/signoz/otelcol-sidecar.yaml"
                ],
                "Effect": "Allow"
            }
        ]
    }
    ```

### Deploy the task definition

If your application runs as an ECS service, you update the service to use the new revision of your task definition. This tells ECS to start new tasks based on this updated definition and gracefully replace the old tasks with the new ones, ensuring minimal disruption to your application.

If you're not using a service and instead run tasks directly, you manually start a new task using the updated task definition. This approach is less common for long-running applications but might be used for testing or one-off executions.

**NOTE:** Once the task is running, you should be able to see SigNoz sidecar container logs in CloudWatch Logs because we have set the `logDriver` parameter to be `awslogs` in our task definition.

### Verify data in SigNoz

To verify that your sidecar container is running, go to the Dashboard
section of SigNoz and import the dashboard `ECS - Container Metrics` Dashboard from
[here](https://github.com/SigNoz/dashboards/raw/main/ecs-infra-metrics/container-metrics.json).

---

## Send Data from Applications

In this section, we will see how to send traces data from applications deployed in ECS
to SigNoz using sidecar container that we deployed in the previous section.

### Add OpenTelemetry Instrumentation to your Application

Instrumentation involves modifying your application code to generate telemetry data. This could mean adding specific OpenTelemetry API calls or relying on auto-instrumentation provided by the SDK for common libraries and frameworks.

To add OpenTelemetry instrumentation to your application, follow the docs [here](https://signoz.io/docs/instrumentation/).

**NOTE:** This step can also include adding the OpenTelemetry SDK as well as the initialization code to your application codebase and rebuilding the application container.

### Configure OTLP Endpoint

Once your application is instrumented to generate telemetry data, you need to configure it to send this data to the SigNoz collector sidecar container. For this, you need to set the OTLP endpoint to the endpoint of the sidecar container. This can be done by setting theenvironment variable `OTEL_EXPORTER_OTLP_ENDPOINT` to the endpoint of
the sidecar container.

Select the network mode of your ECS task definition:

<Tabs>
<TabItem value="bridge" label="Bridge" default>

```json
{
    ...
    "containerDefinitions": [
        {
            "name": "<your-container-name>",
            "environment": [
                {
                    "name": "OTEL_EXPORTER_OTLP_ENDPOINT",
                    "value": "http://signoz-collector:4317"
                },
                {
                    "name": "OTEL_RESOURCE_ATTRIBUTES",
                    "value": "service.name=<your-service-name>"
                }
            ],
            "links": [
                "signoz-collector"
            ],
            ...
        }
    ]
}
```

</TabItem>
<TabItem value="awsvpc" label="AWS VPC">

```json
{
    ...
    "containerDefinitions": [
        {
            "name": "<your-container-name>",
            "environment": [
                {
                    "name": "OTEL_EXPORTER_OTLP_ENDPOINT",
                    "value": "http://localhost:4317"
                },
                {
                    "name": "OTEL_RESOURCE_ATTRIBUTES",
                    "value": "service.name=<your-service-name>"
                }
            ],
            ...
        }
    ]
}
```

</TabItem>
</Tabs>

### Rebuild and Deploy Application Container

After instrumenting your application and configuring the OTLP endpoint, you'll need to rebuild your application container with these changes and deploy it to ECS cluster using the same task definition that we used in the previous section.

### Verify data in SigNoz

To verify that the data is being sent to SigNoz, you will need to
generate some traffic to your application. Now go to the
**Services** section of SigNoz and you should be able to see your application in the services list.

---
