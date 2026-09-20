# Pixelence — Original Website Implementation

> **Development stopped:** This repository contains the incomplete website implementation of Pixelence. Development of this version stopped when the group decided to build a Flutter application instead. It is retained as a record of the original web development work.

## Overview

Pixelence began as a group website project. The code retained here is an initial Laravel application scaffold, with the default welcome page and standard user, session, cache, and queue infrastructure. It does not establish the intended product purpose, target users, or domain-specific workflows, so those details cannot be documented from this repository alone.

This repository does not contain the Flutter application or a completed migration to Flutter. The website is unfinished and is not production-ready.

## Technology stack

Versions below are declared dependency ranges unless a locked version is stated.

| Area | Technology and repository evidence |
| --- | --- |
| Backend | PHP `^8.2`, Laravel `^11.31` (`composer.json`); Laravel `v11.36.1` in `composer.lock` |
| Templates | Blade; the only view is `resources/views/welcome.blade.php` |
| Frontend assets | Vite `^6.0`, Laravel Vite plugin `^1.0`, JavaScript ES modules |
| Styling | Tailwind CSS `^3.4.13`, PostCSS `^8.4.47`, Autoprefixer `^10.4.20` |
| HTTP client | Axios `^1.7.4`, initialized in `resources/js/bootstrap.js` |
| Database | MySQL selected in `.env.example`; other standard database connections remain in `config/database.php` |
| Local tooling | Laravel Tinker, Pint, Pail, Sail, and `concurrently` are declared dependencies |
| Tests | Pest `^3.7`, Laravel Pest plugin `^3.0`; PHPUnit configuration and starter tests are included |

A Composer lockfile is included. No npm lockfile or Node.js version pin is included. Declared tools and framework configuration do not imply completed application features.

## Existing code and functionality

These descriptions come from source inspection, not runtime verification.

| Area | Present in the repository | Status |
| --- | --- | --- |
| Home page | `GET /` in `routes/web.php` returns the `welcome` view | Route and template are present; content is the default Laravel welcome page, not a Pixelence interface |
| Health endpoint | `/up` configured in `bootstrap/app.php` | Framework health route is configured; response was not tested |
| User data | `User` model with name, email, password, password hashing, and hidden authentication fields | Data-model scaffold only |
| Authentication | Standard auth configuration, users and password-reset-token tables, conditional login/register/dashboard links in the welcome view | Partial foundation only; no login, registration, password-reset, or dashboard routes, controllers, or pages are included |
| Database infrastructure | Migrations for `users`, `password_reset_tokens`, `sessions`, `cache`, `cache_locks`, `jobs`, `job_batches`, and `failed_jobs` | Standard framework tables; no Pixelence-specific schema |
| Frontend setup | Tailwind directives, Vite entry points, and Axios initialization | Build configuration is present; no project-specific interactions or API calls |
| Controllers and services | Empty abstract base controller and empty application service-provider methods | Placeholders, with no application business logic |
| Sample data | User factory and a seeder that creates one example user | Development scaffolding, not a completed account workflow |
| Console | Starter `inspire` command with an hourly schedule | Framework example, not a Pixelence feature |
| Tests | Example root-page HTTP-status test and a trivial unit assertion | Starter tests only; not executed during this documentation review |

No application API route file or custom feature controllers are included. The welcome view uses built assets when available and otherwise falls back to inline styling. It also references external fonts and Laravel images.

## Repository structure

```text
app/
bootstrap/
config/
database/
public/
resources/
routes/
storage/
tests/
.env.example
artisan
composer.json
composer.lock
package.json
phpunit.xml
postcss.config.js
tailwind.config.js
vite.config.js
README.md
```

- `app/Models/User.php` contains the standard user model. `app/Http/Controllers/Controller.php` is an empty base controller.
- `bootstrap/app.php` connects web routes, console commands, and the health endpoint.
- `config/` contains application, database, authentication, session, cache, queue, mail, and filesystem settings.
- `database/migrations/` defines the framework tables; `database/factories/` and `database/seeders/` contain example user data generation.
- `resources/views/welcome.blade.php` is the sole page template. `resources/css/app.css` and `resources/js/app.js` are Vite entry points.
- `routes/web.php` defines the home page; `routes/console.php` contains the example console command.
- `public/` contains the web entry point and public assets. `storage/` holds runtime files; `bootstrap/cache/` holds framework cache files.
- `tests/` and `phpunit.xml` provide the starter test setup.
- `composer.json` and `package.json` define dependencies and scripts; the Vite, Tailwind, and PostCSS files configure asset processing.

## Local setup and usage

**Verification status:** The steps below are based on the checked-in configuration and scripts. Dependency installation, asset builds, database migrations, application startup, and tests were not run during this review. PHP and Composer were unavailable in the review environment.

### Prerequisites

- Git, PHP 8.2–8.4, and Composer 2. The root PHP requirement is `^8.2`, but some locked development dependencies constrain support to PHP 8.2–8.4.
- PHP extensions required by the locked packages, plus `pdo_mysql` for the MySQL setup below. Composer's platform check can identify missing requirements.
- Node.js and npm compatible with the declared Vite 6 tooling. The repository does not pin a Node.js version.
- A local MySQL server, an empty development database, and a local database user with permission to create and modify its tables.
- Write access for the PHP process to `storage/` and `bootstrap/cache/`.

### 1. Obtain the source and prepare the environment

```sh
git clone https://github.com/levyy15/PixelenceWebsite.git
cd PixelenceWebsite
```

The repository includes a tracked `.env`. For a fresh local setup, replace that file with a copy of `.env.example` rather than reusing its values. If you already have your own local configuration, preserve it privately before replacing it.

```sh
cp .env.example .env
```

On Windows PowerShell, use `Copy-Item .env.example .env` instead. Keep local secrets out of commits; `.env` is already tracked, so ignore rules alone do not prevent its changes from being committed.

Edit `.env` for your local environment. For example:

```dotenv
APP_NAME=Pixelence
APP_ENV=local
APP_KEY=
APP_DEBUG=true
APP_URL=http://127.0.0.1:8000

DB_CONNECTION=mysql
DB_HOST=127.0.0.1
DB_PORT=3306
DB_DATABASE=pixelence_local
DB_USERNAME=your_local_database_user
DB_PASSWORD="replace_with_your_local_database_password"

SESSION_DRIVER=database
CACHE_STORE=database
QUEUE_CONNECTION=database
MAIL_MAILER=log
```

Replace the database placeholders with your own local values and create the matching empty MySQL database before running migrations. Leave `APP_KEY` empty until the generation command below. Debug mode is for local development only. The default log mailer does not send email; external mail, cloud-storage, or Redis credentials are not needed for this default setup.

### 2. Install dependencies and initialize the database

```sh
composer install
composer check-platform-reqs
npm install
php artisan key:generate
php artisan migrate
```

Use `composer install` to respect `composer.lock`. Use `npm install`, since there is no npm lockfile for `npm ci`. npm may create a local lockfile.

Migrations are needed for the configured database-backed sessions, cache, and queues. The example user seeder is optional and is not needed to view the welcome page. It does not enable login.

The Composer project-creation hook mentions creating a SQLite file, but `.env.example` selects MySQL. Do not treat that hook as a substitute for configuring and migrating your local database after cloning.

### 3. Start the development servers

The repository provides a combined development command:

```sh
composer run dev
```

It runs the PHP development server, `php artisan queue:listen --tries=1`, and the Vite development server through `concurrently`.

Alternatively, run these commands in separate terminals:

```sh
php artisan serve
```

```sh
npm run dev
```

No project-specific queued jobs are included, so a queue listener is not needed simply to inspect the welcome page. Open [http://127.0.0.1:8000](http://127.0.0.1:8000); the root route is configured to display the Laravel starter view.

To build frontend assets instead of running the Vite development server:

```sh
npm run build
```

This builds assets only; it does not start the PHP application or establish production readiness.

### 4. Inspect routes and run the starter tests

After installing dependencies and configuring the environment:

```sh
php artisan route:list
php artisan test
```

These commands were not executed during this review. The existing tests do not validate Pixelence product functionality. The SQLite test-database overrides in `phpunit.xml` are commented out; no isolated test database is configured there.

## Known limitations

- The repository retains an initial scaffold rather than a completed website. Its intended product domain and target audience are not established by the source.
- The only page is Laravel's default welcome view. Authentication links are conditional template placeholders; their corresponding application routes and screens are absent.
- There is no project-specific business logic, database schema, API, or frontend interaction code.
- `vendor/`, `node_modules/`, and compiled frontend assets are not included. PHP dependencies must be installed before Artisan or the application can run; frontend dependencies are needed for Vite commands.
- Local startup requires a valid application key, working database configuration, migrated tables, and writable runtime directories. The tracked environment file should not be treated as a portable local configuration.
- Frontend dependency resolution is not locked, and Node.js is not pinned, so installation reproducibility has not been established.
- Sail is declared as a development dependency, but no Docker Compose configuration is included for a ready-to-run container setup.
- The welcome page references externally hosted fonts and images, which may not load offline.
- Installation, rendering, builds, migrations, and test results remain unverified. No production deployment or completed Flutter implementation is documented here.
