# Blackbox exporter

The blackbox exporter allows blackbox probing of endpoints over
HTTP, HTTPS and TCP.

## Building and running

    make
    ./blackbox_exporter <flags>

Visiting [http://localhost:9115/probe?address=google.com&module=http2xx](http://localhost:9115/probe?address=google.com&module=http2xx)
will return metrics for a HTTP probe against google.com.

## Configuration

A configuration showing all options is below:
```
modules:
  http2xx:
    prober: http
    timeout: 5s
    http:
      valid_status_codes: []  # Defaults to 2xx
      no_follow_redirects: false
      fail_if_ssl: false
      fail_if_not_ssl: false
  tcpconnect:
    prober: tcp
    timeout: 5s
  icmp:
    prober: icmp
    timeout: 5s
```

HTTP, HTTPS (via the `http` prober), TCP socket and ICMP (v4 only) are currently supported.
Additiona modules can be defined to meet your needs.


## Prometheus Configuration

The blackbox exporter needs to be passed the target as a parameter, this can be
done with relabelling.

Example config:
```
scrape_config:
  - job_name: 'blackbox'
    metrics_path: /probe
    params:
      module: [http2xx]  # Look for a HTTP 200 response.
    target_groups:
      - targets:
        - http://mywebsite.com  # Target to probe
    relabel_configs:
      - source_labels: [__address__]
        regex: (.*):80
        target_label: __param_target
        replacement: ${1}
      - source_labels: [__param_address]
        regex: (.*)
        target_label: instance
        replacement: ${1}
      - source_labels: []
        regex: .*
        target_label: __address__
        replacement: 127.0.0.1:9115  # Blackbox exporter.
```
