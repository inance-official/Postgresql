# PostgreSQL Install

- root에서 실행

## Install the repository RPM
```bash
dnf install -y https://download.postgresql.org/pub/repos/yum/reporpms/EL-9-x86_64/pgdg-redhat-repo-latest.noarch.rpm
```

## Disable the built-in PostgreSQL module(모듈 비활성화)
```bash
dnf -qy module disable postgresql
```

## PostgreSQL 설치
```bash
dnf install -y postgresql15-server
```

## 초기화 및 서비스 자동 실행 설정:

```bash
/usr/pgsql-15/bin/postgresql-15-setup initdb
```

 ```bash
systemctl enable postgresql-15
```

```bash
systemctl start postgresql-15
```

# 외부 접속을 위한 초기 설정

-  설정파일 오픈
```bash
vi /var/lib/pgsql/15/data/postgresql.conf
```

- 설정 파일에서 listen_addresses를 listen_addresses = '\*' 로 수정해 준다
```bash

...
#listen_addresses = '*'
...

```

## 특정 ip 접속 허용

- 설정파일 오픈
```bash
vi /var/lib/pgsql/15/data/pg_hba.conf
```

```plaintext
TYPE   DATABASE         USER            ADDRESS                 METHOD
host    all             all             0.0.0.0/0               md5
```

- 이후 해당 라인의 METHOD를 md5로 수정 후 원하는 ip 주소로 제한

- TYPE local의 경우 md5로 METHOD를 바꾸면 로그인 시, 비밀번호를 요구한다.
  따라서 local의 경우 md5로 바꾸기 전, postgres 사용자에 대한 비밀번호를 psql로 접속 후 설정해주자.
```bash
sudo su -postgres
qsql
```

```postgresql
alter user postgres with password '[password]';
```

## Postgresql-devel 설치

기본 postgresql-server 에는 ecpg가 없다. postgresql15-devel 설치를 해주어야 한다.
postgresql15-devel 다운을 위한 명령어는 다음과 같다
```bash
dnf install postgresql15-devel
```

## Postgresql-devel 설치 시 OS로 인한 에러

```bash
마지막 메타자료 만료확인(2:38:45 이전): 2024년 04월 02일 (화) 오전 08시 28분 17초.
오류: 
 문제: 작업에 가장 적합한 선택을 설치 할 수 없습니다
  - nothing provides perl(IPC::Run) needed by postgresql15-devel-15.6-1PGDG.rhel9.x86_64 from pgdg15
(설치 할 수 없는 꾸러미를 건너 뛰려면 '--skip-broken'을 (를) 추가하십시오 또는 '--nobest'는 최적 후보의 꾸러미만 사용합니다)
```

# Rocky Linux 9 perl-ipc-run 패키지

현재 사용하는 OS 버전으로 인해 perl-ipc 종속성을 해결해주지 못해서 발생하는 문제이다.
perl-ipc-run 패키지를 설치해주면 된다.

+) 버전 9.x 와 8.x 가 다르다

**RHEL, ROCKY 8.x**  
```bash
wget https://repo.almalinux.org/almalinux/8/PowerTools/aarch64/os/Packages/perl-IPC-Run-0.99-1.el8.noarch.rpm
wget https://repo.almalinux.org/almalinux/8/PowerTools/x86_64/os/Packages/perl-IO-Tty-1.12-11.el8.x86_64.rpm
yum -y install perl
rpm -i perl-IO-Tty-1.12-11.el8.x86_64.rpm
rpm -i perl-IPC-Run-0.99-1.el8.noarch.rpm
yum install -y https://download.postgresql.org/pub/repos/yum/reporpms/EL-8-x86_64/pgdg-redhat-repo-latest.noarch.rpm 
yum install postgresql15-devel  
```
  
**RHEL, ROCKY 9.x**
```bash
wget https://repo.almalinux.org/almalinux/9/CRB/x86_64/os/Packages/perl-IO-Tty-1.16-4.el9.x86_64.rpm
wget https://repo.almalinux.org/almalinux/9/CRB/x86_64/os/Packages/perl-IPC-Run-20200505.0-6.el9.noarch.rpm
yum -y install perl
rpm -i perl-IO-Tty-1.16-4.el9.x86_64.rpm
rpm -i perl-IPC-Run-20200505.0-6.el9.noarch.rpm
yum install -y https://download.postgresql.org/pub/repos/yum/reporpms/EL-9-x86_64/pgdg-redhat-repo-latest.noarch.rpm
yum install postgresql15-devel
```


--------------------------------------------------------------------------
참고자료
[devel 설치 시, perl 종속성 문제 해결 1](https://database.sarang.net/?inc=read&aid=10427&criteria=pgsql&subcrit=qna&id=0&limit=20&keyword=&page=1)    
[devel 설치 시, perl 종속성 문제 해결 2](https://postgrespro.com/list/thread-id/2684262)
[postgrssql 구조](https://bitnine.tistory.com/549)
