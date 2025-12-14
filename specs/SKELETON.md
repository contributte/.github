# Contributte Skeleton Development Specification

This document describes the standards and conventions for developing Contributte skeleton projects.

## Reference Repositories

- [contributte/doctrine-skeleton](https://github.com/contributte/doctrine-skeleton)
- [contributte/messenger-skeleton](https://github.com/contributte/messenger-skeleton)

## Purpose

Skeleton projects serve as:
- **Starting templates** for new applications
- **Reference implementations** showing best practices
- **Demo applications** for Contributte libraries
- **Learning resources** for developers

## Directory Structure

```
├── .build/                   # Build artifacts
├── .data/                    # Persistent data (databases, etc.)
├── .docs/                    # Documentation files
│   └── assets/               # Screenshots, diagrams
├── .github/                  # GitHub workflows and templates
│   └── workflows/            # CI/CD workflow files
├── app/                      # Application source code
│   ├── Domain/               # Domain logic and entities
│   ├── Model/                # Business models
│   ├── Presenters/           # Nette presenters
│   ├── Router/               # Application routing
│   └── UI/                   # UI components and templates
├── bin/                      # Console commands and scripts
├── config/                   # Configuration files
│   ├── common.neon           # Common configuration
│   ├── local.neon            # Local environment config
│   └── local.neon.dist       # Template for local config
├── db/                       # Database files
│   ├── Fixtures/             # Data fixtures
│   └── Migrations/           # Database migrations
├── tests/                    # Test suite
│   └── Cases/                # Test cases
├── var/                      # Runtime data
│   ├── log/                  # Log files
│   └── tmp/                  # Temporary files
├── www/                      # Public web root
│   └── index.php             # Application entry point
├── .editorconfig             # Editor configuration
├── .gitignore                # Git ignore rules
├── composer.json             # Composer configuration
├── docker-compose.yml        # Docker services
├── LICENSE                   # MIT License
├── Makefile                  # Build automation
├── phpstan.neon              # PHPStan configuration
├── README.md                 # Project readme
└── ruleset.xml               # PHP CodeSniffer rules
```

## Requirements

- **PHP Version**: 8.2+ (minimum 8.3 recommended)
- **Nette Framework**: 3.2+
- **Docker**: For local development services

## Composer Configuration

### composer.json Structure

```json
{
  "name": "contributte/example-skeleton",
  "description": "Example skeleton project description",
  "license": "MIT",
  "type": "project",
  "authors": [
    {
      "name": "Milan Felix Sulc",
      "homepage": "https://f3l1x.io"
    }
  ],
  "require": {
    "php": ">=8.2",
    "nette/application": "^3.2",
    "nette/bootstrap": "^3.2",
    "nette/caching": "^3.3",
    "nette/di": "^3.2",
    "nette/http": "^3.3",
    "nette/robot-loader": "^4.0",
    "nette/routing": "^3.1",
    "nette/utils": "^4.0",
    "tracy/tracy": "^2.10",
    "latte/latte": "^3.0",
    "contributte/console": "^0.10"
  },
  "require-dev": {
    "contributte/dev": "^0.5",
    "contributte/qa": "^0.4",
    "contributte/phpstan": "^0.1",
    "nette/tester": "^2.5",
    "mockery/mockery": "^1.6"
  },
  "autoload": {
    "psr-4": {
      "App\\": "app"
    }
  },
  "autoload-dev": {
    "psr-4": {
      "Tests\\": "tests"
    }
  },
  "config": {
    "sort-packages": true,
    "allow-plugins": {
      "dealerdirect/phpcodesniffer-composer-installer": true
    }
  },
  "minimum-stability": "dev",
  "prefer-stable": true
}
```

## Makefile

Skeleton projects have extended Makefile with project management tasks:

```makefile
.PHONY: project init install setup clean qa cs csf phpstan tests coverage dev build docker-up deploy

#
# Project
#

project: install setup

init:
	cp config/local.neon.dist config/local.neon

install:
	composer install

setup:
	mkdir -p var/tmp var/log
	chmod -R 0777 var/tmp var/log

clean:
	find var/tmp -mindepth 1 ! -name '.gitignore' -type f,d -exec rm -rf {} + 2>/dev/null; true
	find var/log -mindepth 1 ! -name '.gitignore' -type f,d -exec rm -rf {} + 2>/dev/null; true

#
# QA
#

qa: phpstan cs

cs:
ifdef GITHUB_ACTION
	vendor/bin/phpcs --standard=ruleset.xml --encoding=utf-8 --extensions=php,phpt --tab-width=4 -sp app tests --report=checkstyle | cs2pr
else
	vendor/bin/phpcs --standard=ruleset.xml --encoding=utf-8 --extensions=php,phpt --tab-width=4 -sp app tests
endif

csf:
	vendor/bin/phpcbf --standard=ruleset.xml --encoding=utf-8 --extensions=php,phpt --tab-width=4 -sp app tests

phpstan:
	vendor/bin/phpstan analyse -c phpstan.neon

tests:
	vendor/bin/tester -s -p php --colors 1 -C tests/Cases

coverage:
ifdef GITHUB_ACTION
	vendor/bin/tester -s -p phpdbg --colors 1 -C --coverage coverage.xml --coverage-src app tests/Cases
else
	vendor/bin/tester -s -p phpdbg --colors 1 -C --coverage coverage.html --coverage-src app tests/Cases
endif

#
# Dev
#

dev:
	NETTE_DEBUG=1 php -S 0.0.0.0:8000 -t www

#
# Docker
#

docker-up:
	docker compose up -d

#
# Deploy
#

build:
	# Add build steps here

deploy: clean project build clean
```

### Available Commands

| Command | Description |
|---------|-------------|
| `make project` | Full project setup (install + setup) |
| `make init` | Copy local config template |
| `make install` | Install Composer dependencies |
| `make setup` | Create required directories |
| `make clean` | Clean temporary and log files |
| `make qa` | Run all quality checks |
| `make cs` | Check coding standards |
| `make csf` | Fix coding standards |
| `make phpstan` | Run static analysis |
| `make tests` | Run test suite |
| `make coverage` | Generate coverage report |
| `make dev` | Start development server |
| `make docker-up` | Start Docker services |
| `make deploy` | Build for deployment |

## Docker Configuration

### docker-compose.yml

Skeleton projects typically include Docker setup for required services:

#### Doctrine Skeleton (Database focused)

```yaml
services:
  postgres:
    image: postgres:15-alpine
    restart: always
    environment:
      POSTGRES_PASSWORD: contributte
      POSTGRES_USER: contributte
      POSTGRES_DB: contributte
    ports:
      - "5432:5432"
    volumes:
      - ./.data/postgres/data/:/var/lib/postgresql/data/

  mariadb:
    image: mariadb:10.10
    restart: always
    environment:
      MARIADB_ROOT_PASSWORD: contributte
      MARIADB_USER: contributte
      MARIADB_PASSWORD: contributte
      MARIADB_DATABASE: contributte
    ports:
      - "3306:3306"
    volumes:
      - ./.data/mariadb/data/:/var/lib/mysql/
```

#### Messenger Skeleton (Queue focused)

```yaml
services:
  web:
    image: php:8.1
    volumes:
      - .:/srv
    ports:
      - "8080:8080"
    depends_on:
      - database
      - redis
    environment:
      - NETTE_DEBUG=1

  database:
    image: postgres:15
    environment:
      POSTGRES_PASSWORD: contributte
      POSTGRES_DB: contributte
    ports:
      - "5432:5432"

  redis:
    image: redis:7
    command: redis-server --requirepass contributte
    ports:
      - "6379:6379"

  adminer:
    image: adminer
    ports:
      - "8081:8080"
    depends_on:
      - database
      - redis
```

### Docker Patterns

| Service | Default Credentials |
|---------|---------------------|
| PostgreSQL | user: `contributte`, password: `contributte` |
| MariaDB | user: `contributte`, password: `contributte` |
| Redis | password: `contributte` |

## Configuration

### config/common.neon

Base configuration shared across all environments:

```neon
parameters:

application:
    errorPresenter: Error
    mapping:
        *: App\UI\*\*Presenter

session:
    autoStart: true

di:
    export:
        tags: false

extensions:
    console: Contributte\Console\DI\ConsoleExtension(%consoleMode%)
```

### config/local.neon.dist

Template for local environment configuration:

```neon
parameters:
    database:
        host: 127.0.0.1
        port: 5432
        user: contributte
        password: contributte
        database: contributte
```

## PHPStan Configuration

### phpstan.neon

```neon
includes:
    - vendor/contributte/phpstan/phpstan.neon

parameters:
    level: 9
    phpVersion: 80200

    scanDirectories:
        - app

    fileExtensions:
        - php

    paths:
        - app
        - .docs

    ignoreErrors:
```

## Coding Standards

### ruleset.xml

```xml
<?xml version="1.0"?>
<ruleset>
    <rule ref="./vendor/contributte/qa/ruleset-8.2.xml"/>

    <rule ref="SlevomatCodingStandard.Files.TypeNameMatchesFileName">
        <properties>
            <property name="rootNamespaces" type="array">
                <element key="app" value="App"/>
                <element key="tests" value="Tests"/>
            </property>
        </properties>
    </rule>

    <exclude-pattern>/tests/tmp</exclude-pattern>
</ruleset>
```

## Application Entry Point

### www/index.php

```php
<?php declare(strict_types = 1);

require __DIR__ . '/../vendor/autoload.php';

App\Bootstrap::boot()
    ->createContainer()
    ->getByType(Nette\Application\Application::class)
    ->run();
```

### app/Bootstrap.php

```php
<?php declare(strict_types = 1);

namespace App;

use Nette\Bootstrap\Configurator;

final class Bootstrap
{
    public static function boot(): Configurator
    {
        $configurator = new Configurator();
        $appDir = dirname(__DIR__);

        $configurator->setDebugMode(true);
        $configurator->enableTracy($appDir . '/var/log');

        $configurator->setTimeZone('Europe/Prague');
        $configurator->setTempDirectory($appDir . '/var/tmp');

        $configurator->createRobotLoader()
            ->addDirectory(__DIR__)
            ->register();

        $configurator->addConfig($appDir . '/config/common.neon');
        $configurator->addConfig($appDir . '/config/local.neon');

        return $configurator;
    }
}
```

## Console Commands

### bin/console

```php
#!/usr/bin/env php
<?php declare(strict_types = 1);

require __DIR__ . '/../vendor/autoload.php';

exit(App\Bootstrap::boot()
    ->createContainer()
    ->getByType(Contributte\Console\Application::class)
    ->run());
```

## Documentation

### README.md Structure

Skeleton READMEs should include:

1. **Title and badges** - Name, build status, coverage
2. **Description** - What the skeleton demonstrates
3. **Screenshots** - Visual preview (in `.docs/assets/`)
4. **Requirements** - PHP, Docker, etc.
5. **Installation** - Step-by-step guide
6. **Configuration** - Environment setup
7. **Usage** - How to run the application
8. **Development** - How to contribute
9. **License** - MIT

### Installation Section Example

```markdown
## Installation

1. Clone the repository:
   ```bash
   git clone https://github.com/contributte/example-skeleton.git
   cd example-skeleton
   ```

2. Copy configuration template:
   ```bash
   make init
   # or: cp config/local.neon.dist config/local.neon
   ```

3. Start Docker services:
   ```bash
   make docker-up
   ```

4. Install dependencies:
   ```bash
   make install
   ```

5. Start development server:
   ```bash
   make dev
   ```

6. Open in browser: http://localhost:8000
```

## Git Configuration

### .gitignore

```
# Composer
/vendor/
composer.lock

# Local configuration
/config/local.neon

# Runtime
/var/log/*
!/var/log/.gitignore
/var/tmp/*
!/var/tmp/.gitignore

# Data
/.data/

# Coverage
coverage.html
coverage.xml

# IDE
.idea/
.vscode/
*.swp
```

## Testing

### tests/bootstrap.php

```php
<?php declare(strict_types = 1);

require __DIR__ . '/../vendor/autoload.php';

Tester\Environment::setup();

date_default_timezone_set('Europe/Prague');
```

### Test Organization

```
tests/
├── Cases/
│   ├── E2E/              # End-to-end tests
│   ├── Integration/      # Integration tests
│   └── Unit/             # Unit tests
├── Fixtures/             # Test data and fixtures
├── bootstrap.php         # Test bootstrap
└── .coveralls.yml        # Coverage config (optional)
```

## Checklist for New Skeletons

- [ ] Create directory structure
- [ ] Configure `composer.json` (type: project)
- [ ] Set up `Makefile` with project commands
- [ ] Configure `phpstan.neon`
- [ ] Configure `ruleset.xml`
- [ ] Add `.editorconfig`
- [ ] Add `.gitignore`
- [ ] Create `docker-compose.yml`
- [ ] Add `config/local.neon.dist` template
- [ ] Create `app/Bootstrap.php`
- [ ] Create `www/index.php` entry point
- [ ] Create `bin/console` for CLI
- [ ] Add `LICENSE` (MIT)
- [ ] Create `README.md` with screenshots
- [ ] Add documentation in `.docs`
- [ ] Write tests
- [ ] Verify all CI checks pass
- [ ] Test Docker setup works

## Differences: Library vs Skeleton

| Aspect | Library | Skeleton |
|--------|---------|----------|
| Type | `library` | `project` |
| Source directory | `src/` | `app/` |
| Entry point | N/A | `www/index.php` |
| Docker | Not included | Required |
| Local config | N/A | `config/local.neon` |
| Runtime data | N/A | `var/` directory |
| Console | N/A | `bin/console` |
| Web server | N/A | Built-in or Docker |
| Dependencies | Minimal | Full application |
