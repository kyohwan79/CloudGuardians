# Use chrony package to install the NTP server
gedit /etc/chrony.conf
서버가 참조할 외부 NTP 웹사이트는 아래 3곳이다. 모두 한국 오픈소스 ntp서버 FQDN주소이다.
server kr.pool.ntp.org iburst
server ntp.kornet.net iburst
server ntp.bora.net iburst
그리고 서버가 서비스를 제공할 클라이언트 네트워크 대역대는 아래 9곳이다.
allow 192.168.10.0/24
allow 192.168.20.0/24
allow 192.168.30.0/24
allow 192.168.40.0/24
allow 192.168.50.0/24
allow 192.168.60.0/24
allow 192.168.70.0/24
allow 192.168.80.0/24
allow 192.168.90.0/24

![image](https://github.com/user-attachments/assets/ae6f7b6d-cf81-4574-b4b0-258828c07c1e)
