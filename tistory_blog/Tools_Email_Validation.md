> 직접 구현에 대한 내용은 이렇게 동작하는구나 참고만 하시고  
> 유료 서비스나 일정양을 무료로 제공하는 서비스를 잘 찾아서 사용하시기 바랍니다.

# 이메일 유효성 검증

1. 직접 구현
2. Online Email Validators 혹은 라이브러리 사용

## 1. 직접 구현

> `구문 검증` <g-emoji>👉</g-emoji> `DNS 조회` <g-emoji>👉</g-emoji> `Email Box 통신 확인` 순으로 진행

> <g-emoji>🚫</g-emoji> 메일회사 정책에 따라 DNS 조회나 Email Box 통신 확인이 정상적으로 작동하지 않을 수 있고,  
> <g-emoji>👩‍💻</g-emoji> 반복적으로 여러번 시도할 경우 해킹 시도로 오해받을 수 있으니 주의하시기 바랍니다.

### 구문 검증

- Javascript 정규식을 통해 처리 가능
- RFC 2822 standard email validation  
  https://www.w3resource.com/javascript/form/email-validation.php
  ```js
  function validateEmail(email) {
    const re = "정규식추가";
    return re.test(String(email).toLowerCase());
  }
  ```

### DNS 조회

```bash
$> nslookup -type=mx gmail.com
Server:         172.29.128.1
Address:        172.29.128.1#53

Non-authoritative answer:
gmail.com       mail exchanger = 5 gmail-smtp-in.l.google.com.
gmail.com       mail exchanger = 10 alt1.gmail-smtp-in.l.google.com.
gmail.com       mail exchanger = 20 alt2.gmail-smtp-in.l.google.com.
gmail.com       mail exchanger = 40 alt4.gmail-smtp-in.l.google.com.
gmail.com       mail exchanger = 30 alt3.gmail-smtp-in.l.google.com.
Name:   gmail-smtp-in.l.google.com
Address: 64.233.189.27
Name:   gmail-smtp-in.l.google.com
Address: 2404:6800:4008:c07::1b
Name:   alt1.gmail-smtp-in.l.google.com
Address: 142.250.141.27
Name:   alt1.gmail-smtp-in.l.google.com
Address: 2607:f8b0:4023:c0b::1b
Name:   alt2.gmail-smtp-in.l.google.com
Address: 142.250.115.26
Name:   alt2.gmail-smtp-in.l.google.com
Address: 2607:f8b0:4023:1004::1a
Name:   alt4.gmail-smtp-in.l.google.com
Address: 142.250.152.26
Name:   alt4.gmail-smtp-in.l.google.com
Address: 2607:f8b0:4001:c56::1b
Name:   alt3.gmail-smtp-in.l.google.com
Address: 64.233.171.26
Name:   alt3.gmail-smtp-in.l.google.com
Address: 2607:f8b0:4003:c15::1b

Authoritative answers can be found from:
```

### Email Box 통신 확인

```bash
$> telnet gmail-smtp-in.l.google.com 25
Trying 74.125.23.26...
Connected to gmail-smtp-in.l.google.com.
Escape character is '^]'.
220 mx.google.com ESMTP b17si36903571pgb.238 - gsmtp
# SMTP와 대화 시작
EHLO google.com
250-mx.google.com at your service, [121.133.136.6]
250-SIZE 157286400
250-8BITMIME
250-STARTTLS
250-ENHANCEDSTATUSCODES
250-PIPELINING
250-CHUNKING
250 SMTPUTF8
# 발신자 입력
mail from:<awoodongs@gmail.com>
250 2.1.0 OK b17si36903571pgb.238 - gsmtp
# 정상 수신자 입력
rcpt to:<cerealtigerpower@gmail.com>
250 2.1.5 OK b17si36903571pgb.238 - gsmtp
# 잘못된 수신자 입력
rcpt to:<!test!@gmail.com>
550-5.1.1 The email account that you tried to reach does not exist. Please try
550-5.1.1 double-checking the recipient's email address for typos or
550-5.1.1 unnecessary spaces. Learn more at
550 5.1.1  https://support.google.com/mail/?p=NoSuchUser b17si36903571pgb.238 - gsmtp
QUIT
221 2.0.0 closing connection c13si10201160pgw.160 - gsmtp
Connection closed by foreign host.
```

## 2. Online Email Validators 혹은 라이브러리 사용

### Online Email Validators : ClearOut

- https://app.clearout.io/
- 비용 지불 방식 ([📑 링크](https://app.clearout.io/dashboard/account/pricing))

### 라이브러리 : check-if-email-exists

- Repository : https://github.com/reacherhq/check-if-email-exists
- Live Demo : https://reacher.email/
- Docker 로 사용 가능
  ```bash
  $> docker run -p 3000:3000 reacherhq/check-if-email-exists
  ```
- Return Value
  ```json
  {
    "input": "cerealtigerpower@gmail.com",
    "is_reachable": "safe",
    "misc": {
      "is_disposable": false,
      "is_role_account": false
    },
    "mx": {
      "accepts_mail": true,
      "records": ["alt3.gmail-smtp-in.l.google.com.", "alt1.gmail-smtp-in.l.google.com.", "alt4.gmail-smtp-in.l.google.com.", "alt2.gmail-smtp-in.l.google.com.", "gmail-smtp-in.l.google.com."]
    },
    "smtp": {
      "can_connect_smtp": true,
      "has_full_inbox": false,
      "is_catch_all": false,
      "is_deliverable": true,
      "is_disabled": false
    },
    "syntax": {
      "address": "cerealtigerpower@gmail.com",
      "domain": "gmail.com",
      "is_valid_syntax": true,
      "username": "cerealtigerpower"
    }
  }
  ```
