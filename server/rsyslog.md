```bash
# rsyslog 설치 (Ubuntu 18.04에는 기본 설치되어 있음)
apt update
apt install rsyslog -y

# rsyslog 상태 확인
systemctl status rsyslog

# 설정 파일 백업
cp /etc/rsyslog.conf /etc/rsyslog.conf.backup

# 설정 파일 편집
nano /etc/rsyslog.conf
```

```bash
#################
#### MODULES ####
#################

module(load="imuxsock") # provides support for local system logging
module(load="imklog")   # provides kernel logging support

# 원격 로그 수신을 위한 모듈
module(load="imudp")
input(type="imudp" port="514")

module(load="imtcp")
input(type="imtcp" port="514")

###########################
#### GLOBAL DIRECTIVES ####
###########################

# 작업 디렉토리 설정
$WorkDirectory /var/spool/rsyslog

# 파일 권한 설정
$FileOwner syslog
$FileGroup adm
$FileCreateMode 0640
$DirCreateMode 0755
$Umask 0022

# 원격 로그 저장 템플릿
$template RemoteHost,"/var/log/remote/%HOSTNAME%/syslog.log"
$template RemoteHostAuth,"/var/log/remote/%HOSTNAME%/auth.log"

###############
#### RULES ####
###############

# 원격 호스트에서 온 로그 처리
if $fromhost-ip != '127.0.0.1' then {
    # 인증 관련 로그
    auth,authpriv.*                 ?RemoteHostAuth
    # 기타 모든 로그
    *.*                             ?RemoteHost
    # 원격 로그는 로컬 로그 파일에 중복 저장하지 않음
    stop
}

# 로컬 로그 규칙
auth,authpriv.*                 /var/log/auth.log
*.*;auth,authpriv.none          -/var/log/syslog
daemon.*                        -/var/log/daemon.log
kern.*                          -/var/log/kern.log
mail.*                          -/var/log/mail.log
user.*                          -/var/log/user.log

# 긴급 메시지는 모든 사용자에게 전송
*.emerg                         :omusrmsg:*
```

```bash
# 로그 디렉터리 생성
mkdir -p /var/log/remote
chown syslog:adm /var/log/remote
chmod 755 /var/log/remote

# 구문 검사
rsyslogd -N1

# 오류가 없으면 서비스 재시작
systemctl restart rsyslog

# 상태 확인
systemctl status rsyslog
netstat -tulnp | grep 514
```

```bash
# 514번 포트 UDP/TCP 개방
ufw allow 514/udp
ufw allow 514/tcp

# 방화벽 상태 확인
ufw status

# 테스트 로그 전송
logger -n 127.0.0.1 -P 514 "Test log message from local"

# 로그 파일 확인
tail -f /var/log/syslog
```

### 웹 대시보드에서 로그 확인
```bash
sudo apt install apache2 php libapache2-mod-php -y
sudo systemctl start apache2
sudo systemctl enable apache2

# 웹 디렉토리로 이동
cd /var/www/html

# www-data 사용자가 로그 파일을 읽을 수 있도록 권한 부여
usermod -a -G adm www-data
systemctl restart apache2

# 로그 뷰어 PHP 파일 생성
nano logviewer.php
```

```html
<!DOCTYPE html>
<html>
<head>
    <title>SSH Log Monitor</title>
    <meta http-equiv="refresh" content="5">
</head>
<body>
    <h1>SSH 로그 모니터</h1>
    <h2>최근 SSH 로그:</h2>
    <pre>
    <?php
    // SSH 관련 로그만 표시
    $logFile = '/var/log/remote/192-168-100-71/sshd.log';
    if (file_exists($logFile)) {
        echo htmlspecialchars(shell_exec("tail -50 $logFile"));
    } else {
        echo "로그 파일을 찾을 수 없습니다.";
    }
    ?>
    </pre>
    
    <h2>전체 인증 로그:</h2>
    <pre>
    <?php
    $authLog = '/var/log/remote/192-168-100-71/auth.log';
    if (file_exists($authLog)) {
        echo htmlspecialchars(shell_exec("tail -30 $authLog"));
    } else {
        echo "인증 로그 파일을 찾을 수 없습니다.";
    }
    ?>
    </pre>
</body>
</html>
```
