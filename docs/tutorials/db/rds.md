# Relational Database Service (RDS)
Monitoring relational databases on the cloud is very similar to monitoring databases on-premise with New Relic Infrastructure agent and on-host integration.  Consider the following architecture diagram.


```mermaid
flowchart LR

    subgraph nr1[New Relic]
    US
    EU
    end

    subgraph id2[New Relic Infrastructure Agent]
    nri-mssql
    nri-oracledb
    nri-mongodb
    nri-databricks
    nri-db*
    end

    subgraph id1[Cloud Provider]
    id99[(rds-databases)]
    end

    id1-->id2-->nr1
 ```


## Architecture
Because RDS instances are managed by the cloud provider, you won't be able to install any agents on the cloud hosting it.  However, you can have a dedicated "integration" server with the New Relic Infrastructure agent installed to monitor all of your RDS instances.

```mermaid
flowchart LR

    subgraph newrelic-infra
      direction TB
      subgraph nri-mssql
          direction BT
          mssql-custom-query.yml-->mssql-config.yml
      end
      subgraph nri-xxx
          direction BT
          xxx-custom-query.yml-->xxx-config.yml
      end
      subgraph nri-oracledb
          direction BT
          oracledb-custom-query.yml-->oracledb-config.yml
      end
    end

    subgraph AWS
    id1[(oracle-prod.ca-central-1.rds.amazonaws.com)]
    id2[(oracle-dev.ca-central-1.rds.amazonaws.com)]
    id3[(mssql-qa.ca-central-1.rds.amazonaws.com)]
    end

    subgraph Azure
    id4[(mssql-prod.database.windows.net)]
    id5[(mssql-dev.database.windows.net)]
    id6[(oracledb-qa.database.windows.net)]
    end

    id1-->nri-oracledb
    id2-->nri-oracledb
    id3-->nri-mssql
    id4-->nri-mssql
    id5-->nri-mssql
    id6-->nri-oracledb

 ```

## Setup
To set this up you'll need the following:
1. A machine you can install agents, e.g., AWS EC2 instance, Azure Virtual machine, or some server in a data center.
2. Install the New Relic Infrastructure Agent
3. Install the relavant on-host integration packages, e.g., `nri-mssql`, `nri-oracledb`, etc.
4. Update the `config.yml` to include every RDS instance that needs to be monitored
5. The only difference between on-premise database monitoring and RDS monitoring is the `HOSTNAME`.  For RDS, your `HOSTNAME` is the RDS endpoint.

For example, to monitor a Microsoft SQL Server RDS instance on AWS, follow the same instructions for [setting up monitoring MSSQL](/tutorials/db/mssql.md) on-host.  In the `mssql-config.yml` replace the `HOSTNAME` with the RDS endpoint from AWS and any credentials to proceed.

## Concerns
One bottleneck with this method of RDS monitoring is the network throughput.  Many datacenters may limit your network ingress/egress.  For example, if you have a 100 Mbps limit, that is about 12.5 MB of telemetry data you can send to New Relic per second (or ~1TB per day).  Most users won't hit this networking limit though.