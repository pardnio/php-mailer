# php-mailer - 架構

最後更新：2026-10-25

> 返回 [README](./README.zh.md)

## 概覽

```mermaid
graph TB
    A[呼叫端] --> B[Mailer::send]
    A --> BK[Mailer::sendBulk]
    BK -->|逐封 + 隨機延遲| B
    ENV[MAIL_SERVICE_* 環境變數] --> C[建構子 SMTP 設定]
    B --> C
    C --> D[_send 組裝郵件]
    D --> E[PHPMailer]
    E --> F[SMTP 伺服器]
```

## Module: Mailer

`pardnchiu\Mailer` 以兩個靜態方法對外，每次寄送都建立新的 PHPMailer 實例。

```mermaid
graph TB
    subgraph Mailer
        CT[__construct] --> SMTP[isSMTP / SMTPAuth / Timeout 30]
        CT --> PORT{Port == 465?}
        PORT -->|是| SSL[SMTPS]
        PORT -->|否| TLS[STARTTLS]
        S[send] --> CT
        S --> PS[_send]
        PS --> FROM[setFrom]
        PS --> TO[addAddress]
        PS --> CC[addCC / addBCC]
        PS --> PR[Priority 1 / 3 / 5]
        PS --> BODY[isHTML / Body / AltBody]
        PS --> SEND[PHPMailer::send]
        SEND --> CLR[finally 清空收件人]
        BULK[sendBulk] --> S
    end
    PM[phpmailer/phpmailer] --> CT
```

## Data Flow

```mermaid
sequenceDiagram
    participant 呼叫端
    participant Mailer
    participant PHPMailer
    participant SMTP
    呼叫端->>Mailer: sendBulk(recipients, subject, body, options)
    loop 每位收件人
        Mailer->>Mailer: send(config)
        Mailer->>PHPMailer: 依環境變數建立並設定
        Mailer->>PHPMailer: 收件人 / 副本 / 本文 / 優先級
        PHPMailer->>SMTP: 連線並寄送
        alt 成功
            SMTP-->>PHPMailer: 250 OK
            PHPMailer-->>Mailer: true
        else 失敗
            PHPMailer-->>Mailer: 例外
            Mailer-->>呼叫端: 拋出 郵件發送失敗
        end
        Mailer->>Mailer: sleep(1~3 秒)
    end
    Mailer-->>呼叫端: 結果陣列
```

***

©️ 2025 [邱敬幃 Pardn Chiu](https://www.linkedin.com/in/pardnchiu)
