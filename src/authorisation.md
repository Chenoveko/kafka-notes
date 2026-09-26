# Authorisation ACL
ACLs como in tow flavours
- Topics: restrcit client can read/write data
- Consumers Groups: which client can use a specific consumer group
- Cluster: which client can create/delete topcis or apply settings

A super user in Kafka that can do everything without any special kind of ACL

There are no concept of User Groups so far in kafka. Each ACL have to be written for each client/user

## Properties on Brokers

```properties
# Enable ACL
authorizer.call.name=kafka.security.auth.SimpleAclAuthorizer

# Set super user
super.users=User:admin,User:kafka

# Deny acces for a user which is not part of the ACL
allow.everyone.if.no.acl.found=false

# SSL comunication between brokers using kerberos
security.inter.broker.protocol=SASL_SSL

# Authenticate between brokers
inter.broker.protocol=SASL_SSL
```

## `kafka-acl.sh`

```bash

```