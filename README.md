# CitOmni App Skeleton

Neutral application skeleton for CitOmni.

`citomni/app-skeleton` is the mode-neutral starting point for CitOmni applications. It provides the application container, app-owned root files, neutral source-layer folders, runtime state folders, `.gitignore` defaults, and installer lifecycle tooling. The common config entry points come from `citomni/kernel` and the defensive `.htaccess` files from `citomni/http`; `citomni/installer` materializes both.

It does **not** assume that the application is an HTTP app, a CLI app, or both. HTTP and CLI support are added explicitly through `citomni/http` and `citomni/cli`.

Lean, deterministic, and pleasantly boring where boring is a feature.

---

## Highlights

- **Mode-neutral application container** for CitOmni projects.
- **No HTTP assumption** by default: No `public/`, no `templates/`, no HTTP routes, and no web front controller until `citomni/http` is installed.
- **No CLI assumption** by default: No `bin/citomni` launcher until `citomni/cli` is installed.
- **Installer-ready lifecycle model** through `citomni/installer`.
- **App-owned common config entry points** for shared cfg, providers, and local services, created from `citomni/kernel` by the first install.
- **Canonical source-layer folders** for operations, repositories, services, policies, state handlers, support classes, enums, utilities, and exceptions.
- **Shared-hosting defense-in-depth** through deny rules in internal folders once `citomni/http` is installed.
- **Deterministic line endings** through `.gitattributes`.

---

## What this package is

`citomni/app-skeleton` is the neutral application root for CitOmni.

It exists so new applications can start from a clean, mode-free structure and then explicitly opt into the runtime modes they need.

A new application may become:

- An HTTP application.
- A CLI application.
- A combined HTTP and CLI application.
- A base for a richer DevKit-driven workflow.

The skeleton itself stays small. Runtime-near scaffold files are owned by the package that needs them.

Examples:

- `public/index.php` is owned by `citomni/http`.
- `templates/` is owned by `citomni/http`.
- `bin/citomni` is owned by `citomni/cli`.
- Scaffold materialization is handled by `citomni/installer`.

This keeps ownership visible and avoids turning the starter project into a drawer full of maybe-useful files.

---

## What this package provides

`citomni/app-skeleton` provides the app-owned baseline structure.

That includes:

- Root Composer project metadata.
- PSR-4 application autoloading.
- Composer scripts for the installer commands.
- Neutral source-layer directories.
- Language directories.
- Runtime state directories under `var/`.
- `.gitattributes` for deterministic line endings.
- `.gitignore` defaults in a `citomni/app-skeleton` block, ahead of the block that `citomni/installer` maintains.
- `citomni/installer` as lifecycle tooling.

It gives the application a safe place to exist before any runtime mode is installed.

The common configuration files (`config/citomni_cfg.php`, its environment overlays, `config/services.php` and `config/providers.php`) are scaffold of `citomni/kernel`, and the defensive `.htaccess` files in internal folders are scaffold of `citomni/http`. `citomni/installer` materializes them; see [Installation](#installation).

---

## What this package does not provide

`citomni/app-skeleton` deliberately does **not** provide:

- HTTP runtime behavior.
- CLI runtime behavior.
- HTTP front controller files.
- CLI launcher files.
- HTTP routes.
- CLI commands.
- HTML templates.
- Public webroot files.
- Deployment profiles.
- Application business logic.

Those belong to runtime packages, provider packages, DevKit, or the application itself.

No ceremonial `public/` directory for pure CLI apps. No `bin/citomni` launcher for pure HTTP apps. The filesystem should not cosplay as a framework decision.

---

## Requirements

- PHP **8.5+**
- Composer
- `citomni/kernel`
- `citomni/installer`

OPcache is strongly recommended in production once a runtime mode has been installed.

---

## Installation

Create a new neutral CitOmni application:

```bash
composer create-project citomni/app-skeleton my-app
cd my-app
```

### Configure scaffold placeholders

Package scaffold stubs contain placeholders such as `{{APP_NAME}}`, and the installer stops on a placeholder without a value. Provide the values in `config/citomni_installer.php` before the first install. The file is app-owned and belongs in Git:

```php
<?php
declare(strict_types=1);

return [
	'placeholders' => [
		// citomni/kernel: config/citomni_cfg.php
		'APP_NAME' => 'My App',
		'LANGUAGE' => 'en',
		'ICU_LOCALE' => 'en_US',
		'TIMEZONE' => 'UTC',
		'CHARSET' => 'UTF-8',
		'OPERATOR_NAME' => 'Example Ltd.',
		'OPERATOR_EMAIL' => 'hello@example.com',
		'OPERATOR_URL' => 'https://www.example.com',
		'PUBLIC_CONTACT_NAME' => 'Example Ltd.',
		'PUBLIC_CONTACT_EMAIL' => 'hello@example.com',
		'PUBLIC_CONTACT_PHONE' => '+45 12 34 56 78',
		'PUBLIC_CONTACT_URL' => 'https://www.example.com/contact',
		'TECHNICAL_SUPPORT_NAME' => 'Example Ltd.',
		'TECHNICAL_SUPPORT_EMAIL' => 'support@example.com',
		'TECHNICAL_SUPPORT_PHONE' => '+45 12 34 56 78',
		'TECHNICAL_SUPPORT_URL' => 'https://www.example.com/support',
		'DEVELOPER_NAME' => 'Example Ltd.',
		'DEVELOPER_EMAIL' => 'dev@example.com',
		'DEVELOPER_URL' => 'https://www.example.com',

		// citomni/http (stage and prod config) and citomni/cli (stage and prod launcher)
		'STAGE_ROOT_URL' => 'https://stage.example.com',
		'PROD_ROOT_URL' => 'https://www.example.com',
	],
];
```

`STAGE_ROOT_URL` and `PROD_ROOT_URL` are base URLs without a trailing slash. `citomni/http` needs them already at install; `citomni/cli` needs them when the application switches to stage or prod. `STAGE_ROOT_HOST` and `PROD_ROOT_HOST` are derived from them and must not be set. A single run can override a value with `--placeholder=KEY=VALUE`.

### Materialize the scaffold

```bash
composer citomni:install -- --environment=dev
```

Composer passes the arguments after `--` to the installer. The first install requires an explicit environment and covers every installed package: it records the environment, creates the common configuration files from `citomni/kernel`, and writes the `citomni/installer` block into `.gitignore`.

At this point the application is intentionally neutral. It has no HTTP or CLI runtime mode yet.

### Composer scripts

The skeleton maps the installer commands to Composer scripts:

```text
composer citomni:doctor                          Validate environment, manifests, installer config and write access.
composer citomni:status                          Report scaffold state. Read-only.
composer citomni:install -- --environment=dev    Materialize scaffold for the recorded or a first environment.
composer citomni:migrate -- --dry-run            Reconcile scaffold with current package sources.
composer citomni:environment -- prod             Switch environment-aware files and the Composer posture.
composer citomni:repair                          Recreate missing files from recorded state.
composer citomni:sync                            Move package-managed scaffold to the current package baseline.
```

`citomni:install` has no default environment on purpose. Without `--environment` it fails instead of materializing dev files in a stage or production checkout. The scripts belong to the application; `citomni/installer` never edits `composer.json`.

---

## Add HTTP support

Install the HTTP runtime package:

```bash
composer require citomni/http
composer citomni:install -- --environment=dev
```

Direct equivalent, if you do not use Composer scripts:

```bash
vendor/bin/citomni-installer install --environment=dev
```

Use the environment the application was installed with. Install creates the scaffold of newly added packages and leaves existing files alone; use `citomni:environment` to switch environments.

This materializes HTTP-owned scaffold such as:

```text
.htaccess
public/index.php
public/.htaccess
public/robots.txt
public/assets/.gitkeep
public/uploads/.htaccess
config/citomni_http_cfg.php
config/citomni_http_cfg.dev.php
config/citomni_http_cfg.stage.php
config/citomni_http_cfg.prod.php
config/citomni_http_routes.php
config/citomni_http_routes.dev.php
config/citomni_http_routes.stage.php
config/citomni_http_routes.prod.php
config/services_http.php
src/Http/Controller/AppController.php
templates/public/maintenance.php.tpl
templates/member/.gitkeep
templates/admin/.gitkeep
```

It also installs the deny-all `.htaccess` files in `bin/`, `config/`, `language/`, `src/`, `templates/`, `tests/` and `var/` (see [Security notes](#security-notes)).

After this, point your web server document root to:

```text
public/
```

Use a web server in development too. PHP's built-in server (`php -S`) is deliberately not supported.

For the full scaffold lifecycle manual, see the [CitOmni Installer README](https://github.com/citomni/installer/blob/main/README.md).

---

## Add CLI support

Install the CLI runtime package:

```bash
composer require citomni/cli
composer citomni:install -- --environment=dev
```

Direct equivalent, if you do not use Composer scripts:

```bash
vendor/bin/citomni-installer install --environment=dev
```

This materializes CLI-owned scaffold such as:

```text
bin/citomni
config/citomni_cli_cfg.php
config/citomni_cli_cfg.dev.php
config/citomni_cli_cfg.stage.php
config/citomni_cli_cfg.prod.php
config/citomni_cli_commands.php
config/citomni_cli_commands.dev.php
config/citomni_cli_commands.stage.php
config/citomni_cli_commands.prod.php
config/services_cli.php
```

For the full scaffold lifecycle manual, see the [CitOmni Installer README](https://github.com/citomni/installer/blob/main/README.md).

---

## Add both HTTP and CLI

A combined application can install both modes:

```bash
composer require citomni/http citomni/cli
composer citomni:install -- --environment=dev
```

HTTP and CLI share the same app root, config folder, common cfg, providers, common services, source layers, language files, and runtime state folders.

The mode-specific adapters stay separate:

```text
src/Http/
src/Cli/
```

Shared application logic should normally live in:

```text
src/Operation/
src/Repository/
src/Service/
src/Policy/
src/State/
src/Support/
src/Enum/
src/Util/
src/Exception/
```

For details about installer status checks, repair, sync, conflict handling, manifests, policies, state, and JSON output, see the [CitOmni Installer README](https://github.com/citomni/installer/blob/main/README.md).

---

## Fresh project layout

A fresh `citomni/app-skeleton` project contains only the neutral app structure:

```text
/app-root
	/config
		/CONFIGURATION.md

	/language
		/da
			/.gitkeep
		/en
			/.gitkeep

	/src
		/Enum
			/.gitkeep
		/Exception
			/.gitkeep
		/Operation
			/.gitkeep
		/Policy
			/.gitkeep
		/Repository
			/.gitkeep
		/Service
			/.gitkeep
		/State
			/.gitkeep
		/Support
			/.gitkeep
		/Util
			/.gitkeep

	/var
		/backups
			/.gitkeep
		/cache
			/.gitkeep
		/flags
			/.gitkeep
		/logs
			/.gitkeep
		/nonces
			/.gitkeep
		/secrets
			/.gitkeep
		/state
			/.gitkeep
		/storage
			/.gitkeep

	.gitattributes
	.gitignore
	composer.json
	LICENSE
	NOTICE
	README.md
	TRADEMARKS.md
```

The first install adds the common configuration files from `citomni/kernel`:

```text
config/citomni_cfg.php
config/citomni_cfg.dev.php
config/citomni_cfg.stage.php
config/citomni_cfg.prod.php
config/providers.php
config/services.php
```

Mode-specific folders and files such as `public/`, `templates/`, `bin/`, `tests/`, `src/Http/`, `src/Cli/` and the defensive `.htaccess` files are intentionally absent until their owning package is installed and materialized.

---

## Configuration files

The first install creates the common app-level config entry points from `citomni/kernel`. Runtime packages may add mode-specific overlays when HTTP or CLI support is installed. Once created, these files belong to the application.

For the detailed merge contract, see [`config/CONFIGURATION.md`](config/CONFIGURATION.md).

### `config/citomni_cfg.php`

Defines application-owned common configuration shared by HTTP and CLI.

```php
<?php
declare(strict_types=1);

return [
	'identity' => [
		'app_name' => 'My App',
	],
];
```

This file is the stable app base. It should contain durable values that are not inherently HTTP-specific or CLI-specific.

### `config/citomni_cfg.<env>.php`

Defines optional common environment overlays, normally `dev`, `stage`, and `prod`.

These files are loaded after `config/citomni_cfg.php` and before mode-specific environment overlays. They should express narrow environment differences, not duplicate the base file because future you already has enough to review.

### `config/providers.php`

Lists provider package registries loaded by the application.

```php
<?php
declare(strict_types=1);

return [
	// \Vendor\Package\Boot\Registry::class,
];
```

Provider order matters. Provider cfg is merged in the listed order, while provider service-map precedence is derived from the same order with app service maps still winning above provider and vendor definitions.

### `config/services.php`

Defines app-local service map entries and common overrides.

```php
<?php
declare(strict_types=1);

return [
	// 'example' => \App\Service\ExampleService::class,
];
```

`services.php` is shared by HTTP and CLI. A service definition placed here should therefore be safe for both modes, or intentionally override a common service contract.

### Mode-specific cfg files

HTTP mode may add:

```text
config/citomni_http_cfg.php
config/citomni_http_cfg.dev.php
config/citomni_http_cfg.stage.php
config/citomni_http_cfg.prod.php
```

CLI mode may add:

```text
config/citomni_cli_cfg.php
config/citomni_cli_cfg.dev.php
config/citomni_cli_cfg.stage.php
config/citomni_cli_cfg.prod.php
```

Mode-specific cfg files override common cfg for their mode. Environment overlays override the matching base files.

### Mode-specific service files

HTTP mode may add:

```text
config/services_http.php
```

CLI mode may add:

```text
config/services_cli.php
```

Service maps use PHP array union semantics. First matching service ID wins. In practice, app mode-specific service maps have the highest precedence, followed by app common services, then provider maps, then the selected vendor mode baseline.

### Dispatch files

HTTP routes remain HTTP-specific:

```text
config/citomni_http_routes.php
config/citomni_http_routes.dev.php
config/citomni_http_routes.stage.php
config/citomni_http_routes.prod.php
```

CLI commands remain CLI-specific:

```text
config/citomni_cli_commands.php
config/citomni_cli_commands.dev.php
config/citomni_cli_commands.stage.php
config/citomni_cli_commands.prod.php
```

There is no common route map and no common command map. Shared orchestration belongs in operations, repositories, services, policies, or other shared source layers, not in shared dispatch files.

## Source-layer model

CitOmni keeps responsibility boundaries explicit.

- `src/Operation/`: Transport-agnostic orchestration and application-level decision logic.
- `src/Repository/`: Persistence boundary. All SQL and datastore IO belongs here.
- `src/Service/`: App-aware services registered in the service map.
- `src/Policy/`: App-aware policy classes for explicit rules and requirements.
- `src/State/`: Runtime state handlers for state contracts, keys, locks, TTLs, and transitions.
- `src/Support/`: Focused support classes that may need application context without being singleton services.
- `src/Enum/`: Bounded value sets such as modes, statuses, purposes, or stable internal flags.
- `src/Util/`: Pure helpers only. No App, no config reads, no IO, no logging, no SQL.
- `src/Exception/`: Transport-agnostic application and domain exceptions.

HTTP-specific code should live under `src/Http/` once HTTP mode is installed.

CLI-specific code should live under `src/Cli/` once CLI mode is installed.

---

## Scaffold ownership

CitOmni packages own the scaffold they require.

```text
citomni/app-skeleton
	Owns the neutral app container.

citomni/kernel
	Owns common config scaffold such as citomni_cfg.php, services.php, and providers.php.

citomni/http
	Owns HTTP runtime and HTTP scaffold.

citomni/cli
	Owns CLI runtime and CLI scaffold.

citomni/installer
	Materializes package-owned scaffold into the app.
```

The details of scaffold discovery, rendering, checksums, managed files, create-only files, `.new` files, repair, and sync belong in the [CitOmni Installer README](https://github.com/citomni/installer/blob/main/README.md).

---

## Security notes

`citomni/http` installs defensive `.htaccess` files in internal folders:

```text
bin/.htaccess
config/.htaccess
language/.htaccess
src/.htaccess
templates/.htaccess
tests/.htaccess
var/.htaccess
```

These files are defense-in-depth for shared hosting or misconfigured web roots. A correct HTTP deployment should still point the web server document root to `public/` after HTTP mode is installed.

They are package-managed: the installer renders them per installation and the `citomni/installer` block keeps them out of Git, together with `public/index.php` and the other managed targets. A fresh checkout or a new server therefore runs `citomni:install` with its environment before it serves traffic. A neutral or CLI-only application has none of these files.

Do not rely on `.htaccess` as the only security boundary. The correct public webroot is still the real fix. The `.htaccess` files are the airbag, not the steering wheel.

Secrets should not be committed.

Runtime secrets belong under:

```text
var/secrets/
```

or should be provided by environment, deployment tooling, or a dedicated secret mechanism.

---

## Runtime state

The `var/` folder is the application runtime write area.

Typical subfolders:

```text
var/backups/
var/cache/
var/flags/
var/logs/
var/nonces/
var/secrets/
var/state/
var/storage/
```

`var/storage/` holds private files such as user uploads that must not be served from `public/`. Treat it like `var/secrets/`.

Production deployments should generally make application code read-only and allow writes only where the application deliberately needs runtime state.

Runtime content stays out of Git; each directory keeps only its `.gitkeep`, so a fresh checkout has the directories. See [Git ignore rules](#git-ignore-rules).

---

## Git ignore rules

The skeleton's `.gitignore` is the first of three tiers. Git uses the last matching pattern, so each tier takes precedence over the ones above it:

1. The `citomni/app-skeleton` block at the top of the file: defaults for the runtime directories above, plus local conventions such as `/vendor/` and root folders starting with `_`. It belongs to the application after `composer create-project`; no tool rewrites it.
2. The `citomni/installer` block, which `citomni/installer` inserts right after the skeleton block and rewrites on `install`, `sync` and `migrate`. It ignores the runtime paths of the installed packages and their package-managed scaffold targets such as `public/index.php`, while keeping the files the packages want versioned.
3. Application rules at the end of the file. They take precedence over both blocks, for example to version a fixture below `var/state/`, or to ignore a storage directory that the application configured elsewhere.

Edits inside the installer block are lost on the next run. `composer.lock` is not ignored: applications should commit it for deterministic installs. The [CitOmni Installer README](https://github.com/citomni/installer/blob/main/README.md#gitignore-block) describes how the installer block is generated.

---

## Autoloading and performance

The skeleton uses PSR-4 autoloading:

```json
{
	"autoload": {
		"psr-4": {
			"App\\": "src/"
		}
	}
}
```

For development, avoid `classmap-authoritative` as a default unless you are comfortable running autoload dumps whenever classes are added or moved.

For production, use optimized Composer autoloading:

```bash
composer install --no-dev --optimize-autoloader --classmap-authoritative
```

or:

```bash
composer dump-autoload -a
```

OPcache is strongly recommended in production.

---

## Line endings

The skeleton ships with `.gitattributes` to keep text files normalized.

This reduces false diffs, makes scaffold checksums more predictable, and keeps Windows development from turning line endings into a tiny procedural crime scene.

---

## Testing

The skeleton is ready for PHPUnit if you add it:

```bash
composer require --dev phpunit/phpunit:^10.5
```

Tests should live under:

```text
tests/
```

with namespace:

```text
App\Tests\
```

---

## Troubleshooting

### `vendor/bin/citomni-installer` is missing

Run Composer install:

```bash
composer install
```

Confirm that `citomni/installer` is required in `composer.json`.

### `install requires --environment=<dev|stage|prod>`

Pass the environment after `--`:

```bash
composer citomni:install -- --environment=dev
```

### `Unknown placeholder: {{APP_NAME}}`

A scaffold stub needs a value that `config/citomni_installer.php` does not provide. Add the key named in the message; see [Configure scaffold placeholders](#configure-scaffold-placeholders).

### HTTP install did not create `public/`

Confirm that `citomni/http` is installed, then materialize the HTTP scaffold:

```bash
composer show citomni/http
composer citomni:install -- --environment=dev
```

### CLI install did not create `bin/citomni`

Confirm that `citomni/cli` is installed, then materialize the CLI scaffold:

```bash
composer show citomni/cli
composer citomni:install -- --environment=dev
```

### New PHP class is not found

Run:

```bash
composer dump-autoload
```

If using authoritative classmaps, run:

```bash
composer dump-autoload -a
```

### Web server shows internal files

Point the web server document root to:

```text
public/
```

If `public/` does not exist, HTTP mode has not been installed yet.

---

## Coding and documentation conventions

All CitOmni projects follow the shared conventions documented here:

[CitOmni Coding and Documentation Conventions](https://github.com/citomni/docs/blob/main/contribute/CONVENTIONS.md)

Core conventions:

- PHP **8.5+**
- PSR-1 and PSR-4
- PascalCase classes
- camelCase methods and variables
- UPPER_SNAKE_CASE constants
- K&R brace style
- Tabs for indentation
- PHPDoc and inline comments in English
- Explicit wiring over discovery
- Deterministic behavior over magic

---

## License

**CitOmni App Skeleton** is open-source under the **MIT License**.

See [LICENSE](LICENSE).

**Trademark notice:** "CitOmni" and the CitOmni logo are trademarks of **Lars Grove Mortensen**. Usage of the name or logo must follow the policy in [NOTICE](NOTICE). Do not imply endorsement or affiliation without prior written permission.

---

## Trademarks

"CitOmni" and the CitOmni logo are trademarks of **Lars Grove Mortensen**.

You may make factual references to "CitOmni", but do not modify the marks, create confusingly similar logos, or imply sponsorship, endorsement, or affiliation without prior written permission.

Do not register or use "citomni" or confusingly similar terms in company names, domains, social handles, or top-level vendor/package names.

For details, see [NOTICE](NOTICE).

---

## Author

Developed by Lars Grove Mortensen © 2012-present.

---

CitOmni - low overhead, high performance, ready for anything.
