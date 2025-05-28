## mysql DB 구축 과정

## READ ME!
![anko1](https://github.com/user-attachments/assets/505d395f-4894-498b-b780-386b0d5a7def)
<br>

## 개요  
- **개발 환경** :   Ubuntu 18.04
- **서버 IP** : `192.168.80.2`
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
