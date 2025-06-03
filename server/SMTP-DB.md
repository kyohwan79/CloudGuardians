## 메일 데이터베이스 서버 설정

### 서버 정보
- OS: Ubuntu 18.04
- IP: 192.168.70.71
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

# MariaDB 재시작
sudo systemctl restart mariadb
```

1.4 방화벽 설정
```bash
# UFW 방화벽에서 MySQL 포트 허용
sudo ufw allow 3306/tcp

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
CREATE USER 'mailuser'@'192.168.70.%' IDENTIFIED BY 'rootoor';
GRANT ALL PRIVILEGES ON mailserver.* TO 'mailuser'@'192.168.70.%';
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
mysql -h 192.168.70.71 -u mailuser -prootoor mailserver

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
mysql -h 192.168.70.71 -u mailuser -prootoor mailserver

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
SHOW GRANTS FOR 'mailuser'@'192.168.70.%';
```

</br>
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
