# Joomla 3 Component Upgrade Rectors

Rector rules to easily upgrade Joomla 3 components to Joomla 4 MVC

Copyright (C) 2022  Nicholas K. Dionysopoulos

This program is free software; you can redistribute it and/or modify
it under the terms of the GNU General Public License as published by
the Free Software Foundation; either version 2 of the License, or
(at your option) any later version.

This program is distributed in the hope that it will be useful,
but WITHOUT ANY WARRANTY; without even the implied warranty of
MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
GNU General Public License for more details.

You should have received a copy of the GNU General Public License along
with this program; if not, write to the Free Software Foundation, Inc.,
51 Franklin Street, Fifth Floor, Boston, MA 02110-1301 USA.

## Sponsors welcome

Do you have a Joomla extensions development business? Are you a web agency using tons of custom components? Maybe you can sponsor this work! It will save you tons of time — in the order of dozens of hours per component.

Sponsorships will help me spend more time working on this tool, the Joomla extension developer's documentation and core Joomla code.

If you're interested hit me up at [the Contact Me page](https://www.dionysopoulos.me/contact-me.html?view=item)! You'll get my gratitude and your logo on this page.

## Requirements

* Rector 0.14
* PHP 7.2 or later; 8.1 or later with XDebug _turned off_ recommended for best performance
* Composer 2.x

Your component project must have the structure described below.

* Your component's backend code must be in a folder named `administrator`, `admin`, `backend` or `administrator/components/com_yourcomponent` (where `com_yourcomponent` is the name of your component).

* Your component's frontend code must be in a folder named `site`, `frontend`, or `components/com_yourcomponent` (where `com_yourcomponent` is the name of your component).

* Your component's media files must be in a folder named `media`, or `media/com_yourcomponent` (where `com_yourcomponent` is the name of your component).

## What is this all about?

This repository provides Rector rules to automatically refactor your legacy Joomla 3 component into Joomla 4 MVC. It does not do everything, it just handles the boring, repeated and soul–crushing work. You will still need to do a few things.

**What it does**
* Namespace all of your MVC (Model, Controller, View and Table) classes and place them into the appropriate directories.

**What I would like to add**
* Refactor and namespace helper classes.
* Refactor and namespace custom form field classes.
* Refactor and namespace custom form rule classes.
* Refactor static getInstance calls to the base model and table classes.
* Refactor getModel and getView calls in controllers.
* Rename language files so that they do NOT have a language prefix.
* Move view templates into the new folder structure.
* Update the XML manifest with the namespace prefix.
* Update the XML manifest with the new language file prefixes.
* Create a basic `services/provider.php` file.
* Move backend and frontend XML forms to the appropriate folders.
* Replace `addfieldpath` with `addfieldprefix` in XML forms.

**What it CAN NOT and WILL NOT do**
* Change typehints in PHP docblocks. While possible in Rector it's too convoluted and time–consuming. Sorry.
* Remove your old entry point file, possibly converting it to a custom Dispatcher.
* Convert custom HTML helpers into a custom HTML service for your component.
* Refactor static getInstance calls to _descendants of_ the base model and table classes.

## How to use

Checkout your component's repository.

Update your `composer.json` file with the following:

```json
{
  "repositories": [
    {
      "name": "nikosdion/joomla_typehints",
      "type": "vcs",
      "url": "https://github.com/nikosdion/joomlatypehints"
    },
    {
      "name": "nikosdion/joomla_com_upgrader",
      "type": "vcs",
      "url": "https://github.com/nikosdion/joomla_com_upgrader"
    }
  ],
  "require-dev": {
    "rector/rector": "^0.14.0",
    "nikosdion/joomla_typehints": "*",
    "nikosdion/joomla_com_upgrader": "*",
    "friendsofphp/php-cs-fixer": "^3.0"
  }
}
```

Run `composer update --dev` to install the dependencies.

Create a new `rector.php` in your component project's root with the following contents:

```php
<?php

declare(strict_types=1);

use Rector\Config\RectorConfig;
use Rector\Set\ValueObject\LevelSetList;
use Rector\Set\ValueObject\SetList;
use Supercalifragilisticexpialidocious\Config\JoomlaLegacyPrefixToNamespace;
use Supercalifragilisticexpialidocious\Rector\ConvertJoomlaMVCNamespaces;

return static function (RectorConfig $rectorConfig): void {
    $rectorConfig->disableParallel();

    $rectorConfig->paths([
        __DIR__ . '/admin',
        __DIR__ . '/site',
        __DIR__ . '/script.php',
        // Add any more directories or files your project may be using here
    ]);

        $rectorConfig->sets([
            // Auto-refactor code to at least PHP 7.2 (minimum Joomla version)
            LevelSetList::UP_TO_PHP_72,
            // Replace legacy class names with the namespaced ones
            __DIR__ . '/vendor/nikosdion/joomla_typehints/rector/joomla_4_0.php',
            // Use early returns in if-blocks (code quality)
            SetList::EARLY_RETURN,
        ]);

    // Auto-refactor the Joomla MVC classes
    $rectorConfig->ruleWithConfiguration(ConvertJoomlaMVCNamespaces::class, [
        new JoomlaLegacyPrefixToNamespace('Helloworld', 'Acme\HelloWorld', []),
        new JoomlaLegacyPrefixToNamespace('HelloWorld', 'Acme\HelloWorld', []),
    ]);

    // Replace Fully Qualified Names (FQN) of classes with `use` imports at the top of the file.
    $rectorConfig->importNames();
    // Do NOT import short class names such as `DateTime`
    $rectorConfig->importShortClasses(false);
};
```

The lines you need to change are:
```php
    // Auto-refactor the Joomla MVC classes
    $rectorConfig->ruleWithConfiguration(ConvertJoomlaMVCNamespaces::class, [
        new JoomlaLegacyPrefixToNamespace('Helloworld', 'Acme\HelloWorld', []),
        new JoomlaLegacyPrefixToNamespace('HelloWorld', 'Acme\HelloWorld', []),
    ]);
```
where `HelloWorld` is the name of your component without the `com_` prefix and `Acme\HelloWorld` is the namespace prefix you want to use for your component. It is recommended to use the convention `CompanyName\ComponentNameWithoutCom` or `CompanyName\Component\ComponentNameWithoutCom` for your namespace prefix.

Now you can run Rector to do _a hell of a lot_ of the refactoring necessary to convert your component to Joomla 4 MVC:

```bash
php ./vendor/bin/rector --dry-run
```

The `--dry-run` parameter prints out the changes. Removing it will apply these changes _for real_ (files will be modified).