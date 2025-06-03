## SMTP 메일 서버 설정

서버 정보
- **OS: Rocky Linux 8**
- IP: 192.168.70.172
- 호스트명: mail.ysck.kr
- 도메인: ysck.kr

### 1. 시스템 기본 설정

1.1 패키지 설치
```bash
# 시스템 업데이트
sudo dnf update -y

# 메일 서버 패키지 설치
sudo dnf install -y postfix dovecot mailx telnet nc
sudo dnf install -y httpd php php-mysql php-imap php-mysqli php-mbstring php-intl php-zip mariadb

# 웹메일 관련 패키지
sudo dnf install -y wget unzip
```

1.2 방화벽 설정
```bash
# 메일 서비스 포트 열기
sudo firewall-cmd --permanent --add-service=smtp          # 25
sudo firewall-cmd --permanent --add-service=smtps         # 465
sudo firewall-cmd --permanent --add-service=imap          # 143
sudo firewall-cmd --permanent --add-service=imaps         # 993
sudo firewall-cmd --permanent --add-service=pop3          # 110
sudo firewall-cmd --permanent --add-service=pop3s         # 995
sudo firewall-cmd --permanent --add-service=http          # 80
sudo firewall-cmd --permanent --add-service=https         # 443
sudo firewall-cmd --reload
```

1.3 SELinux 설정
```bash
# 테스트 환경에서 SELinux 비활성화
sudo setenforce 0
```

1.4 호스트명 설정
```bash
# 호스트명 설정
sudo hostnamectl set-hostname mail.ysck.kr

# /etc/hosts 파일 수정
echo "192.168.70.172 mail.ysck.kr mail" | sudo tee -a /etc/hosts
```

### 2. Postfix (SMTP 서버) 설정

2.1 main.cf 설정 파일
```bash
# 기존 설정 백업
sudo cp /etc/postfix/main.cf /etc/postfix/main.cf.backup

# main.cf 설정
sudo tee /etc/postfix/main.cf > /dev/null << 'EOF'
# 기본 설정
myhostname = mail.ysck.kr
mydomain = ysck.kr
myorigin = $mydomain
inet_interfaces = all
inet_protocols = ipv4
mydestination = $myhostname, localhost.$mydomain, localhost, $mydomain

# 네트워크 설정
mynetworks = 192.168.70.0/24, 127.0.0.0/8

# 메일 저장 방식 (로컬 배송)
home_mailbox = Maildir/

# 릴레이 설정
mynetworks_style = subnet
smtpd_recipient_restrictions = permit_mynetworks, reject_unauth_destination
smtpd_relay_restrictions = permit_mynetworks, defer_unauth_destination

# 기본 설정
biff = no
append_dot_mydomain = no
readme_directory = no
compatibility_level = 2
EOF
```

2.2 Postfix 서비스 관리
```bash
# Postfix 서비스 시작 및 활성화
sudo systemctl enable postfix
sudo systemctl start postfix
sudo systemctl status postfix

# 포트 확인
sudo ss -tlnp | grep :25
```

### 3. Dovecot (IMAP 서버) 설정

3.1 dovecot.conf 설정 파일
```bash
# 기존 설정 백업
sudo cp /etc/dovecot/dovecot.conf /etc/dovecot/dovecot.conf.backup

# dovecot.conf 설정
sudo tee /etc/dovecot/dovecot.conf > /dev/null << 'EOF'
protocols = imap
listen = *

# 로그 설정
log_path = /var/log/dovecot.log
auth_verbose = yes
auth_debug = yes

# SSL 비활성화 (테스트 환경)
ssl = no

# 인증 설정
auth_mechanisms = plain login
disable_plaintext_auth = no

# 메일 위치
mail_location = maildir:~/Maildir

# 간단한 네임스페이스
namespace inbox {
  inbox = yes
}

# IMAP 서비스
service imap-login {
  inet_listener imap {
    port = 143
  }
}

# 간단한 인증 (passwd 파일 사용)
passdb {
  driver = passwd-file
  args = /etc/dovecot/passwd
}

userdb {
  driver = passwd-file
  args = /etc/dovecot/passwd
}
EOF
```

3.2 Dovecot 사용자 파일 설정
```bash
# Dovecot 사용자 파일 생성
sudo mkdir -p /etc/dovecot
sudo tee /etc/dovecot/passwd > /dev/null << 'EOF'
testuser:{PLAIN}testpass123:1001:1001::/home/testuser::
EOF

# 권한 설정
sudo chown dovecot:dovecot /etc/dovecot/passwd
sudo chmod 600 /etc/dovecot/passwd
```

3.3 Dovecot 권한 및 로그 설정
```bash
# 로그 파일 권한 설정
sudo touch /var/log/dovecot.log
sudo chown dovecot:dovecot /var/log/dovecot.log
sudo chmod 644 /var/log/dovecot.log

# 런타임 디렉토리 권한 설정
sudo mkdir -p /var/run/dovecot
sudo chown dovecot:dovecot /var/run/dovecot
sudo chmod 755 /var/run/dovecot
```

3.4 Dovecot 서비스 관리
```bash
# Dovecot 서비스 시작 및 활성화
sudo systemctl enable dovecot
sudo systemctl start dovecot
sudo systemctl status dovecot

# 포트 확인
sudo ss -tlnp | grep :143

# 인증 테스트
sudo doveadm auth test testuser testpass123
```

### 4. 시스템 사용자 설정

4.1 테스트 사용자 생성
```bash
# 테스트 사용자 생성
sudo useradd -m testuser
sudo passwd testuser
# 비밀번호: testpass123

# 메일박스 디렉토리 생성
sudo mkdir -p /home/testuser/Maildir/{new,cur,tmp}
sudo chown -R testuser:testuser /home/testuser/Maildir
sudo chmod -R 700 /home/testuser/Maildir

# mail 그룹에 사용자 추가
sudo usermod -a -G mail testuser
```

### 5. Apache 웹 서버 설정

5.1 Apache 기본 설정
```bash
# Apache 서비스 시작 및 활성화
sudo systemctl enable httpd
sudo systemctl start httpd
sudo systemctl status httpd

# 포트 확인
sudo ss -tlnp | grep :80
```

### 6. RoundCube 웹메일 설치

6.1 RoundCube 다운로드 및 설치
```bash
# RoundCube 다운로드
cd /tmp
wget https://github.com/roundcube/roundcubemail/releases/download/1.6.5/roundcubemail-1.6.5-complete.tar.gz
tar -xzf roundcubemail-1.6.5-complete.tar.gz
sudo rm -rf /var/www/html/roundcube
sudo mv roundcubemail-1.6.5 /var/www/html/roundcube

# 권한 설정
sudo chown -R apache:apache /var/www/html/roundcube
sudo chmod -R 755 /var/www/html/roundcube

# 로그 및 임시 디렉토리 생성
sudo mkdir -p /var/www/html/roundcube/{logs,temp}
sudo chown -R apache:apache /var/www/html/roundcube/{logs,temp}
sudo chmod -R 755 /var/www/html/roundcube/{logs,temp}
```

6.2 RoundCube 설정 파일
```bash
# config.inc.php 파일 생성
sudo tee /var/www/html/roundcube/config/config.inc.php > /dev/null << 'EOF'
<?php
// 기본 설정
$config['db_dsnw'] = 'mysql://mailuser:rootoor@192.168.70.71/mailserver';
$config['default_host'] = 'localhost';
$config['default_port'] = 143;
$config['smtp_server'] = 'localhost';
$config['smtp_port'] = 25;
$config['smtp_user'] = '%u';
$config['smtp_pass'] = '%p';
$config['support_url'] = '';
$config['product_name'] = 'YSCK Webmail';
$config['des_key'] = 'rcmail-!24BytesDESkey*Str';
$config['log_driver'] = 'file';
$config['log_dir'] = '/var/www/html/roundcube/logs/';
$config['temp_dir'] = '/var/www/html/roundcube/temp/';
$config['auto_create_user'] = true;
$config['force_https'] = false;
$config['debug_level'] = 1;
?>
EOF

# 권한 설정
sudo chown apache:apache /var/www/html/roundcube/config/config.inc.php
sudo chmod 644 /var/www/html/roundcube/config/config.inc.php
```

### 7. 서비스 상태 확인

7.1 모든 서비스 상태 확인
```bash
# 서비스 상태 확인
sudo systemctl status postfix dovecot httpd

# 포트 리스닝 확인
sudo ss -tlnp | grep -E ':25|:143|:80'

# 로그 파일 위치
echo "메일 로그: /var/log/maillog"
echo "Dovecot 로그: /var/log/dovecot.log"
echo "Apache 로그: /var/log/httpd/error_log"
echo "RoundCube 로그: /var/www/html/roundcube/logs/errors.log"
```

7.2 테스트 명령어
```bash
# 로컬 메일 발송 테스트
echo "Local test message" | mail -s "Test Subject" testuser

# 메일박스 확인
sudo ls -la /home/testuser/Maildir/new/

# IMAP 연결 테스트
echo -e "a1 LOGIN testuser testpass123\na2 LIST \"\" \"*\"\na3 LOGOUT" | telnet localhost 143

# 웹메일 접속
echo "브라우저에서 http://192.168.70.172/roundcube 접속"
echo "로그인: testuser / testpass123"
```

### 8. 접속 정보

8.1 웹메일 접속
- **URL**: http://192.168.70.172/roundcube
- **사용자명**: testuser
- **비밀번호**: testpass123

8.2 메일 클라이언트 설정 정보
- **IMAP 서버**: 192.168.70.172:143 (암호화 없음)
- **SMTP 서버**: 192.168.70.172:25 (암호화 없음)
- **사용자명**: testuser
- **비밀번호**: testpass123

### 9. 주요 설정 파일 위치

```bash
# Postfix 설정
/etc/postfix/main.cf

# Dovecot 설정
/etc/dovecot/dovecot.conf
/etc/dovecot/passwd

# Apache 설정
/etc/httpd/conf/httpd.conf

# RoundCube 설정
/var/www/html/roundcube/config/config.inc.php

# 로그 파일
/var/log/maillog
/var/log/dovecot.log
/var/log/httpd/error_log
/var/www/html/roundcube/logs/errors.log
```
핵심 구성 요소:</br>
✅ Postfix - SMTP 서버 (포트 25) </br>
✅ Dovecot - IMAP 서버 (포트 143) </br>
✅ Apache - 웹 서버 (포트 80)</br>
✅ RoundCube - 웹메일 클라이언트</br>
✅ 시스템 사용자 - testuser (testpass123)</br>
</br>
주요 특징:</br>
로컬 메일 배송: Maildir 형식으로 사용자 홈 디렉토리에 저장</br>
간단한 인증: passwd 파일 기반 인증 시스템</br>
네트워크 범위: 192.168.70.0/24 대역 허용</br>
보안: SSL/TLS 비활성화 (테스트 환경)</br>
</br>
접속 정보:</br>
웹메일: http://192.168.70.172/roundcube</br>
로그인: testuser / testpass123</br>
