# BackDoor

# 공격기법
### 해커가 reverse shell 파일을 “shell.war” 로 생성하고 USB에 담아서 내부 스파이가 서버에 저장한다
![image](https://github.com/user-attachments/assets/b9d140f7-5ad8-4c15-a91b-75304d50845e)

### 내부 스파이가 서버에 저장한 shell.war파일을 apache tomcat manager 권한으로 배포한 후 실행한다 
![image](https://github.com/user-attachments/assets/709e8ae1-c25a-48ff-b2a7-780f3656cdaa)

### 공격자는 4444포트로 Listen 하고있다가 shell.war 파일이 실행되는 순간 tcp로 port reversing 이 되고 공격자는 서버의 주요 정보를 탈취할 수 있다
![image](https://github.com/user-attachments/assets/a7780b7f-ab63-4d7b-b454-3d5c894441ba)

# 대응방법
### apache tomcat manager 계정명과 비밀번호를 어렵게 설정하고
![image](https://github.com/user-attachments/assets/fad7d9f3-bade-48e4-99e8-99f20d2fb24e)

### chattr +i 로 관리자조차도 파일을 배포하지 못하도록 막아두고 .war 파일을 배포할 때만 일시적으로 풀어준다
![image](https://github.com/user-attachments/assets/19ed4fc8-386d-4aa9-8111-494080a5cffd)

### 서버에 함부로  외장하드나 USB같은 외부 저장장치를 삽입하지 못하도록 조치한다
![image](https://github.com/user-attachments/assets/133a20e2-b8ab-4be3-948b-c0a8c4799240)
