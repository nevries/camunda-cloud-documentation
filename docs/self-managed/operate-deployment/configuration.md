---
id: configuration
title: Configuration
---

Operate is a Spring Boot application. This means every way to [configure](https://docs.spring.io/spring-boot/docs/current/reference/html/spring-boot-features.html#boot-features-external-config)
a Spring Boot application can be applied.

By default, the configuration for Operate is stored in a YAML file (`application.yml`). All Operate-related settings are prefixed with `camunda.operate`. The following parts are configurable:

* [Webserver](#webserver)
* [REST API access](#rest-api-access)
* [Elasticsearch connection](#elasticsearch)
* [Zeebe Broker connection](#zeebe-broker-connection)
* [Zeebe Elasticsearch Exporter](#zeebe-elasticsearch-exporter)
* [Operation Executor](#operation-executor)
* [Authentication](authentication.md)
* [Scaling Operate](importer-and-archiver.md)
* [Monitoring possibilities](#monitoring-operate)
* [Logging configuration](#logging)

## Configurations

### Webserver

Operate supports customizing the **context-path** using default Spring configuration.

Example for `application.yml`:
`server.servlet.context-path: /operate`

Example for environment variable:
`SERVER_SERVLET_CONTEXT_PATH=/operate`

The default context-path is `/`.

### REST API access

Operate provides a REST API under the endpoint `/v1`. Clients can access this API using a JWT access token in an authorization header `Authorization: Bearer <JWT>` 
or by login as a user and use the cookie in each following request.  

#### Access with cookie

The client needs to send the cookie with each request. The cookie can be obtained by using API endpoint `/api/login`.

__Example:__
1. Log in as user 'demo' and store the cookie in the file `cookie.txt`
```shell
curl -c cookie.txt -X POST 'http://localhost:8080/api/login?username=demo&password=demo'
```
2. Send the cookie (as a header) in each API request. In this case request all process definitions. 
```shell
curl -b cookie.txt -X POST 'http://localhost:8080/v1/process-definitions/list' -H 'Content-Type: application/json' -d '{}'
```

#### Access with JWT access token

Operate requires the following settings to validate the token:

| Setting                                                            | Description                                         |Example|
|--------------------------------------------------------------------|-----------------------------------------------------|--------|
| camunda.operate.client.audience                                    | Operate tries to match this with `aud` in JWT.      | operate.camunda.io|
| camunda.operate.client.clusterId                                   | Operate tries to match this with `scope` in JWT.    | cafe-0815-0235-a221-21cc6df91dc5|
| spring.security.oauth2.resourceserver.jwt.jwk-set-uri (recommended) | Complete URI to get public keys for JWT validation. | https://weblogin.cloud.company.com/.well-known/jwks.json|
| *OR*                                                               |                                                     |
| spring.security.oauth2.resourceserver.jwt.issuer-uri               | URI to get public keys for JWT validation.          | https://weblogin.cloud.company.com/|

__Example:__
1. Obtain a token for access the REST API 
You need `client_id`, `client_secret`, `audience`, and the url of the authorization server. For more information on how to get these for Camunda Cloud look
at [Manage API Clients](/docs/components/cloud-console/manage-clusters/manage-api-clients/).

```shell
curl -X POST -H 'content-type: application/json' -d '{"client_id": "RgVdPv...", "client_secret":"eDS1~Hg...","audience":"operate.camunda.io","grant_type":"client_credentials"}' https://login.cloud.camunda.com/oauth/token    
```

You will get something like that:
```json
{"access_token":"eyJhbG...","scope":"f408ca38-....","expires_in":58847,"token_type":"Bearer"}
```
Take the `access_token` value from the response object and store it as your token.

2. Send the token as an authorization header in each request. In this case request all process definitions.
```shell
curl -X POST 'http://localhost:8080/v1/process-definitions/list' -H 'Content-Type: application/json' -H 'Authorization: Bearer eyJhb...' -d '{}'
```

### Elasticsearch

Operate stores and reads data in/from Elasticsearch.

### Settings to connect

Operate supports [basic authentication](https://www.elastic.co/guide/en/elasticsearch/reference/7.12/setting-up-authentication.html) for Elasticsearch.

Set the appropriate username/password combination in the configuration to use it.

#### Settings to connect to a secured Elasticsearch instance
To connect to a secured (https) Elasticsearch instance you need normally only set the url protocol 
part to `https` instead of `http`. A secured Elasticsearch instance needs also `username` and `password`. 
The other ssl settings should only be used in case of connection problems, for example disable
host verification.

:::note
You may need to import the certificate into JVM runtime.
:::


Either set `host` and `port` (deprecated) or `url` (recommended).

Name | Description | Default value |
| -- | -- | -- |
| camunda.operate.elasticsearch.indexPrefix | Prefix for index names | operate |
| camunda.operate.elasticsearch.clusterName | Cluster name of Elasticsearch | elasticsearch |
| camunda.operate.elasticsearch.url | URL of Elasticsearch REST API | http://localhost:9200 |
| camunda.operate.elasticsearch.username | Username to access Elasticsearch REST API | - |
| camunda.operate.elasticsearch.password | Password to access Elasticsearch REST API | - |
| camunda.operate.elasticsearch.ssl.certificatePath | Path to certificate used by Elasticsearch | - |
| camunda.operate.elasticsearch.ssl.selfSigned | Certificate was self-signed | false |
| camunda.operate.elasticsearch.ssl.verifyHostname | Should the hostname be validated | false

### Settings for shards and replicas

Operate creates the template with index settings named `operate-<version>_template` that Elasticsearch uses for all Operate indices. These settings can be changed.

The following configuration parameters define the settings:

Name | Description | Default value |
| -- | -- | -- |
| camunda.operate.elasticsearch.numberOfShards| How many shards Elasticsearch uses for all Operate indices | 1 |
| camunda.operate.elasticsearch.numberOfReplicas| How many replicas Elasticsearch uses for all Operate indices | 0 |

These values are applied only on first startup of Operate or during version upgrade. After the Operate
schema is created, settings may be adjusted directly in the Elasticsearch template, and the new settings are applied
to indices created after adjustment.

### A snippet from application.yml

```yaml
camunda.operate:
  elasticsearch:
    # Cluster name
    clusterName: elasticsearch
    # Url
    url: https://localhost:9200
    ssl:
      selfSigned: true
```

## Zeebe broker connection

Operate needs a connection to the Zeebe Broker to start the import and execute user operations.

### Settings to connect

| Name                                 | Description | Default value|
|--------------------------------------|-------------|--------------|
| camunda.operate.zeebe.gatewayAddress | Gateway address that points to Zeebe as hostname and port. | localhost:26500|

:::note
Currently, Operate does not support TLS communication with Zeebe.
:::

### A snippet from application.yml

```yaml
camunda.operate:
  zeebe:
    # Gateway host and port
    gatewayAddress: localhost:26500
```

## Zeebe Elasticsearch exporter

Operate imports data from Elasticsearch indices created and filled in by the [Zeebe Elasticsearch Exporter](https://github.com/camunda-cloud/zeebe/tree/develop/exporters/elasticsearch-exporter).

Therefore, settings for this Elasticsearch connection must be defined and must correspond to the settings on the Zeebe side.

### Settings to connect and import
See also [Settings to connect to a secured Elasticsearch instance](#settings-to-connect-to-a-secured-elasticsearch-instance)

| Name                                                   | Description | Default value|
|--------------------------------------------------------|-------------|--------------|
| camunda.operate.zeebeElasticsearch.clusterName         | Cluster name of Elasticsearch | elasticsearch|
| camunda.operate.zeebeElasticsearch.url                 | URL of Zeebe Elasticsearch REST API | http://localhost:9200|
| camunda.operate.zeebeElasticsearch.prefix              | Index prefix as configured in Zeebe Elasticsearch exporter | zeebe-record|
| camunda.operate.zeebeElasticsearch.username            | Username to access Elasticsearch REST API | -|
| camunda.operate.zeebeElasticsearch.password            | Password to access Elasticsearch REST API | -|
| camunda.operate.zeebeElasticsearch.ssl.certificatePath | Path to certificate used by Elasticsearch | -|
| camunda.operate.zeebeElasticsearch.ssl.selfSigned      | Certificate was self signed | false|
| camunda.operate.zeebeElasticsearch.ssl.verifyHostname  | Should the hostname be validated | false|

### A snippet from application.yml:

```yaml
camunda.operate:
  zeebeElasticsearch:
    # Cluster name
    clusterName: elasticsearch
    # Url
    url: https://localhost:9200
    # Index prefix, configured in Zeebe Elasticsearch exporter
    prefix: zeebe-record
```

## Operation executor

Operations are user operations, like cancellation of process instance(s) or updating the variable value. 

Operations are executed in a multi-threaded manner.

| Name                                           | Description | Default value|
|------------------------------------------------|-------------|--------------|
| camunda.operate.operationExecutor.threadsCount | How many threads should be used. | 3|

### A snippet from application.yml

```yaml
camunda.operate:
  operationExecutor:
  	threadsCount: 3
```

## Monitoring Operate

Operate includes [Spring Boot Actuator](https://docs.spring.io/spring-boot/docs/current/reference/html/production-ready-features.html#production-ready) inside. This provides the number of monitoring possibilities.

Operate uses the following Actuator configuration by default:

```yaml
# Disable default health indicators
# https://docs.spring.io/spring-boot/docs/current/reference/html/production-ready-features.html#production-ready-health-indicators
management.health.defaults.enabled: false
# enable Kubernetes health groups:
# https://docs.spring.io/spring-boot/docs/current/reference/html/production-ready-features.html#production-ready-kubernetes-probes
management.health.probes.enabled: true
# enable health check, metrics and loggers endpoints
management.endpoints.web.exposure.include: health,prometheus,loggers
```

With this configuration, the following endpoints are available for use out of the box:

```<server>:8080/actuator/prometheus``` Prometheus metrics

```<server>:8080/actuator/health/liveness``` Liveness probe

```<server>:8080/actuator/health/readiness``` Readiness probe

### Versions before 0.25.0

In versions before 0.25.0, management endpoints look different. Therefore, we recommend reconfiguring for next versions.

|Name|Before 0.25.0| Starting with 0.25.0|
|----|-------------|--------|
|Readiness|/api/check|/actuator/health/readiness|
|Liveness|/actuator/health|/actuator/health/liveness|

## Logging

Operate uses the Log4j2 framework for logging. In the distribution archive, as well as inside a Docker image, `config/log4j2.xml` logging configuration files are included and can be further adjusted to your needs:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<Configuration status="WARN" monitorInterval="30">
  <Properties>
    <Property name="LOG_PATTERN">%clr{%d{yyyy-MM-dd HH:mm:ss.SSS}}{faint} %clr{%5p} %clr{${sys:PID}}{magenta} %clr{---}{faint} %clr{[%15.15t]}{faint} %clr{%-40.40c{1.}}{cyan} %clr{:}{faint} %m%n%xwEx</Property>
  </Properties>
  <Appenders>
    <Console name="Console" target="SYSTEM_OUT" follow="true">
      <PatternLayout pattern="${LOG_PATTERN}"/>
    </Console>
	<Console name="Stackdriver" target="SYSTEM_OUT" follow="true">
      <StackdriverJSONLayout/>
    </Console>
  </Appenders>
  <Loggers>
    <Logger name="io.camunda.operate" level="info" />
    <Root level="info">
      <AppenderRef ref="${env:OPERATE_LOG_APPENDER:-Console}"/>
    </Root>
  </Loggers>
</Configuration>
```

By default, `ConsoleAppender` is used.

#### JSON logging configuration

You can choose to output logs in JSON format (Stackdriver compatible). To enable it, define
the environment variable ```OPERATE_LOG_APPENDER``` like this:

```sh
OPERATE_LOG_APPENDER=Stackdriver
```

### Change logging level at runtime

Operate supports the default scheme for changing logging levels as provided by [Spring Boot](https://docs.spring.io/spring-boot/docs/2.4.3/actuator-api/htmlsingle/#loggers).

The log level for Operate can be changed by following the [Setting a Log Level](https://docs.spring.io/spring-boot/docs/2.4.3/actuator-api/htmlsingle/#loggers-setting-level) section.

#### Set all Operate loggers to DEBUG

```shell
curl 'http://localhost:8080/actuator/loggers/io.camunda.operate' -i -X POST \
-H 'Content-Type: application/json' \
-d '{"configuredLevel":"debug"}'
```

## An example of application.yml file

The following snippet represents the default Operate configuration, which is shipped with the distribution. This can be found inside the `config` folder (`config/application.yml`) and can be used to adjust Operate to your needs.

```yaml
# Operate configuration file

camunda.operate:
  # Set operate userId, displayName and password.
  # If user with <userId> does not exists it will be created.
  # Default: demo/demo/demo
  userId: anUserId
  displayName: nameShownInWebpage
  password: aPassword
  roles:
    - OWNER
    - USER
  # ELS instance to store Operate data
  elasticsearch:
    # Cluster name
    clusterName: elasticsearch
    # Url
    url: http://localhost:9200
  # Zeebe instance
  zeebe:
    # Gateway address to zeebe
    gatewayAddress: localhost:26500
  # ELS instance to export Zeebe data to
  zeebeElasticsearch:
    # Cluster name
    clusterName: elasticsearch
    # url
    url: http://localhost:9200
    # Index prefix, configured in Zeebe Elasticsearch exporter
    prefix: zeebe-record
```
