
## Install vaping

First, you will always need to install vaping with:

```sh
pip install vaping
```

## Example Standalone Latency

The example config file (from `examples/standalone_dns`) uses both vodka and graphsrv plugins, so those will need to be installed with:

```sh
pip install vodka
pip install graphsrv
```


Then just start vaping with:

```sh
vaping start --home=examples/standalone_dns/ --debug
```

By default it uses a generic layout template and listens on http://0.0.0.0:7021 - going to that in a browser should produce the summary paging which looks like:

![Vaping](https://raw.githubusercontent.com/20c/vaping/master/docs/img/standalone_dns.png)

And clicking on the title will goto a detail page which looks like:

![Vaping](https://raw.githubusercontent.com/20c/vaping/master/docs/img/standalone_dns-detail.png)

Below is the config file, common things to change include:

- hosts: probes.public_dns.hosts
- listening address: plugins.vodka.plugins.http.host and port.
- fping frequency: plugins.std_fping.count and interval

Config file:

`examples/standalone_dns/config.yml`:
```yml
{!examples/standalone_dns/config.yml!}
```


## Example Distributed Latency

The example config file (from `examples/distributed_dns`) is the same as the standalone one and uses both vodka and graphsrv, so those will need to be installed with:

```sh
pip install vodka
pip install graphsrv
```

The main difference is the collector is running in a separate process than the
web server, which allows you to graph things from multiple locations, as well
as using another webserver, such as nginx to serve client requests.


To try it out, start vaping with:

```sh
vaping start --home=examples/distributed_dns/vaping/ --debug
```

### Gunicorn

To test with gunicorn, first install it:

```sh
pip install gunicorn
```

plugins['http'].server is already set to 'gunicorn' in
`examples/distributed_dns/vodka/config.yml` so run:

```sh
export VODKA_HOME=examples/distributed_dns/vodka
gunicorn -b 0.0.0.0:7021 vodka.runners.wsgi:application
```

You should be able to browse to port 7021 to see the display.


### nginx

To test with nginx, install uwsgi:

```sh
pip install uwsgi
```

In `examples/distributed_dns/vodka/config.yml` change plugins['http'].server
to 'uwsgi' and then configure nginx with an upstream to connect to it.

`nginx.conf`:
```
upstream vaping {
    server 127.0.0.1:7021;
}
```

Start the uwsgi process with:

```sh
export VODKA_HOME=examples/distributed_dns/vodka
uwsgi -H $VIRTUAL_ENV --socket=0.0.0.0:7026 -w vodka.runners.wsgi:application --enable-threads
```

And you should be able to point your browser to the address nginx is listening
on to view it.

!!! Tip "Note"
    If you're running selinux, you'll need to allow nginx to connect to it
    with `setsebool -P httpd_can_network_connect 1`.

Config files:

`examples/distributed_dns/vaping/config.yml`:
```yml
{!examples/distributed_dns/vaping/config.yml!}
```

`examples/distributed_dns/vodka/config.yml`:
```yml
{!examples/distributed_dns/vodka/config.yml!}
```
