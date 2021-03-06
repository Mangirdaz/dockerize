dockerize ![version v0.0.3](https://img.shields.io/badge/version-v0.0.3-brightgreen.svg) ![License MIT](https://img.shields.io/badge/license-MIT-blue.svg)
=============

Utility to simplify running applications in docker containers.

dockerize is a utility to simplify running applications in docker containers.  It allows you
to generate application configuration files at container startup time from templates and
container environment variables.  It also allows log files to be tailed to stdout and/or
stderr.

The typical use case for dockerize is when you have an application that has one or more
configuration files and you would like to control some of the values using environment variables.

For example, a Python application using Sqlalchemy may be able to use environment variables directly.
It may require that the database URL be read from a python settings file with a variable named
`SQLALCHEMY_DATABASE_URI`.  dockerize allows you to set an environment variable such as
`DATABASE_URL` and update the python file when the container starts.

Another use case is when the application logs to specific files on the filesystem and not stdout
or stderr.  This makes it difficult to troubleshoot the container using the `docker logs` command.
For example, nginx will log to `/var/log/nginx/access.log' and
'/var/log/nginx/error.log' by default.  While you can sometimes work around this, it's tedious to find
the a solution for every application.  dockerize allows you to specify which logs files should
be tailed and where they should be sent.

See [A Simple Way To Dockerize Applications](http://jasonwilder.com/blog/2014/10/13/a-simple-way-to-dockerize-applications/)

## Installation

Download the latest version in your container:

* [linux/amd64](https://github.com/jwilder/dockerize/releases/download/v0.0.3/dockerize-linux-amd64-v0.0.3.tar.gz)

For Ubuntu Images:

```
RUN apt-get update && apt-get install -y wget
RUN wget https://github.com/jwilder/dockerize/releases/download/v0.0.3/dockerize-linux-amd64-v0.0.3.tar.gz
RUN tar -C /usr/local/bin -xzvf dockerize-linux-amd64-v0.0.3.tar.gz
```

## Usage

dockerize works by wrapping the call to your application using the `ENTRYPOINT` or `CMD` directives.

This would generate `/etc/nginx/nginx.conf` from the template located at `/etc/nginx/nginx.tmpl` and
send `/var/log/nginx/access.log' to `STDOUT` and `/var/log/nginx/error.log` to `STDERR` after running
`nginx`.

```
CMD dockerize -template /etc/nginx/nginx.tmpl:/etc/nginx/nginx.conf -stdout /var/log/nginx/access.log -stderr /var/log/nginx/error.log nginx
```

### Command-line Options

You can specify multiple template by passing using `-template` multiple times:

```
$ dockerize -template template1.tmpl:file1.cfg -template template2.tmpl:file3

```

You can tail multiple files to `STDOUT` and `STDERR` by passing the options multiple times.

```
$ dockerize -stdout info.log -stdout perf.log

```

If your file uses `{{` and `}}` as part of it's syntax, you can change the template escape characters using the `-delims`.

```
$ dockerize -delims "<%:%>"
```

## Using Templates

Templates use Golang [text/template](http://golang.org/pkg/text/template/). You can access environment
variables within a template with `.Env`.

```
{{ .Env.PATH }} is my path
```

There are a few built in functions as well:

  * `default $var $default` - Returns a default value for one that does not exist. `{{ default .Env.VERSION "0.1.2" }}`
  * `contains $map $key` - Returns true if a string is within another string
  * `exists $path` - Determines if a file path exists or not. `{{ exists "/etc/default/myapp" }}`
  * `split $string $sep` - Splits a string into an array using a separator string. Alias for [`strings.Split`][go.string.Split]. `{{ split .Env.PATH ":" }}`
  * `replace $string $old $new $count` - Replaces all occurences of a string within another string. Alias for [`strings.Replace`][go.string.Replace]. `{{ replace .Env.PATH ":" }}`
  * `parseUrl $url` - Parses a URL into it's [protocol, scheme, host, etc. parts][go.url.URL]. Alias for [`url.Parse`][go.url.Parse]

## License

MIT


[go.string.Split]: https://golang.org/pkg/strings/#Split
[go.string.Replace]: https://golang.org/pkg/strings/#Replace
[go.url.Parse]: https://golang.org/pkg/net/url/#Parse
[go.url.URL]: https://golang.org/pkg/net/url/#URL
