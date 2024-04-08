
DB 접속
```bash
~# su postgres
bash$ psql
```
사용자 계정을 postgresql의 마스터인 postgres로 변경 후 접속한다.

DB 사용자 생성
```postgresql
CREATE USER [username] WITH PASSWORD '[password]';
```

DB 생성
```postgresql
CREATE DATABASE [dbname];
```

DB 소유 할당
```postgresql
ALTER DATABASE [dbname] OWNER TO [username];
```

postgresql 접속
```postgresql
psql -h [host] -p [port] psql -U [username] -d [dbname]
```
-h 와 -p 를 입력하지 않을 시, localhost와 5432 port에 접속을 시도한다.

postgresql 종료
```
\q
```
