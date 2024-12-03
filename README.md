# OpenNCP + openEHR = 🔥

## Introduction

This [project](https://github.com/konateq/openncp.git) is a fork of the OpenNCP project with the goal of adding support
for openEHR.

This repository contains instructions for testing the integration of openEHR with OpenNCP.

## Prerequisites

- Java 11
- Maven
- Docker
- Docker Compose
- Bruno

## Getting Started

Clone the repository:

```shell
git clone -b openehr-support https://github.com/konateq/openncp.git
```

Move to the project directory:

```shell
cd openncp
```

Build the project:

```shell
mvn clean install -P national-connector-mock
```

Before running the Docker containers, you need to edit the URLs of the backend services used by the National Connector
Mock.
Open the `openncp-docker/docker-compose.yml` and add the following environment variables to the`tomcat_node_a` service:

```yaml
environment:
  FHIRSERVER_URL: "http://host.docker.internal:9999/fhir"  # URL of your FHIR backend
  OPENEHR_ENDPOINTS_QUERY_URL: "http://host.docker.internal:9999/rest/openehr/v1/query"  # URL of your openEHR backend (without /aql path segment)
```

Move to the `openncp-docker` directory:

```shell
  cd openncp-docker
```

Build and run the Docker containers:

```shell
docker-compose up --build
```

When the containers are running, access the MySQL database and insert the following records into the `EHNCP_PROPERTY` 
table:

```sql
INSERT INTO ehealth_properties.EHNCP_PROPERTY (NAME, VALUE, IS_SMP)
VALUES ('DE.openEHRQueryService.WSE', 'https://openncp-server:8443/openncp-ws-server/openehr/v1/query/aql', false),
       ('DE.FhirService.WSE', 'https://openncp-server:8443/fhir', false),
       ('<iso2_country_code>.openEHRQueryService.WSE', '<tomcat_node_a_base_url>/openncp-ws-server/openehr/v1/query/aql', false),
       ('<iso2_country_code>.FhirService.WSE', '<tomcat_node_a_base_url>/openncp-ws-server/fhir', false);
```

> [!TIP]
> Replace `<iso2_country_code>` with the ISO 3166-1 alpha-2 country code of the country and `<tomcat_node_a_base_url>`
> with the base URL of the `tomcat_node_a` service.

For local testing, start WireMock using the `compose.yml` file in this repository:

```shell
docker compose up
```

Start Bruno and import the collection located in the `bruno` directory. 



> [!IMPORTANT]
> The FHIR request is not working yet due to the missing SAML assertion.