##vyatta.iso 파일 경로는 "ISO 및 각종 툴 모음" 폴더 안에 있습니다
##ID & PW: vyatta
install system  //destroy sda 만 디폴트 설정값이 다르게 yes 이고 나머진 디폴트로 설정
configure
set interfaces ethernet eth0 address '192.168.100.254/24'
set interfaces ethernet eth1 address '192.168.200.254/24'
commit
save

![image](https://github.com/user-attachments/assets/3e0fc723-af3a-4ae2-a414-6d0449f92f12)
