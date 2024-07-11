[![REUSE status](https://api.reuse.software/badge/github.com/SAP-samples/managed-auth-envoy-buildpack)](https://api.reuse.software/info/github.com/SAP-samples/managed-auth-envoy-buildpack)

# Managed Authentication Buildpack

This buildpack starts an envoy that acts as reverse proxy in front of the application that authenticates incoming requests by validating the presented token.
For this it generates an envoy configuration based on the bound XSUAA/IAS instance.
The envoy validates the issuer, signature, expire time and tenant data of the token.

If no XSUAA/IAS instance is bound to the application, all requests are declined (beside requests to the paths configured via `CAUTH_PUBLIC_PATHS`). If multiple instances are bound the startup of the application will fail.

## Usage

1. Adapt the manifest file of your application to include the buildpack and place it before the final buildpack.
2. Change your application to listen on port 8000. It can also be any other port beside `8080`, then setting `CAUTH_PORT` (see [configuration](#configuration)) is required.

<pre>
applications:
- name: example-app
  memory: 256MB
  buildpacks:
    <b>- https://github.com/SAP-samples/managed-auth-envoy-buildpack</b>
    - python_buildpack
  command: gunicorn -b 0.0.0.0:<b>8000</b> -k gevent httpbin:app
</pre>


For CAP node applications the default port can be changed by setting the `PORT` environment variable. `export PORT=8000`.
For spring boot applications the `server.port` property must be set in the application.properties file `server.port=8000`.

## Configuration

Configuration of managed authentication is done via environment variables:

| Name | Description |
|------|-------------|
|CAUTH_PUBLIC_PATHS| JSON array of paths that are public and do not require token for access, e.g. `["/health"]` |
|CAUTH_PORT| Port of the application where envoy will forward the requests to. Default: `8000` |
|CAUTH_MEM| Memory of the envoy in `MB`, default `100`|
|CAUTH_LOG_LEVEL| Log level of the envoy, default `warn`|
|CAUTH_MAX_DOWNSTREAM_CON| Max connections of envoy for downstream connections, default `1000`|

## Limitations

- If port 8080 is occupied, the application will keep crashing.
- If your application uses http health check, the health endpoint needs to be configured in `CAUTH_PUBLIC_PATHS`
- No custom issuer for IAS supported (only `https://*.accounts.ondemand.com`)

## How to obtain support
[Create an issue](https://github.com/SAP-samples/managed-auth-envoy-buildpack/issues) in this repository if you find a bug or have questions about the content.
 
For additional support, [ask a question in SAP Community](https://answers.sap.com/questions/ask.html).

## Contributing
If you wish to contribute code, offer fixes or improvements, please send a pull request. Due to legal reasons, contributors will be asked to accept a DCO when they create the first pull request to this project. This happens in an automated fashion during the submission process. SAP uses [the standard DCO text of the Linux Foundation](https://developercertificate.org/).

## License
Copyright (c) 2024 SAP SE or an SAP affiliate company. All rights reserved. This project is licensed under the Apache Software License, version 2.0 except as noted otherwise in the [LICENSE](LICENSE) file.
