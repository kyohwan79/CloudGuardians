
```bash
apt update
rm -rf /var/lib/dpkg/lock*

apt install -y mysql-client mysql-server

# mysql 보안 설정
mysql_secure_installation

mysql -u root -p
# Bacula 카탈로그용 데이터베이스와 사용자 생성
CREATE DATABASE bacula;
CREATE USER 'bacula'@'localhost' IDENTIFIED BY 'rootoor';
GRANT ALL PRIVILEGES ON bacula.* TO 'bacula'@'localhost';
FLUSH PRIVILEGES;
SHOW DATABASES;
EXIT;

apt install bacula bacula-common-mysql -y

systemctl enable bacula-director
systemctl enable bacula-sd
systemctl enable bacula-fd



mkdir -p /opt/bacula/scripts
nano /opt/bacula/scripts/mysql_backup.sh

nano /opt/bacula/scripts/mysql_restore.sh

chmod +x /opt/bacula/scripts/mysql_backup.sh
chmod +x /opt/bacula/scripts/mysql_restore.sh

cp /etc/bacula/bacula-dir.conf /etc/bacula/bacula-dir.conf.backup
nano /etc/bacula/bacula-dir.conf

# Director 리소스 (파일 상단 부근)
Director {
  Name = bacula-dir
  DIRport = 9101
  QueryFile = "/etc/bacula/scripts/query.sql"
  WorkingDirectory = "/var/lib/bacula"
  PidDirectory = "/run/bacula"
  Maximum Concurrent Jobs = 20
  Password = "rootoor"
  Messages = Daemon
  DirAddress = 192.168.100.96
}

# MySQL 백업용 Job 정의 추가 (기존 Job 뒤에 추가)
Job {
  Name = "MySQL-Backup"
  Type = Backup
  Level = Full
  Client = bacula-fd
  FileSet = "MySQL-FileSet"
  Schedule = "WeeklyCycle"
  Storage = File1
  Messages = Standard
  Pool = File
  SpoolAttributes = yes
  Priority = 10
  Write Bootstrap = "/var/lib/bacula/%c.bsr"
}

# MySQL 복구용 Job 정의
Job {
  Name = "MySQL-Restore"
  Type = Restore
  Client = bacula-fd
  Storage = FileStorage
  FileSet = "MySQL-FileSet"
  Pool = File
  Messages = Standard
  Where = /bacula-restores
  RunAfterJob = "/opt/bacula/scripts/mysql_restore_apply.sh"
}

# 기존 Autochanger 수정
Autochanger {
  Name = File1
  Address = 192.168.100.96
  SDPort = 9103
  Password = "rootoor"
  Device = FileChgr1-Dev1
  Media Type = File1
  Maximum Concurrent Jobs = 10
  Autochanger = File1
}

# MySQL FileSet 정의 (기존 FileSet 뒤에 추가)
FileSet {
  Name = "MySQL-FileSet"
  Include {
    Options {
      signature = MD5
      compression = GZIP
    }
    Plugin = "bpipe:/opt/bacula/scripts/mysql_backup.sh:/opt/bacula/scripts/mysql_restore.sh"
  }
}

# Catalog 확인
Catalog {
  Name = MyCatalog
  dbname = "bacula"
  dbuser = "bacula"
  dbpassword = "rootoor"
}

# Client (File Daemon) 정의
Client {
  Name = bacula-fd
  Address = 192.168.100.96
  FDPort = 9102
  Catalog = MyCatalog
  Password = "rootoor"
  File Retention = 60 days
  Job Retention = 6 months
  AutoPrune = yes
}




cp /etc/bacula/bacula-sd.conf /etc/bacula/bacula-sd.conf.backup
nano /etc/bacula/bacula-sd.conf

Storage {
  Name = bacula-sd
  SDPort = 9103
  WorkingDirectory = "/var/lib/bacula"
  Pid Directory = "/run/bacula"
  Maximum Concurrent Jobs = 20
  SDAddress = 192.168.100.96
}

Director {
  Name = bacula-dir
  Password = "rootoor"
}

Device {
  Name = FileChgr1-Dev1
  Media Type = File1
  Archive Device = /var/lib/bacula/storage
  LabelMedia = yes
  Random Access = Yes
  AutomaticMount = yes
  RemovableMedia = no
  AlwaysOpen = no
  Maximum Concurrent Jobs = 5
}


nano /etc/bacula/bconsole.conf

Director {
  Name = bacula-dir
  DIRport = 9101
  address = 192.168.100.96
  Password = "rootoor"
}




cp /etc/bacula/bacula-fd.conf /etc/bacula/bacula-fd.conf.backup
nano /etc/bacula/bacula-fd.conf

Director {
  Name = bacula-dir
  Password = "rootoor"
}

FileDaemon {
  Name = bacula-fd
  FDport = 9102
  WorkingDirectory = /var/lib/bacula
  Pid Directory = /run/bacula
  Maximum Concurrent Jobs = 20
  Plugin Directory = /usr/lib/bacula
  FDAddress = 192.168.100.96
}

Messages {
  Name = Standard
  director = bacula-dir = all, !skipped, !restored
}


# 백업 저장 디렉토리 생성
mkdir -p /var/lib/bacula/storage
chown bacula:bacula /var/lib/bacula/storage
chmod 755 /var/lib/bacula/storage

# 복구용 디렉토리 생성
mkdir -p /bacula-restores
chown bacula:bacula /bacula-restores

# 설정 문법 검사
bacula-dir -tc /etc/bacula/bacula-dir.conf
bacula-sd -tc /etc/bacula/bacula-sd.conf
bacula-fd -tc /etc/bacula/bacula-fd.conf

# 서비스 재시작 및 상태 확인
systemctl restart bacula-director
systemctl restart bacula-sd
systemctl restart bacula-fd
systemctl status bacula-director bacula-sd bacula-fd


# 스크립트 디렉토리 생성
sudo mkdir -p /opt/bacula/scripts

# 백업 스크립트 생성
sudo nano /opt/bacula/scripts/mysql_backup.sh

# 복구 스크립트 생성
nano /opt/bacula/scripts/mysql_restore_apply.sh


/usr/share/bacula-director/create_mysql_database
/usr/share/bacula-director/make_mysql_tables
/usr/share/bacula-director/grant_mysql_privileges


# 스크립트 디렉토리 생성
mkdir -p /opt/bacula/scripts
# 백업 스크립트 생성
nano /opt/bacula/scripts/mysql_backup.sh

#!/bin/bash
mysqldump -h 192.168.100.71 -u backup -prootoor --single-transaction --routines --triggers --all-databases

nano /opt/bacula/scripts/mysql_restore.sh

#!/bin/bash
mysql -h 192.168.100.71 -u backup -prootoor

chmod +x /opt/bacula/scripts/mysql_backup.sh
chmod +x /opt/bacula/scripts/mysql_restore.sh

```


bconsole
```bash
# 상태 확인
status director
status client
status storage

# 정의된 리소스들 확인
show jobs
show clients
show storages
show filesets



```
