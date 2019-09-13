# RabbitMQ integration with LDAP

The main goal of this guide is to demonstrate, step by step, how to set up RabbitMQ to authenticate and authorize via the LDAP plugin. It starts with a very simple scenario, [Only Authentication](only-authentication/README.md), which just configures RabbitMQ to authenticate users via LDAP.

Every scenario helps the user launch an OpenLDAP server, import required LDAP entries to work with the scenario and configure RabbitMQ accordingly. It also helps the user verify the configuration.  

The guide continues further configuring RabbitMQ with LDAP to secure vhost access, secure resource (i.e. *exchanges* and *queues*) access and management plugin access too.

The last scenario, [Authentication and Authorization (tags, vhosts, resources)](auth-and-authz/README.md), is the most complete one and it is one possible LDAP+RabbitMQ scenario out of the many we may encounter in real-world.

The aim of this repository is to address more scenarios in the future.

## Prerequisites to follow this guide

This guide assumes RabbitMQ is running locally (on port 5672 and 15672). It also provides an script to deploy **OpenLDAP** locally via Docker. Additionally, we need the following prerequisites:
- `ldap` tools are installed such as `ldapsearch` and/or `ldapadd`.
- Ruby is installed. We will use it to run some AMQP clients.
- Python is installed. We will use it to run [rabbitmqadmin](https://www.rabbitmq.com/management-cli.html)
- `rabbitadmin` is installed.  Go to [http://localhost:15672/cli/rabbitmqadmin](http://localhost:15672/cli/rabbitmqadmin]), copy the downloaded file to your preferred location in your `PATH`

If you are currently running [RabbitMQ for PCF](https://docs.pivotal.io/rabbitmq-cf/1-17/index.html), this [guide](rabbitmq-4-pcf.md) walks you through the steps on how to set it up with LDAP authentication so that you can try any of the scenarios we will cover the next sections. It also shows you how to deploy **OpenLDAP** in a separate VM and make it accessible from your RabbitMQ for PCF. 

## Implemented Integration scenarios

- [Only Authentication](only-authentication/README.md)
- [Authentication and User tags](authentication-and-tags/README.md)
- [Authentication, User tags and Vhosts](auth-tags-vhost/README.md)
- [Authentication and Authorization (tags, vhosts, resources)](auth-and-authz/README.md)
- [Users are organized in a hierarchical fashion (e.g. under different *Organizatinal Units*)](hierarchical-user-organization/README.md)
- [Use multiple Authentication backends](multiple-auth-backends/README.md)
- [Cache Authentication and Authorization backend results](cache-auth-results/README.md)
- [Retrieve RabbitMQ's client identify from the client's certificate](tls-identity/README.md)

## Future Integration scenarios

- [Many RabbitMQ Clusters](many-rabbitmq-clusters/README.md)

# Best Practices | Recommendations

In addition to all the recommendations done in the [RabbitMQ LDAP documentation](https://www.rabbitmq.com/ldap.html), it is worth keeping an eye on these other ones.

## Use rabbitmq-auth-backend-cache

With external authz backends like the LDAP one we highly recommend using https://github.com/rabbitmq/rabbitmq-auth-backend-cache in production because under load RabbitMQ is known to hammer LDAP servers hard enough with queries that they can't keep up.

Check out the scenario [Cache Authentication and Authorization backend results](cache-auth-results/README.md).

## Properly configure LDAP timeouts

Make sure the connection timeouts in your LDAP server are larger than your configured timeout (`auth_ldap.timeout`) otherwise your LDAP server may terminate the connection and the ldap plugin may fail to operate afterwards.

## Cache LDAP connection to avoid excessive connection churn

LDAP server connections are pooled to avoid excessive connection churn and LDAP server load. By default the pool has up to 64 connections. This can be controlled using the `auth_ldap.connection_pool_size `. Pooled connections without activity are closed after a period of time configurable via `auth_ldap.idle_timeout` which by default it is set to `300000` msec.

## Monitor log file to detect when RabbitMQ lost connection with LDAP server

TODO : Add more sample log statements and the minimum configuration to enable it

```
[warning] <0.1777.0> HTTP access denied: rabbit_auth_backend_ldap failed authenticating bob: ldap_connect_error
```
