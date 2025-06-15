![image](https://github.com/user-attachments/assets/05d7f787-cf4e-475d-8f9e-122c3dcb0373)# DNS
/etc/named.conf \
정방향 DNS Zone 이름과 설정파일 저장경로를 지정 \
![image](https://github.com/user-attachments/assets/039dac82-5513-4c31-834a-dbf0d019cb7b)

/var/named/fwd.ysck.kr 정방향 DNS Zone 상세설정 \
www.ysck.kr 로 dns query를 보내면 192.168.70.101 에서 query response를 응답해준다 \
![image](https://github.com/user-attachments/assets/3cc95d97-989a-48d7-884d-bb9865ed2fa9)

Ubuntu Client에서 nslookup 정방향 dns 조회 성공 \
![image](https://github.com/user-attachments/assets/c56a6a1e-6dc9-40d6-9d7c-1e0e2c3cc335)

# DHCP
/etc/dhcp/dhcpd.conf \
192.168.10.10 부터 192.168.10.100 까지, 총 91개 주소를 임대해줄 수 있도록 설정 \
![image](https://github.com/user-attachments/assets/36bfcc72-f595-4eef-a0f2-55096f41d8e3)

ubuntu Client에서 192.168.10.10 부터 임대를 받았다 \
![image](https://github.com/user-attachments/assets/8def4a13-4cf9-4847-a8a0-dc3f62153aaa)

/var/lib/dhcpd/dhcpd.leases 에서 IP주소 임대 이력을 확인 \
![image](https://github.com/user-attachments/assets/15661995-75a1-4719-9c45-b78c8ce91903)

# Proxy
서버의 dns주소를 먼저 확인한다 \
해당 서버로 요청된 쿼리는 모두 8.8.8.8 구글 global dns address로 전달한다 \
![image](https://github.com/user-attachments/assets/90941fc0-ed22-4106-9dd2-ff714b73a53e)

/etc/squid/squid.conf 에서 서비스를 허용할 네트워크 대역대를 지정한다 \
![image](https://github.com/user-attachments/assets/214e7867-8bfc-4b6c-82ac-c9966fa80e20)

위에서 허용한 대역대로 http 를 허용해준다 \
※ vlan10은 192.168.10.0/24 의 "acl alias"이다 \
![image](https://github.com/user-attachments/assets/b3c2bdad-9ba2-4c71-af85-fc9fc1bda29b)

Ubuntu Client에서 Proxy 설정 \
![image](https://github.com/user-attachments/assets/6230b381-8a8a-48bb-8461-973746729361)

proxy 서버로부터 Ubuntu Client에 서비스되었다 \
![image](https://github.com/user-attachments/assets/adcc3574-5002-4586-a58d-8db79f645e19)

/var/log/squid/access.log 에서 서비스 해준 이력 확인 \
![image](https://github.com/user-attachments/assets/02c948cd-9b41-439e-9531-e98d18e6892a)

---
# FTP
![image](https://github.com/user-attachments/assets/e122fc4f-5e7b-41eb-b977-451c1c95ae75)

Ubuntu Client에서 서버의 “test1_ftp.txt” 파일을 get 명령어로 가져온다 \
![image](https://github.com/user-attachments/assets/cbb93a44-30c5-4960-8bf5-2f2e747a11c7)

# NTP
NTP패키지는 chrony로 사용한다 \
서버는 kr.pool.ntp.org 포함 총 3곳에서 시간데이터를 가져온다 \
![image](https://github.com/user-attachments/assets/c7e518d1-d23f-48e0-8204-69549714b387)

서비스를 허용할 네트워크 대역대를 지정한다 \
![image](https://github.com/user-attachments/assets/a09a9e34-dc51-4cb4-ba6e-86e0e7a84623)

Ubuntu Client에서 시간데이터 소스를 가져올 서버의 주소를 확인한다
![image](https://github.com/user-attachments/assets/fce56e19-25c6-4a05-9f37-4af660abceb5)

chrony 서버에서 UTC기준으로 시간데이터를 가져온 후, Client의 Time zone 정보(Asia/Seoul) 에 따라서 시간데이터를 KST로 변환하여 Client에게 표시해준다
![image](https://github.com/user-attachments/assets/d14d595b-d0c5-42ab-80d9-d1672fce3a3d)
