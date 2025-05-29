## mysql DB 구축 과정

## READ ME!
![anko1](https://github.com/user-attachments/assets/505d395f-4894-498b-b780-386b0d5a7def)
<br>

## 개요  
- **개발 환경** :   Ubuntu 18.04
- **서버 IP** : `192.168.100.7`
- **서비스 port** : `3306`
<br>

## 1️⃣ MySQL 설치 
```bash
$ sudo apt update
$ sudo apt install mysql-server -y
```
<br>

## 2️⃣동작 확인
```bash
$ sudo systemctl status mysql
```
<br>

## 3️⃣방화벽 설정
```bash
$ sudo ufw allow 3306/tcp
# 또는 특정 IP만 허용(권장)
$ sudo ufw allow from <허용할_IP주소> to any port 3306 proto tcp

$ sudo ufw enable
$ sudo ufw status
```
<br>

## 4️⃣바인딩 주소 확인
```bash
$ sudo nano /etc/mysql/mysql.conf.d/mysqld.cnf
```
- 기본은 127.0.0.1 → 로컬 접속만 허용
- 0.0.0.0 → 모든 IP에서 접속 허용
<br>

## 5️⃣계정 생성
MySQL 5.7부터 root 사용자가 기본적으로 auth_socket 플러그인을 사용하기 때문에 mysql_native_password 플러그인으로 변경
```bash
$ sudo mysql -u root -p

-- root 사용자의 plugin 확인
> SELECT user, host, plugin FROM mysql.user WHERE user = 'root';
-- root 사용자의 인증 플러그인을 변경하고 비밀번호 설정
> ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY 'rootoor';
> GRANT ALL PRIVILEGES ON *.* TO 'root'@'%' IDENTIFIED BY 'rootoor' WITH GRANT OPTION;

-- 권한 적용
> FLUSH PRIVILEGES;

-- 변경 확인
> SELECT user, host, plugin FROM mysql.user WHERE user = 'root';

-- MySQL 종료
> EXIT;
```

## 6️⃣다른 머신에서 확인
MySQL client 설치 후 root계정 접속 확인
```bash
sudo apt update
sudo apt install mysql-client -y
mysql -h 192.168.100.7 -u root -p

```
