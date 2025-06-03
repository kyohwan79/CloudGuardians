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


## 메일 데이터베이스 서버 설정

### 서버 정보
- OS: Ubuntu 18.04
- IP: 192.168.100.71
- 역할: MariaDB 데이터베이스 서버
- 데이터베이스명: mailserver

### 1. MariaDB 설치 및 기본 설정

1.1 MariaDB 설치
```bash
# 패키지 업데이트
sudo apt update

# MariaDB 서버 설치
sudo apt install -y mariadb-server mariadb-client

# MariaDB 서비스 시작 및 활성화
sudo systemctl enable mariadb
sudo systemctl start mariadb
sudo systemctl status mariadb
```

1.2 MariaDB 보안 설정
```bash
# MariaDB 보안 설정 실행
sudo mysql_secure_installation

# 설정 옵션:
# - Enter current password for root: [Enter] (초기에는 비밀번호 없음)
# - Set root password: Y
# - New password: rootoor (또는 원하는 비밀번호)
# - Remove anonymous users: Y
# - Disallow root login remotely: N (원격 접속 허용)
# - Remove test database: Y
# - Reload privilege tables: Y
```

1.3 원격 접속 설정
```bash
# MariaDB 설정 파일 수정
sudo nano /etc/mysql/mariadb.conf.d/50-server.cnf

# bind-address 설정 변경
# bind-address = 127.0.0.1  =>  bind-address = 0.0.0.0

# 또는 sed 명령으로 자동 변경
sudo sed -i 's/bind-address.*=.*127.0.0.1/bind-address = 0.0.0.0/' /etc/mysql/mariadb.conf.d/50-server.cnf

# MariaDB 재시작
sudo systemctl restart mariadb
```

1.4 방화벽 설정
```bash
# UFW 방화벽에서 MySQL 포트 허용
sudo ufw allow 3306/tcp

# 특정 네트워크 대역만 허용 (더 안전)
sudo ufw allow from 192.168.100.0/24 to any port 3306

# 방화벽 상태 확인
sudo ufw status
```

### 2. 메일 서버용 데이터베이스 설정

2.1 데이터베이스 및 사용자 생성
```sql
-- MariaDB에 root로 접속
sudo mysql -u root -p

-- 메일서버용 데이터베이스 생성
CREATE DATABASE mailserver;

-- 메일서버용 사용자 생성 및 권한 부여
CREATE USER 'mailuser'@'192.168.100.%' IDENTIFIED BY 'rootoor';
GRANT ALL PRIVILEGES ON mailserver.* TO 'mailuser'@'192.168.100.%';
FLUSH PRIVILEGES;

-- 데이터베이스 선택
USE mailserver;
```

2.2 메일 시스템 기본 테이블 생성
```sql
-- 도메인 테이블 생성
CREATE TABLE domains (
    id INT AUTO_INCREMENT PRIMARY KEY,
    domain VARCHAR(100) NOT NULL UNIQUE
);

-- 사용자 테이블 생성
CREATE TABLE users (
    id INT AUTO_INCREMENT PRIMARY KEY,
    email VARCHAR(100) NOT NULL UNIQUE,
    password VARCHAR(100) NOT NULL,
    domain_id INT,
    FOREIGN KEY (domain_id) REFERENCES domains(id)
);

-- 별칭(알리아스) 테이블 생성
CREATE TABLE aliases (
    id INT AUTO_INCREMENT PRIMARY KEY,
    source VARCHAR(100) NOT NULL,
    destination VARCHAR(100) NOT NULL
);
```

2.3 기본 데이터 삽입
```sql
-- 도메인 추가
INSERT INTO domains (domain) VALUES ('ysck.kr');

-- 사용자 추가
INSERT INTO users (email, password, domain_id) VALUES 
('admin@ysck.kr', '{PLAIN}rootoor', 1),
('test@ysck.kr', '{PLAIN}rootoor', 1);

-- 데이터 확인
SELECT * FROM domains;
SELECT * FROM users;

-- MariaDB 종료
EXIT;
```

### 3. RoundCube 웹메일용 테이블 생성

3.1 RoundCube 필수 테이블
```sql
-- MariaDB에 mailuser로 접속
mysql -h 192.168.100.71 -u mailuser -prootoor mailserver

-- RoundCube 사용자 테이블 (중복되지 않도록 수정)
CREATE TABLE IF NOT EXISTS `rc_users` (
  `user_id` int(10) UNSIGNED NOT NULL AUTO_INCREMENT,
  `username` varchar(128) CHARACTER SET utf8mb4 COLLATE utf8mb4_bin NOT NULL,
  `mail_host` varchar(128) NOT NULL,
  `created` datetime NOT NULL DEFAULT current_timestamp(),
  `last_login` datetime DEFAULT NULL,
  `failed_login` datetime DEFAULT NULL,
  `failed_login_counter` int(10) UNSIGNED DEFAULT NULL,
  `language` varchar(16) DEFAULT NULL,
  `preferences` longtext DEFAULT NULL,
  PRIMARY KEY (`user_id`),
  UNIQUE KEY `users_username_index` (`username`,`mail_host`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;

-- RoundCube 세션 테이블
CREATE TABLE IF NOT EXISTS `session` (
  `sess_id` varchar(128) NOT NULL,
  `expires` datetime DEFAULT NULL,
  `vars` mediumtext NOT NULL,
  `ip` varchar(40) NOT NULL DEFAULT '',
  `changed` datetime NOT NULL DEFAULT current_timestamp() ON UPDATE current_timestamp(),
  PRIMARY KEY (`sess_id`),
  KEY `expires_index` (`expires`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;

-- RoundCube 연락처 테이블
CREATE TABLE IF NOT EXISTS `identities` (
  `identity_id` int(10) UNSIGNED NOT NULL AUTO_INCREMENT,
  `user_id` int(10) UNSIGNED NOT NULL,
  `changed` datetime NOT NULL DEFAULT current_timestamp() ON UPDATE current_timestamp(),
  `del` tinyint(1) NOT NULL DEFAULT 0,
  `standard` tinyint(1) NOT NULL DEFAULT 0,
  `name` varchar(128) NOT NULL,
  `organization` varchar(128) NOT NULL DEFAULT '',
  `email` varchar(128) NOT NULL,
  `reply-to` varchar(128) NOT NULL DEFAULT '',
  `bcc` varchar(128) NOT NULL DEFAULT '',
  `signature` longtext DEFAULT NULL,
  `html_signature` tinyint(1) NOT NULL DEFAULT 0,
  PRIMARY KEY (`identity_id`),
  KEY `identities_user_index` (`user_id`,`del`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;

-- RoundCube 캐시 테이블
CREATE TABLE IF NOT EXISTS `cache` (
  `user_id` int(10) UNSIGNED NOT NULL,
  `cache_key` varchar(128) CHARACTER SET ascii COLLATE ascii_general_ci NOT NULL,
  `expires` datetime DEFAULT NULL,
  `data` longtext NOT NULL,
  PRIMARY KEY (`user_id`,`cache_key`),
  KEY `expires_index` (`expires`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;

-- RoundCube 연락처 테이블
CREATE TABLE IF NOT EXISTS `contacts` (
  `contact_id` int(10) UNSIGNED NOT NULL AUTO_INCREMENT,
  `user_id` int(10) UNSIGNED NOT NULL,
  `changed` datetime NOT NULL DEFAULT current_timestamp() ON UPDATE current_timestamp(),
  `del` tinyint(1) NOT NULL DEFAULT 0,
  `name` varchar(128) NOT NULL,
  `email` text NOT NULL,
  `firstname` varchar(128) NOT NULL,
  `surname` varchar(128) NOT NULL,
  `vcard` longtext DEFAULT NULL,
  `words` text DEFAULT NULL,
  PRIMARY KEY (`contact_id`),
  KEY `contacts_user_index` (`user_id`,`del`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;

-- RoundCube 연락처 그룹 테이블
CREATE TABLE IF NOT EXISTS `contactgroups` (
  `contactgroup_id` int(10) UNSIGNED NOT NULL AUTO_INCREMENT,
  `user_id` int(10) UNSIGNED NOT NULL,
  `changed` datetime NOT NULL DEFAULT current_timestamp() ON UPDATE current_timestamp(),
  `del` tinyint(1) NOT NULL DEFAULT 0,
  `name` varchar(128) NOT NULL,
  PRIMARY KEY (`contactgroup_id`),
  KEY `contactgroups_user_index` (`user_id`,`del`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;
```

### 4. 데이터베이스 상태 확인

4.1 MariaDB 서비스 상태
```bash
# MariaDB 서비스 상태 확인
sudo systemctl status mariadb

# MariaDB 프로세스 확인
sudo ps aux | grep mysql

# 포트 리스닝 확인
sudo netstat -tlnp | grep :3306
sudo ss -tlnp | grep :3306
```

4.2 데이터베이스 연결 테스트
```bash
# 로컬에서 root 접속 테스트
sudo mysql -u root -p

# 원격에서 mailuser 접속 테스트 (SMTP 서버에서 실행)
mysql -h 192.168.100.71 -u mailuser -prootoor mailserver

# 테이블 확인
SHOW TABLES;

# 데이터 확인
SELECT * FROM domains;
SELECT * FROM users;
```

4.3 권한 확인
```sql
-- 사용자 및 권한 확인
SELECT User, Host FROM mysql.user WHERE User = 'mailuser';
SHOW GRANTS FOR 'mailuser'@'192.168.100.%';

-- 데이터베이스 크기 확인
SELECT 
    table_schema AS 'Database',
    ROUND(SUM(data_length + index_length) / 1024 / 1024, 2) AS 'Size (MB)'
FROM information_schema.tables 
WHERE table_schema = 'mailserver'
GROUP BY table_schema;
```

### 5. 백업 및 복원 설정

5.1 자동 백업 스크립트
```bash
# 백업 디렉토리 생성
sudo mkdir -p /var/backups/mariadb

# 백업 스크립트 생성
sudo tee /usr/local/bin/backup_mailserver.sh > /dev/null << 'EOF'
#!/bin/bash
# 메일서버 데이터베이스 백업 스크립트

BACKUP_DIR="/var/backups/mariadb"
DATE=$(date +%Y%m%d_%H%M%S)
DB_NAME="mailserver"
DB_USER="root"
DB_PASS="rootoor"

# 백업 실행
mysqldump -u ${DB_USER} -p${DB_PASS} ${DB_NAME} > ${BACKUP_DIR}/mailserver_${DATE}.sql

# 7일 이상 된 백업 파일 삭제
find ${BACKUP_DIR} -name "mailserver_*.sql" -mtime +7 -delete

echo "Backup completed: ${BACKUP_DIR}/mailserver_${DATE}.sql"
EOF

# 실행 권한 부여
sudo chmod +x /usr/local/bin/backup_mailserver.sh

# 크론탭에 일일 백업 추가
echo "0 2 * * * /usr/local/bin/backup_mailserver.sh" | sudo crontab -
```

5.2 복원 방법
```bash
# 백업 파일로부터 복원
mysql -u root -prootoor mailserver < /var/backups/mariadb/mailserver_YYYYMMDD_HHMMSS.sql
```

### 6. 보안 설정

6.1 추가 보안 설정
```bash
# MariaDB 설정 파일에 보안 옵션 추가
sudo tee -a /etc/mysql/mariadb.conf.d/99-security.cnf > /dev/null << 'EOF'
[mysqld]
# 보안 설정
skip-symbolic-links
local-infile = 0
max_connections = 100
max_user_connections = 50

# 로그 설정
log-error = /var/log/mysql/error.log
slow-query-log = 1
slow-query-log-file = /var/log/mysql/slow.log
long_query_time = 2
EOF

# MariaDB 재시작
sudo systemctl restart mariadb
```

6.2 접속 제한 설정
```sql
-- 특정 IP에서만 접속 가능하도록 제한
-- 필요시 추가 사용자 생성
CREATE USER 'mailuser_local'@'localhost' IDENTIFIED BY 'rootoor';
GRANT ALL PRIVILEGES ON mailserver.* TO 'mailuser_local'@'localhost';

-- 불필요한 계정 삭제 (필요시)
-- DROP USER 'mailuser'@'%';  -- 모든 IP 접속 허용하는 계정 삭제
```


### 접속 정보
- **서버 IP**: 192.168.70.71
- **포트**: 3306
- **데이터베이스**: mailserver
- **사용자명**: mailuser
- **비밀번호**: rootoor
- **접속 권한**: 192.168.70.% (192.168.100.0/24 대역)

### 주요 테이블
```bash
# 메일 시스템 테이블
domains          # 도메인 정보
users            # 사용자 정보  
aliases          # 메일 별칭

# RoundCube 테이블
rc_users         # RoundCube 사용자
session          # 웹 세션
identities       # 메일 계정 정보
cache            # 캐시 데이터
contacts         # 연락처
contactgroups    # 연락처 그룹
```

### 연결 테스트 명령어
```bash
# SMTP 서버에서 DB 연결 테스트
mysql -h 192.168.70.71 -u mailuser -prootoor mailserver -e "SHOW TABLES;"

# 웹에서 DB 연결 테스트
curl http://192.168.70.172/db_test.php
```
