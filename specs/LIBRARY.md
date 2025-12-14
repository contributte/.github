# Contributte Library Development Specification

This document describes the standards and conventions for developing Contributte libraries.

## Reference Repositories

- [contributte/doctrine-dbal](https://github.com/contributte/doctrine-dbal)
- [contributte/doctrine-orm](https://github.com/contributte/doctrine-orm)
- [contributte/messenger](https://github.com/contributte/messenger)

## Directory Structure

```
├── .docs/                    # Documentation files
├── .github/                  # GitHub workflows and templates
│   └── workflows/            # CI/CD workflow files
├── src/                      # Source code
├── tests/                    # Test suite
│   └── Cases/                # Test cases
├── .editorconfig             # Editor configuration
├── .gitattributes            # Git export attributes
├── .gitignore                # Git ignore rules
├── composer.json             # Composer configuration
├── LICENSE                   # MIT License
├── Makefile                  # Build automation
├── phpstan.neon              # PHPStan configuration
├── README.md                 # Project readme
└── ruleset.xml               # PHP CodeSniffer rules
```

## Requirements

- **PHP Version**: 8.2+ (minimum)
- **Nette Framework**: 3.2+
- **Coding Standard**: Contributte QA ruleset

## Composer Configuration

### composer.json Structure

```json
{
  "name": "nettrine/example",
  "description": "Package description",
  "license": "MIT",
  "type": "library",
  "homepage": "https://github.com/contributte/example",
  "authors": [
    {
      "name": "Milan Felix Sulc",
      "homepage": "https://f3l1x.io"
    }
  ],
  "require": {
    "php": ">=8.2"
  },
  "require-dev": {
    "contributte/qa": "^0.4",
    "contributte/phpstan": "^0.1",
    "mockery/mockery": "^1.6",
    "nette/tester": "^2.5",
    "tracy/tracy": "^2.10"
  },
  "autoload": {
    "psr-4": {
      "Nettrine\\Example\\": "src"
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
  "extra": {
    "branch-alias": {
      "dev-master": "0.1.x-dev"
    }
  },
  "minimum-stability": "dev",
  "prefer-stable": true
}
```

### Key Dependencies

| Package | Purpose |
|---------|---------|
| `contributte/qa` | Coding standards ruleset |
| `contributte/phpstan` | PHPStan configuration |
| `nette/tester` | Testing framework |
| `mockery/mockery` | Mocking library |
| `tracy/tracy` | Debugging tools |

## Makefile

The Makefile provides standardized development commands:

```makefile
.PHONY: install qa cs csf phpstan tests coverage

install:
	composer update

qa: phpstan cs

cs:
ifdef GITHUB_ACTION
	vendor/bin/phpcs --standard=ruleset.xml --encoding=utf-8 --extensions=php,phpt --tab-width=4 -sp src tests --report=checkstyle | cs2pr
else
	vendor/bin/phpcs --standard=ruleset.xml --encoding=utf-8 --extensions=php,phpt --tab-width=4 -sp src tests
endif

csf:
	vendor/bin/phpcbf --standard=ruleset.xml --encoding=utf-8 --extensions=php,phpt --tab-width=4 -sp src tests

phpstan:
	vendor/bin/phpstan analyse -c phpstan.neon

tests:
	vendor/bin/tester -s -p php --colors 1 -C tests/Cases

coverage:
ifdef GITHUB_ACTION
	vendor/bin/tester -s -p phpdbg --colors 1 -C --coverage coverage.xml --coverage-src src tests/Cases
else
	vendor/bin/tester -s -p phpdbg --colors 1 -C --coverage coverage.html --coverage-src src tests/Cases
endif
```

### Available Commands

| Command | Description |
|---------|-------------|
| `make install` | Install/update Composer dependencies |
| `make qa` | Run all quality assurance checks (phpstan + cs) |
| `make cs` | Run code style checks |
| `make csf` | Fix code style issues automatically |
| `make phpstan` | Run static analysis |
| `make tests` | Execute test suite |
| `make coverage` | Generate code coverage report |

## PHPStan Configuration

### phpstan.neon

```neon
includes:
    - vendor/contributte/phpstan/phpstan.neon

parameters:
    level: 9
    phpVersion: 80200

    scanDirectories:
        - src

    fileExtensions:
        - php

    paths:
        - src
        - .docs

    ignoreErrors:
```

### Configuration Details

- **Level**: 9 (strictest)
- **PHP Version**: Matches minimum required version
- **Scan paths**: `src` and `.docs` directories

## Coding Standards

### ruleset.xml

```xml
<?xml version="1.0"?>
<ruleset>
    <rule ref="./vendor/contributte/qa/ruleset-8.2.xml"/>

    <rule ref="SlevomatCodingStandard.Files.TypeNameMatchesFileName">
        <properties>
            <property name="rootNamespaces" type="array">
                <element key="src" value="Nettrine\Example"/>
                <element key="tests" value="Tests"/>
            </property>
        </properties>
    </rule>

    <exclude-pattern>/tests/tmp</exclude-pattern>
</ruleset>
```

### Key Points

- Uses `contributte/qa` base ruleset
- Requires PHP 8.2 compatible ruleset
- Enforces file/class name matching
- Excludes temporary test directories

## GitHub Workflows

Libraries use reusable workflows from `contributte/.github` repository.

### Required Workflow Files

```
.github/workflows/
├── codesniffer.yml
├── coverage.yml
├── phpstan.yml
└── tests.yml
```

### Workflow Templates

#### tests.yml

```yaml
name: "Nette Tester"

on:
  pull_request:
  workflow_dispatch:
  push:
    branches:
      - "**"
  schedule:
    - cron: "0 8 * * 1"

jobs:
  test84:
    name: "Nette Tester"
    uses: contributte/.github/.github/workflows/nette-tester.yml@master
    with:
      php: "8.4"

  test83:
    name: "Nette Tester"
    uses: contributte/.github/.github/workflows/nette-tester.yml@master
    with:
      php: "8.3"

  test82:
    name: "Nette Tester"
    uses: contributte/.github/.github/workflows/nette-tester.yml@master
    with:
      php: "8.2"

  testlower:
    name: "Nette Tester"
    uses: contributte/.github/.github/workflows/nette-tester.yml@master
    with:
      php: "8.2"
      composer: "composer update --no-interaction --no-progress --prefer-dist --prefer-lowest --prefer-stable"
```

#### phpstan.yml

```yaml
name: "Phpstan"

on:
  pull_request:
  workflow_dispatch:
  push:
    branches:
      - "**"
  schedule:
    - cron: "0 8 * * 1"

jobs:
  phpstan:
    name: "Phpstan"
    uses: contributte/.github/.github/workflows/phpstan.yml@master
    with:
      php: "8.3"
```

#### codesniffer.yml

```yaml
name: "Codesniffer"

on:
  pull_request:
  workflow_dispatch:
  push:
    branches:
      - "**"
  schedule:
    - cron: "0 8 * * 1"

jobs:
  codesniffer:
    name: "Codesniffer"
    uses: contributte/.github/.github/workflows/codesniffer.yml@master
    with:
      php: "8.3"
```

#### coverage.yml

```yaml
name: "Coverage"

on:
  pull_request:
  workflow_dispatch:
  push:
    branches:
      - "**"
  schedule:
    - cron: "0 8 * * 1"

jobs:
  coverage:
    name: "Nette Tester"
    uses: contributte/.github/.github/workflows/nette-tester-coverage-v2.yml@master
    with:
      php: "8.3"
    secrets: inherit
```

### Workflow Triggers

All workflows are triggered by:
- Pull requests
- Manual dispatch (`workflow_dispatch`)
- Push to any branch
- Weekly schedule (Monday 8:00 UTC)

## Testing

### Test Structure

```
tests/
├── Cases/              # Test cases organized by feature
│   ├── Unit/           # Unit tests
│   └── Integration/    # Integration tests
├── Fixtures/           # Test fixtures and data
├── bootstrap.php       # Test bootstrap file
└── .coveralls.yml      # Coveralls configuration (optional)
```

### Test Bootstrap

```php
<?php declare(strict_types = 1);

require __DIR__ . '/../vendor/autoload.php';

Tester\Environment::setup();

date_default_timezone_set('Europe/Prague');
```

### Writing Tests

```php
<?php declare(strict_types = 1);

namespace Tests\Cases\Unit;

use Tester\Assert;
use Tester\TestCase;

require_once __DIR__ . '/../../bootstrap.php';

final class ExampleTest extends TestCase
{
    public function testExample(): void
    {
        Assert::true(true);
    }
}

(new ExampleTest())->run();
```

## Editor Configuration

### .editorconfig

```ini
root = true

[*]
charset = utf-8
end_of_line = lf
insert_final_newline = true
trim_trailing_whitespace = true
indent_style = tab
indent_size = 4

[*.{yml,yaml}]
indent_style = space
indent_size = 2

[*.md]
trim_trailing_whitespace = false

[*.neon]
indent_style = tab
indent_size = 4
```

## Git Configuration

### .gitattributes

```
/.build         export-ignore
/.docs          export-ignore
/.github        export-ignore
/tests          export-ignore
.editorconfig   export-ignore
.gitattributes  export-ignore
.gitignore      export-ignore
Makefile        export-ignore
phpstan.neon    export-ignore
ruleset.xml     export-ignore
```

### .gitignore

```
/vendor/
/tests/tmp/
composer.lock
coverage.html
coverage.xml
```

## Documentation

### .docs Structure

Documentation is stored in the `.docs` directory:

```
.docs/
├── README.md           # Main documentation
├── index.md            # Index/getting started
└── assets/             # Images and diagrams
```

### README.md Template

The main README.md should include:

1. Package name and badges
2. Brief description
3. Documentation link
4. Installation instructions
5. Quick usage example
6. Version compatibility matrix
7. Development section
8. License information

## Versioning

- Follow [Semantic Versioning](https://semver.org/)
- Use branch aliases in composer.json
- Tag releases appropriately

### Version Matrix Example

| State | Version | Branch | Nette | PHP |
|-------|---------|--------|-------|-----|
| dev | `^0.11` | `master` | 3.2+ | >=8.2 |
| stable | `^0.10` | `master` | 3.2+ | >=8.2 |
| stable | `^0.9` | `master` | 3.1+ | >=8.1 |

## Checklist for New Libraries

- [ ] Create directory structure
- [ ] Configure `composer.json`
- [ ] Set up `Makefile`
- [ ] Configure `phpstan.neon`
- [ ] Configure `ruleset.xml`
- [ ] Add `.editorconfig`
- [ ] Add `.gitattributes`
- [ ] Add `.gitignore`
- [ ] Create GitHub workflows
- [ ] Add `LICENSE` (MIT)
- [ ] Create `README.md`
- [ ] Add documentation in `.docs`
- [ ] Write initial tests
- [ ] Verify all CI checks pass
