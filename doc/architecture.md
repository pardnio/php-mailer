# php-mailer - Architecture

Last updated: 2026-10-25

> Back to [README](../README.md)

## Overview

```mermaid
graph TB
    A[Caller] --> B[Mailer::send]
    A --> BK[Mailer::sendBulk]
    BK -->|per message + random delay| B
    ENV[MAIL_SERVICE_* env vars] --> C[Constructor SMTP setup]
    B --> C
    C --> D[_send message assembly]
    D --> E[PHPMailer]
    E --> F[SMTP server]
```

## Module: Mailer

`pardnchiu\Mailer` exposes two static methods and creates a fresh PHPMailer instance for every send.

```mermaid
graph TB
    subgraph Mailer
        CT[__construct] --> SMTP[isSMTP / SMTPAuth / Timeout 30]
        CT --> PORT{Port == 465?}
        PORT -->|Yes| SSL[SMTPS]
        PORT -->|No| TLS[STARTTLS]
        S[send] --> CT
        S --> PS[_send]
        PS --> FROM[setFrom]
        PS --> TO[addAddress]
        PS --> CC[addCC / addBCC]
        PS --> PR[Priority 1 / 3 / 5]
        PS --> BODY[isHTML / Body / AltBody]
        PS --> SEND[PHPMailer::send]
        SEND --> CLR[finally clear recipients]
        BULK[sendBulk] --> S
    end
    PM[phpmailer/phpmailer] --> CT
```

## Data Flow

```mermaid
sequenceDiagram
    participant Caller
    participant Mailer
    participant PHPMailer
    participant SMTP
    Caller->>Mailer: sendBulk(recipients, subject, body, options)
    loop each recipient
        Mailer->>Mailer: send(config)
        Mailer->>PHPMailer: create and configure from env vars
        Mailer->>PHPMailer: recipients / CC / body / priority
        PHPMailer->>SMTP: connect and send
        alt success
            SMTP-->>PHPMailer: 250 OK
            PHPMailer-->>Mailer: true
        else failure
            PHPMailer-->>Mailer: exception
            Mailer-->>Caller: throw send failure
        end
        Mailer->>Mailer: sleep(1~3 s)
    end
    Mailer-->>Caller: result array
```

***

©️ 2025 [邱敬幃 Pardn Chiu](https://www.linkedin.com/in/pardnchiu)
