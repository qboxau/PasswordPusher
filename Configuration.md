
# Overview

Configure everything from defaults, features, branding, languages and more.

# How to Configure the Application

Password Pusher uses a centralized configuration that is stored in [config/settings.yml](https://github.com/pglombardo/PasswordPusher/blob/master/config/settings.yml).  This file contains all of the settings that is configurable for the application.

There are two ways to modify the settings in this file:

1. Use environment variable that override this file
2. Modify the file itself

For a few modifications, environment variables are the easy route.  For more extensive configuration, it's suggested to maintain your own custom `settings.yml` file across updates.

Read on for details on both methods.

## Configuring via Environment variables

The settings in the `config/settings.yml` file can be overridden by environment variables.  A listing and description of these environment variables is available in this documentation below and also in the [settings.yml](https://github.com/pglombardo/PasswordPusher/blob/master/config/settings.yml) file itself.

### Shell Example

```sh
# Change the default language for the application to French
export PWP__DEFAULT_LOCALE='fr'
```
### Docker Example

```sh
# Change the default language for the application to French
docker run -d --env PWP__DEFAULT_LOCALE=fr -p "5100:5100" pglombardo/pwpush-ephemeral:release
```

## Configuring via a Custom `settings.yml` File

If you prefer, you can take the [default settings.yml file](https://github.com/pglombardo/PasswordPusher/blob/master/config/settings.yml), modify it and apply it to the Password Pusher Docker container.

Inside the Password Pusher Docker container:
* application code exists in the path `/opt/PasswordPusher/`
* the `settings.yml` file is located at `/opt/PasswordPusher/config/settings.yml`

To replace this file with your own custom version, you can launch the Docker container with a bind mount option:

```sh
    docker run -d \
      --mount type=bind,source=/path/settings.yml,target=/opt/PasswordPusher/config/settings.yml \
      -p "5100:5100" pglombardo/pwpush-ephemeral:release
```

# Application Encryption

Password Pusher encrypts sensitive data in the database. This requires a randomly generated encryption key for each application instance.

To set a custom encryption key for your application, set the environment variable `PWPUSH_MASTER_KEY`:

    PWPUSH_MASTER_KEY=0c110f7f9d93d2122f36debf8a24bf835f33f248681714776b336849b801f693

## Generate a New Encryption Key

Key generation can be done through the [helper tool](https://pwpush.com/pages/generate_key) or on the command line in the application source using `Lockbox.generate_key`:

```ruby
bundle
rails c
Lockbox.generate_key
```

Notes:

* If an encryption key isn't provided, a default key will be used.
* The best security for private instances of Password Pusher is to use your own custom encryption key although it is not required.
* The risk in using the default key is lessened if you keep your instance secure and your push expirations short. e.g. 1 day/1 view versus 100 days/100 views.
* Once a push expires, all encrypted data is deleted.
* Changing an encryption key where old pushes already exist will make those older pushes unreadable. In other words, the payloads will be garbled. New pushes going forward will work fine.


# Changing Application Defaults

## Application General

| Environment Variable | Description | Default Value |
| --------- | ------------------ | --- |
| PWP__DEFAULT_LOCALE | Sets the default language for the application.  See the [documentation](https://github.com/pglombardo/PasswordPusher#internationalization). | `en` |
| PWP__RELATIVE_ROOT | Runs the application in a subfolder.  e.g. With a value of `pwp` the front page will then be at `https://url/pwp` | `Not set` |

## Push Form Defaults

| Environment Variable | Description | Default Value |
| --------- | ------------------ | --- |
| PWP__EXPIRE_AFTER_DAYS_DEFAULT | Controls the "Expire After Days" default value in Password#new | `7` |
| PWP__EXPIRE_AFTER_DAYS_MIN | Controls the "Expire After Days" minimum value in Password#new | `1` |
| PWP__EXPIRE_AFTER_DAYS_MAX | Controls the "Expire After Days" maximum value in Password#new | `90` |
| PWP__EXPIRE_AFTER_VIEWS_DEFAULT | Controls the "Expire After Views" default value in Password#new | `5` |
| PWP__EXPIRE_AFTER_VIEWS_MIN | Controls the "Expire After Views" minimum value in Password#new | `1` |
| PWP__EXPIRE_AFTER_VIEWS_MAX | Controls the "Expire After Views" maximum value in Password#new | `100` |
| PWP__ENABLE_DELETABLE_PUSHES | Can passwords be deleted by viewers? When true, passwords will have a link to optionally delete the password being viewed | `false` |
| PWP__DELETABLE_PASSWORDS_DEFAULT | When the above is `true`, this sets the default value for the option. | `true` |
| PWP__ENABLE_RETRIEVAL_STEP | When `true`, adds an option to have a preliminary step to retrieve passwords.  | `true` |
| PWP__RETRIEVAL_STEP_DEFAULT | Sets the default value for the retrieval step for newly created passwords. | `false` |

# Enabling Logins

To enable logins in your instance of Password Pusher, you must have an SMTP server available to send emails through.  These emails are sent for events such as password reset, unlock, registration etc..

To use logins, you should be running a databased backed version of Password Pusher.  Logins will likely work in ephemeral but aren't suggested since all data is wiped with every restart.

_All_ of the following environments need to be set (except SMTP authentication if none) for application logins to function properly.

| Environment Variable | Description | Default |
| --------- | ------------------ | --- |
| PWP__ENABLE_LOGINS | On/Off switch for logins. | `false` |
| PWP__ALLOW_ANONYMOUS | When false, requires a login for the front page (to push new passwords). | `true` |
| PWP__MAIL__RAISE_DELIVERY_ERRORS | Email delivery errors will be shown in the application | `true` |
| PWP__MAIL__SMTP_ADDRESS | Allows you to use a remote mail server. Just change it from its default "localhost" setting. | `smtp.domain.com` |
| PWP__MAIL__SMTP_PORT | Port of the SMTP server | `587` |
| PWP__MAIL__SMTP_USER_NAME | If your mail server requires authentication, set the username in this setting. | `smtp_username` |
| PWP__MAIL__SMTP_PASSWORD | If your mail server requires authentication, set the password in this setting. | `smtp_password` |
| PWP__MAIL__SMTP_AUTHENTICATION | If your mail server requires authentication, you need to specify the authentication type here. This is a string and one of :plain (will send the password in the clear), :login (will send password Base64 encoded) or :cram_md5 (combines a Challenge/Response mechanism to exchange information and a cryptographic Message Digest 5 algorithm to hash important information) | `plain` |
| PWP__MAIL__SMTP_STARTTLS | Use STARTTLS when connecting to your SMTP server and fail if unsupported. | `true` |
| PWP__MAIL__OPEN_TIMEOUT | Number of seconds to wait while attempting to open a connection. | `10` |
| PWP__MAIL__READ_TIMEOUT | Number of seconds to wait until timing-out a read(2) call. | `10` |
| PWP__HOST_DOMAIN | Used to build fully qualified URLs in emails.  Where is your instance hosted? | `pwpush.com` |
| PWP__HOST_PROTOCOL | The protocol to access your Password Pusher instance.  HTTPS advised. | `https` |
| PWP__MAIL__MAILER_SENDER | This is the "From" address in sent emails. | '"Company Name" <user@example.com>' |
| PWP__DISABLE_SIGNUPS| Once your user accounts are created, you can set this to disable any further user account creation.  Sign up links and related backend functionality is disabled when `true`. | `false` |

## Shell Example

```
export PWP__ENABLE_LOGINS=true
export PWP__MAIL__RAISE_DELIVERY_ERRORS=true
export PWP__MAIL__SMTP_ADDRESS=smtp.mycompany.org
export PWP__MAIL__SMTP_PORT=587
export PWP__MAIL__SMTP_USER_NAME=yolo
export PWP__MAIL__SMTP_PASSWORD=secret
export PWP__MAIL__SMTP_AUTHENTICATION=plain
export PWP__MAIL__SMTP_STARTTLS=true
export PWP__MAIL__OPEN_TIMEOUT=10
export PWP__MAIL__READ_TIMEOUT=10
export PWP__HOST_DOMAIN=pwpush.mycompany.org
export PWP__HOST_PROTOCOL=https
export PWP__MAIL__MAILER_SENDER='"Spiderman" <thespider@mycompany.org>'
```

* See also this [Github discussion](https://github.com/pglombardo/PasswordPusher/issues/265#issuecomment-964432942).
* [External Documentation on mailer configuration](https://guides.rubyonrails.org/action_mailer_basics.html#action-mailer-configuration) for the underlying technology if you need more details for configuration issues.

# Enabling File Pushes

To enable file uploads (File Pushes) in your instance of Password Pusher, you must have logins enabled (see above) and specify a place to store uploaded files.

The following settings enable/disable the feature and specify where to store uploaded files.

This feature can store uploads on local disk (not valid for Docker containers), Amazon S3, Google Cloud Storage or Azure Storage.

## General Settings

| Environment Variable | Description | Value(s) |
| --------- | ------------------ | --- |
| PWP__ENABLE_FILE_PUSHES | On/Off switch for File Pushes. | `false` |
| PWP__FILES__STORAGE | Chooses the storage area for uploaded files. | `local`, `s3`, `gcs` or `as` |

## Local Storage

The default location for local storage is `./storage`.

If using containers and you prefer local storage, you can add a volume mount to the container at the path `/opt/PasswordPusher/storage`:

`docker run -d -p "5100:5100" -v /var/lib/pwpush/files:/opt/PasswordPusher/storage pglombardo/pwpush-postgres:release`

## Amazon S3

| Environment Variable | Description | Value(s) |
| --------- | ------------------ | --- |
| PWP__FILES__S3__ENDPOINT | S3 Endpoint | None |
| PWP__FILES__S3__ACCESS_KEY_ID | Access Key ID | None |
| PWP__FILES__S3__SECRET_ACCESS_KEY | Secret Access Key| None |
| PWP__FILES__S3__REGION | S3 Region| None |
| PWP__FILES__S3__BUCKET | The S3 bucket name | None |

## Google Cloud Storage

| Environment Variable | Description | Value(s) |
| --------- | ------------------ | --- |
| PWP__FILES__GCS__PROJECT | GCS Project | None |
| PWP__FILES__GCS__CREDENTIALS | GCS Credentials | None |
| PWP__FILES__GCS__BUCKET | The GCS bucket name | None |

## Azure Storage

| Environment Variable | Description | Value(s) |
| --------- | ------------------ | --- |
| PWP__FILES__AS__STORAGE_ACCOUNT_NAME | Azure Storage Account Name | None |
| PWP__FILES__AS__STORAGE_ACCESS_KEY | Azure Storage Account Key | None |
| PWP__FILES__AS__CONTAINER | Azure Storage Container Name | None |

# Enabling URL Pushes

Similar to file pushes, URL pushes also require logins to be enabled.

| Environment Variable | Description | Default |
| --------- | ------------------ | --- |
| PWP__ENABLE_URL_PUSHES | On/Off switch for URL Pushes. | `false` |

# Rebranding

Password Pusher has the ability to be [re-branded](https://twitter.com/pwpush/status/1557658305325109253) with your own site title, tagline and logo.

This can be done with the following environment variables:

| Environment Variable | Description | Default Value |
| --------- | ------------------ | --- |
| PWP__BRAND__TITLE | Title for the site. | `Password Pusher` |
| PWP__BRAND__TAGLINE | Tagline for the site.  | `Go Ahead.  Email Another Password.` |
| PWP__BRAND__SHOW_FOOTER_MENU | On/Off switch for the footer menu. | `true` |
| PWP__BRAND__LIGHT_LOGO | Site logo image for the light theme. | `media/img/logo-transparent-sm-bare.png` |
| PWP__BRAND__DARK_LOGO | Site logo image for the dark theme. | `media/img/logo-transparent-sm-bare.png` |

## See Also

* the `brand` section of [settings.yml](https://github.com/pglombardo/PasswordPusher/blob/master/config/settings.yml) for more details, examples and description.
* [this issue comment](https://github.com/pglombardo/PasswordPusher/issues/432#issuecomment-1282158006) on how to mount images into the contianer and set your environment variables accordingly


# Google Analytics

| Environment Variable | Description |
| --------- | ------------------ |
| GA_ENABLE | The existence of this variable will enable the Google Analytics for the application.  See `app/views/layouts/_ga.html.erb`.|
| GA_ACCOUNT | The Google Analytics account id.  E.g. `UA-XXXXXXXX-X` |
| GA_DOMAIN | The domain where the application is hosted.  E.g. `pwpush.com` |

# Throttling

Throttling enforces a minimum time interval
between subsequent HTTP requests from a particular client, as
well as by defining a maximum number of allowed HTTP requests
per a given time period (per second, minute, hourly, or daily).

| Environment Variable | Description | Default Value |
| --------- | ------------------ | --- |
| PWP__THROTTLING__DAILY | The maximum number of allowed HTTP requests per day | `1000` |
| PWP__THROTTLING__HOURLY | The maximum number of allowed HTTP requests per hour | `100` |
| PWP__THROTTLING__MINUTE | The maximum number of allowed HTTP requests per minute | `30` |
| PWP__THROTTLING__SECOND | The maximum number of allowed HTTP requests per second | `2` |


# Logging

| Environment Variable | Description |
| --------- | ------------------ |
| PWP__LOG_LEVEL | Set the logging level for the application.  Valid values are: `debug`, `info`, `warn`, `error` and `fatal`.  Note: lowercase.
| PWP__LOG_TO_STDOUT | Set to 'true' to have log output sent to STDOUT instead of log files.  Default: `false`


# Forcing SSL Links

See also the Proxies section below.

| Environment Variable | Description |
| --------- | ------------------ |
| FORCE_SSL | The existence of this variable will set `config.force_ssl` to `true` and generate HTTPS based secret URLs

# Proxies

An occasional issue is that when using Password Pusher behind a proxy, the generated secret URLs are incorrect.  They often have the backend URL & port instead of the public fully qualified URL - or use HTTP instead of HTTPS (or all of the preceding).

To resolve this, make sure your proxy properly forwards the `X-Forwarded-Host`, `X-Forwarded-Port` and `X-Forwarded-Proto` headers.

The values in these headers represent the front end request.  When these headers are sent, Password Pusher can then build the correct URLs.

If you are unable to have these headers passed to the application for any reason, you could instead force an override of the base URL using the `PWP__OVERRIDE_BASE_URL` environment variable.

| Environment Variable | Description | Example Value |
| --------- | ------------------ | --- |
| PWP__OVERRIDE_BASE_URL | Set this value (without a trailing slash) to force the base URL of generated links. | 'https://subdomain.domain.dev'
