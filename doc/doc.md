# php-mailer - Documentation

Last updated: 2026-10-25

> Back to [README](../README.md)

## Prerequisites

- PHP 8.0 or higher
- Composer
- `phpmailer/phpmailer` `^6.9` (installed automatically)
- A working SMTP account (Gmail requires an app password)
- Environment variables available in `$_ENV` (`variables_order` containing `E`, or a loader such as `vlucas/phpdotenv`)

## Installation

### Using Composer

```bash
composer require pardnchiu/mailer
```

### From Source

```bash
git clone https://github.com/pardnio/php-mailer.git
cd php-mailer
composer install
```

The package maps `pardnchiu\` to `src/` via PSR-4.

## Configuration

### Environment Variables

| Variable | Required | Default | Description |
|----------|----------|---------|-------------|
| `MAIL_SERVICE` | Yes | `""` | SMTP host, e.g. `smtp.gmail.com` |
| `MAIL_SERVICE_USER` | Yes | `""` | SMTP username, also the default sender address |
| `MAIL_SERVICE_PASSWORD` | Yes | `""` | SMTP password or app password |
| `MAIL_SERVICE_PORT` | No | `465` | `465` uses SMTPS; every other port uses STARTTLS |
| `MAIL_SERVICE_CHARSET` | No | `UTF-8` | Message charset |

```env
MAIL_SERVICE=smtp.gmail.com
MAIL_SERVICE_USER=your@gmail.com
MAIL_SERVICE_PASSWORD=your_app_password
MAIL_SERVICE_PORT=587
MAIL_SERVICE_CHARSET=UTF-8
```

Fixed behavior: `SMTPAuth = true` and a 30-second connection timeout.

## Usage

### Basic

```php
<?php

require __DIR__ . '/vendor/autoload.php';

use pardnchiu\Mailer;

try {
    Mailer::send([
        "email"   => "recipient@example.com",
        "subject" => "Test Email",
        "body"    => "<h1>Hello</h1>",
        "isHtml"  => true,
    ]);
    echo "Sent";
} catch (\Exception $e) {
    error_log($e->getMessage());
}
```

### Multiple Recipients, CC, and Priority

```php
<?php

require __DIR__ . '/vendor/autoload.php';

use pardnchiu\Mailer;

try {
    Mailer::send([
        "email" => [
            "user1@example.com" => "User One",
            "user2@example.com",
        ],
        "cc"        => ["cc@example.com" => "CC User"],
        "bcc"       => "audit@example.com",
        "subject"   => "Release Notice",
        "body"      => "<p>v1.0.0 is out.</p>",
        "altBody"   => "v1.0.0 is out.",
        "fromEmail" => "noreply@example.com",
        "fromName"  => "Release Bot",
        "priority"  => "high",
        "isHtml"    => true,
    ]);
} catch (\Exception $e) {
    error_log($e->getMessage());
}
```

### Advanced: Bulk Sending

`sendBulk()` calls `send()` once per recipient; any failure throws and stops the remaining sends, so the result array only holds messages that went out.

```php
<?php

require __DIR__ . '/vendor/autoload.php';

use pardnchiu\Mailer;

$recipients = [
    "user1@example.com" => "User One",
    "user2@example.com" => "User Two",
    "user3@example.com" => "User Three",
];

try {
    $results = Mailer::sendBulk(
        $recipients,
        "Monthly Report",
        "<h1>October Report</h1>",
        ["isHtml" => true, "fromName" => "Reports"]
    );
    echo count($results) . " / " . count($recipients) . " sent";
} catch (\Exception $e) {
    error_log($e->getMessage());
}
```

## API Reference

### Mailer::send

```php
public static function send(array $config): bool
```

Creates a new instance and sends one message, returning `true` on success; failures throw `\Exception("郵件發送失敗: <PHPMailer ErrorInfo> - <original message>")`. Recipients, CC, BCC, and reply-to lists are cleared after each send.

| Key | Type | Required | Default | Description |
|-----|------|----------|---------|-------------|
| `email` | `string \| array` | Yes | - | Recipients; arrays may mix `"addr"` and `"addr" => "name"` |
| `subject` | `string` | Yes | - | Subject |
| `body` | `string` | Yes | - | Body |
| `altBody` | `string` | No | - | Plain-text alternative body |
| `fromEmail` | `string` | No | `MAIL_SERVICE_USER` | Sender address |
| `fromName` | `string` | No | - | Sender name |
| `cc` | `string \| array` | No | `[]` | CC, same format as `email` |
| `bcc` | `string \| array` | No | `[]` | BCC, same format as `email` |
| `priority` | `string` | No | - | `high` (1) / `normal` (3) / `low` (5), case-insensitive |
| `isHtml` | `bool` | No | `false` | Whether the body is HTML |

### Mailer::sendBulk

```php
public static function sendBulk(array $recipients, string $subject, string $body, array $options = []): array
```

Sends one message per recipient; `$options` takes the remaining `send()` keys, with `sleep(rand(1, 3))` between messages. Returns an array keyed by the `$recipients` keys with each `send()` result as value.

| `$recipients` format | Recipient | Result key |
|----------------------|-----------|------------|
| `["a@x.com" => "Name"]` | `a@x.com` (name `Name`) | `a@x.com` |
| `["a@x.com"]` | `a@x.com` | `0` |

***

©️ 2025 [邱敬幃 Pardn Chiu](https://www.linkedin.com/in/pardnchiu)
