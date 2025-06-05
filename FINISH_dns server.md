### Packages used : 
###### bind(Berkeley Internet Name Domain) 
###### bind-utils(dig, nslookup, host)
###### bind-libs bind-chroot bind-devel
### Server IP : 192.168.70.101/24
### Server OS : Centos 7
### Client IP : 192.168.10.10/24
### Client OS : Ubuntu 18.04
---
# Set configuration file in Server (/etc/named.conf)
### 아래 9개의 네트워크 대역대에만 dns request query를 허용하도록 명시하고 
![image](https://github.com/user-attachments/assets/9009997a-3898-48ed-ac84-d7ae0676a331)
### 서비스를 제공할 정방향 DNS Zone과 역방향 DNS Zone을 명시한다.
![image](https://github.com/user-attachments/assets/70a9db9f-429f-4899-9fc6-6f18000d9f6d)

# Set configuration file (/var/named/fwd.ysck.kr)
### 정항뱡 DNS Zone이 참조할 설정파일에 서비스할 내용들을 명시한다.
![image](https://github.com/user-attachments/assets/d70f5264-c8fc-4a53-a88a-ad0ab0f90257)

# Set configuration file (/var/named/rev.ysck.kr)
### 정방향 DNS를 설정한 내용들을 토대로 역방향 DNS를 설정한다. DNS Zone이 참조할 설정파일에 서비스할 내용들을 명시한다.
![image](https://github.com/user-attachments/assets/6a492ca0-32d8-4e20-b854-44b7721961f7)

# Check fwd_file and rev_file
### 문법검사할때 DNS Zone 이름은 'ysck.kr' 이고, 역방향 DNS Zone 이름은 '70.168.192.in-addr.arpa' 이다.
![image](https://github.com/user-attachments/assets/77baa8c1-fdf9-48dd-9690-d998ae17c3e6)

# Test DNS in Client (dig)
![image](https://github.com/user-attachments/assets/27aeacf4-28af-4c39-a3ee-d9f3a55a22e4)

# Test reverse DNS in Client (dig -x)
![image](https://github.com/user-attachments/assets/76d5a578-e0fd-4c77-83af-1c43e04dd2a9)
![image](https://github.com/user-attachments/assets/2b04f4ba-f920-4530-a165-139e423d369c)

# Test nslookup in Client (nslookup)
![image](https://github.com/user-attachments/assets/0049cd06-4b98-4b5d-ba5a-fadb038d9def)
