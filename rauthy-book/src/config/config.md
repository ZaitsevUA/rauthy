# Reference Config

This shows a full example config with (hopefully) every value nicely described.

```admonish caution
When you go into production, make sure that you provide the included secrets / sensistive information in this
file in an appropriate way. With docker, you can leave them inside this file, but when deploying with Kubernetes,
extract these values, create Kubernetes Secrets and provide them as environment variables.
```

```
#####################################
############## ACCESS ###############
#####################################

# If the User Registration endpoint should be accessible by anyone. If not, an admin
# must create each new user. (default: false)
#OPEN_USER_REG=true

# Can be used when 'OPEN_USER_REG=true' to restrict the domains for a registration.
# For instance, set it to 'USER_REG_DOMAIN_RESTRICTION=gmail.com' to allow only
# registrations with 'user@gmail.com' (default: '')
#USER_REG_DOMAIN_RESTRICTION=some-domain.com

#####################################
############# BACKUPS ###############
#####################################

# Cron job for automatic data store backups (default: "0 0 4 * * * *")
# sec min hour day_of_month month day_of_week year
#BACKUP_TASK="0 0 4 * * * *"

# The name for the data store backups. The current timestamp will always be appended
# automatically. (default: rauthy-backup-)
#BACKUP_NAME="rauthy-backup-"

# All backups older than the specified hours will be cleaned up automatically
# (default: 720)
#BACKUP_RETENTION_LOCAL=720

#####################################
############## CACHE ################
#####################################

# If the cache should start in HA mode or standalone
# accepts 'true|false', defaults to 'false'
#HA_MODE=false

# The connection strings (with hostnames) of the HA instances as a CSV
# Format: 'scheme://hostname:port'
#HA_HOSTS="http://rauthy-0.rauthy:8000, http://rauthy-1.rauthy:8000, http://rauthy-2.rauthy:8000"

# Overwrite the hostname which is used to identify each cache member.
# Useful in scenarios, where for instance all members are on the same host with
# different ports or for testing.
#HOSTNAME_OVERWRITE="rauthy-0.rauthy:8080"

## Define buffer sizes for channels between the components
# Buffer for client requests on the incoming stream - server side (default: 128)
# Make sense to have the CACHE_BUF_SERVER set to:
# `(number of total HA cache hosts - 1) * CACHE_BUF_CLIENT`
# In a non-HA deployment, set the same size for both
#CACHE_BUF_SERVER=128
# Buffer for client requests to remote servers for all cache operations
# (default: 128)
#CACHE_BUF_CLIENT=128

# Secret token, which is used to authenticate the cache members
#CACHE_AUTH_TOKEN=SomeSuperSecretAndVerySafeToken1337

## Connections Timeouts

# The Server sends out keepalive pings with configured timeouts
# The keepalive ping interval in seconds (default: 5)
#CACHE_KEEPALIVE_INTERVAL=5
# The keepalive ping timeout in seconds (default: 5)
#CACHE_KEEPALIVE_TIMEOUT=5

# The timeout for the leader election. If a newly saved leader request has
# not reached quorum after the timeout, the leader will be reset and a new
# request will be sent out.
# CAUTION: This should not be lower than CACHE_RECONNECT_TIMEOUT_UPPER,
# since cold starts and elections will be problematic in that case.
# value in seconds, default: 15
#CACHE_ELECTION_TIMEOUT=15

# These 2 values define the reconnect timeout for the HA Cache Clients.
# The values are in ms and a random between these 2 will be chosen each
# time to avoid conflicts and race conditions
# (default: 2500)
#CACHE_RECONNECT_TIMEOUT_LOWER=2500
# (default: 5000)
#CACHE_RECONNECT_TIMEOUT_UPPER=5000

#####################################
############ DATABASE ###############
#####################################

# The database driver will be chosen at runtime depending on the given
# DATABASE_URL format. Examples:
# Sqlite: 'sqlite:data/rauthy.db' or 'sqlite::memory:'
# Postgres: 'postgresql://User:PasswordWithoutSpecialCharacters@localhost:5432/DatabaseName'
#
# NOTE: The password in this case should be alphanumeric. Special
# characters could cause problems in the connection string.
#
# CAUTION: To make the automatic migrations work with Postgres15, when
# you do not want to just use the `postgres` user, You need to have a
# user with the same name as the DB / schema. For instance, the
# following would work without granting extra access to the `public`
# schema which is disabled by default since PG15:
# database: rauthy
# user: rauthy
# schema: rauthy with owner rauthy
#
#DATABASE_URL=sqlite::memory:
#DATABASE_URL=sqlite:data/rauthy.db
#DATABASE_URL=postgresql://rauthy:123SuperSafe@localhost:5432/rauthy

# Max DB connections - irrelevant for SQLite (default: 5)
#DATABASE_MAX_CONN=5

# If specified, the current Database, set with DATABASE_URL, will be
# DELETED and OVERWRITTEN with a migration from the given database
# with this variable. Can be used to migrate between different databases.
# !!! USE WITH CARE !!!
#MIGRATE_DB_FROM=sqlite:data/rauthy.db

# Disables the housekeeping schedulers (default: false)
#SCHED_DISABLE=true

#####################################
############# E-MAIL ################
#####################################

SMTP_USERNAME=
#SMTP_PASSWORD=
SMTP_URL=
# Format: "Rauthy <rauthy@localhost.de>"
SMTP_FROM=

#####################################
###### Encryption / Hashing #########
#####################################

# Format: "key_id/enc_key another_key_id/another_enc_key" - the
# enc_key itself must be exactly 32 characters long and and
# should not contain special characters.
# The ID must match '[a-zA-Z0-9]{2,20}'
#ENC_KEYS="bVCyTsGaggVy5yqQ/S9n7oCen53xSJLzcsmfdnBDvNrqQ63r4 q6u26onRvXVG4427/3CEC8RJWBcMkrBMkRXgx65AmJsNTghSA"
ENC_KEY_ACTIVE=bVCyTsGaggVy5yqQ

# M_COST should never be below 32768 in production
ARGON2_M_COST=32768
# T_COST should never be below 1 in production
ARGON2_T_COST=3
# P_COST should never be below 2 in production
ARGON2_P_COST=2

# Limits the maximum amount of parallel password hashes at
# the exact same time to never exceed system memory while
# still allowing a good amount of memory for the argon2id
# algorithm (default: 2)
# CAUTION: You must make sure, that you have at least
# (MAX_HASH_THREADS * ARGON2_M_COST / 1024) + 30 MB of memory
# available.
MAX_HASH_THREADS=1

# The time in ms when to log a warning, if a request waited
# longer than this time. This is an indicator, that you have
# more concurrent logins than allowed and may need config adjustments,
# if this happens more often. (default: 500)
#HASH_AWAIT_WARN_TIME=500

#####################################
####### LIFETIMES / TIMEOUTS ########
#####################################

# Set the grace time in seconds for how long in seconds the
# refresh token should still be valid after usage. Keep this
# value small, but do not set it to 0 with an HA deployment
# to not get issues with small HA cache latencies.
#
# If you have an external client, which does concurrent
# requests, from which the request interceptor wants to refresh
# the token, you may have multiple hits on the endpoint and all
# of them should be valid.
#
# Caching is done on the endpoint itself, but grace time of 0
# will only be good for a single instance of rauthy.
# default: 5
#REFRESH_TOKEN_GRACE_TIME=5

# Lifetime for offline tokens in hours (default: 720)
#OFFLINE_TOKEN_LIFETIME=720

# Session lifetime in seconds - the session can not be
# extended beyond this time and a new login will be forced.
# This is the session for the authorization code flow. (default: 14400)
#SESSION_LIFETIME=14400

# If 'true', a 2FA / MFA check will be done with each automatic
# token generation, even with an active session, which kind of
# makes the session useless with Webauthn enabled, but provides
# maximum amount of security.
# If 'false', the user will not get a MFA prompt with an active
# session at the authorization endpoint.
# (default: false)
#SESSION_RENEW_MFA=false

# Session timeout in seconds
# When a new token / login is requested before this timeout hits
# the limit, the user will be authenticated without prompting for
# the credentials again.
# This is the value which can extend the session, until it hits
# its maximum lifetime set with SESSION_LIFETIME.
#SESSION_TIMEOUT=5400

# ML: magic link
# LT: lifetime
# Lifetime in minutes for reset password magic links (default: 30)
#ML_LT_PWD_RESET=30

# Lifetime in minutes for the first password magic link, for
# setting the initial password. (default: 86400)
#ML_LT_PWD_FIRST=86400

#####################################
############# LOGGING ###############
#####################################

# This is the log level for stdout logs
# Accepts: error, info, debug, trace (default: info)
#LOG_LEVEL=info

# This is a special config which allows the configuration of
# customized access logs. These logs will be logged with each
# request in addition to the normal LOG_LEVEL logs.
# The following values are valid:
# - Debug
#   CAUTION: The Debug setting logs every information available
#   to the middleware which includes SENSITIVE HEADERS
#   DO NOT use the Debug level in a working production environment!
# - Verbose
#   Verbose logging without headers - generates huge outputs
# - Basic
#   Logs access to all endpoints apart from the Frontend ones
#   which all js, css, ...
# - Modifying
#   Logs only requests to modifying endpoints and skips all GET
# - Off
# (default: Modifying)
LOG_LEVEL_ACCESS=Basic

#####################################
################ MFA ################
#####################################

# If 'true', MFA for an account must be enabled to access the
# rauthy admin UI (default: true)
ADMIN_FORCE_MFA=false

# If set to true, you can access rauthy's admin API only with
# a valid session + CSRF token. If you need some external access
# via JWT tokens, since sessions are managed with cookies, set
# this to false. (default: true)
ADMIN_ACCESS_SESSION_ONLY=true

#####################################
############## POW  #################
#####################################

## Proof of Work (PoW) configuration for Client Endpoints like
# User Registration. The iteration count for the PoW calculation
# (default: 1000000)
#POW_IT=1000000

# The expiration duration in seconds when a saved PoW should be
# cleaned up (default: 300)
#POW_EXP=300

#####################################
############# SERVER ################
#####################################

# The server address to listen on. Can bind to a specific IP.
# (default: 0.0.0.0)
#LISTEN_ADDRESS=0.0.0.0

# The listen ports for HTTP / HTTPS, depending on the
# activated 'LISTEN_SCHEME'
# default: 8080
#LISTEN_PORT_HTTP=8080
# default: 8443
#LISTEN_PORT_HTTPS=8443

# The scheme to use locally, valid values:
# http | https | http_https (default: http_https)
LISTEN_SCHEME=http

# The Public URL of the whole deployment
# The LISTEN_SCHEME + PUB_URL must match the HTTP ORIGIN
# HEADER later on, which is especially important when running
# rauthy behind a reverse proxy. In case of a non-standard
# port (80/443), you need to add the port to the PUB_URL
PUB_URL=localhost:8080

# default value: number of available physical cores
#HTTP_WORKERS=1

# When rauthy is running behind a reverse proxy, set to true
# (default: false)
PROXY_MODE=false

#####################################
############### TLS #################
#####################################

## Rauthy TLS

# Overwrite the path to the TLS certificate file in PEM
# format for rauthy (default: tls/tls.crt)
#TLS_CERT=tls/tls.crt
# Overwrite the path to the TLS private key file in PEM
# format for rauthy. If the path / filename ends with
# '.der', rauthy will parse it as DER, otherwise as PEM.
# (default: tls/tls.key)
#TLS_KEY=tls/tls.key

## CACHE TLS

# Enable / disable TLS for the cache communication
# (default: true)
CACHE_TLS=true
# The path to the server TLS certificate PEM file
# (default: tls/redhac.local.cert.pem)
CACHE_TLS_SERVER_CERT=tls/redhac.local.cert.pem
# The path to the server TLS key PEM file
# (default: tls/redhac.local.key.pem)
CACHE_TLS_SERVER_KEY=tls/redhac.local.key.pem
# If not empty, the PEM file from the specified location
# will be added as the CA certificate chain for validating
# the servers TLS certificate (default: tls/ca-chain.cert.pem)
CACHE_TLS_CA_SERVER=tls/ca-chain.cert.pem

# The path to the client mTLS certificate PEM file
# (default: tls/redhac.local.cert.pem)
CACHE_TLS_CLIENT_CERT=tls/redhac.local.cert.pem
# The path to the client mTLS key PEM file
# (default: tls/redhac.local.key.pem)
CACHE_TLS_CLIENT_KEY=tls/redhac.local.key.pem
# If not empty, the PEM file from the specified location will
# be added as the CA certificate chain for validating
# the clients mTLS certificate (default: tls/ca-chain.cert.pem)
CACHE_TLS_CA_CLIENT=tls/ca-chain.cert.pem

# The domain / CN the client should validate the certificate
# against. This domain MUST be inside the
# 'X509v3 Subject Alternative Name' when you take a look at the
# servers certificate with the openssl tool.
# default: redhac.local
CACHE_TLS_CLIENT_VALIDATE_DOMAIN=redhac.local

# Can be used, if you need to overwrite the SNI when the client
# connects to the server, for instance if you are behind
# a loadbalancer which combines multiple certificates. (default: "")
#CACHE_TLS_SNI_OVERWRITE=

#####################################
############# WEBAUTHN ##############
#####################################

# The 'Relaying Party (RP) ID' - effective domain name
# (default: localhost)
# CAUTION: When this changes, already registered devices will stop
# working and users cannot log in anymore!
RP_ID=localhost

# Url containing the effective domain name
# (default: http://localhost:8080)
# CAUTION: Must include the port number!
RP_ORIGIN=http://localhost:8080

# Non critical RP Name
# Has no security properties and may be changed without issues
# (default: Rauthy Webauthn)
RP_NAME='Rauthy Webauthn'

# The Cache lifetime in seconds for Webauthn requests. Within
# this time, a webauthn request must have been validated.
# (default: 60)
#WEBAUTHN_REQ_EXP=60

# The Cache lifetime for additional Webauthn Data like auth
# codes and so on. Should not be lower than WEBAUTHN_REQ_EXP.
# The value is in seconds (default: 90)
#WEBAUTHN_DATA_EXP=90

# With webauthn enabled for a user, he needs to enter
# username / password on a new system. If these credentials are
# verified, rauthy will set an additional cookie, which will
# determine how long the user can then use only (safe) MFA
# passwordless webauthn login with yubikeys, apple touch id,
# Windows hello, ... until he needs to verify his credentials
# again.
# Passwordless login is generally much safer than logging in
# with a password. But sometimes it is possible, that the
# Webauthn devices do not force the user to include a second
# factor, which in that case would be a single factor login
# again. That is why we should ask for the original password
# in addition once in a while to set the cookie.
# The value is in hours (default: 2160)
#WEBAUTHN_RENEW_EXP=2160

```