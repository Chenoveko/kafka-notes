# Monitoring with Grafana + Prometheus
[Kafka Monitoring - Redpanda](https://www.redpanda.com/guides/kafka-performance-kafka-monitoring)
[Kafka Monitoring - Conduktor](https://www.conduktor.io/glossary/kafka-cluster-monitoring-and-metrics)

Kafka is independent from your monitoring stack. By default, kafka exposes its metrics using **JMX** (Java Management Extensions)and therefore most monitoring back-end should have a way to integrate with JMX directly or using and ETC process

Self monitoring vailable solutions:
- **Prometheus + Grafana**
- **ElasticSearch + Kibana (ELK Stack)** 
- **Confluent Control Centre** (paid monitoring tool by Confluent)
- **LinkedIn Cruise Control** (no UI, automated monitoring)

Managed monitoring solutions:
- **Datadog**
- **New Relic**
- **Splunk**
- **CloudWatch Monitoring**

## Kafka monitoring with Prometheus

## Kafka monitoring with Prometheus
Grafana Dashboards:
- [Kafka Metrics](https://grafana.com/grafana/dashboards/11962-kafka-metrics/)
- [Kafka Exporter Overview](https://grafana.com/grafana/dashboards/7589-kafka-exporter-overview/)
- [Kafka Overview](https://grafana.com/grafana/dashboards/13684-kafka-overview/)
- [Confluent Open Source Grafana Dashboard](https://github.com/provectus/provectus-cp-helm-charts/blob/master/grafana-dashboard/confluent-open-source-grafana-dashboard.json)
