# Camunda Platform 8

* this repo
  * links to Camunda Platform 8 resources / official release artifacts (binaries)
  * supporting config files -- for running Docker Compose -- as a local development option
* Recommendations
  * ⚠️ **Docker Compose is ONLY recommended for local development** ⚠️
  * [SaaS](https://camunda.com/get-started/) or
  * [Helm/Kubernetes](https://docs.camunda.io/docs/self-managed/setup/overview/)
    * recommended for production
    * [Helm charts](https://docs.camunda.io/docs/self-managed/setup/install/)
    * [helm.camunda.io](https://helm.camunda.io/) 
  * [Self-Managed](https://docs.camunda.io/docs/self-managed/setup/overview/)

## Links to additional Camunda Platform 8 repos and assets

- [Documentation](https://docs.camunda.io)
- [Camunda Platform SaaS](https://camunda.io)
- [Getting Started Guide](https://github.com/camunda/camunda-platform-get-started)
- [Releases](https://github.com/camunda/camunda-platform/releases)
- [Helm Charts](https://helm.camunda.io/)
- [Zeebe Workflow Engine](https://github.com/camunda/zeebe)
- [Contact](https://docs.camunda.io/contact/)

## Via Docker Compose

* requirements
  * Docker 20.10.16+
    * Reason: 🧠 [docker compose version 1.27.0+ specifications are being used | "dockerCompose.yaml"](https://docs.docker.com/compose/compose-file/) 🧠
### complete Camunda
* [docker-compose.yaml](docker-compose.yaml)
  * == complete Camunda Platform 8 Self-Managed environment
    * Zeebe
    * Operate
    * Tasklist
    * Connectors
    * Optimize
    * Identity
    * Elasticsearch
    * Keycloak
    * PostgreSQL
    * Web Modeler
      * ❌NOT included by default ❌
      * [Follow the instructions below](#web-modeler-self-managed) to install it
  * `docker compose up -d`
  * Check
    * Keycloak
      * 's container logs -- to ensure -- components have started
      * allows
        * managing users
          * [http://localhost:18080/auth/](http://localhost:18080/auth/) / login with 
            * `demo` - `demo`
            * `admin` - `admin`
              * Problem: it does NOT work
                * Solution: TODO:
    * navigate to the different web apps & log in with the user `demo` and password `demo`
      * Operate: [http://localhost:8081](http://localhost:8081)
        * it's network != Identity's network
        * Problem: "NOT up"
          * Solution: TODO:
      * Tasklist: [http://localhost:8082](http://localhost:8082)
        * it's network != Identity's network
        * Problem: "NOT up"
          * Solution: TODO:
      * Optimize: [http://localhost:8083](http://localhost:8083)
        * Problem: "NOT up"
          * Solution: TODO:
      * Identity: [http://localhost:8084](http://localhost:8084)
        * Problem: "NOT up"
          * Solution: TODO:
      * Elasticsearch: [http://localhost:9200](http://localhost:9200)
    * Zeebe
      * -- available -- gRPC | `localhost:26500`
      * it's network != Identity's network
  * `docker compose down -v`
    * down the WHOLE environment

### basic components

* == NOT needed
  * Optimize
  * Identity
  * Keycloak -> use [docker-compose-core.yaml](docker-compose-core.yaml)
* `docker compose -f docker-compose-core.yaml up -d`
* Open in your browser "localhost:8081" / pass `demo` - `demo` as credentials

### Deploying BPMN diagrams

* TODO:
In addition to the local environment setup with docker compose, use the [Camunda Desktop Modeler](#desktop-modeler) to locally model BPMN diagrams for execution and directly deploy them to your local environment.
As an enterprise customer, you can [use Web Modeler](#web-modeler-self-managed).

Feedback and updates are welcome!

## Securing the Zeebe API

By default, the Zeebe GRPC API is publicly accessible without requiring any client credentials for development purposes.

You can however enable authentication of GRPC requests in Zeebe by setting the environment variable `ZEEBE_AUTHENTICATION_MODE` to `identity`, e.g. via running:
```
ZEEBE_AUTHENTICATION_MODE=identity docker compose up -d
```
or by modifying the default value in the [`.env`](.env) file.

## Connectors

Both docker-compose files contain our [out-of-the-box Connectors](https://docs.camunda.io/docs/components/integration-framework/connectors/out-of-the-box-connectors/available-connectors-overview/).

Refer to the [Connector installation guide](https://docs.camunda.io/docs/self-managed/connectors-deployment/install-and-start/) for details on how to provide the related Connector templates for modeling.

To inject secrets into the Connector runtime they can be added to the
[`connector-secrets.txt`](connector-secrets.txt) file inside the repository in the format `NAME=VALUE`
per line. The secrets will then be available in the Connector runtime with the
format `secrets.NAME`.

To add custom Connectors either create a new docker image bundling them as
described [here](https://github.com/camunda/connectors-bundle/tree/main/runtime).

Alternatively, you can mount new Connector JARs as volumes into the `/opt/app` folder by adding this to the docker-compose file. Keep in mind that the Connector JARs need to bring along all necessary dependencies inside the JAR.

## Kibana

A `kibana` profile is available in the provided docker compose files to support inspection and exploration of the Camunda Platform 8 data in Elasticsearch.
It can be enabled by adding `--profile kibana` to your docker compose command.
In addition to the other components, this profile spins up [Kibana](https://www.elastic.co/kibana/).
Kibana can be used to explore the records exported by Zeebe into Elasticsearch, or to discover the data in Elasticsearch used by the other components (e.g. Operate).

You can navigate to the Kibana web app and start exploring the data without login credentials:

- Kibana: [http://localhost:5601](http://localhost:5601)

> **Note**
> You need to configure the index patterns in Kibana before you can explore the data.
> - Go to `Management > Stack Management > Kibana > Index Patterns`.
> - Create a new index pattern. For example, `zeebe-record-*` matches the exported records.
>   - If you don't see any indexes then make sure to export some data first (e.g. deploy a process). The indexes of the records are created when the first record of this type is exported.
> - Go to `Analytics > Discover` and select the index pattern.

## Desktop Modeler

> :information_source: The Desktop Modeler is [open source, free to use](https://github.com/camunda/camunda-modeler).

[Download the Desktop Modeler](https://camunda.com/download/modeler/) and start modeling BPMN, DMN and Camunda Forms on your local machine.

### Deploy or execute a process

#### Without authentication
Once you are ready to deploy or execute processes use these settings to deploy to the local Zeebe instance:
* Authentication: `None`
* URL: `http://localhost:26500`

#### With Zeebe request authentication
If you enabled authentication for GRPC requests on Zeebe you need to provide client credentials when deploying and executing processes:
* Authentication: `OAuth`
* URL: `http://localhost:26500`
* Client ID: `zeebe`
* Client secret: `zecret`
* OAuth URL: `http://localhost:18080/auth/realms/camunda-platform/protocol/openid-connect/token`
* Audience: `zeebe-api`

## Web Modeler Self-Managed

> :information_source: Web Modeler Self-Managed is available to Camunda enterprise customers only.

The Docker images for Web Modeler are available in a private registry. Enterprise customers either already have credentials to this registry, or they can request access to this registry through their CSM contact at Camunda.

To run Camunda Platform with Web Modeler Self-Managed clone this repo and issue the following commands:

```
$ docker login registry.camunda.cloud
Username: your_username
Password: ******
Login Succeeded
$ docker compose -f docker-compose.yaml -f docker-compose-web-modeler.yaml up -d
```

To tear down the whole environment run the following command

```
$ docker compose -f docker-compose.yaml -f docker-compose-web-modeler.yaml down -v
```

If you want to delete everything (including any data you created).
Alternatively, if you want to keep the data run:

```
$ docker compose -f docker-compose.yaml -f docker-compose-web-modeler.yaml down
```

### Login
You can access Web Modeler Self-Managed and log in with the user `demo` and password `demo` at [http://localhost:8070](http://localhost:8070).

### Deploy or execute a process

#### Without authentication
Once you are ready to deploy or execute processes use these settings to deploy to the local Zeebe instance:
* Authentication: `None`
* URL: `http://zeebe:26500`

#### With Zeebe request authentication
If you enabled authentication for GRPC requests on Zeebe you need to provide client credentials when deploying and executing processes:
* Authentication: `OAuth`
* URL: `http://zeebe:26500`
* Client ID: `zeebe`
* Client secret: `zecret`
* OAuth URL: `http://keycloak:8080/auth/realms/camunda-platform/protocol/openid-connect/token`
* Audience: `zeebe-api`

### Emails
The setup includes [Mailpit](https://github.com/axllent/mailpit) as a test SMTP server. It captures all emails sent by Web Modeler, but does not forward them to the actual recipients. 

You can access emails in Mailpit's Web UI at [http://localhost:8075](http://localhost:8075).

## Troubleshooting

### Submitting Issues
When submitting an issue on this repository, please make sure your issue is related to the docker compose deployment
method of the Camunda Platform. All questions regarding to functionality of the web applications should be instead
posted on the [Camunda Forum](https://forum.camunda.io/). This is the best way for users to query for existing answers
that others have already encountered. We also have a category on that forum specifically for [Deployment Related Topics](https://forum.camunda.io/c/camunda-platform-8-topics/deploying-camunda-platform-8/33).

### Running on arm64 based hardware
When using arm64-based hardware like a M1 or M2 Mac the Keycloak container might not start because Bitnami only
provides amd64-based images for versions <  22. You can build and tag an arm-based
image locally using the following command. After building and tagging the image you can start the environment as
described in [Using docker-compose](#using-docker-compose).

```
$ DOCKER_BUILDKIT=0 docker build -t bitnami/keycloak:19.0.3 "https://github.com/camunda/camunda-platform.git#8.2.15:.keycloak/"
```

## Resource based authorizations

You can control access to specific processes and decision tables in Operate and Tasklist with [resource-based authorization](https://docs.camunda.io/docs/self-managed/concepts/access-control/resource-authorizations/).

This feature is disabled by default and can be enabled by setting 
`RESOURCE_AUTHORIZATIONS_ENABLED` to `true`, either via the [`.env`](.env) file or through the command line:

```
RESOURCE_AUTHORIZATIONS_ENABLED=true docker compose up -d
```

## Multi-Tenancy

You can use [multi-tenancy](https://docs.camunda.io/docs/self-managed/concepts/multi-tenancy/) to achieve tenant-based isolation.

This feature is disabled by default and can be enabled by setting
`MULTI_TENANCY_ENABLED` to `true`, either via the [`.env`](.env) file or through the command line:

```
ZEEBE_AUTHENICATION_MODE=identity MULTI_TENANCY_ENABLED=true docker compose up -d
```

As seen above the feature also requires you to use `identity` as an authentication provider.

Ensure you [setup tenants in identity](https://docs.camunda.io/docs/self-managed/identity/user-guide/tenants/managing-tenants/) after you start the platform.

## Camunda Platform 7

Looking for information on Camunda Platform 7? Check out the links below:

- [Documentation](https://docs.camunda.org/)
- [GitHub](https://github.com/camunda/camunda-bpm-platform)
