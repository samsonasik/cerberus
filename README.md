# Cerberus

UNDER DEVELOPMENT!

## Introduction

This is a Circuit Breaker pattern implementation in PHP.

This library helps you to handle external services timeouts and outages.

It detects service failures and adapts itself. 

You can combine this library with [Kharon](https://github.com/mt-olympus/kharon) to have a realiable service Load Balance.   

## Requirements

* PHP >= 5.5
* [Zend Cache](https://github.com/zendframework/zend-cache)

## Instalation

```
composer require mt-olympus/cerberus:~1.0
```

## Configuration

You can manually create a Cerberus instance or use a Factory

### Factory

If you use a [Container Interopt](https://github.com/container-interop/container-interop) campatible project,
you can define a factory:

```php
'factories' => [
    Cerberus\Cerberus::class => Cerberus\Factory::class
],
```

and copy the configuration file config/cerberus.global.php.dist to your config/autoload/cerberus.global.php and change to your needs.

```php
return [
    'cerberus' => [
        'max_failues' => 5,
        'timeout' => 60,
        'storage' => [
            'adapter' => [
                'name' => 'filesystem',
                'options' => [
                    'cache_dir' => 'data/cache',
                    'namespace' => 'my_project'
                ]
            ],
            'plugins' => [
                // Don't throw exceptions on cache errors
                'exception_handler' => [
                    'throw_exceptions' => false
                ]
            ]
        ]
    ]
];
```

The `maxFailure` parameter is the number of failures after which the circuit is opened and the service becomes not available.

When the `timeout` is reached, the circuit becomes half opened and one attempt is possible and the status is updated.

The storage key is a zend-cache configuration. You can check the [official documentation](https://github.com/zendframework/zend-cache).

The `namespace` key inside the storage is important. It defines de default namespace for Cerberus cache capabilities. If
you choose to call Cerberus methods with service name (see Usage bellow), you can ommit this as it will be ignored.  

### Manually

You can create a Cerberus instance manually:

```php
$storage = StorageFactory::factory($storageConfig);
$cerberus = new Cerberus($storage, 5, 60);
```

The $storageConfig is the zend-cache configuration as above.

## Usage

The usage is simple. Each time you will access a remote resource (like an Web Service), check for its availability and report its success or failure:

```php
if ($cerberus->isAvailable()) {
    try {
        $http->makeRequest();
        $cerberus->reportSuccess();
    } catch (\Exception $ex) {
        $cerberus->reportFailure();
    }
}
``` 

You can use Cerberus to control more than one service. In this scenario, use the methods passing a service name:

```php
if ($cerberus->isAvailable('service-one')) {
    try {
        $http->makeRequest();
        $cerberus->reportSuccess('service-one');
    } catch (\Exception $ex) {
        $cerberus->reportFailure('service-one');
    }
}

if ($cerberus->isAvailable('service-two')) {
    try {
        $http->makeRequest();
        $cerberus->reportSuccess('service-two');
    } catch (\Exception $ex) {
        $cerberus->reportFailure('service-two');
    }
}
```
