

```bash
yum update -y
yum install -y wget curl telnet mailx

# hostname 임의로 guardians.edu로 설정
hostnamectl set-hostname guardians.edu

# /etc/hosts에 추가
echo "192.168.100.163 guardians.edu guardians" | tee -a /etc/hosts

# SMTP 서비스와 포트 587을 반드시 허용
firewall-cmd --permanent --add-service=smtp
firewall-cmd --permanent --add-port=587/tcp
firewall-cmd --reload
setenforce 0

# 기존 sendmail 메일 서버 확인 및 제거
rpm -qa | grep sendmail
# sendmail이 있다면 제거
# yum remove -y sendmail

# Postfix 설치
yum install -y postfix

# 설치 확인
postconf -d | head -5

# 기본 구성 파일 백업
cp /etc/postfix/main.cf /etc/postfix/main.cf.backup
cp /etc/postfix/master.cf /etc/postfix/master.cf.backup

# 현재 Postfix 설정 확인
sudo postconf -n

# 기본 설정 파일 위치 확인
ls -la /etc/postfix/

# Postfix 서비스 시작 및 자동 시작 설정
systemctl start postfix
systemctl enable postfix
systemctl status postfix

# 포트 확인
sudo netstat -tulnp | grep :25

```


```bash
# 현재 설정 확인
postconf myhostname
postconf mydomain
postconf myorigin
postconf inet_interfaces
postconf mynetworks

# 호스트명 및 도메인 설정
# myhostname 설정 (FQDN 형태로 설정)
postconf -e "myhostname = guardians.edu"

# mydomain 설정 (도메인명)
postconf -e "mydomain = edu"

# myorigin 설정 (메일 발신 시 사용할 도메인)
postconf -e "myorigin = \$mydomain"

# 네트워크 인터페이스 설정
# inet_interfaces 설정
postconf -e "inet_interfaces = all"

# inet_protocols 설정 (IPv4만 사용)
postconf -e "inet_protocols = ipv4"

# 신뢰할 수 있는 네트워크 설정
# mynetworks 설정 (릴레이를 허용할 네트워크 대역)
postconf -e "mynetworks = 127.0.0.0/8, 192.168.100.0/24"

# 목적지 도메인 설정
# mydestination 설정 (로컬로 배송할 도메인)
postconf -e "mydestination = \$myhostname, localhost.\$mydomain, localhost, \$mydomain"

# 메일박스 설정 (Maildir 형식 사용)
postconf -e "home_mailbox = Maildir/"

# 메일 큐 디렉토리 확인
postconf -e "queue_directory = /var/spool/postfix"

# 메일 크기 제한 설정
sudo postconf -e "mailbox_size_limit = 51200000"  # 50MB

# SMTP 배너 설정
postconf -e "smtpd_banner = \$myhostname ESMTP"

# 기본 보안 설정
# 릴레이 제한 설정
postconf -e "smtpd_relay_restrictions = permit_mynetworks, permit_sasl_authenticated, defer_unauth_destination"

# 설정 파일 문법 검사
postfix check

# 변경된 설정 확인
postconf -n | grep -E "(myhostname|mydomain|myorigin|inet_interfaces|mynetworks|mydestination|home_mailbox)"

# Postfix 서비스 재시작
systemctl restart postfix
systemctl status postfix
```

## 메일박스 및 사용자 설정
```bash
useradd -m -s /bin/bash testuser1
useradd -m -s /bin/bash testuser2
useradd -m -s /bin/bash admin
echo "testuser1:testuser1" | chpasswd
echo "testuser2:testuser2" | chpasswd
echo "admin:admin" | chpasswd

for user in testuser1 testuser2 admin; do
  if id "$user" &>/dev/null; then
    user_home=$(getent passwd "$user" | cut -d: -f6)
    mkdir -p "$user_home/Maildir"/{cur,new,tmp}
    chown -R "$user:$user" "$user_home/Maildir"
    chmod -R 700 "$user_home/Maildir"
  fi
done

# 기본 별칭 설정
cat << 'EOF' | tee -a /etc/aliases

# 메일 서버 관리용 별칭
postmaster: mailadmin
webmaster: mailadmin
abuse: mailadmin
root: mailadmin

# 테스트용 별칭
admin: mailadmin
test: testuser1
EOF

newaliases

# 로컬 메일 배송 테스트
echo "This is a test mail for testuser1" | mail -s "Test Mail 1" testuser1
echo "This is a test mail for testuser2" | mail -s "Test Mail 2" testuser2
echo "This is a test mail to admin alias" | mail -s "Admin Test Mail" admin

# 메일 배송 결과 확인

```
