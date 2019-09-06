# Proposal: Support External syslog endpoint

Author: `<Qian Deng / ninjadq>`

## Abstract

This proposal will provide a solution to support external syslog service. In which user can configure if harbor send logs to outside or store it in local.

## Background

Currently, Harbor stores the log files in local. However most modern application especially the Cloud-Native apps will send their logs to remote host. In that way users do not need connect the machine to check the logs anymore, and they can access a central log service to analyse the logs which often provides user friendly interface and more powerful analysis tools. To support this Harbor should provide a way to output its logs to external endpoint.

Moreover Harbor already has a component harbor-log in which a rsyslog server is running to collect logs generated by other containers and store them to local machine.

## Proposal

1. Add the ability to harbor-log container to forwarding collected logs to other source
2. The log's time and format conforms the RFC5424 standard if forward to remote endpoint
3. Add external endpoint config item to harbor.yml and using these info to decide if store log in local or forward

## Implementation

1. Add external log endpoint related options in harbor.yml

```yaml
# Log configurations
log:
  level: info
  rotate_count: 50
  rotate_size: 200M
  # location: /var/log/harbor
  external_endpoint:
    protocol: tcp
    host: localhost
    port: 5140
  ```
  
2. Modify the Dockerfile of harbor-log container remove hardcoded rsyslog config rules

3. Forwarding rules of rsyslogd should dynamically generated by configs in harbor.yml

```conf
#  Rsyslog configuration file for docker.

template(name="DynaFile" type="string" string="/var/log/docker/%programname%.log")

if $programname != "rsyslogd" then {
{% if endpoint == "local"}
        action(type="omfile" dynaFile="DynaFile")
{% else %}
        action(type="omfwd" Target="10.192.233.234" Port="5140" Protocol="tcp" Template="RSYSLOG_SyslogProtocol23Format")
{% end %}
}
```

4. Date and log format should conform RFC5424

5. Test it in fluent-bit and log-insight