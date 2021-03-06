# Customization of NGINX Configuration

You can customize the NGINX configuration using ConfigMaps or Annotations.

The table below summarizes some of the options. More options (extensions) are available, refer to the [examples](..) folder.

| Annotation | ConfigMaps Key | Description | Default |
| ---------- | -------------- | ----------- | ------- |
| `nginx.org/proxy-read-timeout` | `proxy-read-timeout` | Sets the value of the [proxy_connect_timeout](http://nginx.org/en/docs/http/ngx_http_proxy_module.html#proxy_connect_timeout) directive. | `60s` |
| `nginx.org/proxy-read-timeout` | `proxy-read-timeout` | Sets the value of the [proxy_read_timeout](http://nginx.org/en/docs/http/ngx_http_proxy_module.html#proxy_read_timeout) directive. | `60s` |
| `nginx.org/client-max-body-size` | `client-max-body-size` | Sets the value of the [client_max_body_size](http://nginx.org/en/docs/http/ngx_http_core_module.html#client_max_body_size) directive. | `1m` |
| `nginx.org/proxy-buffering` | `proxy-buffering` | Enables or disables [buffering of responses](http://nginx.org/en/docs/http/ngx_http_proxy_module.html#proxy_buffering) from the proxied server. | `True` |
| `nginx.org/proxy-buffers` | `proxy-buffers` | Sets the value of the [proxy_buffers](http://nginx.org/en/docs/http/ngx_http_proxy_module.html#proxy_buffers) directive. | Depends on the platform. |
| `nginx.org/proxy-buffer-size` | `proxy-buffer-size` | Sets the value of the [proxy_buffer_size](http://nginx.org/en/docs/http/ngx_http_proxy_module.html#proxy_buffer_size) directive | Depends on the platform. |
| `nginx.org/proxy-max-temp-file-size` | `proxy-max-temp-file-size` | Sets the value of the  [proxy_max_temp_file_size](http://nginx.org/en/docs/http/ngx_http_proxy_module.html#proxy_max_temp_file_size) directive. | `1024m` |
| N/A | `server-names-hash-bucket-size` | Sets the value of the [server_names_hash_max_size](http://nginx.org/en/docs/http/ngx_http_core_module.html#server_names_hash_max_size) directive. | Depends on the size of the processor’s cache line. |
| N/A | `server-names-hash-max-size` | Sets the value of the [server_names_hash_bucket_size](http://nginx.org/en/docs/http/ngx_http_core_module.html#server_names_hash_bucket_size) directive. | `512` |
| `nginx.org/http2` | `http2` | Enables HTTP/2 in servers with SSL enabled. To support HTTP/2 for Chrome users, use the provided controller image based on the alpine Linux. It includes OpenSSL with ALPN support, [necessary for Chrome users](https://www.nginx.com/blog/supporting-http2-google-chrome-users/). | `False` |
| N/A | `log-format` | Sets the custom [log format](http://nginx.org/en/docs/http/ngx_http_log_module.html#log_format).  | See the [template file](../../nginx-controller/nginx/nginx.conf.tmpl). |  
| `nginx.org/hsts` | `hsts` | Enables [HTTP Strict Transport Security (HSTS)](https://www.nginx.com/blog/http-strict-transport-security-hsts-and-nginx/): the HSTS header is added to the responses from backends. The `preload` directive is included in the header. | `False` |
| `nginx.org/hsts-max-age` | `hsts-max-age` | Sets the value of the `max-age` directive of the HSTS header. | `2592000` (1 month) |
| `nginx.org/hsts-include-subdomains` | `hsts-include-subdomains` | Adds the `includeSubDomains` directive to the HSTS header. | `False`|

## Using ConfigMaps

1. Make sure that you specify the configmaps resource to use when you start an Ingress controller.
For example, `-nginx-configmaps=default/nginx-config`, where we specify
the config map to use with the following format: `<namespace>/<name>`. See [nginx-ingress-rc.yaml](../complete-example/nginx-ingress-rc.yaml) or
[nginx-plus-ingress-rc.yaml](../complete-example/nginx-plus-ingress-rc.yaml) files.

1. Create a configmaps file with the name *nginx-config.yaml* and set the values
that make sense for your setup:
  ```yaml
  kind: ConfigMap
  apiVersion: v1
  metadata:
    name: nginx-config
  data:
    proxy-connect-timeout: "10s"
    proxy-read-timeout: "10s"
    client-max-body-size: "2m"
  ```
  See the **nginx-config.yaml** from this directory for a complete example.

2. Create a configmaps resource:
  ```
  $ kubectl create -f nginx-config.yaml
  ```
  The NGINX configuration will be updated.

3. If you want to update the configmaps, update the file and replace the config map:
  ```
  $ kubectl replace -f nginx-config.yaml
  ```
  The NGINX configuration will be updated.

## Using Annotations

If you want to customize the configuration for a particular Ingress resource only, you can use Annotations.
Here is an example (**cafe-ingress-with-annotations.yaml**):
```yaml
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: cafe-ingress-with-annotations
  annotations:
    nginx.org/proxy-connect-timeout: "30s"
    nginx.org/proxy-read-timeout: "20s"
    nginx.org/client-max-body-size: "4m"
spec:
  rules:
  - host: cafe.example.com
    http:
      paths:
      - path: /tea
        backend:
          serviceName: tea-svc
          servicePort: 80
      - path: /coffee
        backend:
          serviceName: coffee-svc
          servicePort: 80
```
Annotations take precedence over ConfigMaps.
