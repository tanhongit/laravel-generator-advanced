# Using Multiple Language on 

This is a tutorial on how to use multiple language support in the new version of the theme.

I will use the _example.php_ file to review this feature. 

## 1. Create a new config lang file for each language

- Language List: en, ja

Create example config file for each language

```
lang
├── en
│   └── example.php
└── ja
    └── example.php
```

## 2. Add the language key and value to this config file

**Note:** The language key must be the same as same on all lang folder

Its structure has the following form:

```php
<?php

return [
    'key' => 'value',
    'key2' => 'value2',
    'key3' => 'value3',
];
```

This is the example config file for **en** and **ja** language

```php
<?php
// ja/example.php

return [
    'invalid_name' => 'The name field is invalid.',
    'invalid_email' => 'The email field is invalid.',
];
```

```php
<?php
// ja/example.php

return [
    'invalid_name' => '名前フィールドが無効です。',
    'invalid_email' => 'メールフィールドが無効です。',
];
```

## 3. Use the language key in the template file

```php
<?php
// exampleView.php
echo App::getLocale();
echo '<br>';
echo __('example.invalid_name');
```

This is the result

En:
```
en
The name field is invalid.
```

Ja:
```
ja
名前フィールドが無効です。
```

## 4. Set key language with ResponseUtil

on **Utils/ResponseUtil.php**

import some constants with the value is the language key:

```php
public const ERR_NAME_INVALID = 'example.invalid_name';
public const ERR_EMAIL_INVALID = 'example.invalid_email';
```

and you can use it same as the example above:

```php
echo __(ResponseUtil::ERR_NAME_INVALID);
```

Another example:

```php
if (!$request->name) {
    return $this->errorResponse(__(ResponseUtil::ERR_NAME_INVALID), 422);
}
```

---

If you want to add some attributes to the language key, you can use the following this document: [https://laravel.com/docs/8.x/localization#replacing-parameters-in-translation-strings](https://laravel.com/docs/8.x/localization#replacing-parameters-in-translation-strings)

If you wish, you may define placeholders in your translation strings. All placeholders are prefixed with a :. For example, you may define a welcome message with a placeholder name:

```php
'welcome' => 'Welcome, :name',
```

on **Utils/ResponseUtil.php**

Import some constants with the value is the language key:

```php
public const WELCOME = 'example.welcome';
```

To replace the placeholders when retrieving a translation string, you may pass an array of replacements as the second argument to the __ function:

```php
echo __(ResponseUtil::WELCOME, ['name' => 'Everyone']);
```

**_However, you can see the value of name is not translated._**

The text 'Everyone' is not translated.

To solve this problem, you can create a new config file for each language and add this value of name to this config file.

Example:

Create the example2.php file in the lang folder

```
lang
├── en
│   ├── example2.php
│   └── example.php
└── ja
    ├── example2.php
    └── example.php
```

and add the following code to this file:

```
// en/example2.php
return [
    'name' => 'Everyone',
];
```

```
// ja/example2.php
return [
    'name' => 'みんな',
];
```

Now, you can use the language key in the template file:

```php
echo __(ResponseUtil::WELCOME, ['name' => __('example2.name')]);
```
