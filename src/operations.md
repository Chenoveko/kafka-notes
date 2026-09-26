# Operations
[Kafka Operations - Conduktor](https://www.conduktor.io/glossary/kafka-admin-operations-and-maintenance)

## Jolokia 
Jolokia is a tool that lets applications expose Java JMX information through HTTP and JSON.
In simple terms, it acts as a bridge between a Java application and monitoring tools. It allows you to check metrics, memory usage, threads, and other application information using a web request instead of a direct JMX connection.
It can be used with Java applications such as Apache Kafka.

We can of course use any HTTP client to access Jolokia agent. One of the most popular and powerful tools is the curl command.

We can access the agent with simple GET requests:
```bash
$ curl -s -u \
jolokia:jolokia http://localhost:8080/jolokia/read/java.lang:type=Memory/HeapMemoryUsage | jq .
{
  "request": {
    "mbean": "java.lang:type=Memory",
    "attribute": "HeapMemoryUsage",
    "type": "read"
  },
  "value": {
    "init": 524288000,
    "committed": 532676608,
    "max": 8334082048,
    "used": 53842272
  },
  "status": 200,
  "timestamp": 1701866205
}
```
but it’s also easy to send POST requests for example:
```bash
$ curl -s -u jolokia:jolokia \
-H'Content-Type: application/json' \
-XPOST -d '{"mbean":"java.lang:type=Memory","attribute":"HeapMemoryUsage","type":"read"}' \
http://localhost:8080/jolokia | jq .
{
  "request": {
    "mbean": "java.lang:type=Memory",
    "attribute": "HeapMemoryUsage",
    "type": "read"
  },
  "value": {
    "init": 524288000,
    "committed": 532676608,
    "max": 8334082048,
    "used": 53842272
  },
  "status": 200,
  "timestamp": 1701866282
}
```

![Jolokia](https://jolokia.org/images/jolokia_browser_version.png)