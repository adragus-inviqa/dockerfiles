# PHP 7.0 base

For PHP 7.0
```Dockerfile
FROM quay.io/continuouspipe/php7-apache:v1.0
ARG GITHUB_TOKEN=

COPY . /app
RUN container build
```

For PHP 5.6
```Dockerfile
FROM quay.io/continuouspipe/php5.6-apache:v1.0
ARG GITHUB_TOKEN=

COPY . /app
RUN container build
```

## How to build
```bash
# For PHP 7.0
docker-compose build php70_apache
docker-compose push php70_apache

# For PHP 5.6
docker-compose build php56_apache
docker-compose push php56_apache
```

### Basic authentication

This image has support for protecting websites with basic authentication.

To use this functionality:

1. Generate a suitable password string using a tool such as Lastpass.
2. Decide upon a username to authenticate with.
3. Run the following to generate the htpasswd line: `htpasswd -n <username>`
4. Provide this htpasswd line securely in the environment for this image as `AUTH_HTTP_HTPASSWD`
5. Also provide the following variable with some values either through docker-compose environment or in
   `/usr/local/share/env/`:
  ```
  export AUTH_HTTP_ENABLED=true
  ```
6. You may also optionally configure the following in the same way:
  ```
  export AUTH_HTTP_REALM=Protected System
  export AUTH_HTTP_TYPE=Basic
  export AUTH_HTTP_FILE=/etc/apache2/custom-htpasswd-path
  ```

#### Environment variables

The following variables are supported

Variable | Description | Expected values | Default
--- | --- | --- | ----
SENDMAIL_RELAY_HOST | The MTA host to relay PHP's mail() to. PHP mail() will return false if not set | a domain
SENDMAIL_RELAY_PORT | The MTA port to relay PHP's mail() to | 0-65535 | 25
SENDMAIL_RELAY_USER | The user to authenticate with the relay. Anonymous SMTP used if not set | relay's username
SENDMAIL_RELAY_PASSWORD | The password to authenticate with the relay | relay's password
SENDMAIL_RELAY_TLS_SECURITY_LEVEL | Controls whether to use TLS, and what authentication of TLS | http://www.postfix.org/postconf.5.html#smtp_tls_security_level | may

### Custom build and startup scripts

To run commands during the build and startup sequences that the base images add,
please add `usr/local/share/container/plan.sh` for a project, or
`usr/local/share/container/baseimage-{number}.sh` if creating another base image.

This allows you to define and override bash functions that the base images add.

In addition to the bash functions defined in this base image's parent images:
* [the base image functions](../ubuntu/16.04/README.md#custom-build-and-startup-scripts)

This base image adds the following bash functions:

function | description | executed on
--- | --- | ---
do_composer | Runs composer install in /app if it's not been run yet | do_build, do_development_start
do_build_permissions | Ensures that /app is owned by the build user and not www-data or root, for security and ability to run composer as a non-root user | do_build, do_development_start

These functions can be triggered via the /usr/local/bin/container command, dropping off the "do_" part. e.g:

/usr/local/bin/container build # runs do_build
/usr/local/bin/container start_supervisord # runs do_start_supervisord
