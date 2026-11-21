# libpmform

libpmform is a small form library for Axolotl-PM plugins. It is a virion, not a standalone plugin. Load the source from the server's `virions/` directory during development or build a virion PHAR with DevTools for deployment.

[![Build](https://github.com/CzechPMDevs/libpmform/actions/workflows/build.yml/badge.svg)](https://github.com/CzechPMDevs/libpmform/actions/workflows/build.yml)
[![Apache-2.0 license](https://img.shields.io/badge/license-Apache--2.0-blue.svg)](LICENSE)

## Requirements

| Requirement | Version |
| --- | --- |
| PHP | `^8.4` |
| Axolotl-PM API | `5.0.0` |
| PHPStan development target | Axolotl-PM 5.51.2 |
| DevTools PHAR builds | DevTools 4.1.1 |

The manifest declares PHP 8.4 and API 5.0.0. Composer locks the static-analysis dependency to Axolotl-PM 5.51.2. PHPStan analysis and a successful PHAR build do not establish that the library was loaded by a running server.

## Install

For development, place the virion source folder in the server's `virions/` directory. DevTools can load it directly alongside plugin folders.

```text
server/
├── plugins/
│   └── MyPlugin/
└── virions/
    └── libpmform/
        ├── virion.yml
        └── src/
```

For deployment, build a virion PHAR from the repository root with the DevTools CLI. Install DevTools' locked dependencies first.

```sh
composer install --working-dir=/path/to/DevTools --no-interaction --no-progress --no-scripts --no-plugins
php -d phar.readonly=0 /path/to/DevTools/bin/devtools.php build --type=virion --project=/path/to/libpmform --out=/path/to/build
```

Copy `build/libpmform.phar` into the server's `virions/` directory. DevTools also builds and inspects this virion in the GitHub Actions workflow. The workflow uploads the PHAR and build metadata as an artifact.

## Create a form

Send a form from a plugin using the standard PocketMine form API. This example ignores a closed form and handles a button response only when it is an integer.

```php
use czechpmdevs\libpmform\response\FormResponse;
use czechpmdevs\libpmform\type\SimpleForm;
use pocketmine\player\Player;

$form = new SimpleForm("Example", "Choose an option", true);
$form->addButton("Continue");
$form->setCallback(static function (Player $player, FormResponse $response): void {
	$data = $response->getData();
	if (!is_int($data)) {
		return;
	}

	$player->sendMessage("Selected button {$data}.");
});

$player->sendForm($form);
```

The response contains the value received from the client. `FormResponse::getData()` returns it as `mixed`, so check its type before using it. A closed form has a `null` response. `FormResponse::isValid()` reports whether the response is non-null. The `SimpleForm` and `CustomForm` constructors, and `Form::ignoreInvalidResponse()`, can suppress the callback for a null response.

## Form types

| Class | Methods |
| --- | --- |
| `SimpleForm` | `addButton(string $text)` |
| `ModalForm` | Set the title, content, and two button labels in the constructor |
| `CustomForm` | `addInput()`, `addLabel()`, `addToggle()`, and `addDropdown()` |

Every form accepts a callback through `setCallback(Closure $callback)`. The callback receives the responding `Player` and a `FormResponse`. Custom forms preserve their element order in the serialized `content` list. Dropdown options are supplied as an array of strings.

## Development

Run the checks with PHP 8.4 and the PHP extensions required by Axolotl-PM's locked Composer dependencies.

```sh
composer install --prefer-dist --no-interaction --no-progress
composer validate --strict
composer analyse
```

`composer analyse` runs PHPStan at level `max`. The GitHub Actions workflow checks this configuration and builds a virion PHAR with DevTools 4.1.1.

## License

libpmform is licensed under the Apache License 2.0. See [LICENSE](LICENSE) for the full terms and original attribution.
