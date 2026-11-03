# Metabase on Wodby

What Wodby sets up for Metabase on this service. Metabase runs from the official `metabase/metabase` image (open source edition) as user 2000. Configuration is passed as `MB_*` environment variables.

## Linked services

| Link | Variables |
| --- | --- |
| PostgreSQL (required) | `MB_DB_HOST`, `MB_DB_PORT`, `MB_DB_DBNAME`, `MB_DB_USER`, `MB_DB_PASS` |
| SMTP server (optional) | `MB_EMAIL_SMTP_HOST`, `MB_EMAIL_SMTP_PORT`, `MB_EMAIL_SMTP_SECURITY` (`none`) |

`MB_DB_TYPE` is `postgres`. The linked database is Metabase's application database: users, questions, dashboards, saved connections and settings. The service has no volume. The databases people analyze are added in Metabase's admin settings, not as links.

## What the manifest sets

- `MB_SITE_URL` is the service's URL in the environment. Metabase uses it in emails, sharing links and embeds. Without a domain it is empty, and Metabase takes the address of the first administrator's visit.
- `MB_ENCRYPTION_SECRET_KEY` comes from the generated token `encryption_key`. Metabase encrypts the connection details and other secrets it stores with it. Changing it makes them unreadable, so keep it.
- `MB_SESSION_SECRET_KEY` comes from the generated token `session_key`. Metabase signs sessions with it. Changing it signs everyone out.
- `JAVA_OPTS` lets the JVM use 75% of the container's memory limit.
- TLS ends at Wodby's router. HTTP requests are redirected to HTTPS by default.

## First administrator

The service settings Admin email and Admin password are required. On the first start with an empty application database, the container's start script creates the administrator from them through Metabase's setup API, sets the site name to the app's title and turns off Metabase's anonymous usage tracking. The pod reports ready only after that, so Metabase's setup page is never reachable. The password must meet Metabase's password rules; if Metabase refuses it, the container stops with Metabase's reason in its log. Changing the settings later does not change an existing account.

## Versions and upgrades

Metabase migrates the application database when a newer version starts, and an older version can't use a migrated database. Deployments use the recreate strategy: all replicas stop before the new version starts. Moving to an older option version fails; restore a backup of the database instead. Back up the PostgreSQL database before an upgrade.

## Changing configuration

Most settings are changed in Metabase's admin settings and stored in the application database. A setting given as an `MB_*` environment variable takes precedence and can't be changed in Metabase. The container runs the start script instead of the image's entrypoint; the script starts Metabase with the image's `/app/run_metabase.sh`, which reads `JAVA_OPTS`.

## Check the result

From inside the environment, `/api/health` on port 3000 answers `{"status":"ok"}` when Metabase is ready.
