Administrator documentation
Settings
settings.yml
engines:
brand:
general:
search:
server:
ui:
redis:
outgoing:
categories_as_tabs:
plugins:
Installation
Docker Container
Get Docker
searxng/searxng
Build the image
Command line
Installation Script
Step by step installation
Install packages
Create user
Install SearXNG & dependencies
Configuration
Check
uWSGI
Origin uWSGI
Distributors
uWSGI maintenance
uWSGI setup
Pitfalls of the Tyrant mode
NGINX
The nginx HTTP server
NGINX’s SearXNG site
Disable logs
Apache
The Apache HTTP server
Apache’s SearXNG site
disable logs
SearXNG maintenance
How to update
How to inspect & debug
Migrate and stay tuned!
Answer CAPTCHA from server’s IP
Favicons
Infrastructure
Setting up the cache
Proxy configuration
Limiter
Enable Limiter
Configure Limiter
limiter.toml
Implementation
LIMITER_CFG_SCHEMA
pre_request()
is_installed()
initialize()
Administration API
Get configuration data
Embed search bar
Architecture
uWSGI Setup
List of plugins
Buildhosts
Build and Development tools
Build docs
Lint shell scripts
-------------------
settings.yml
This page describe the options possibilities of the git://searx/settings.yml file.

Further reading ..

Configuration

Search API

settings.yml location

use_default_settings

settings.yml location
The initial settings.yml we be load from these locations:

the full path specified in the SEARXNG_SETTINGS_PATH environment variable.

/etc/searxng/settings.yml

If these files don’t exist (or are empty or can’t be read), SearXNG uses the git://searx/settings.yml file. Read use_default_settings to see how you can simplify your user defined settings.yml.

use_default_settings
use_default_settings: true

settings.yml location

Configuration

/etc/searxng/settings.yml

The user defined settings.yml is loaded from the settings.yml location and can relied on the default configuration git://searx/settings.yml using:

use_default_settings: true

server:
In the following example, the actual settings are the default settings defined in git://searx/settings.yml with the exception of the secret_key and the bind_address:

use_default_settings: true
server:
    secret_key: "ultrasecretkey"   # change this!
    bind_address: "[::]"
engines:
With use_default_settings: true, each settings can be override in a similar way, the engines section is merged according to the engine name. In this example, SearXNG will load all the default engines, will enable the bing engine and define a token for the arch linux engine:

use_default_settings: true
server:
  secret_key: "ultrasecretkey"   # change this!
engines:
  - name: arch linux wiki
    tokens: ['$ecretValue']
  - name: bing
    disabled: false
engines: / remove:
It is possible to remove some engines from the default settings. The following example is similar to the above one, but SearXNG doesn’t load the the google engine:

use_default_settings:
  engines:
    remove:
      - google
server:
  secret_key: "ultrasecretkey"   # change this!
engines:
  - name: arch linux wiki
    tokens: ['$ecretValue']
engines: / keep_only:
As an alternative, it is possible to specify the engines to keep. In the following example, SearXNG has only two engines:

use_default_settings:
  engines:
    keep_only:
      - google
      - duckduckgo
server:
  secret_key: "ultrasecretkey"   # change this!
engines:
  - name: google
    tokens: ['$ecretValue']
  - name: duckduckgo
    tokens: ['$ecretValue']
-----
engines:
Further reading ..

Configured Engines

Engine Overview

In the section engines: is a list of the engines that are to be made available in the instance. Each list entry is in turn a key/value mapping.

engines:

  - name: dummy.online
    engine: dummy
    ..
  - name: dummy.offline
    engine: dummy-offline
    ..
  ..
In the code example below a full fledged example of a YAML setup from a dummy engine is shown. Most of the options have a default value or even are optional.

Hint
A few more options are possible, but they are pretty specific to some engines (Engine Implementations).

- name: example
  engine: example
  shortcut: demo
  base_url: 'https://{language}.example.com/'
  send_accept_language_header: false
  categories: general
  timeout: 3.0
  api_key: 'apikey'
  disabled: false
  language: en_US
  tokens: [ 'my-secret-token' ]
  weight: 1
  display_error_messages: true
  about:
     website: https://example.com
     wikidata_id: Q306656
     official_api_documentation: https://example.com/api-doc
     use_official_api: true
     require_api_key: true
     results: HTML

  # overwrite values from section 'outgoing:'
  enable_http2: false
  retries: 1
  max_connections: 100
  max_keepalive_connections: 10
  keepalive_expiry: 5.0
  using_tor_proxy: false
  proxies:
    http:
      - http://proxy1:8080
      - http://proxy2:8080
    https:
      - http://proxy1:8080
      - http://proxy2:8080
      - socks5://user:password@proxy3:1080
      - socks5h://user:password@proxy4:1080

  # other network settings
  enable_http: false
  retry_on_http_error: true # or 403 or [404, 429]
name :
Name that will be used across SearXNG to define this engine. In settings, on the result page…

engine :
Name of the python file used to handle requests and responses to and from this search engine.

shortcut :
Code used to execute bang requests (in this case using !bi)

base_urloptional
Part of the URL that should be stable across every request. Can be useful to use multiple sites using only one engine, or updating the site URL without touching at the code.

send_accept_language_header :
Several engines that support languages (or regions) deal with the HTTP header Accept-Language to build a response that fits to the locale. When this option is activated, the language (locale) that is selected by the user is used to build and send a Accept-Language header in the request to the origin search engine.

categoriesoptional
Specifies to which categories the engine should be added. Engines can be assigned to multiple categories.

Categories can be shown as tabs (categories_as_tabs:) in the UI. A search in a tab (in the UI) will query all engines that are active in this tab. In the preferences page (/preferences) – under engines – users can select what engine should be active when querying in this tab.

Alternatively, !bang can be used to search all engines in a category, regardless of whether they are active or not, or whether they are in a tab of the UI or not. For example, !dictionaries can be used to query all search engines in that category (group).

timeoutoptional
Timeout of the search with the current search engine. Overwrites request_timeout from outgoing:. Be careful, it will modify the global timeout of SearXNG.

api_keyoptional
In a few cases, using an API needs the use of a secret key. How to obtain them is described in the file.

disabledoptional
To disable by default the engine, but not deleting it. It will allow the user to manually activate it in the settings.

inactive: optional
Remove the engine from the settings (disabled & removed).

languageoptional
If you want to use another language for a specific engine, you can define it by using the ISO code of language (and region), like fr, en-US, de-DE.

tokensoptional
A list of secret tokens to make this engine private, more details see Private Engines (tokens).

weightdefault 1
Weighting of the results of this engine.

display_error_messagesdefault true
When an engine returns an error, the message is displayed on the user interface.

networkoptional
Use the network configuration from another engine. In addition, there are two default networks:

ipv4 set local_addresses to 0.0.0.0 (use only IPv4 local addresses)

ipv6 set local_addresses to :: (use only IPv6 local addresses)

enable_httpoptional
Enable HTTP for this engine (by default only HTTPS is enabled).

retry_on_http_erroroptional
Retry request on some HTTP status code.

Example:

true : on HTTP status code between 400 and 599.

403 : on HTTP status code 403.

[403, 429]: on HTTP status code 403 and 429.

proxies :
Overwrites proxy settings from outgoing:.

using_tor_proxy :
Using tor proxy (true) or not (false) for this engine. The default is taken from using_tor_proxy of the outgoing:.

max_keepalive_connection#s :
Pool limit configuration, overwrites value pool_maxsize from
outgoing: for this engine.

max_connections :
Pool limit configuration, overwrites value pool_connections from outgoing: for this engine.

keepalive_expiry :
Pool limit configuration, overwrites value keepalive_expiry from outgoing: for this engine.

Private Engines (tokens)
Administrators might find themselves wanting to limit access to some of the enabled engines on their instances. It might be because they do not want to expose some private information through Offline Engines. Or they would rather share engines only with their trusted friends or colleagues.

info

Initial sponsored by Search and Discovery Fund of NLnet Foundation.

To solve this issue the concept of private engines exists.

A new option was added to engines named tokens. It expects a list of strings. If the user making a request presents one of the tokens of an engine, they can access information about the engine and make search requests.

Example configuration to restrict access to the Arch Linux Wiki engine:

- name: arch linux wiki
  engine: archlinux
  shortcut: al
  tokens: [ 'my-secret-token' ]
Unless a user has configured the right token, the engine is going to be hidden from them. It is not going to be included in the list of engines on the Preferences page and in the output of /config REST API call.

Tokens can be added to one’s configuration on the Preferences page under “Engine tokens”. The input expects a comma separated list of strings.

The distribution of the tokens from the administrator to the users is not carved in stone. As providing access to such engines implies that the admin knows and trusts the user, we do not see necessary to come up with a strict process. Instead, we would like to add guidelines to the documentation of the feature.

Example: Multilingual Search
SearXNG does not support true multilingual search. You have to use the language prefix in your search query when searching in a different language.

But there is a workaround: By adding a new search engine with a different language, SearXNG will search in your default and other language.

Example configuration in settings.yml for a German and English speaker:

search:
    default_lang : "de"
    ...

engines:
  - name : google english
    engine : google
    language : en
    ...
When searching, the default google engine will return German results and “google english” will return English results.
----

brand:
brand:
  issue_url: https://github.com/searxng/searxng/issues
  docs_url: https://docs.searxng.org
  public_instances: https://searx.space
  wiki_url: https://github.com/searxng/searxng/wiki
issue_url :
If you host your own issue tracker change this URL.

docs_url :
If you host your own documentation change this URL.

public_instances :
If you host your own https://searx.space change this URL.

wiki_url :
Link to your wiki (or false)



---------
general:
general:
  debug: false
  instance_name:  "SearXNG"
  privacypolicy_url: false
  donation_url: false
  contact_url: false
  enable_metrics: true
  open_metrics: ''
debug$SEARXNG_DEBUG
Allow a more detailed log if you run SearXNG directly. Display detailed error messages in the browser too, so this must be deactivated in production.

donation_url :
Set value to true to use your own donation page written in the searx/info/en/donate.md and use false to disable the donation link altogether.

privacypolicy_url:
Link to privacy policy.

contact_url:
Contact mailto: address or WEB form.

enable_metrics:
Enabled by default. Record various anonymous metrics available at /stats, /stats/errors and /preferences.

open_metrics:
Disabled by default. Set to a secret password to expose an OpenMetrics API at /metrics, e.g. for usage with Prometheus. The /metrics endpoint is using HTTP Basic Auth, where the password is the value of open_metrics set above. The username used for Basic Auth can be randomly chosen as only the password is being validated.


-----------

search:
search:
  safe_search: 0
  autocomplete: ""
  favicon_resolver: ""
  default_lang: ""
  ban_time_on_fail: 5
  max_page: 0
  max_ban_time_on_fail: 120
  suspended_times:
    SearxEngineAccessDenied: 86400
    SearxEngineCaptcha: 86400
    SearxEngineTooManyRequests: 3600
    cf_SearxEngineCaptcha: 1296000
    cf_SearxEngineAccessDenied: 86400
    recaptcha_SearxEngineCaptcha: 604800
  formats:
    - html
safe_search:
Filter results.

0: None

1: Moderate

2: Strict

autocomplete:
Existing autocomplete backends, leave blank to turn it off.

360search

baidu

brave

dbpedia

duckduckgo

google

mwmbl

quark

qwant

seznam

sogou

stract

swisscows

wikipedia

yandex

favicon_resolver:
To activate favicons in SearXNG’s result list select a default favicon-resolver, leave blank to turn off the feature. Don’t activate the favicons before reading the Favicons documentation.

default_lang:
Default search language - leave blank to detect from browser information or use codes from git://searx/languages.py.

languages:
List of available languages - leave unset to use all codes from git://searx/languages.py. Otherwise list codes of available languages. The all value is shown as the Default language in the user interface (in most cases, it is meant to send the query without a language parameter ; in some cases, it means the English language) Example:

languages:
  - all
  - en
  - en-US
  - de
  - it-IT
  - fr
  - fr-BE
max_page:
If engine supports paging, 0 means unlimited numbers of pages. The value is only applied if the engine itself does not have a max value that is lower than this one.

ban_time_on_fail:
Ban time in seconds after engine errors.

max_ban_time_on_fail:
Max ban time in seconds after engine errors.

suspended_times:
Engine suspension time after error (in seconds; set to 0 to disable)

SearxEngineAccessDenied: 86400
For error “Access denied” and “HTTP error [402, 403]”

SearxEngineCaptcha: 86400
For error “CAPTCHA”

SearxEngineTooManyRequests: 3600
For error “Too many request” and “HTTP error 429”

Cloudflare CAPTCHA:
cf_SearxEngineCaptcha: 1296000

cf_SearxEngineAccessDenied: 86400

Google CAPTCHA:
recaptcha_SearxEngineCaptcha: 604800

formats:
Result formats available from web, remove format to deny access (use lower case).

html

csv

json

rss


----------
server:
server:
    base_url: http://example.org/location  # change this!
    port: 8888
    bind_address: "127.0.0.1"
    secret_key: "ultrasecretkey"           # change this!
    limiter: false
    public_instance: false
    image_proxy: false
    default_http_headers:
      X-Content-Type-Options : nosniff
      X-Download-Options : noopen
      X-Robots-Tag : noindex, nofollow
      Referrer-Policy : no-referrer
base_url$SEARXNG_URL
The base URL where SearXNG is deployed. Used to create correct inbound links.

port & bind_address: $SEARXNG_PORT & $SEARXNG_BIND_ADDRESS
Port number and bind address of the SearXNG web application if you run it directly using python searx/webapp.py. Doesn’t apply to a SearXNG services running behind a proxy and using socket communications.

secret_key$SEARXNG_SECRET
Used for cryptography purpose.

limiter$SEARXNG_LIMITER
Rate limit the number of request on the instance, block some bots. The Limiter requires a redis: database.

public_instance : $SEARXNG_PUBLIC_INSTANCE

Setting that allows to enable features specifically for public instances (not needed for local usage). By set to true the following features are activated:

searx.botdetection.link_token in the Limiter

image_proxy$SEARXNG_IMAGE_PROXY
Allow your instance of SearXNG of being able to proxy images. Uses memory space.

default_http_headers :
Set additional HTTP headers, see #755


-----------
ui:
ui:
  static_use_hash: false
  default_locale: ""
  query_in_title: false
  infinite_scroll: false
  center_alignment: false
  cache_url: https://web.archive.org/web/
  default_theme: simple
  theme_args:
    simple_style: auto
  search_on_category_select: true
  hotkeys: default
  url_formatting: pretty
static_use_hash$SEARXNG_STATIC_USE_HASH
Enables cache busting of static files.

default_locale :
SearXNG interface language. If blank, the locale is detected by using the browser language. If it doesn’t work, or you are deploying a language specific instance of searx, a locale can be defined using an ISO language code, like fr, en, de.

query_in_title :
When true, the result page’s titles contains the query it decreases the privacy, since the browser can records the page titles.

infinite_scroll:
When true, automatically loads the next page when scrolling to bottom of the current page.

center_alignmentdefault false
When enabled, the results are centered instead of being in the left (or RTL) side of the screen. This setting only affects the desktop layout (min-width: @tablet)

cache_urlhttps://web.archive.org/web/
URL prefix of the internet archive or cache, don’t forget trailing slash (if needed). The default is https://web.archive.org/web/ alternatives are:

https://webcache.googleusercontent.com/search?q=cache:

https://archive.today/

default_theme :
Name of the theme you want to use by default on your SearXNG instance.

theme_args.simple_style:
Style of simple theme: auto, light, dark, black

results_on_new_tab:
Open result links in a new tab by default.

search_on_category_select:
Perform search immediately if a category selected. Disable to select multiple categories.

hotkeys:
Hotkeys to use in the search interface: default, vim (Vim-like).

url_formatting:
Formatting type to use for result URLs: pretty, full or host.


-----------------

redis:
A redis DB can be connected by an URL, in searx.redisdb you will find a description to test your redis connection in SearXNG. When using sockets, don’t forget to check the access rights on the socket:

ls -la /usr/local/searxng-redis/run/redis.sock
srwxrwx--- 1 searxng-redis searxng-redis ... /usr/local/searxng-redis/run/redis.sock
In this example read/write access is given to the searxng-redis group. To get access rights to redis instance (the socket), your SearXNG (or even your developer) account needs to be added to the searxng-redis group.

url$SEARXNG_REDIS_URL
URL to connect redis database, see Redis.from_url(url) & Redis DB:

redis://[[username]:[password]]@localhost:6379/0
rediss://[[username]:[password]]@localhost:6379/0
unix://[[username]:[password]]@/path/to/socket.sock?db=0
Redis Developer Notes
To set up a local redis instance, first set the socket path of the Redis DB in your YAML setting:

redis:
  url: unix:///usr/local/searxng-redis/run/redis.sock?db=0
Then use the following commands to install the redis instance (./manage redis.help):

$ ./manage redis.build
$ sudo -H ./manage redis.install
$ sudo -H ./manage redis.addgrp "${USER}"
# don't forget to logout & login to get member of group

-----------------
outgoing:
Communication with search engines.

outgoing:
  request_timeout: 2.0       # default timeout in seconds, can be override by engine
  max_request_timeout: 10.0  # the maximum timeout in seconds
  useragent_suffix: ""       # information like an email address to the administrator
  pool_connections: 100      # Maximum number of allowable connections, or null
                             # for no limits. The default is 100.
  pool_maxsize: 10           # Number of allowable keep-alive connections, or null
                             # to always allow. The default is 10.
  enable_http2: true         # See https://www.python-httpx.org/http2/
  # uncomment below section if you want to use a custom server certificate
  # see https://www.python-httpx.org/advanced/#changing-the-verification-defaults
  # and https://www.python-httpx.org/compatibility/#ssl-configuration
  #  verify: ~/.mitmproxy/mitmproxy-ca-cert.cer
  #
  # uncomment below section if you want to use a proxyq see: SOCKS proxies
  #   https://2.python-requests.org/en/latest/user/advanced/#proxies
  # are also supported: see
  #   https://2.python-requests.org/en/latest/user/advanced/#socks
  #
  #  proxies:
  #    all://:
  #      - http://proxy1:8080
  #      - http://proxy2:8080
  #
  #  using_tor_proxy: true
  #
  # Extra seconds to add in order to account for the time taken by the proxy
  #
  #  extra_proxy_timeout: 10.0
  #
request_timeout :
Global timeout of the requests made to others engines in seconds. A bigger timeout will allow to wait for answers from slow engines, but in consequence will slow SearXNG reactivity (the result page may take the time specified in the timeout to load). Can be override by timeout in the engines:.

useragent_suffix :
Suffix to the user-agent SearXNG uses to send requests to others engines. If an engine wish to block you, a contact info here may be useful to avoid that.

pool_maxsize:
Number of allowable keep-alive connections, or null to always allow. The default is 10. See max_keepalive_connections Pool limit configuration.

pool_connections :
Maximum number of allowable connections, or null # for no limits. The default is 100. See max_connections Pool limit configuration.

keepalive_expiry :
Number of seconds to keep a connection in the pool. By default 5.0 seconds. See keepalive_expiry Pool limit configuration.

proxies :
Define one or more proxies you wish to use, see httpx proxies. If there are more than one proxy for one protocol (http, https), requests to the engines are distributed in a round-robin fashion.

source_ips :
If you use multiple network interfaces, define from which IP the requests must be made. Example:

0.0.0.0 any local IPv4 address.

:: any local IPv6 address.

192.168.0.1

[ 192.168.0.1, 192.168.0.2 ] these two specific IP addresses

fe80::60a2:1691:e5a2:ee1f

fe80::60a2:1691:e5a2:ee1f/126 all IP addresses in this network.

[ 192.168.0.1, fe80::/126 ]

retries :
Number of retry in case of an HTTP error. On each retry, SearXNG uses an different proxy and source ip.

enable_http2 :
Enable by default. Set to false to disable HTTP/2.

verify:$SSL_CERT_FILE, $SSL_CERT_DIR
Allow to specify a path to certificate. see httpx verification defaults.

In addition to verify, SearXNG supports the $SSL_CERT_FILE (for a file) and $SSL_CERT_DIR (for a directory) OpenSSL variables. see httpx ssl configuration.

max_redirects :
30 by default. Maximum redirect before it is an error.

using_tor_proxy :
Using tor proxy (true) or not (false) for all engines. The default is false and can be overwritten in the engines:


------------------
categories_as_tabs:
A list of the categories that are displayed as tabs in the user interface. Categories not listed here can still be searched with the Search syntax.

categories_as_tabs:
  general:
  images:
  videos:
  news:
  map:
  music:
  it:
  science:
  files:
  social media:
Engines are added to categories: (compare categories), the categories listed in categories_as_tabs are shown as tabs in the UI. If there are no active engines in a category, the tab is not displayed (e.g. if a user disables all engines in a category).

On the preferences page (/preferences) – under engines – there is an additional tab, called other. In this tab are all engines listed that are not in one of the UI tabs (not included in categories_as_tabs).


----------------------
plugins:
Attention
The enabled_plugins: section in SearXNG’s settings no longer exists. There is no longer a distinction between built-in and external plugin, all plugins are registered via the settings in the plugins: section.

Further reading ..

List of plugins

Plugin Development

In SearXNG, plugins can be registered in the PluginStore via a fully qualified class name.

A configuration (PluginCfg) can be transferred to the plugin, e.g. to activate it by default / opt-in or opt-out from user’s point of view.

Please note that some plugins, such as the Hostnames plugin, require further configuration before they can be made available for selection.

built-in plugins
The built-in plugins are all located in the namespace searx.plugins.

plugins:

  searx.plugins.calculator.SXNGPlugin:
    active: true

  searx.plugins.hash_plugin.SXNGPlugin:
    active: true

  searx.plugins.self_info.SXNGPlugin:
    active: true

  searx.plugins.tracker_url_remover.SXNGPlugin:
    active: true

  searx.plugins.unit_converter.SXNGPlugin:
    active: true

  searx.plugins.ahmia_filter.SXNGPlugin:
    active: true

  searx.plugins.hostnames.SXNGPlugin:
    active: true

  searx.plugins.oa_doi_rewrite.SXNGPlugin:
    active: false

  searx.plugins.tor_check.SXNGPlugin:
    active: false
external plugins
SearXNG supports external plugins / there is no need to install one, SearXNG runs out of the box.

Only show green hosted results
Only show green hosted results
A SearXNG plugin to check if a domain is part of the Green WEB.

Installation
USE WITH CARE

Its recommended to install (and regularly update) The Green Domains dataset otherwise the plugin uses the API what sends a lot of requests from the SearXNG instance to the Green WEB foundation and slows down the SearXNG instance massively.

Change to the environment in which the plugin is to be installed:

$ sudo utils/searxng.sh instance cmd python -m \
    pip install git+https://github.com/return42/tgwf-searx-plugins
Development
This project is managed by hatch, for development tasks you should install hatch:

$ pipx install hatch
Format and lint your code before commit:

$ hatch run fix
$ hatch run check
To enter the development environment use shell:

$ hatch shell
To get a developer installation in a SearXNG developer environment:

$ git clone git@github.com:return42/tgwf-searx-plugins.git
$ ./manage pyenv.cmd python -m \
      pip install -e tgwf-searx-plugins
To register the plugin in SearXNG add only_show_green_results.SXNGPlugin to the plugins::

plugins:
  # ...
  only_show_green_results.SXNGPlugin:
    active: false
---------------

Docker Container
info

searxng/searxng @dockerhub

git://Dockerfile

Docker overview

Docker Cheat Sheet

Alpine Linux (wiki) apt packages

Alpine’s /bin/sh is dash

If you intend to create a public instance using Docker, use our well maintained docker container

searxng/searxng @dockerhub.

hint

The rest of this article is of interest only to those who want to create and maintain their own Docker images.

The sources are hosted at searxng-docker and the container includes:

a HTTPS reverse proxy [caddy] and

a Redis DB

The default SearXNG setup of this container:

enables limiter to protect against bots

enables image proxy for better privacy

enables cache busting to save bandwidth

Get Docker
If you plan to build and maintain a docker image by yourself, make sure you have Docker installed. On Linux don’t forget to add your user to the docker group (log out and log back in so that your group membership is re-evaluated):

$ sudo usermod -a -G docker $USER
searxng/searxng
docker run

--rm automatically clean up when container exits

-d start detached container

-v mount volume HOST:CONTAINER

The docker image is based on git://Dockerfile and available from searxng/searxng @dockerhub. Using the docker image is quite easy, for instance you can pull the searxng/searxng @dockerhub image and deploy a local instance using docker run:

$ mkdir my-instance
$ cd my-instance
$ export PORT=8080
$ docker pull searxng/searxng
$ docker run --rm \
             -d -p ${PORT}:8080 \
             -v "${PWD}/searxng:/etc/searxng" \
             -e "BASE_URL=http://localhost:$PORT/" \
             -e "INSTANCE_NAME=my-instance" \
             searxng/searxng
2f998.... # container's ID
The environment variables UWSGI_WORKERS and UWSGI_THREADS overwrite the default number of UWSGI processes and UWSGI threads specified in /etc/searxng/uwsgi.ini.

Open your WEB browser and visit the URL:

$ xdg-open "http://localhost:$PORT"
Inside ${PWD}/searxng, you will find settings.yml and uwsgi.ini. You can modify these files according to your needs and restart the Docker image.

$ docker container restart 2f998
Use command container ls to list running containers, add flag -a to list exited containers also. With container stop a running container can be stopped. To get rid of a container use container rm:

$ docker container ls
CONTAINER ID   IMAGE             COMMAND                  CREATED         ...
2f998d725993   searxng/searxng   "/sbin/tini -- /usr/…"   7 minutes ago   ...

$ docker container stop 2f998
$ docker container rm 2f998
Warning

This might remove all docker items, not only those from SearXNG.

If you won’t use docker anymore and want to get rid of all containers & images use the following prune command:

$ docker stop $(docker ps -aq)       # stop all containers
$ docker system prune                # make some housekeeping
$ docker rmi -f $(docker images -q)  # drop all images
shell inside container
Bashism

A tale of two shells: bash or dash

How to make bash scripts work in dash

Checking for Bashisms

Like in many other distributions, Alpine’s /bin/sh is dash. Dash is meant to be POSIX-compliant. Compared to debian, in the Alpine image bash is not installed. The git://dockerfiles/docker-entrypoint.sh script is checked against dash (make tests.shell).

To open a shell inside the container:

$ docker exec -it 2f998 sh
Build the image
It’s also possible to build SearXNG from the embedded git://Dockerfile:

$ git clone https://github.com/searxng/searxng.git
$ cd searxng
$ make docker.build
...
Successfully built 49586c016434
Successfully tagged searxng/searxng:latest
Successfully tagged searxng/searxng:1.0.0-209-9c823800-dirty

$ docker images
REPOSITORY        TAG                        IMAGE ID       CREATED          SIZE
searxng/searxng   1.0.0-209-9c823800-dirty   49586c016434   13 minutes ago   308MB
searxng/searxng   latest                     49586c016434   13 minutes ago   308MB
alpine            3.13                       6dbb9cc54074   3 weeks ago      5.61MB
Command line
docker run

Use flags -it for interactive processes.

In the git://Dockerfile the ENTRYPOINT is defined as git://dockerfiles/docker-entrypoint.sh

docker run --rm -it searxng/searxng -h
Command line:
  -h  Display this help
  -d  Dry run to update the configuration files.
  -f  Always update on the configuration files (existing files are renamed with
      the .old suffix).  Without this option, the new configuration files are
      copied with the .new suffix
Environment variables:
  INSTANCE_NAME settings.yml : general.instance_name
  AUTOCOMPLETE  settings.yml : search.autocomplete
  BASE_URL      settings.yml : server.base_url
  MORTY_URL     settings.yml : result_proxy.url
  MORTY_KEY     settings.yml : result_proxy.key
  BIND_ADDRESS  uwsgi bind to the specified TCP socket using HTTP protocol.
                Default value: [::]:8080
Volume:
  /etc/searxng  the docker entry point copies settings.yml and uwsgi.ini in
                this directory (see the -f command line option)"

---------------
uWSGI
further reading

systemd.unit

uWSGI Emperor

Origin uWSGI

Distributors

Debian’s uWSGI layout

uWSGI maintenance

uWSGI setup

Pitfalls of the Tyrant mode

Origin uWSGI
How uWSGI is implemented by distributors varies. The uWSGI project itself recommends two methods:

systemd.unit template file as described here One service per app in systemd:

There is one systemd unit template on the system installed and one uwsgi ini file per uWSGI-app placed at dedicated locations. Take archlinux and a searxng.ini as example:

systemd template unit: /usr/lib/systemd/system/uwsgi@.service
        contains: [Service]
                  ExecStart=/usr/bin/uwsgi --ini /etc/uwsgi/%I.ini

SearXNG application:   /etc/uwsgi/searxng.ini
        links to: /etc/uwsgi/apps-available/searxng.ini
The SearXNG app (template /etc/uwsgi/%I.ini) can be maintained as known from common systemd units:

$ systemctl enable  uwsgi@searxng
$ systemctl start   uwsgi@searxng
$ systemctl restart uwsgi@searxng
$ systemctl stop    uwsgi@searxng
The uWSGI Emperor which fits for maintaining a large range of uwsgi apps and there is a Tyrant mode to secure multi-user hosting.

The Emperor mode is a special uWSGI instance that will monitor specific events. The Emperor mode (the service) is started by a (common, not template) systemd unit.

The Emperor service will scan specific directories for uwsgi ini files (also know as vassals). If a vassal is added, removed or the timestamp is modified, a corresponding action takes place: a new uWSGI instance is started, reload or stopped. Take Fedora and a searxng.ini as example:

to install & start SearXNG instance create --> /etc/uwsgi.d/searxng.ini
to reload the instance edit timestamp      --> touch /etc/uwsgi.d/searxng.ini
to stop instance remove ini                --> rm /etc/uwsgi.d/searxng.ini
Distributors
The uWSGI Emperor mode and systemd unit template is what the distributors mostly offer their users, even if they differ in the way they implement both modes and their defaults. Another point they might differ in is the packaging of plugins (if so, compare Install packages) and what the default python interpreter is (python2 vs. python3).

While archlinux does not start a uWSGI service by default, Fedora (RHEL) starts a Emperor in Tyrant mode by default (you should have read Pitfalls of the Tyrant mode). Worth to know; debian (ubuntu) follow a complete different approach, read see Debian’s uWSGI layout.

Debian’s uWSGI layout
Be aware, Debian’s uWSGI layout is quite different from the standard uWSGI configuration. Your are familiar with Debian’s Apache layout? .. they do a similar thing for the uWSGI infrastructure. The folders are:

/etc/uwsgi/apps-available/
/etc/uwsgi/apps-enabled/
The uwsgi ini file is enabled by a symbolic link:

ln -s /etc/uwsgi/apps-available/searxng.ini /etc/uwsgi/apps-enabled/
More details can be found in the uwsgi.README.Debian (/usr/share/doc/uwsgi/README.Debian.gz). Some commands you should know on Debian:

Commands recognized by init.d script
====================================

You can issue to init.d script following commands:
  * start        | starts daemon
  * stop         | stops daemon
  * reload       | sends to daemon SIGHUP signal
  * force-reload | sends to daemon SIGTERM signal
  * restart      | issues 'stop', then 'start' commands
  * status       | shows status of daemon instance (running/not running)

'status' command must be issued with exactly one argument: '<confname>'.

Controlling specific instances of uWSGI
=======================================

You could control specific instance(s) by issuing:

    SYSTEMCTL_SKIP_REDIRECT=1 service uwsgi <command> <confname> <confname>...

where:
  * <command> is one of 'start', 'stop' etc.
  * <confname> is the name of configuration file (without extension)

For example, this is how instance for /etc/uwsgi/apps-enabled/hello.xml is
started:

    SYSTEMCTL_SKIP_REDIRECT=1 service uwsgi start hello
uWSGI maintenance
Ubuntu / debianArch LinuxFedora / RHEL
# init.d --> /usr/share/doc/uwsgi/README.Debian.gz
# For uWSGI debian uses the LSB init process, this might be changed
# one day, see https://bugs.debian.org/cgi-bin/bugreport.cgi?bug=833067

create     /etc/uwsgi/apps-available/searxng.ini
enable:    sudo -H ln -s /etc/uwsgi/apps-available/searxng.ini /etc/uwsgi/apps-enabled/
start:     sudo -H service uwsgi start   searxng
restart:   sudo -H service uwsgi restart searxng
stop:      sudo -H service uwsgi stop    searxng
disable:   sudo -H rm /etc/uwsgi/apps-enabled/searxng.ini
uWSGI setup
Create the configuration ini-file according to your distribution and restart the uwsgi application. As shown below, the Installation Script installs by default:

a uWSGI setup that listens on a socket and

enables cache busting.

Ubuntu / debianArch LinuxFedora / RHEL
# -*- mode: conf; coding: utf-8  -*-
[uwsgi]

# uWSGI core
# ----------
#
# https://uwsgi-docs.readthedocs.io/en/latest/Options.html#uwsgi-core

# Who will run the code / Hint: in emperor-tyrant mode uid & gid setting will be
# ignored [1].  Mode emperor-tyrant is the default on fedora (/etc/uwsgi.ini).
#
# [1] https://uwsgi-docs.readthedocs.io/en/latest/Emperor.html#tyrant-mode-secure-multi-user-hosting
#
uid = searxng
gid = searxng

# set (python) default encoding UTF-8
env = LANG=C.UTF-8
env = LANGUAGE=C.UTF-8
env = LC_ALL=C.UTF-8

# chdir to specified directory before apps loading
chdir = /usr/local/searxng/searxng-src/searx

# SearXNG configuration (settings.yml)
env = SEARXNG_SETTINGS_PATH=/etc/searxng/settings.yml

# disable logging for privacy
disable-logging = true

# The right granted on the created socket
chmod-socket = 666

# Plugin to use and interpreter config
single-interpreter = true

# enable master process
master = true

# load apps in each worker instead of the master
lazy-apps = true

# load uWSGI plugins
plugin = python3,http

# By default the Python plugin does not initialize the GIL.  This means your
# app-generated threads will not run.  If you need threads, remember to enable
# them with enable-threads.  Running uWSGI in multithreading mode (with the
# threads options) will automatically enable threading support. This *strange*
# default behaviour is for performance reasons.
enable-threads = true

# Number of workers (usually CPU count)
workers = %k
threads = 4

# plugin: python
# --------------
#
# https://uwsgi-docs.readthedocs.io/en/latest/Options.html#plugin-python

# load a WSGI module
module = searx.webapp

# set PYTHONHOME/virtualenv
virtualenv = /usr/local/searxng/searx-pyenv

# add directory (or glob) to pythonpath
pythonpath = /usr/local/searxng/searxng-src


# speak to upstream
# -----------------

socket = /usr/local/searxng/run/socket
buffer-size = 8192

# uWSGI serves the static files and in settings.yml we use::
#
#   ui:
#     static_use_hash: true
#
static-map = /static=/usr/local/searxng/searxng-src/searx/static
static-gzip-all = True
offload-threads = %k
Pitfalls of the Tyrant mode
The implementation of the process owners and groups in the Tyrant mode is somewhat unusual and requires special consideration. In Tyrant mode mode the Emperor will run the vassal using the UID/GID of the vassal configuration file (user and group of the app .ini file).

Without option emperor-tyrant-initgroups=true in /etc/uwsgi.ini the process won’t get the additional groups, but this option is not available in 2.0.x branch (see #2099@uWSGI) the feature #752@uWSGI has been merged (on Oct. 2014) to the master branch of uWSGI but had never been released; the last major release is from Dec. 2013, since the there had been only bugfix releases (see #2425uWSGI). To shorten up:

In Tyrant mode, there is no way to get additional groups, and the uWSGI process misses additional permissions that may be needed.

For example on Fedora (RHEL): If you try to install a redis DB with socket communication and you want to connect to it from the SearXNG uWSGI, you will see a Permission denied in the log of your instance:

ERROR:searx.redisdb: [searxng (993)] can't connect redis DB ...
ERROR:searx.redisdb:   Error 13 connecting to unix socket: /usr/local/searxng-redis/run/redis.sock. Permission denied.
ERROR:searx.plugins.limiter: init limiter DB failed!!!
Even if your searxng user of the uWSGI process is added to additional groups to give access to the socket from the redis DB:

$ groups searxng
searxng : searxng searxng-redis
To see the effective groups of the uwsgi process, you have to look at the status of the process, by example:

$ ps -aef | grep '/usr/sbin/uwsgi --ini searxng.ini'
searxng       93      92  0 12:43 ?        00:00:00 /usr/sbin/uwsgi --ini searxng.ini
searxng      186      93  0 12:44 ?        00:00:01 /usr/sbin/uwsgi --ini searxng.ini
Here you can see that the additional “Groups” of PID 186 are unset (missing gid of searxng-redis):

$ cat /proc/186/task/186/status
...
Uid:      993     993     993     993
Gid:      993     993     993     993
FDSize:   128
Groups:
...

---------------
NGINX
This section explains how to set up a SearXNG instance using the HTTP server nginx. If you have used the Installation Script and do not have any special preferences you can install the SearXNG site using searxng.sh:

$ sudo -H ./utils/searxng.sh install nginx
If you have special interests or problems with setting up nginx, the following section might give you some guidance.

further reading

nginx

nginx beginners guide

nginx server configuration

Getting Started wiki

uWSGI support from nginx

The nginx HTTP server

NGINX’s SearXNG site

Disable logs

The nginx HTTP server
If nginx is not installed, install it now.

Ubuntu / debianArch LinuxFedora / RHEL
sudo -H apt-get install nginx
Now at http://localhost you should see a Welcome to nginx! page, on Fedora you see a Fedora Webserver - Test Page. The test page comes from the default nginx server configuration. How this default site is configured, depends on the linux distribution:

Ubuntu / debianArch LinuxFedora / RHEL
less /etc/nginx/nginx.conf
There is one line that includes site configurations from:

include /etc/nginx/sites-enabled/*;
NGINX’s SearXNG site
Now you have to create a configuration file (searxng.conf) for the SearXNG site. If nginx is new to you, the nginx beginners guide is a good starting point and the Getting Started wiki is always a good resource to keep in the pocket.

Depending on what your SearXNG installation is listening on, you need a http or socket communication to upstream.

sockethttp
location /searxng {

    uwsgi_pass unix:///usr/local/searxng/run/socket;

    include uwsgi_params;

    uwsgi_param    HTTP_HOST             $host;
    uwsgi_param    HTTP_CONNECTION       $http_connection;

    # see flaskfix.py
    uwsgi_param    HTTP_X_SCHEME         $scheme;
    uwsgi_param    HTTP_X_SCRIPT_NAME    /searxng;

    # see limiter.py
    uwsgi_param    HTTP_X_REAL_IP        $remote_addr;
    uwsgi_param    HTTP_X_FORWARDED_FOR  $proxy_add_x_forwarded_for;
}

# uWSGI serves the static files and in settings.yml we use::
#
#   ui:
#     static_use_hash: true
#
# location /searxng/static/ {
#     alias /usr/local/searxng/searxng-src/searx/static/;
# }
The Installation Script installs the reference setup and a uWSGI setup that listens on a socket by default.

Ubuntu / debianArch LinuxFedora / RHEL
Create configuration at /etc/nginx/sites-available/ and place a symlink to sites-enabled:

sudo -H ln -s /etc/nginx/sites-available/searxng.conf \
              /etc/nginx/sites-enabled/searxng.conf
Restart services:

Ubuntu / debianArch LinuxFedora / RHEL
sudo -H systemctl restart nginx
sudo -H service uwsgi restart searxng
Disable logs
For better privacy you can disable nginx logs in /etc/nginx/nginx.conf.

http {
    # ...
    access_log /dev/null;
    error_log  /dev/null;
    # ...
}

------------------
Apache
This section explains how to set up a SearXNG instance using the HTTP server Apache. If you did use the Installation Script and do not have any special preferences you can install the SearXNG site using searxng.sh:

$ sudo -H ./utils/searxng.sh install apache
If you have special interests or problems with setting up Apache, the following section might give you some guidance.

further read

Apache Arch Linux

Apache Debian

apache2.README.Debian

Apache Fedora

Apache directives

The Apache HTTP server

Debian’s Apache layout

Apache modules

Apache sites

Apache’s SearXNG site

disable logs

The Apache HTTP server
If Apache is not installed, install it now. If apache is new to you, the Getting Started, Configuration Files and Terms Used to Describe Directives documentation gives first orientation. There is also a list of Apache directives to keep in the pocket.

Ubuntu / debianArch LinuxFedora / RHEL
sudo -H apt-get install apache2
Now at http://localhost you should see some kind of Welcome or Test page. How this default site is configured, depends on the linux distribution (compare Apache directives).

Ubuntu / debianArch LinuxFedora / RHEL
less /etc/apache2/sites-enabled/000-default.conf
In this file, there is a line setting the DocumentRoot directive:

DocumentRoot /var/www/html
And the welcome page is the HTML file at /var/www/html/index.html.

Debian’s Apache layout
Be aware, Debian’s Apache layout is quite different from the standard Apache configuration. For details look at the apache2.README.Debian (/usr/share/doc/apache2/README.Debian.gz). Some commands you should know on Debian:

apache2ctl: Apache HTTP server control interface

a2enmod, a2dismod: switch on/off modules

a2enconf, a2disconf: switch on/off configurations

a2ensite, a2dissite: switch on/off sites

Apache modules
To load additional modules, in most distributions you have to uncomment the lines with the corresponding LoadModule directive, except in Debian’s Apache layout.

Ubuntu / debianArch LinuxFedora / RHEL
Debian’s Apache layout uses a2enmod and a2dismod to activate or disable modules:

sudo -H a2enmod ssl
sudo -H a2enmod headers
sudo -H a2enmod proxy
sudo -H a2enmod proxy_http
sudo -H a2enmod proxy_uwsgi
Apache sites
Ubuntu / debianArch LinuxFedora / RHEL
In Debian’s Apache layout you create a searxng.conf with the <Location /searxng > directive and save this file in the sites available folder at /etc/apache2/sites-available. To enable the searxng.conf use a2ensite:

sudo -H a2ensite searxng.conf
Apache’s SearXNG site
uWSGI

Use mod_proxy_uwsgi / don’t use the old mod_uwsgi anymore.

To proxy the incoming requests to the SearXNG instance Apache needs the mod_proxy module (Apache modules).

HTTP headers

With ProxyPreserveHost the incoming Host header is passed to the proxied host.

Depending on what your SearXNG installation is listening on, you need a http mod_proxy_http) or socket (mod_proxy_uwsgi) communication to upstream.

The Installation Script installs the reference setup and a uWSGI setup that listens on a socket by default. You can install and activate your own searxng.conf like shown in Apache sites.

sockethttp
# -*- coding: utf-8; mode: apache -*-

LoadModule ssl_module           /mod_ssl.so
LoadModule headers_module       /mod_headers.so
LoadModule proxy_module         /mod_proxy.so
LoadModule proxy_uwsgi_module   /mod_proxy_uwsgi.so
# LoadModule setenvif_module      /mod_setenvif.so
#
# SetEnvIf Request_URI /searxng dontlog
# CustomLog /dev/null combined env=dontlog

<Location /searxng>

    Require all granted
    Order deny,allow
    Deny from all
    # Allow from fd00::/8 192.168.0.0/16 fe80::/10 127.0.0.0/8 ::1
    Allow from all

    # add the trailing slash
    RedirectMatch  308 /searxng$ /searxng/

    ProxyPreserveHost On
    ProxyPass unix:/usr/local/searxng/run/socket|uwsgi://uwsgi-uds-searxng/

    # see flaskfix.py
    RequestHeader set X-Scheme %{REQUEST_SCHEME}s
    RequestHeader set X-Script-Name /searxng

    # see limiter.py
    RequestHeader set X-Real-IP %{REMOTE_ADDR}s
    RequestHeader append X-Forwarded-For %{REMOTE_ADDR}s

</Location>

# uWSGI serves the static files and in settings.yml we use::
#
#   ui:
#     static_use_hash: true
#
# Alias /searxng/static/ /usr/local/searxng/searxng-src/searx/static/
Restart service:

Ubuntu / debianArch LinuxFedora / RHEL
sudo -H systemctl restart apache2
sudo -H service uwsgi restart searxng
disable logs
For better privacy you can disable Apache logs. In the examples above activate one of the lines and restart apache:

SetEnvIf Request_URI "/searxng" dontlog
# CustomLog /dev/null combined env=dontlog
The CustomLog directive disables logs for the entire (virtual) server, use it when the URL of the service does not have a path component (/searxng), so when SearXNG is located at root (/).


------------
SearXNG maintenance
further read

DevOps tooling box

uWSGI maintenance

How to update

How to inspect & debug

Migrate and stay tuned!

remove obsolete services

Check after Installation

How to update
How to update depends on the Installation method. If you have used the Installation Script, use the update command from the utils/searxng.sh script.

sudo -H ./utils/searxng.sh instance update
How to inspect & debug
How to debug depends on the Installation method. If you have used the Installation Script, use the inspect command from the utils/searxng.sh script.

sudo -H ./utils/searxng.sh instance inspect
Migrate and stay tuned!
info

PR 1332

PR 456

A comment about rolling release

SearXNG is a rolling release; each commit to the master branch is a release. SearXNG is growing rapidly, the services and opportunities are change every now and then, to name just a few:

Bot protection has been switched from filtron to SearXNG’s limiter, this requires a Redis database.

The image proxy morty is no longer needed, it has been replaced by the image proxy from SearXNG.

To save bandwidth cache busting has been implemented. To get in use, the static-expires needs to be set in the uWSGI setup.

To stay tuned and get in use of the new features, instance maintainers have to update the SearXNG code regularly (see How to update). As the above examples show, this is not always enough, sometimes services have to be set up or reconfigured and sometimes services that are no longer needed should be uninstalled.

Hint
First of all: SearXNG is installed by the script utils/searxng.sh. If you have old filtron, morty or searx setup you should consider complete uninstall/reinstall.

Here you will find a list of changes that affect the infrastructure. Please check to what extent it is necessary to update your installations:

PR 1595: [fix] uWSGI: increase buffer-size
Re-install uWSGI (utils/searxng.sh) or fix your uWSGI searxng.ini file manually.

remove obsolete services
If your searx instance was installed “Step by step” or by the “Installation scripts”, you need to undo the installation procedure completely. If you have morty & filtron installed, it is recommended to uninstall these services also. In case of scripts, to uninstall use the scripts from the origin you installed searx from or try:

$ sudo -H ./utils/filtron.sh remove all
$ sudo -H ./utils/morty.sh   remove all
$ sudo -H ./utils/searx.sh   remove all
Hint
If you are migrate from searx take into account that the .config.sh is no longer used.

If you upgrade from searx or from before PR 1332 has been merged and you have filtron and/or morty installed, don’t forget to remove HTTP sites.

Apache:

$ sudo -H ./utils/filtron.sh apache remove
$ sudo -H ./utils/morty.sh apache remove
nginx:

$ sudo -H ./utils/filtron.sh nginx remove
$ sudo -H ./utils/morty.sh nginx remove
Check after Installation
Once you have done your installation, you can run a SearXNG check procedure, to see if there are some left overs. In this example there exists a old /etc/searx/settings.yml:

$ sudo -H ./utils/searxng.sh instance check

SearXNG checks
--------------
ERROR: settings.yml in /etc/searx/ is deprecated, move file to folder /etc/searxng/
INFO:  [OK] (old) account 'searx' does not exists
INFO:  [OK] (old) account 'filtron' does not exists
INFO:  [OK] (old) account 'morty' does not exists
...
INFO    searx.redisdb                 : connecting to Redis db=0 path='/usr/local/searxng-redis/run/redis.sock'
INFO    searx.redisdb                 : connected to Redis

-------------------
Answer CAPTCHA from server’s IP
With a SSH tunnel we can send requests from server’s IP and solve a CAPTCHA that blocks requests from this IP. If your SearXNG instance is hosted at example.org and your login is user you can setup a proxy simply by ssh:

# SOCKS server: socks://127.0.0.1:8080

$ ssh -q -N -D 8080 user@example.org
The socks://localhost:8080 from above can be tested by:

server’s IPdesktop’s IP
$ curl -x socks://127.0.0.1:8080 http://ipecho.net/plain
n.n.n.n
In the settings of the WEB browser open the “Network Settings” and setup a proxy on SOCKS5 127.0.0.1:8080 (see screenshot below). In the WEB browser check the IP from the server is used:

http://ipecho.net/plain

Now open the search engine that blocks requests from your server’s IP. If you have issues with the qwant engine, solve the CAPTCHA from qwant.com.

Firefox
FFox proxy on SOCKS5, 127.0.0.1:8080
Fig. 1 Firefox’s network settings

ssh manual:
-D [bind_address:]port
Specifies a local “dynamic” application-level port forwarding. This works by allocating a socket to listen to port on the local side .. Whenever a connection is made to this port, the connection is forwarded over the secure channel, and the application protocol is then used to determine where to connect to from the remote machine .. ssh will act as a SOCKS server ..

-N
Do not execute a remote command. This is useful for just forwarding ports.

------------
Favicons
warning

Don’t activate the favicons before reading the documentation.

Infrastructure

Setting up the cache

Maintenance of the cache

Proxy configuration

Register resolvers

Activating the favicons in SearXNG is very easy, but this generates a significantly higher load in the client/server communication and increases resources needed on the server.

To mitigate these disadvantages, various methods have been implemented, including a cache. The cache must be parameterized according to your own requirements and maintained regularly.

To activate favicons in SearXNG’s result list, set a default favicon_resolver in the search settings:

search:
  favicon_resolver: "duckduckgo"
By default and without any extensions, SearXNG serves these resolvers:

duckduckgo

allesedv

google

yandex

With the above setting favicons are displayed, the user has the option to deactivate this feature in his settings. If the user is to have the option of selecting from several resolvers, a further setting is required / but this setting will be discussed later in this article, first we have to setup the favicons cache.

Infrastructure
The infrastructure for providing the favicons essentially consists of three parts:

Favicons-Proxy (aka proxy)

Favicons-Resolvers (aka resolver)

Favicons-Cache (aka cache)

To protect the privacy of users, the favicons are provided via a proxy. This proxy is automatically activated with the above activation of a resolver. Additional requests are required to provide the favicons: firstly, the proxy must process the incoming requests and secondly, the resolver must make outgoing requests to obtain the favicons from external sources.

A cache has been developed to massively reduce both, incoming and outgoing requests. This cache is also activated automatically with the above activation of a resolver. In its defaults, however, the cache is minimal and not well suitable for a production environment!

Setting up the cache
To parameterize the cache and more settings of the favicons infrastructure, a TOML configuration is created in the file /etc/searxng/favicons.toml.

[favicons]

cfg_schema = 1   # config's schema version no.

[favicons.cache]

db_url = "/var/cache/searxng/faviconcache.db"  # default: "/tmp/faviconcache.db"
LIMIT_TOTAL_BYTES = 2147483648                 # 2 GB / default: 50 MB
# HOLD_TIME = 5184000                            # 60 days / default: 30 days
# BLOB_MAX_BYTES = 40960                         # 40 KB / default 20 KB
# MAINTENANCE_MODE = "off"                       # default: "auto"
# MAINTENANCE_PERIOD = 600                       # 10min / default: 1h
cfg_schema:
Is required to trigger any processes required for future upgrades / don’t change it.

cache.db_url:
The path to the (SQLite) database file. The default path is in the /tmp folder, which is deleted on every reboot and is therefore unsuitable for a production environment. The FHS provides the folder /var/cache for the cache of applications, so a suitable storage location of SearXNG’s caches is folder /var/cache/searxng.

In a standard installation (compare Create user), the folder must be created and the user under which the SearXNG process is running must be given write permission to this folder.

$ sudo mkdir /var/cache/searxng
$ sudo chown root:searxng /var/cache/searxng/
$ sudo chmod g+w /var/cache/searxng/
In container systems, a volume should be mounted for this folder. Check whether the process in the container has read/write access to the mounted folder.

cache.LIMIT_TOTAL_BYTES:
Maximum of bytes stored in the cache of all blobs. The limit is only reached at each maintenance interval after which the oldest BLOBs are deleted; the limit is exceeded during the maintenance period.

Attention
If the maintenance period is too long or maintenance is switched off completely, the cache grows uncontrollably.

SearXNG hosters can change other parameters of the cache as required:

cache.HOLD_TIME

cache.BLOB_MAX_BYTES

Maintenance of the cache
Regular maintenance of the cache is required! By default, regular maintenance is triggered automatically as part of the client requests:

cache.MAINTENANCE_MODE (default auto)

cache.MAINTENANCE_PERIOD (default 6000 / 1h)

As an alternative to maintenance as part of the client request process, it is also possible to carry out maintenance using an external process. For example, by creating a crontab entry for maintenance:

$ python -m searx.favicons cache maintenance
The following command can be used to display the state of the cache:

$ python -m searx.favicons cache state
Proxy configuration
Most of the options of the Favicons-Proxy are already set sensibly with settings from the settings.yml and should not normally be adjusted.

[favicons.proxy]

max_age = 5184000             # 60 days / default: 7 days (604800 sec)
max_age:
The HTTP Cache-Control max-age response directive indicates that the response remains fresh until N seconds after the response is generated. This setting therefore determines how long a favicon remains in the client’s cache. As a rule, in the favicons infrastructure of SearXNG’s this setting only affects favicons whose byte size exceeds BLOB_MAX_BYTES (the other favicons that are already in the cache are embedded as data URL in the generated HTML, which can greatly reduce the number of additional requests).

Register resolvers
A resolver is a function that obtains the favicon from an external source. The resolver functions available to the user are registered with their fully qualified name (FQN) in a resolver_map.

If no resolver_map is defined in the favicon.toml, the favicon infrastructure of SearXNG generates this resolver_map automatically depending on the settings.yml. SearXNG would automatically generate the following TOML configuration from the following YAML configuration:

search:
  favicon_resolver: "duckduckgo"
[favicons.proxy.resolver_map]

"duckduckgo" = "searx.favicons.resolvers.duckduckgo"
If this automatism is not desired, then (and only then) a separate resolver_map must be created. For example, to give the user two resolvers to choose from, the following configuration could be used:

[favicons.proxy.resolver_map]

"duckduckgo" = "searx.favicons.resolvers.duckduckgo"
"allesedv" = "searx.favicons.resolvers.allesedv"
# "google" = "searx.favicons.resolvers.google"
# "yandex" = "searx.favicons.resolvers.yandex"
Note
With each resolver, the resource requirement increases significantly.

The number of resolvers increases:

the number of incoming/outgoing requests and

the number of favicons to be stored in the cache.

In the following we list the resolvers available in the core of SearXNG, but via the FQN it is also possible to implement your own resolvers and integrate them into the proxy:

searx.favicons.resolvers.duckduckgo

searx.favicons.resolvers.allesedv

searx.favicons.resolvers.google

searx.favicons.resolvers.yandex


------------
Limiter
info

The limiter requires a Redis database.

Enable Limiter

Configure Limiter

limiter.toml

Implementation

Bot protection / IP rate limitation. The intention of rate limitation is to limit suspicious requests from an IP. The motivation behind this is the fact that SearXNG passes through requests from bots and is thus classified as a bot itself. As a result, the SearXNG engine then receives a CAPTCHA or is blocked by the search engine (the origin) in some other way.

To avoid blocking, the requests from bots to SearXNG must also be blocked, this is the task of the limiter. To perform this task, the limiter uses the methods from the Bot Detection:

Analysis of the HTTP header in the request / Probe HTTP headers can be easily bypassed.

Block and pass lists in which IPs are listed / IP lists are hard to maintain, since the IPs of bots are not all known and change over the time.

Detection & dynamically Rate limit of bots based on the behavior of the requests. For dynamically changeable IP lists a Redis database is needed.

The prerequisite for IP based methods is the correct determination of the IP of the client. The IP of the client is determined via the X-Forwarded-For HTTP header.

Attention
A correct setup of the HTTP request headers X-Forwarded-For and X-Real-IP is essential to be able to assign a request to an IP correctly:

NGINX RequestHeader

Apache RequestHeader

Enable Limiter
To enable the limiter activate:

server:
  ...
  limiter: true  # rate limit the number of request on the instance, block some bots
and set the redis-url connection. Check the value, it depends on your redis DB (see redis:), by example:

redis:
  url: unix:///usr/local/searxng-redis/run/redis.sock?db=0
Configure Limiter
The methods of Bot Detection the limiter uses are configured in a local file /etc/searxng/limiter.toml. The defaults are shown in limiter.toml / Don’t copy all values to your local configuration, just enable what you need by overwriting the defaults. For instance to activate the link_token method in the Method ip_limit you only need to set this option to true:

[botdetection.ip_limit]
link_token = true
limiter.toml
In this file the limiter finds the configuration of the Bot Detection:

IP lists

Rate limit

Probe HTTP headers

[real_ip]

# Number of values to trust for X-Forwarded-For.

x_for = 1

# The prefix defines the number of leading bits in an address that are compared
# to determine whether or not an address is part of a (client) network.

ipv4_prefix = 32
ipv6_prefix = 48

[botdetection.ip_limit]

# To get unlimited access in a local network, by default link-local addresses
# (networks) are not monitored by the ip_limit
filter_link_local = false

# activate link_token method in the ip_limit method
link_token = false

[botdetection.ip_lists]

# In the limiter, the ip_lists method has priority over all other methods -> if
# an IP is in the pass_ip list, it has unrestricted access and it is also not
# checked if e.g. the "user agent" suggests a bot (e.g. curl).

block_ip = [
  # '93.184.216.34',  # IPv4 of example.org
  # '257.1.1.1',      # invalid IP --> will be ignored, logged in ERROR class
]

pass_ip = [
  # '192.168.0.0/16',      # IPv4 private network
  # 'fe80::/10'            # IPv6 linklocal / wins over botdetection.ip_limit.filter_link_local
]

# Activate passlist of (hardcoded) IPs from the SearXNG organization,
# e.g. `check.searx.space`.
pass_searxng_org = true
Implementation
searx.limiter.LIMITER_CFG_SCHEMA = PosixPath('/home/runner/work/searxng/searxng/searx/limiter.toml')
Base configuration (schema) of the botdetection.

searx.limiter.pre_request()[source]
See flask.Flask.before_request

searx.limiter.is_installed()[source]
Returns True if limiter is active and a redis DB is available.

searx.limiter.initialize(app: Flask, settings)[source]
Install the limiter

-------------------------
Administration API
Get configuration data
GET /config  HTTP/1.1
Sample response
{
  "autocomplete": "",
  "categories": [
    "map",
    "it",
    "images",
  ],
  "default_locale": "",
  "default_theme": "simple",
  "engines": [
    {
      "categories": [
        "map"
      ],
      "enabled": true,
      "name": "openstreetmap",
      "shortcut": "osm"
    },
    {
      "categories": [
        "it"
      ],
      "enabled": true,
      "name": "arch linux wiki",
      "shortcut": "al"
    },
    {
      "categories": [
        "images"
      ],
      "enabled": true,
      "name": "google images",
      "shortcut": "goi"
    },
    {
      "categories": [
        "it"
      ],
      "enabled": false,
      "name": "bitbucket",
      "shortcut": "bb"
    },
  ],
  "instance_name": "searx",
  "locales": {
    "de": "Deutsch (German)",
    "en": "English",
    "eo": "Esperanto (Esperanto)",
  },
  "plugins": [
    {
      "enabled": true,
      "name": "HTTPS rewrite"
    }
  ],
  "safe_search": 0
}
Embed search bar
The search bar can be embedded into websites. Just paste the example into the HTML of the site. URL of the SearXNG instance and values are customizable.

<form method="post" action="https://example.org/">
  <!-- search      --> <input type="text" name="q">
  <!-- categories  --> <input type="hidden" name="categories" value="general,social media">
  <!-- language    --> <input type="hidden" name="lang" value="all">
  <!-- locale      --> <input type="hidden" name="locale" value="en">
  <!-- date filter --> <input type="hidden" name="time_range" value="month">
</form>

----------------------
Architecture
Further reading

Reverse Proxy: Apache & nginx

uWSGI: uWSGI

SearXNG: Step by step installation

Herein you will find some hints and suggestions about typical architectures of SearXNG infrastructures.

uWSGI Setup
We start with a reference setup for public SearXNG instances which can be build up and maintained by the scripts from our DevOps tooling box.

arch_public.dot
Fig. 2 Reference architecture of a public SearXNG setup.

The reference installation activates server.limiter, server.image_proxy and ui.static_use_hash (/etc/searxng/settings.yml)

# SearXNG settings

use_default_settings: true

general:
  debug: false
  instance_name: "SearXNG"

search:
  safe_search: 2
  autocomplete: 'duckduckgo'

server:
  # Is overwritten by ${SEARXNG_SECRET}
  secret_key: "ultrasecretkey"
  limiter: true
  image_proxy: true
  # public URL of the instance, to ensure correct inbound links. Is overwritten
  # by ${SEARXNG_URL}.
  # base_url: http://example.com/location

redis:
  # URL to connect redis database. Is overwritten by ${SEARXNG_REDIS_URL}.
  url: unix:///usr/local/searxng-redis/run/redis.sock?db=0

ui:
  static_use_hash: true

------------
List of plugins
Further reading ..

SearXNG settings

Plugin Development

Table 1 Plugins configured at built time (defaults)
Name

Active

Description

Basic Calculator

yes

Calculate mathematical expressions via the search bar

Self Information

yes

Displays your IP if the query is “ip” and your user agent if the query is “user-agent”.

Unit converter plugin

yes

Convert between units

Open Access DOI rewrite

no

Avoid paywalls by redirecting to open-access versions of publications when available

Tracker URL remover

no

Remove trackers arguments from the returned URL

Hash plugin

yes

Converts strings to different hash digests.

Tor check plugin

no

This plugin checks if the address of the request is a Tor exit-node, and informs the user if it is; like check.torproject.org, but from SearXNG.
----------------
Buildhosts
Build and Development tools

Build docs

Lint shell scripts

To get best results from build, it’s recommend to install additional packages on build hosts (see utils/searxng.sh).

Build and Development tools
To Install tools used by build and development tasks in once:

SearXNG’s development tools
$ sudo -H ./utils/searxng.sh install buildhost
This will install packages needed by SearXNG:

Ubuntu / debianArch LinuxFedora / RHEL
$ sudo -H apt-get install -y \
    python3-dev python3-babel python3-venv \
    uwsgi uwsgi-plugin-python3 \
    git build-essential libxslt-dev zlib1g-dev libffi-dev libssl-dev
and packages needed to build documentation and run tests:

Ubuntu / debianArch LinuxFedora / RHEL
$ sudo -H apt-get install -y \
    graphviz imagemagick texlive-xetex librsvg2-bin \
    texlive-latex-recommended texlive-extra-utils fonts-dejavu \
    latexmk shellcheck
Build docs
Sphinx build needs

ImageMagick

Graphviz

XeTeX

dvisvgm

Most of the sphinx requirements are installed from git://setup.py and the docs can be build from scratch with make docs.html. For better math and image processing additional packages are needed. The XeTeX needed not only for PDF creation, it’s also needed for Math equations when HTML output is build.

To be able to do Math support for HTML outputs in Sphinx without CDNs, the math are rendered as images (sphinx.ext.imgmath extension).

Here is the extract from the git://docs/conf.py file, setting math renderer to imgmath:

html_math_renderer = 'imgmath'
imgmath_image_format = 'svg'
imgmath_font_size = 14
If your docs build (make docs.html) shows warnings like this:

WARNING: dot(1) not found, for better output quality install \
         graphviz from https://www.graphviz.org
..
WARNING: LaTeX command 'latex' cannot be run (needed for math \
         display), check the imgmath_latex setting
you need to install additional packages on your build host, to get better HTML output (install buildhost).

Ubuntu / debianArch LinuxFedora / RHEL
$ sudo apt install graphviz imagemagick texlive-xetex librsvg2-bin
For PDF output you also need:

Ubuntu / debianArch LinuxFedora / RHEL
$ sudo apt texlive-latex-recommended texlive-extra-utils ttf-dejavu
Lint shell scripts
To lint shell scripts we use ShellCheck - a shell script static analysis tool (install buildhost).

Ubuntu / debianArch LinuxFedora / RHEL
$ sudo apt install shellcheck

----------------
=================
Welcome to SearXNG
Search without being tracked.

SearXNG is a free internet metasearch engine which aggregates results from up to 238 search services. Users are neither tracked nor profiled. Additionally, SearXNG can be used over Tor for online anonymity.

Get started with SearXNG by using one of the instances listed at searx.space. If you don’t trust anyone, you can set up your own, see Installation.

features

self hosted

no user tracking / no profiling

script & cookies are optional

secure, encrypted connections

238 search engines

58 translations

about 70 well maintained instances on searx.space

easy integration of search engines

professional development: CI, quality assurance & automated tested UI

be a part

SearXNG is driven by an open community, come join us! Don’t hesitate, no need to be an expert, everyone can contribute:

help to improve translations

discuss with the community

report bugs & suggestions

…

the origin

SearXNG development has been started in the middle of 2021 as a fork of the searx project.

User information
Search syntax
Configured Engines
About SearXNG
Why use a private instance?
How does SearXNG protect privacy?
Conclusion
Administrator documentation
Settings
Installation
Docker Container
Installation Script
Step by step installation
uWSGI
NGINX
Apache
SearXNG maintenance
Answer CAPTCHA from server’s IP
Favicons
Limiter
Administration API
Architecture
List of plugins
Buildhosts
Developer documentation
Development Quickstart
Runtime Management
How to contribute
Extended Types
Engine Implementations
Result Types
Simple Theme Templates
Search API
Plugins
Answerers
Translation
Developing in Linux Containers
Makefile & ./manage
reST primer
Tooling box searxng_extra
DevOps tooling box
utils/searxng.sh
utils/lxc.sh
Common command environments
Source-Code
Custom message extractor (i18n)
Bot Detection
SearXNG Exceptions
Favicons (source)
Online /info
Locales
Redis DB
Redis Library
Search
Search processors
Settings Loader
SQLite DB
Utility functions for the engines
====================
