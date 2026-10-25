# php-mailer - 技術文件

最後更新：2026-10-25

> 返回 [README](./README.zh.md)

## 前置需求

- PHP 8.0 以上
- Composer
- `phpmailer/phpmailer` `^6.9`（安裝時自動帶入）
- 可用的 SMTP 帳號（Gmail 需使用應用程式密碼）
- 環境變數需寫入 `$_ENV`（`variables_order` 含 `E`，或使用 `vlucas/phpdotenv` 等載入器）

## 安裝

### 使用 Composer

```bash
composer require pardnchiu/mailer
```

### 從原始碼

```bash
git clone https://github.com/pardnio/php-mailer.git
cd php-mailer
composer install
```

套件以 PSR-4 將 `pardnchiu\` 對應至 `src/`。

## 設定

### 環境變數

| 變數 | 必填 | 預設值 | 說明 |
|------|------|--------|------|
| `MAIL_SERVICE` | 是 | `""` | SMTP 主機，如 `smtp.gmail.com` |
| `MAIL_SERVICE_USER` | 是 | `""` | SMTP 帳號，也是預設寄件地址 |
| `MAIL_SERVICE_PASSWORD` | 是 | `""` | SMTP 密碼或應用程式密碼 |
| `MAIL_SERVICE_PORT` | 否 | `465` | `465` 使用 SMTPS，其餘埠號使用 STARTTLS |
| `MAIL_SERVICE_CHARSET` | 否 | `UTF-8` | 郵件編碼 |

```env
MAIL_SERVICE=smtp.gmail.com
MAIL_SERVICE_USER=your@gmail.com
MAIL_SERVICE_PASSWORD=your_app_password
MAIL_SERVICE_PORT=587
MAIL_SERVICE_CHARSET=UTF-8
```

固定行為：`SMTPAuth = true`、連線逾時 30 秒。

## 使用方式

### 基礎

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

### 多收件人、副本與優先級

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

### 進階：批次寄送

`sendBulk()` 逐封呼叫 `send()`，任一封失敗會拋出例外並中止剩餘寄送；結果陣列只包含已成功寄出的項目。

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

## API 參考

### Mailer::send

```php
public static function send(array $config): bool
```

建立新實例並寄出一封郵件，成功回傳 `true`；失敗拋出 `\Exception("郵件發送失敗: <PHPMailer ErrorInfo> - <原始訊息>")`。每次寄送後清空收件人、副本、密件副本與回覆地址。

| 鍵 | 型別 | 必填 | 預設值 | 說明 |
|----|------|------|--------|------|
| `email` | `string \| array` | 是 | - | 收件人；陣列可混用 `"addr"` 與 `"addr" => "name"` |
| `subject` | `string` | 是 | - | 主旨 |
| `body` | `string` | 是 | - | 本文 |
| `altBody` | `string` | 否 | - | 純文字備援本文 |
| `fromEmail` | `string` | 否 | `MAIL_SERVICE_USER` | 寄件地址 |
| `fromName` | `string` | 否 | - | 寄件人名稱 |
| `cc` | `string \| array` | 否 | `[]` | 副本，格式同 `email` |
| `bcc` | `string \| array` | 否 | `[]` | 密件副本，格式同 `email` |
| `priority` | `string` | 否 | - | `high`（1）／`normal`（3）／`low`（5），不分大小寫 |
| `isHtml` | `bool` | 否 | `false` | 本文是否為 HTML |

### Mailer::sendBulk

```php
public static function sendBulk(array $recipients, string $subject, string $body, array $options = []): array
```

對每位收件人各寄一封，`$options` 接受 `send()` 的其餘鍵；每封之間 `sleep(rand(1, 3))`。回傳以 `$recipients` 鍵為鍵、`send()` 結果為值的陣列。

| `$recipients` 格式 | 收件人 | 結果鍵 |
|--------------------|--------|--------|
| `["a@x.com" => "Name"]` | `a@x.com`（名稱 `Name`） | `a@x.com` |
| `["a@x.com"]` | `a@x.com` | `0` |

***

©️ 2025 [邱敬幃 Pardn Chiu](https://www.linkedin.com/in/pardnchiu)
