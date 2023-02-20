# Traefik


## Add Trailing slash (/)

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

### Remove Response Header

Some times you might need to remove a header from the response.  
In this example we want to remove the `Set-Cookie` header:

```yaml
http:
  middlewares:
    removeCookie:
      headers:
        customResponseHeaders:
          set-cookie: ""
```


### Add Response Header via Entrypoint

TODO


## mitmproxy 

Here is how to add [mitproxy](https://mitmproxy.org/) to your app.

Example:

TODO