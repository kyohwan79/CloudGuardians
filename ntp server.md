![image](https://github.com/user-attachments/assets/db0e4b5b-0d83-4b4d-b897-265196ad0a59)# Use chrony to install the NTP server
yum -y install chrony \
gedit /etc/chrony.conf : 서버가 참조할 외부 NTP 웹사이트는 아래 3곳이다. 모두 한국 오픈소스 ntp서버 FQDN주소이다. \
server kr.pool.ntp.org iburst \
server ntp.kornet.net iburst \
server ntp.bora.net iburst \
#그리고 서버가 서비스를 제공할 클라이언트 네트워크 대역대는 아래 9곳이다. \
allow 192.168.10.0/24 \
allow 192.168.20.0/24 \
allow 192.168.30.0/24 \
allow 192.168.40.0/24 \
allow 192.168.50.0/24 \
allow 192.168.60.0/24 \
allow 192.168.70.0/24 \
allow 192.168.80.0/24 \
allow 192.168.90.0/24 
![image](https://github.com/user-attachments/assets/ae6f7b6d-cf81-4574-b4b0-258828c07c1e)

방화벽에 chrony 서비스 허용하고 SELinux 비활성화하고 서비스실행파일을 실행해준다.
![image](https://github.com/user-attachments/assets/67b74218-12f7-42fa-915c-b49ee0077de7)

chronyc tracking && timedatectl && date : chrony는 UTC 시간을 가져오고  timeZone=AsiaSeoul에 따라서 시스템시간은 KST로 변경된 15:11로 표시된다.
![image](https://github.com/user-attachments/assets/ce75622a-acda-451c-a6b6-d144db7af8a1)
![image](https://github.com/user-attachments/assets/6eb3fa4f-6b73-4fc2-82b4-7b56bccdc49c)

# Use chrony on the client
yum -y install chrony \
gedit /etc/chrony.conf : 서비스 제공 받을 서버를 지정한다
![image](https://github.com/user-attachments/assets/2b0531f8-bebe-4294-a277-8681eedb8531)


서비스 실행파일을 실행한다 
![image](https://github.com/user-attachments/assets/5d5eda48-fded-484a-91e2-a8502c58f9fd)

chronyc sources -v : 서버가 나오면 된다 
![image](https://github.com/user-attachments/assets/16e073e9-f0b9-4098-8190-623634ac50c5)


timedatectl : 서버로부터 받아온 시간
![image](https://github.com/user-attachments/assets/f973161f-e92a-4ad5-ba47-c4942354cb3c)

# Check service log on server
![image](https://github.com/user-attachments/assets/fdf650b7-73d8-434f-af6c-d838e2688b78)

