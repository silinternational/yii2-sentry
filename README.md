# [Sentry](https://sentry.io) logger for Yii2

## Installation

```bash
composer require silinternational/yii2-sentry
```

Add target class in the application config:

```php
return [
    'components' => [
        'log' => [
            'traceLevel' => YII_DEBUG ? 3 : 0,
            'targets' => [
                [
                    'class' => 'Sil\Sentry\SentryTarget',
                    'dsn' => 'http://2682ybvhbs347:235vvgy465346@sentry.io/1',
                    'levels' => ['error', 'warning'],
                    // Write the context information (the default is true):
                    'context' => true,
                    // Additional options for `Sentry\init`:
                    'clientOptions' => ['release' => 'my-project-name@2.3.12']
                ],
            ],
        ],
    ],
];
```

## Usage

Writing simple message:

```php
\Yii::error('message', 'category');
```

Writing messages with extra data:

```php
\Yii::warning([
    'msg' => 'message',
    'extra' => 'value',
], 'category');
```

### Extra callback

`extraCallback` property can modify extra's data as callable function:

```php
    'targets' => [
        [
            'class' => 'Sil\Sentry\SentryTarget',
            'dsn' => 'http://2682ybvhbs347:235vvgy465346@sentry.io/1',
            'levels' => ['error', 'warning'],
            'context' => true, // Write the context information. The default is true.
            'extraCallback' => function ($message, $extra) {
                // some manipulation with data
                $extra['some_data'] = \Yii::$app->someComponent->someMethod();
                return $extra;
            }
        ],
    ],
```

### Tags

Writing messages with additional tags. If need to add additional tags for event, add `tags` key in message. Tags are
various key/value pairs that get assigned to an event, and can later be used as a breakdown or quick access to finding
related events.

Example:

```php
\Yii::warning([
    'msg' => 'message',
    'extra' => 'value',
    'tags' => [
        'extraTagKey' => 'extraTagValue',
    ]
], 'category');
```

If all messages need the same tag, use tagCallback in SentryTarget config:

```php
    'targets' => [
        [
            'class' => 'Sil\Sentry\SentryTarget',
            'dsn' => 'https://11111111111111111111111111111111@11111111111111111.ingest.us.sentry.io/1111111111111111',
            'levels' => ['error', 'warning'],
            'tagCallback' => function ($tags) {
                $tags['foo'] = 'bar';
                return $tags;
            },
        ],
    ],
```

More about tags see https://docs.sentry.io/learn/context/#tagging-events

### Additional context

You can add additional context (such as user information, fingerprint, etc) by calling `\Sentry\configureScope()` before
logging.
For example in main configuration on `beforeAction` event (real place will dependant on your project):

```php
return [
    // ...
    'on beforeAction' => function (\yii\base\ActionEvent $event) {
        /** @var \yii\web\User $user */
        $user = Yii::$app->has('user', true) ? Yii::$app->get('user', false) : null;
        if ($user && ($identity = $user->getIdentity(false))) {
            \Sentry\configureScope(function (\Sentry\State\Scope $scope) use ($identity) {
                $scope->setUser([
                    // User ID and IP will be added by logger automatically
                    'username' => $identity->username,
                    'email' => $identity->email,
                ]);
            });
        }
    
        return $event->isValid;
    },
    // ...
];
```

## Log levels

Yii2 log levels converts to Sentry levels:

```
\yii\log\Logger::LEVEL_ERROR => 'error',
\yii\log\Logger::LEVEL_WARNING => 'warning',
\yii\log\Logger::LEVEL_INFO => 'info',
\yii\log\Logger::LEVEL_TRACE => 'debug',
\yii\log\Logger::LEVEL_PROFILE_BEGIN => 'debug',
\yii\log\Logger::LEVEL_PROFILE_END => 'debug',
```
