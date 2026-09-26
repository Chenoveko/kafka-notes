FROM apache/kafka:3.9.1

USER root

RUN mkdir -p /opt/jmx-exporter

ADD https://github.com/prometheus/jmx_exporter/releases/download/1.6.0/jmx_prometheus_javaagent-1.6.0.jar \
    /opt/jmx-exporter/jmx_prometheus_javaagent.jar

COPY exporter.yaml /opt/jmx-exporter/exporter.yaml

USER appuser