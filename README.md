### Fluentd docker image

This Docker image is to create endpoint to collect logs on your host.

Sample Docker Compose yml file:

```
server:
  image: analyser/fluentd:v0.12.12
  restart: always
  hostname: logserver
  links:
    - es:elasticsearch
    - es:es
  volumes:
    - /var/lib/docker/containers:/var/lib/docker/containers
    - ./fluentd-docker.pos:/fluentd/log/fluentd-docker.pos
    - ./fluent.conf:/fluentd/etc/fluent.conf

es:
  image: elasticsearch:latest
  restart: always
  ports:
    - "9200:9200"
    - "9300:9300"

kibana:
  ports:
    - "5601:5601"
  image: kibana:latest
  restart: always
  links:
    - es:elasticsearch
```

## Configurable ENV variables

Environment variable below are configurable to control how to execute fluentd process:

### FLUENTD_CONF

It's for configuration file name, specified for `-c`.

If you want to use your own configuration file (without any optional plugins), you can use it over this ENV variable and -v option.

1. write configuration file with filename `yours.conf`
2. execute `docker run` with `-v /path/to/dir:/fluentd/etc` to share `/path/to/dir/yours.conf` in container, and `-e FLUENTD_CONF=yours.conf` to read it

### FLUENTD_OPT

Use this variable to specify other options, like `-v` or `-q`.

## How to build your own image

It is very easy to use this image as base image. Write your `Dockerfile` and configuration files, and/or your own plugin files if needed.

```
FROM analyser/fluentd:latest
MAINTAINER your_name <...>
RUN gem install fluent-plugin-secure-forward --no-rdoc --no-ri
EXPOSE 24224
CMD fluentd -c /fluentd/etc/$FLUENTD_CONF -p /fluentd/plugins $FLUENTD_OPT
```

Files below are automatically included in build process:

`fluent.conf`: used instead of default file.
`plugins/*`: copied into `/fluentd/plugins` and loaded at runtime.
