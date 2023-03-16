# Traefik

I love Traefik. Here are some of my snippets.


## Add trailing slash to URL (/)

For some raeson your service requires a trailing slash in order to work.
However, for a better user experience you want to drop this requirement.
This can be done by using:

```yaml
http:
  middlewares:
    slashRedirect:
      redirectRegex:
        regex: '^(https?://.*[^/]+/[a-z0-9_-]+)$'
        replacement: '${1}/'
        permanent: true
```

This will automatically redirect `https://example.com/foo` to `https://exmaple.com/foo/`.


!!! note
    Keep in mind, that RegEx is expensive. Therefore, we will set `permanent` to  `true` to return with a response code `301` instead of `302` so the browser may cache it.


!!! tip
    For other RegEx examples visit the [RegEx101.com](https://regex101.com/r/wY4bFu/) playground.

## Headers

### Remove response header

Some times you might need to remove a header from the response.  
In this example we want to remove the `Set-Cookie` header. 
This can simply be done be repleacing it with an empty string:

```yaml
http:
  middlewares:
    removeCookie:
      headers:
        customResponseHeaders:
          set-cookie: ""
```


### Add Response Header to every Request via Entrypoint

Let's say we have a dynamic middleware called `hostname`:

```yaml
http:
  middlewares:
    hostname:
      headers:
        customResponseHeaders:
          x-proxy: "foo.example.com"
```

In order to a apply the `hostname` middleware to all our respones without adding it to every router,
we may apply to direct to our entrypoint `web` at port `80`:

```yaml
services:
  traefik:
    command:
      ...
      - "--entrypoints.web.address=:80"
      - "--entrypoints.web.http.middlewares=hostname@file"
```

## Query Parameters

### Removing a Query Parameter

To remove a parameter named `_pat` this can be done by using the redirectRegex middleware:

```yaml
http:
  middlewares:
    remove-parameter:
      redirectRegex:
        regex: "^(.+)(_pat=\\w+)(.*)"
        replacement: "${1}${3}"
```

!!! warn
    This causes an additional redirect, which negatively affects the overall performance of your application. Major search engines will downgrade your search ranking, as this is considered a poor user experience.


## mitmproxy 

Here is how to add [mitproxy](https://mitmproxy.org/) to your app.
Let's say we have a service running at `example.com:8000`.


```yaml
services:
  mitmweb:
    image: mitmproxy/mitmproxy:9.0.1
    tty: true
    command: mitmweb --web-host 0.0.0.0 --mode reverse:http://example.com:8000
    labels:
      - "traefik.enable=true"
      - "traefik.http.routers.proxy-ui.rule=Host(`proxy.example.com`)"
      - "traefik.http.routers.proxy-ui.service=proxy-ui"
      - "traefik.http.services.proxy-ui.loadbalancer.server.port=8081"
      - "traefik.http.services.proxy-ui.loadbalancer.passHostHeader=false"
      - "traefik.http.routers.proxy-ui.middlewares=mitmproxy-header"
      - "traefik.http.routers.proxy.rule=Host(`example.com`)"
      - "traefik.http.routers.proxy.service=proxy"
      - "traefik.http.services.proxy.loadbalancer.server.port=8000"
      - "traefik.http.middlewares.mitmproxy-header.headers.customrequestheaders.Origin="
      - "traefik.http.middlewares.mitmproxy-header.headers.customresponseheaders.X-Content-Type-Options="
    networks:
      - traefik
```

This will launch the proxy which can be accessed from `exmaple.com`. The mitmproxy web UI can be accessed by `proxy.example.com`.
