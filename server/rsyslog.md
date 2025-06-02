## log 서버 : Ubuntu18.04 (192.168.100.93)
## 타겟 서버 : CentOS7-DB (192.168.100.71)

```bash
# rsyslog 설치 (Ubuntu 18.04에는 기본 설치되어 있음)
apt update
apt install rsyslog -y

# rsyslog 상태 확인
systemctl status rsyslog

# 설정 파일 백업
cp /etc/rsyslog.conf /etc/rsyslog.conf.backup

```

### 설정 파일 편집
nano /etc/rsyslog.conf
```bash
#  /etc/rsyslog.conf	Configuration file for rsyslog.
#
#			For more information see
#			/usr/share/doc/rsyslog-doc/html/rsyslog_conf.html
#
#  Default logging rules can be found in /etc/rsyslog.d/50-default.conf


#################
#### MODULES ####
#################

module(load="imuxsock") # provides support for local system logging
#module(load="immark")  # provides --MARK-- message capability

# provides UDP syslog reception
module(load="imudp")
input(type="imudp" port="514")

# provides TCP syslog reception
module(load="imtcp")
input(type="imtcp" port="514")

# provides kernel logging support and enable non-kernel klog messages
module(load="imklog" permitnonkernelfacility="on")

###########################
#### GLOBAL DIRECTIVES ####
###########################

#
# Use traditional timestamp format.
# To enable high precision timestamps, comment out the following line.
#
$ActionFileDefaultTemplate RSYSLOG_TraditionalFileFormat

# Filter duplicated messages
$RepeatedMsgReduction on

#
# Set the default permissions for all log files.
#
$FileOwner syslog
$FileGroup adm
$FileCreateMode 0640
$DirCreateMode 0755
$Umask 0022
$PrivDropToUser syslog
$PrivDropToGroup syslog

#
# Where to place spool and state files
#
$WorkDirectory /var/spool/rsyslog

#
# Include all config files in /etc/rsyslog.d/
#
$IncludeConfig /etc/rsyslog.d/*.conf

# 원격 로그 저장 템플릿
$template RemoteHost,"/var/log/remote/%HOSTNAME%/syslog.log"
$template RemoteHostAuth,"/var/log/remote/%HOSTNAME%/secure.log"
$template RemoteHostNormalized,"/var/log/remote/%HOSTNAME:::secpath-replace%/syslog.log"
$template RemoteHostAuthNormalized,"/var/log/remote/%HOSTNAME:::secpath-replace%/secure.log"


###############
#### RULES ####
###############

# 원격 호스트에서 온 로그 처리
if $fromhost-ip != '127.0.0.1' then {
    # CentOS의 인증 관련 로그 (authpriv facility 사용)
    authpriv.*                      ?RemoteHostAuth
    # Ubuntu의 인증 관련 로그 (auth facility 사용)  
    auth.*                          ?RemoteHostAuth
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
# 로그 디렉토리 생성
mkdir -p /var/log/remote

# 권한 설정
chown -R syslog:adm /var/log/remote
chmod -R 755 /var/log/remote

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
usermod -a -G syslog www-data
systemctl restart apache2

# 로그 뷰어 PHP 파일 생성
nano logviewer.php
```

```html
<?php
// SSH 로그 모니터 웹 뷰어
// 디버그 정보 표시 활성화
error_reporting(E_ALL);
ini_set('display_errors', 1);

// 로그 디렉토리 경로
$remote_log_dir = '/var/log/remote';
$target_host_dirs = ['192-168-100-71', '192.168.100.71', 'centos7-1'];

function getLogContent($filepath, $lines = 50) {
    if (file_exists($filepath) && is_readable($filepath)) {
        $content = shell_exec("tail -n $lines " . escapeshellarg($filepath) . " 2>/dev/null");
        return $content ? $content : "로그 파일이 비어있습니다.";
    }
    return null;
}

function findLogFiles($base_dir, $host_dirs) {
    $found_files = [];
    foreach ($host_dirs as $host_dir) {
        $full_path = $base_dir . '/' . $host_dir;
        if (is_dir($full_path)) {
            $files = glob($full_path . '/*.log');
            foreach ($files as $file) {
                $found_files[basename($file)] = $file;
            }
        }
    }
    return $found_files;
}
?>
<!DOCTYPE html>
<html lang="ko">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>SSH Log Monitor</title>
    <meta http-equiv="refresh" content="10">
    <style>
        body { 
            font-family: Arial, sans-serif; 
            margin: 20px; 
            background-color: #f5f5f5; 
        }
        .container { 
            max-width: 1200px; 
            margin: 0 auto; 
            background: white; 
            padding: 20px; 
            border-radius: 8px; 
            box-shadow: 0 2px 10px rgba(0,0,0,0.1); 
        }
        .log-section { 
            margin: 20px 0; 
            border: 1px solid #ddd; 
            border-radius: 5px; 
        }
        .log-header { 
            background: #333; 
            color: white; 
            padding: 10px; 
            font-weight: bold; 
        }
        .log-content { 
            background: #000; 
            color: #00ff00; 
            padding: 15px; 
            font-family: 'Courier New', monospace; 
            font-size: 12px; 
            height: 300px; 
            overflow-y: scroll; 
            white-space: pre-wrap; 
        }
        .status { 
            padding: 10px; 
            margin: 10px 0; 
            border-radius: 4px; 
        }
        .status.success { 
            background-color: #d4edda; 
            border: 1px solid #c3e6cb; 
            color: #155724; 
        }
        .status.error { 
            background-color: #f8d7da; 
            border: 1px solid #f5c6cb; 
            color: #721c24; 
        }
        .info { 
            background-color: #d1ecf1; 
            border: 1px solid #bee5eb; 
            color: #0c5460; 
            padding: 10px; 
            margin: 10px 0; 
            border-radius: 4px; 
        }
    </style>
</head>
<body>
    <div class="container">
        <h1>🔍 DB 서버 로그 모니터</h1>
        <p>마지막 업데이트: <?php echo date('Y-m-d H:i:s'); ?></p>
        
        <?php
        // 디버그 정보
        echo "<div class='info'>";
        echo "<strong>디버그 정보:</strong><br>";
        echo "원격 로그 디렉토리: $remote_log_dir<br>";
        echo "디렉토리 존재 여부: " . (is_dir($remote_log_dir) ? "존재" : "존재하지 않음") . "<br>";
        echo "디렉토리 권한: " . (is_readable($remote_log_dir) ? "읽기 가능" : "읽기 불가능") . "<br>";
        
        // 하위 디렉토리 목록
        if (is_dir($remote_log_dir)) {
            $subdirs = glob($remote_log_dir . '/*', GLOB_ONLYDIR);
        }
        echo "</div>";
        
        // 로그 파일 찾기
        $log_files = findLogFiles($remote_log_dir, $target_host_dirs);
        
        if (empty($log_files)) {
            echo "<div class='status error'>";
            echo "<strong>경고:</strong> CentOS 7 서버(192.168.100.71)의 로그 파일을 찾을 수 없습니다.<br>";
            echo "가능한 원인:<br>";
            echo "1. CentOS 7에서 로그가 아직 전송되지 않음<br>";
            echo "2. rsyslog 설정 문제<br>";
            echo "3. 네트워크 연결 문제<br>";
            echo "4. 파일 권한 문제<br>";
            echo "</div>";
            
            // 전체 원격 로그 디렉토리 내용 표시
            if (is_dir($remote_log_dir)) {
                echo "<div class='info'>";
                echo "<strong>원격 로그 디렉토리 전체 내용:</strong><br>";
                $all_files = shell_exec("find $remote_log_dir -type f -name '*.log' 2>/dev/null");
                echo $all_files ? nl2br(htmlspecialchars($all_files)) : "로그 파일이 없습니다.";
                echo "</div>";
            }
        } else {
            echo "<div class='status success'>";
            echo "" . count($log_files) . "개의 로그 파일 확인";
            echo "</div>";
        }
        
        // SSH 관련 로그 (secure.log 또는 auth.log)
        $ssh_logs = ['secure.log', 'auth.log'];
        foreach ($ssh_logs as $log_name) {
            if (isset($log_files[$log_name])) {
                echo "<div class='log-section'>";
                echo "<div class='log-header'>🔐 SSH 인증 로그 ($log_name)</div>";
                echo "<div class='log-content'>";
                $content = getLogContent($log_files[$log_name], 50);
                if ($content) {
                    // SSH 관련 라인만 필터링
                    $lines = explode("\n", $content);
                    $ssh_lines = array_filter($lines, function($line) {
                        return preg_match('/sshd|ssh|login|session|authentication/i', $line);
                    });
                    echo htmlspecialchars(implode("\n", $ssh_lines));
                } else {
                    echo "SSH 로그를 읽을 수 없습니다.";
                }
                echo "</div></div>";
                break;
            }
        }
        
        // 전체 시스템 로그
        if (isset($log_files['syslog.log'])) {
            echo "<div class='log-section'>";
            echo "<div class='log-header'>📋 전체 시스템 로그</div>";
            echo "<div class='log-content'>";
            $content = getLogContent($log_files['syslog.log'], 30);
            echo $content ? htmlspecialchars($content) : "시스템 로그를 읽을 수 없습니다.";
            echo "</div></div>";
        }
        
        // 기타 로그 파일들
        foreach ($log_files as $filename => $filepath) {
            if (!in_array($filename, ['secure.log', 'auth.log', 'syslog.log'])) {
                echo "<div class='log-section'>";
                echo "<div class='log-header'>📄 $filename</div>";
                echo "<div class='log-content'>";
                $content = getLogContent($filepath, 20);
                echo $content ? htmlspecialchars($content) : "로그를 읽을 수 없습니다.";
                echo "</div></div>";
            }
        }
        
        // 테스트 섹션
        echo "<div class='info'>";
        echo "<strong>테스트 명령어:</strong><br>";
        echo "CentOS 7에서 테스트 로그 전송: <code>logger -p authpriv.info 'Test SSH log'</code><br>";
        echo "SSH 접속 시도: <code>ssh user@192.168.100.71</code><br>";
        echo "로그 서버에서 실시간 확인: <code>sudo tail -f /var/log/remote/*/secure.log</code>";
        echo "</div>";
        ?>
        
        <div style="text-align: center; margin-top: 20px; color: #666;">
            <small>자동 새로고침: 10초마다 | 수동 새로고침하려면 F5를 누르세요</small>
        </div>
    </div>
</body>
</html>
```

```bash


```

### CentOS7
```bash
# rsyslog 상태 확인 (CentOS 7에는 기본 설치됨)
systemctl status rsyslog

# 별도 설정 파일 생성
nano /etc/rsyslog.d/50-remote.conf
```

```
# 원격 로그 서버 설정
# TCP를 사용하여 안정적인 전송
*.* @@192.168.100.93:514

# UDP 사용하려면 (덜 안정적이지만 빠름)
# *.* @192.168.100.93:514

# 특정 로그만 전송
auth,authpriv.* @@192.168.100.93:514
daemon.* @@192.168.100.93:514
```

```bash
# SSH 설정 파일 백업
sudo cp /etc/ssh/sshd_config /etc/ssh/sshd_config.backup

# SSH 설정 파일 편집
sudo nano /etc/ssh/sshd_config
```

다음을 확인하여 수정
```
# 로그 레벨을 상세하게 설정
LogLevel VERBOSE

# 로그인 시도 기록 강화
LoginGraceTime 2m
MaxAuthTries 3
MaxSessions 10

# 인증 방법 설정
PubkeyAuthentication yes
PasswordAuthentication yes

# 로그인 기록 설정
PrintMotd no
PrintLastLog yes

# 세션 정보 기록
UsePAM yes

# 루트 로그인 제한 (보안 강화)
PermitRootLogin no
```

```bash
sudo systemctl restart sshd
sudo systemctl status sshd

# auditd는 CentOS 7에 기본 설치됨
sudo systemctl status auditd
sudo systemctl enable auditd

# audit 규칙 파일 생성
sudo nano /etc/audit/rules.d/ssh-monitoring.rules
```

```
# SSH 데몬 실행 파일 감시
-w /usr/sbin/sshd -p x -k ssh_execution

# SSH 설정 파일 변경 감시
-w /etc/ssh/sshd_config -p wa -k ssh_config

# 인증 로그 파일 감시 (CentOS 7에서는 /var/log/secure)
-w /var/log/secure -p wa -k auth_log

# 사용자 로그인/로그아웃 감시
-w /var/log/wtmp -p wa -k session
-w /var/log/btmp -p wa -k session

# 네트워크 연결 감시
-a always,exit -F arch=b64 -S connect -F a2=16 -k network_connect
-a always,exit -F arch=b32 -S connect -F a2=16 -k network_connect

# 파일 권한 변경 감시
-a always,exit -F arch=b64 -S chmod,fchmod,fchmodat -k perm_mod
-a always,exit -F arch=b32 -S chmod,fchmod,fchmodat -k perm_mod

# 사용자 계정 관리 감시
-w /etc/passwd -p wa -k identity
-w /etc/group -p wa -k identity
-w /etc/shadow -p wa -k identity
```

```bash
sudo service auditd restart
sudo systemctl enable auditd

# firewalld 상태 확인
sudo systemctl status firewalld

# rsyslog 포트 허용 (아웃바운드)
sudo firewall-cmd --permanent --add-port=514/tcp
sudo firewall-cmd --permanent --add-port=514/udp

# 설정 적용
sudo firewall-cmd --reload

# 설정 확인
sudo firewall-cmd --list-all

sudo setenforce 0

sudo systemctl restart rsyslog
sudo systemctl status rsyslog

# 테스트 로그 전송
logger "Test log from CentOS7 target server 192.168.100.71"

# SSH 관련 로그 확인
sudo tail -f /var/log/secure
```
