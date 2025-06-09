### Packages used : chrony 
(2025년 기준, 리눅스 기본 NTP는 'ntpd' 에서 'chronyd' 로 변경됐습니다. 아래 RedHat 공식 문서 링크 참조)
https://docs.redhat.com/en/documentation/red_hat_enterprise_linux/7/html/system_administrators_guide/ch-configuring_ntp_using_the_chrony_suite
### Server IP : 192.168.70.101/24
### Server OS : Centos 7
### Client IP : 192.168.10.10/24 (DHCP)
### Client OS : Ubuntu 18.04
---
# *Set configuration file in Server (/etc/chrony.conf)
*서버가 참조할 외부 NTP 서버는 총 3개이다. 모두 한국의 오픈소스 ntp서버 FQDN주소이다. \
그리고 서비스를 제공할 클라이언트가 속한 네트워크 대역대는 총 9개이다. 
![image](https://github.com/user-attachments/assets/ae6f7b6d-cf81-4574-b4b0-258828c07c1e)

# Set Firewall&SELinux&systemd
방화벽에 chrony 서비스 허용하고 SELinux 비활성화하고 서비스 파일을 실행해준다.
![image](https://github.com/user-attachments/assets/67b74218-12f7-42fa-915c-b49ee0077de7)

# Check the result in Server
chronyc tracking && timedatectl && date : chrony가 UTC 시간을 가져온 다음에,  timeZone=AsiaSeoul이므로 시스템시간은 KST로 변경된 15:11로 표시된다.
![image](https://github.com/user-attachments/assets/ce75622a-acda-451c-a6b6-d144db7af8a1)
![image](https://github.com/user-attachments/assets/6eb3fa4f-6b73-4fc2-82b4-7b56bccdc49c)

# *Set configuration file in Client (/etc/chrony/chrony.conf)
*서비스를 제공 받을 서버주소를 지정한다
![image](https://github.com/user-attachments/assets/2b0531f8-bebe-4294-a277-8681eedb8531)

# Set systemd
서비스 파일을 실행한다 
![image](https://github.com/user-attachments/assets/5d5eda48-fded-484a-91e2-a8502c58f9fd)

# Check the connection to Server in Client
chronyc sources -v 출력값에 서버주소가 나오면 된다 
![image](https://github.com/user-attachments/assets/16e073e9-f0b9-4098-8190-623634ac50c5)

# *Check the result in Client
*timedatectl : 서버로부터 받아온 시간
![image](https://github.com/user-attachments/assets/f973161f-e92a-4ad5-ba47-c4942354cb3c)

# *Check the logs served by the Server to the Client
![image](https://github.com/user-attachments/assets/fdf650b7-73d8-434f-af6c-d838e2688b78)

