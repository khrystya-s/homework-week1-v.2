# Repository Analysis

## Project Summary

`khrystya-s/rgr-os` is an academic course project (розрахункова робота — RGR) for an Operating Systems class. It contains Docker-based infrastructure configurations to deploy WordPress sites with HTTPS support using Nginx as a reverse proxy and MySQL as a database backend. The repository includes two parallel deployment setups, each representing a team member's environment: `wordpress-Khrystyna` (created on Windows) and `wordpress-struk`.

## Technologies

- **Docker / Docker Compose** — container orchestration via `docker-compose.yml` (Compose version 3)
- **MySQL 8.0** — relational database backend for WordPress
- **WordPress 5.1.1-fpm-alpine** — CMS, served via PHP-FPM on Alpine Linux
- **Nginx 1.15.12-alpine** — web server and TLS-terminating reverse proxy
- **TLS/SSL** — self-signed certificates (`.crt` / `.key` files) for HTTPS

## Project Structure

```
rgr-os/
├── README.md                              # Brief project description (Ukrainian)
├── wordpress-Khrystyna/                   # Environment 1 — created on Windows
│   ├── docker-compose.yml                 # 3-service stack: MySQL, WordPress, Nginx
│   └── nginx-conf/
│       ├── nginx.conf                     # Nginx config: HTTP→HTTPS redirect + HTTPS server block
│       ├── Struk-Khrystyna.crt            # Self-signed TLS certificate
│       └── Struk-Khrystyna.key            # TLS private key
└── wordpress-struk/                       # Environment 2 — second team member's setup
    ├── docker-compose.yml                 # 3-service stack: MySQL, WordPress, Nginx
    └── nginx-conf/
        ├── nginx.conf                     # Nginx config: HTTP→HTTPS redirect + HTTPS server block
        ├── Khrystyna-Struk.crt            # Self-signed TLS certificate
        └── Khrystyna-Struk.key            # TLS private key
```

Each directory contains a self-contained Docker Compose stack with three services:
1. **db-*** — MySQL 8.0 database, credentials loaded via `.env`
2. **wordpress-*** — WordPress 5.1.1-fpm-alpine, connected to the db service
3. **webserver-*** — Nginx 1.15.12-alpine, exposes ports 80 and 443, terminates TLS, proxies to WordPress

## Strengths

- **Consistent structure** — both deployment environments follow identical patterns, making comparison and learning easy.
- **HTTPS enabled** — Nginx is configured to enforce HTTPS via HTTP-to-HTTPS redirect (`return 301`) in both setups.
- **Credential externalization** — database credentials are loaded via `.env` file and environment variables rather than being hardcoded in the compose file.
- **Named volumes** — `wordpress` and `dbdata` volumes are properly declared, ensuring data persistence across container restarts.
- **Bridge network isolation** — a custom `app-network` bridge network is defined, isolating the services.
- **Alpine-based images** — use of Alpine-based Docker images (WordPress FPM Alpine, Nginx Alpine) keeps image sizes small.
- **Service dependencies** — `depends_on` is used to enforce correct startup order (Nginx depends on WordPress, WordPress depends on MySQL).

## Potential Issues

- **Private TLS keys committed to the repository** — `.key` files (private SSL keys) are tracked in git and publicly visible. This is a significant security concern, even for self-signed/local certificates.
- **No `.env` file included** — the compose files reference a `.env` file for database credentials, but no `.env.example` or template is provided, making it unclear what variables need to be set.
- **Very minimal README** — the README contains only one line of context in Ukrainian and is incomplete; it does not explain how to run the project or what environment variables are needed.
- **No `.gitignore`** — there is no `.gitignore` file, which contributed to the private key files being committed.
- **Outdated image versions** — `wordpress:5.1.1-fpm-alpine` (released 2019) and `nginx:1.15.12-alpine` (released 2019) are significantly out of date; newer, patched versions should be used.
- **nginx.conf appears truncated** — the nginx configuration files are missing the PHP-FPM `location ~ \.php$` block that is needed to proxy PHP requests to WordPress; the config as visible would not forward PHP to the WordPress FPM container.
- **Duplicate SSL certificate names** — in `wordpress-Khrystyna` the cert is named `Struk-Khrystyna`, while in `wordpress-struk` it is `Khrystyna-Struk` — these are swapped, which may indicate copy-paste confusion.
- **No test or CI configuration** — no automated testing or CI/CD pipeline is present.

## Recommendations

1. **Remove private keys from the repository immediately** — add `*.key` and `*.crt` to `.gitignore` and revoke/regenerate any affected certificates. Use git history rewriting (e.g., `git filter-branch` or BFG) to remove them from history.
2. **Add a `.gitignore`** — at minimum exclude `*.key`, `*.crt`, `.env`, and common OS/editor artifacts.
3. **Provide a `.env.example`** — document the required environment variables (`MYSQL_DATABASE`, `MYSQL_USER`, `MYSQL_PASSWORD`, `MYSQL_ROOT_PASSWORD`) with placeholder values.
4. **Expand the README** — add setup instructions, prerequisites (Docker, Docker Compose), steps to generate self-signed certificates, and how to add the local domain to `/etc/hosts`.
5. **Update Docker image versions** — upgrade to current stable versions of WordPress (e.g., `wordpress:latest` or a recent pinned tag), Nginx (e.g., `nginx:stable-alpine`), and MySQL (e.g., `mysql:8.0` is fine but confirm patching).
6. **Complete the nginx.conf** — add the `location ~ \.php$` proxy_pass block to forward PHP requests to the WordPress FPM container (e.g., `fastcgi_pass wordpress-Khrystyna:9000`).
7. **Consider merging or parameterizing the two setups** — the two directories are nearly identical; they could be unified into a single parameterized compose file using Docker profiles or environment overrides.
