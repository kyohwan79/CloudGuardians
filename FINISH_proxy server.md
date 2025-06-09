### Packages used : squid
### Server IP : 192.168.70.101/24
### Server OS : Centos 7
### Client IP : 192.168.10.10/24
### Client OS : Ubuntu 18.04

# Set DNS in Server
![image](https://github.com/user-attachments/assets/7e3ec3ec-a9ae-4fb4-b438-0eb649dbf90a)
# *Set configuration file in Server (/etc/squid/squid.conf)
### *서비스를 제공받을 클라이언트들이 속한 네트워드 대역대와 acl alias(별칭)을 지정한다.
![image](https://github.com/user-attachments/assets/87f4c361-a662-445f-bbdf-c2ffd2142792)
### *위에서 정의한 acl alias들에 http 서비스를 허용한다.
![image](https://github.com/user-attachments/assets/7ddd2dc6-5816-4ad8-b127-bb0ee488f09f)
### 자주 사용되는 캐시데이터를 저장할 디렉터리 경로를 별도로 지정해준다. 총 1000MB , 최대 32개 상위 디렉터리에 최대 256개 하위 디렉터리까지 생성가능하다.
### 맨 밑줄에 'visible_hostname proxycentos' 는 /var/log/squid/access.log 파일에 명시될 서버의 이름이다.(마지막 사진 참조)
![image](https://github.com/user-attachments/assets/0922f4a2-ed70-4f0b-a8b3-fe3c3a157a14)

# *Set proxy in Client (Set preference proxy IP)
![image](https://github.com/user-attachments/assets/bfaaf198-2e81-4386-b570-477ea0921770)

# *Successful in Client 
![image](https://github.com/user-attachments/assets/93977b70-7d64-47d6-8779-39e663f3875a)

# *Check logs file in server (/var/log/squid/access.log)
![image](https://github.com/user-attachments/assets/7adeb541-0c7a-4909-a915-a93db94d430a)
