LoginGateBundle
==============

[![Build Status](https://travis-ci.org/anyx/LoginGateBundle.svg?branch=master)](https://travis-ci.org/anyx/LoginGateBundle)

This bundle detects brute-force attacks on Symfony applications. It then will disable login for attackers for a certain period of time.
This bundle also provides special events to execute custom handlers when a brute-force attack is detected.

## Compatability
Dev version for symfony 5 project

## Installation
Add this bundle via Composer:
```
composer require 41man/login-gate-bundle
```
## Configuration:

Also require manuall sql request call (for migration):
```php
$this->addSql('CREATE TABLE failure_login_attempt (id INT AUTO_INCREMENT NOT NULL, ip VARCHAR(45) NOT NULL, created_at DATETIME NOT NULL, data LONGTEXT NOT NULL COMMENT \'(DC2Type:array)\', INDEX ip (ip), PRIMARY KEY(id)) DEFAULT CHARACTER SET utf8mb4 COLLATE `utf8mb4_unicode_ci` ENGINE = InnoDB');
```

bundles.php
Add 
```php
Anyx\LoginGateBundle\LoginGateBundle::class => ['all' => true]
```

Add in /config/packages/login_gate.yaml:

```yml
login_gate:
    storages: ['orm'] # Attempts storages. Available storages: ['orm', 'session', 'mongodb']
    options:
        max_count_attempts: 3
        timeout: 600 #Ban period
        watch_period: 3600 #Only for databases storage. Period of actuality attempts
 ```
### Register event handler (optional).
```yml
services.yaml
Anyx\LoginGateBundle\Service\BruteForceChecker:
        class: Anyx\LoginGateBundle\Service\BruteForceChecker
        arguments: ['@anyx.login_gate.attempt_storage', '%anyx.login_gate.brute_force_checker_options%']
```

## Usage
In the following example we import the checker via dependency injection in SecurityController.php.
```php
namespace App\Controller;

use Anyx\LoginGateBundle\Service\BruteForceChecker;

/**
 * @var BruteForceChecker $bruteForceChecker
 */
private $bruteForceChecker;

/**
 * SecurityController constructor.
 * @param BruteForceChecker $bruteForceChecker
 */
public function __construct(BruteForceChecker $bruteForceChecker)
{
    $this->bruteForceChecker = $bruteForceChecker;
}
```
We can now use the checker to see if a person is allowed to login.
```php
$this->bruteForceChecker->canLogin($request)
```
We can also clear the loginattempts when a login is succesful.
```php
$this->bruteForceChecker->getStorage()->clearCountAttempts($request);
```

For more examples take a look at the [tests](https://github.com/anyx/LoginGateBundle/tree/master/Tests).
