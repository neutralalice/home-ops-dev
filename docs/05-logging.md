## Collector

Currently using vlagent for everything in cluster which is then forwarded to the
log management of victoria-logs using nativeinserts

## Aggregator

I'm currently with fluent-bit to collect node-level logs, do some minor
transformations, and then pass that off to victoria-logs as json_line

## Log management

Currently Victoria-logs. No real complaints. I'd like to look in to creating
some grafana dashboards for stat analysis, particularly on access-logs.

- syslog at udp 541 - will probably switch this to tcp
- other jsonlogs at 9428

# Strategies

In general, I am trying to set all applications I run to natively emit jsonline.
This is preferable as it reduces the amount of transforms I have to think about.

Also looking at instrumenting certain things with otel
