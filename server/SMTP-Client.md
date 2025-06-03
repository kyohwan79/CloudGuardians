## 클라이언트 설정

### 클라이언트 정보
- **OS**: Ubuntu 18.04
- **IP 대역**: 192.168.70.* (메일 서버 제외)
- **역할**: 메일 발송 클라이언트
- **메일 서버**: 192.168.70.172 (mail.ysck.kr)

### 1. 기본 시스템 설정

1.1 패키지 설치
```bash
# 시스템 업데이트
sudo apt update

# 메일 발송용 패키지 설치
sudo apt install -y mailutils postfix

# 웹 기반 메일 폼용 패키지 (선택사항)
sudo apt install -y apache2 php libapache2-mod-php

# 기타 유용한 도구
sudo apt install -y telnet curl wget
```

1.2 네트워크 설정 확인
```bash
# IP 주소 확인
ip addr show

# 메일 서버 연결 테스트
ping 192.168.70.172
telnet 192.168.70.172 25
```

### 2. Postfix 클라이언트 설정 (메일 릴레이)

2.1 Postfix 설치 시 설정
```bash
# Postfix 설치 중 설정 선택사항:
# 1. "General type of mail configuration" -> "Satellite system" 선택
# 2. "SMTP relay host" -> "192.168.70.172" 입력
```

2.2 Postfix 수동 설정
```bash
# 기존 설정 백업
sudo cp /etc/postfix/main.cf /etc/postfix/main.cf.backup

# main.cf 설정 파일 생성
sudo tee /etc/postfix/main.cf > /dev/null << 'EOF'
# Debian/Ubuntu 기본 설정
smtpd_banner = $myhostname ESMTP $mail_name (Ubuntu)
biff = no
append_dot_mydomain = no
readme_directory = no
compatibility_level = 2

# TLS 설정 (기본값 유지)
smtpd_tls_cert_file=/etc/ssl/certs/ssl-cert-snakeoil.pem
smtpd_tls_key_file=/etc/ssl/private/ssl-cert-snakeoil.key
smtpd_use_tls=yes
smtpd_tls_session_cache_database = btree:${data_directory}/smtpd_scache
smtp_tls_session_cache_database = btree:${data_directory}/smtp_scache

# 메일 서버 릴레이 설정 (핵심 부분)
relayhost = [192.168.70.172]:25
myorigin = $mydomain
mydomain = ysck.kr
myhostname = $(hostname).ysck.kr

# 로컬 배송 비활성화 (모든 메일을 릴레이 서버로 전송)
mydestination = 
local_transport = error:local mail delivery is disabled

# 네트워크 설정
mynetworks = 127.0.0.0/8 [::ffff:127.0.0.0]/104 [::1]/128
inet_interfaces = loopback-only
inet_protocols = ipv4

# 릴레이 제한
smtpd_relay_restrictions = permit_mynetworks, permit_sasl_authenticated, defer_unauth_destination

# 기본 설정
alias_maps = hash:/etc/aliases
alias_database = hash:/etc/aliases
mailbox_size_limit = 0
recipient_delimiter = +
EOF
```

2.3 도메인 설정
```bash
# /etc/mailname 파일 설정
echo "ysck.kr" | sudo tee /etc/mailname

# 호스트명 확인 및 설정 (필요시)
hostnamectl status
# 필요시: sudo hostnamectl set-hostname client1.ysck.kr
```

2.4 Postfix 서비스 관리
```bash
# Postfix 재시작
sudo systemctl restart postfix

# 서비스 상태 확인
sudo systemctl status postfix

# 설정 확인
sudo postconf | grep -E 'relayhost|mydestination|myorigin|mydomain'

# 메일 큐 확인
sudo postqueue -p
```

### 3. 명령줄 메일 발송 방법

3.1 mail 명령어 사용
```bash
# 기본 메일 발송
echo "테스트 메시지입니다." | mail -s "테스트 제목" testuser@ysck.kr

# 파일 내용을 메일로 발송
mail -s "파일 첨부 테스트" testuser@ysck.kr < /path/to/file.txt

# 여러 수신자에게 발송
echo "단체 메일입니다." | mail -s "단체 발송" testuser@ysck.kr,admin@ysck.kr

# 발신자 지정하여 발송
echo "발신자 지정 메일" | mail -s "From 테스트" -a "From: sender@ysck.kr" testuser@ysck.kr
```

3.2 sendmail 명령어 사용
```bash
# sendmail로 직접 발송
sendmail testuser@ysck.kr << EOF
Subject: Sendmail 테스트
From: client@ysck.kr
To: testuser@ysck.kr

이것은 sendmail로 보낸 테스트 메일입니다.
.
EOF
```

### 4. 웹 기반 메일 폼 설정

4.1 Apache 웹 서버 설정
```bash
# Apache 서비스 시작 및 활성화
sudo systemctl enable apache2
sudo systemctl start apache2
sudo systemctl status apache2

# 방화벽 설정 (필요시)
sudo ufw allow 80/tcp
```

4.2 PHP 메일 폼 생성
```bash
# 웹 메일 폼 생성
sudo tee /var/www/html/mailform.php > /dev/null << 'EOF'
<!DOCTYPE html>
<html lang="ko">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>메일 발송 폼</title>
    <style>
        body { 
            font-family: 'Segoe UI', Arial, sans-serif; 
            max-width: 700px; 
            margin: 50px auto; 
            padding: 30px; 
            background-color: #f5f5f5;
        }
        .container {
            background-color: white;
            padding: 40px;
            border-radius: 10px;
            box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1);
        }
        h1 {
            color: #333;
            text-align: center;
            margin-bottom: 30px;
            border-bottom: 3px solid #007cba;
            padding-bottom: 10px;
        }
        .form-group { 
            margin-bottom: 20px; 
        }
        label { 
            display: block; 
            margin-bottom: 8px; 
            font-weight: bold; 
            color: #555;
        }
        input[type="text"], input[type="email"], textarea, select { 
            width: 100%; 
            padding: 12px; 
            border: 2px solid #ddd; 
            border-radius: 6px; 
            font-size: 14px;
            transition: border-color 0.3s;
        }
        input[type="text"]:focus, input[type="email"]:focus, textarea:focus, select:focus {
            border-color: #007cba;
            outline: none;
        }
        textarea { 
            height: 120px; 
            resize: vertical; 
            font-family: inherit;
        }
        button { 
            background-color: #007cba; 
            color: white; 
            padding: 12px 30px; 
            border: none; 
            border-radius: 6px; 
            cursor: pointer; 
            font-size: 16px; 
            font-weight: bold;
            transition: background-color 0.3s;
            width: 100%;
        }
        button:hover { 
            background-color: #005a87; 
        }
        .success { 
            color: #28a745; 
            font-weight: bold; 
            background-color: #d4edda;
            padding: 10px;
            border: 1px solid #c3e6cb;
            border-radius: 4px;
            margin-bottom: 20px;
        }
        .error { 
            color: #dc3545; 
            font-weight: bold; 
            background-color: #f8d7da;
            padding: 10px;
            border: 1px solid #f5c6cb;
            border-radius: 4px;
            margin-bottom: 20px;
        }
        .info {
            background-color: #e7f3ff;
            border: 1px solid #b8daff;
            color: #0c5460;
            padding: 15px;
            border-radius: 6px;
            margin-bottom: 20px;
        }
        .footer {
            text-align: center;
            margin-top: 30px;
            color: #666;
            font-size: 12px;
        }
    </style>
</head>
<body>
    <div class="container">
        <h1>📧 메일 발송 시스템</h1>
        
        <div class="info">
            <strong>📍 클라이언트 정보:</strong><br>
            서버: <?php echo $_SERVER['SERVER_NAME']; ?> (<?php echo $_SERVER['SERVER_ADDR']; ?>)<br>
            메일 서버: mail.ysck.kr (192.168.100.172)
        </div>
        
        <?php
        if ($_POST) {
            $to = $_POST['to'];
            $subject = $_POST['subject'];
            $message = $_POST['message'];
            $from = $_POST['from'];
            $priority = $_POST['priority'];
            
            // 메일 헤더 구성
            $headers = array();
            $headers[] = "From: " . $from;
            $headers[] = "Reply-To: " . $from;
            $headers[] = "X-Mailer: PHP/" . phpversion();
            $headers[] = "Content-Type: text/plain; charset=UTF-8";
            
            // 우선순위 설정
            if ($priority == 'high') {
                $headers[] = "X-Priority: 1";
                $headers[] = "Importance: High";
            } elseif ($priority == 'low') {
                $headers[] = "X-Priority: 5";
                $headers[] = "Importance: Low";
            }
            
            $header_string = implode("\r\n", $headers);
            
            // 메일 발송
            if (mail($to, $subject, $message, $header_string)) {
                echo '<div class="success">✅ 메일이 성공적으로 발송되었습니다!<br>';
                echo '<strong>수신자:</strong> ' . htmlspecialchars($to) . '<br>';
                echo '<strong>제목:</strong> ' . htmlspecialchars($subject) . '</div>';
            } else {
                echo '<div class="error">❌ 메일 발송에 실패했습니다.<br>';
                echo '메일 서버 연결을 확인해주세요.</div>';
            }
        }
        ?>
        
        <form method="POST" action="">
            <div class="form-group">
                <label for="from">📤 보내는 사람:</label>
                <input type="email" id="from" name="from" value="client@ysck.kr" required>
            </div>
            
            <div class="form-group">
                <label for="to">📥 받는 사람:</label>
                <select id="to" name="to" required>
                    <option value="">-- 수신자 선택 --</option>
                    <option value="testuser@ysck.kr">testuser@ysck.kr</option>
                    <option value="admin@ysck.kr">admin@ysck.kr</option>
                    <option value="test@ysck.kr">test@ysck.kr</option>
                </select>
            </div>
            
            <div class="form-group">
                <label for="priority">🔥 우선순위:</label>
                <select id="priority" name="priority">
                    <option value="normal">보통</option>
                    <option value="high">높음</option>
                    <option value="low">낮음</option>
                </select>
            </div>
            
            <div class="form-group">
                <label for="subject">📝 제목:</label>
                <input type="text" id="subject" name="subject" placeholder="메일 제목을 입력하세요" required>
            </div>
            
            <div class="form-group">
                <label for="message">💬 내용:</label>
                <textarea id="message" name="message" placeholder="메일 내용을 입력하세요" required></textarea>
            </div>
            
            <button type="submit">📨 메일 발송</button>
        </form>
        
        <div class="footer">
            YSCK 메일 시스템 | Client: <?php echo gethostname(); ?>
        </div>
    </div>
</body>
</html>
EOF

# 권한 설정
sudo chown www-data:www-data /var/www/html/mailform.php
sudo chmod 644 /var/www/html/mailform.php
```

4.3 웹 폼 접속 정보
```bash
# 클라이언트 IP 확인
CLIENT_IP=$(hostname -I | awk '{print $1}')
echo "웹 메일 폼 접속 URL: http://${CLIENT_IP}/mailform.php"
```


### 5. 테스트 및 검증

5.1 연결 테스트
```bash
# SMTP 서버 연결 테스트
telnet 192.168.70.172 25
# 연결 후 다음 명령어 테스트:
# EHLO client.ysck.kr
# MAIL FROM: client@ysck.kr
# RCPT TO: testuser@ysck.kr
# DATA
# Subject: Connection Test
# 
# This is a connection test.
# .
# QUIT

# IMAP 서버 연결 테스트
telnet 192.168.70.172 143
```

5.2 메일 발송 테스트
```bash
# 명령줄 테스트
echo "클라이언트 테스트 메일입니다." | mail -s "클라이언트 테스트" testuser@ysck.kr

# Python 스크립트 테스트
python3 ~/send_mail.py

# 웹 폼 테스트
curl -X POST -d "from=client@ysck.kr&to=testuser@ysck.kr&subject=cURL테스트&message=cURL로 보낸 메일입니다." http://localhost/mailform.php
```

5.3 로그 확인
```bash
# Postfix 로그 확인
sudo tail -f /var/log/mail.log

# Apache 로그 확인 (웹 폼 사용시)
sudo tail -f /var/log/apache2/access.log
sudo tail -f /var/log/apache2/error.log

# 메일 큐 상태 확인
sudo postqueue -p

# 메일 통계 확인
sudo pflogsumm /var/log/mail.log
```



### 클라이언트 정보 요약
접속 정보</br>
- **메일 서버**: 192.168.70.172 (mail.ysck.kr)</br>
- **SMTP 포트**: 25</br>
- **IMAP 포트**: 143 (메일 읽기용)</br>
- **도메인**: ysck.kr</br>

주요 설정 파일
```bash
# Postfix 설정
/etc/postfix/main.cf

# 도메인 설정
/etc/mailname

# 웹 폼
/var/www/html/mailform.php

# 사용자 스크립트
~/send_mail.py
~/bulk_mail.sh
~/mail_status.sh
```

접속 URL (웹 폼)
```bash
# 각 클라이언트에서 웹 폼 접속
http://[클라이언트_IP]/mailform.php
```
