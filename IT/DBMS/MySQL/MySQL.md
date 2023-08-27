# MySQL Community version 설치, 접속하기

### 설치하기 (rockylinux 9 / mysql 8.0.32 기준)

[이곳에서 다운로드 가능](https://dev.mysql.com/downloads/repo/yum/)  

혹은 기본 yum, apt 레포지토리에서도 인스톨 가능  
```bash
# 레드햇 계열
dnf -y install mysql mysql-server
yum -y install mysql mysql-server

# 데비안 계열
apt-get -y install mysql mysql-server
```

### 접속하기

```bash
# 바이너리 설치의 경우 초기 비밀번호 없음으로 그냥 엔터로 접속.
mysql -uroot -p
```

### mysql db 사용하기
```bash
use mysql;
```

### 패스워드 변경
```bash
alter user 'root'@'localhost' identified with mysql_native_password by '[new password]';
```

### 계정 생성하기
```bash
create user '[user name]'@'[connection ip]' identified with mysql_native_password by '[password]'
```

### 커넥션 빠져나가기
```bash
\q or exit
```

# mysql 아키텍처

MySQL은 프로세스 기반이 아닌 스레드 기반으로 동작한다. 때문에 프로세스를 검색해보면 mysqld 프로세스만이 검색되는 것을 볼 수 있다.  
```bash
ps -ef | grep mysql
mysql      13076       1  0 21:21 ?        00:00:08 /usr/libexec/mysqld --basedir=/usr
root       13136    1270  0 21:39 pts/0    00:00:00 grep --color=auto mysql
```

만약 실행중인 스레드를 확인하고 싶다면 mysql에 접속하여 아래의 쿼리를 통해 확인이 가능하다.
```
mysql> select thread_id, name,type, processlist_user,processlist_host
    -> from performance_schema.threads
    -> order by type, thread_id;
+-----------+---------------------------------------------+------------+------------------+------------------+
| thread_id | name                                        | type       | processlist_user | processlist_host |
+-----------+---------------------------------------------+------------+------------------+------------------+
|         1 | thread/sql/main                             | BACKGROUND | NULL             | NULL             |
|         3 | thread/innodb/io_ibuf_thread                | BACKGROUND | NULL             | NULL             |
|         4 | thread/innodb/io_log_thread                 | BACKGROUND | NULL             | NULL             |
|         5 | thread/innodb/io_read_thread                | BACKGROUND | NULL             | NULL             |
|         6 | thread/innodb/io_read_thread                | BACKGROUND | NULL             | NULL             |
|         7 | thread/innodb/io_read_thread                | BACKGROUND | NULL             | NULL             |
|         8 | thread/innodb/io_read_thread                | BACKGROUND | NULL             | NULL             |
|         9 | thread/innodb/io_write_thread               | BACKGROUND | NULL             | NULL             |
|        10 | thread/innodb/io_write_thread               | BACKGROUND | NULL             | NULL             |
|        11 | thread/innodb/io_write_thread               | BACKGROUND | NULL             | NULL             |
|        12 | thread/innodb/io_write_thread               | BACKGROUND | NULL             | NULL             |
|        13 | thread/innodb/page_flush_coordinator_thread | BACKGROUND | NULL             | NULL             |
|        14 | thread/innodb/log_checkpointer_thread       | BACKGROUND | NULL             | NULL             |
|        15 | thread/innodb/log_flush_notifier_thread     | BACKGROUND | NULL             | NULL             |
|        16 | thread/innodb/log_flusher_thread            | BACKGROUND | NULL             | NULL             |
|        17 | thread/innodb/log_write_notifier_thread     | BACKGROUND | NULL             | NULL             |
|        18 | thread/innodb/log_writer_thread             | BACKGROUND | NULL             | NULL             |
|        19 | thread/innodb/log_files_governor_thread     | BACKGROUND | NULL             | NULL             |
|        22 | thread/innodb/srv_lock_timeout_thread       | BACKGROUND | NULL             | NULL             |
|        23 | thread/innodb/srv_error_monitor_thread      | BACKGROUND | NULL             | NULL             |
|        24 | thread/innodb/srv_monitor_thread            | BACKGROUND | NULL             | NULL             |
|        25 | thread/innodb/buf_resize_thread             | BACKGROUND | NULL             | NULL             |
|        26 | thread/innodb/srv_master_thread             | BACKGROUND | NULL             | NULL             |
|        27 | thread/innodb/dict_stats_thread             | BACKGROUND | NULL             | NULL             |
|        28 | thread/innodb/fts_optimize_thread           | BACKGROUND | NULL             | NULL             |
|        29 | thread/mysqlx/worker                        | BACKGROUND | NULL             | NULL             |
|        30 | thread/mysqlx/acceptor_network              | BACKGROUND | NULL             | NULL             |
|        31 | thread/mysqlx/worker                        | BACKGROUND | NULL             | NULL             |
|        35 | thread/innodb/buf_dump_thread               | BACKGROUND | NULL             | NULL             |
|        36 | thread/innodb/clone_gtid_thread             | BACKGROUND | NULL             | NULL             |
|        37 | thread/innodb/srv_purge_thread              | BACKGROUND | NULL             | NULL             |
|        38 | thread/innodb/srv_worker_thread             | BACKGROUND | NULL             | NULL             |
|        39 | thread/innodb/srv_worker_thread             | BACKGROUND | NULL             | NULL             |
|        40 | thread/innodb/srv_worker_thread             | BACKGROUND | NULL             | NULL             |
|        42 | thread/sql/signal_handler                   | BACKGROUND | NULL             | NULL             |
|        43 | thread/mysqlx/acceptor_network              | BACKGROUND | NULL             | NULL             |
|        41 | thread/sql/event_scheduler                  | FOREGROUND | event_scheduler  | localhost        |
|        45 | thread/sql/compress_gtid_table              | FOREGROUND | NULL             | NULL             |
|        48 | thread/sql/one_connection                   | FOREGROUND | root             | localhost        |
+-----------+---------------------------------------------+------------+------------------+------------------+
39 rows in set (0.00 sec)
```

쿼리의 입장에서의 동작순서는 다음과 같다.  

```
쿼리파서 -> 전처리기 -> 쿼리 옵티마이저 -> 실행 엔진 -> 핸들러(스토리지 엔진) -> 값 반환
```

### 쿼리파서
쿼리 파서는 받아들인 쿼리를 어휘소 단위로 분해하여 트리를 만드는 역할을 한다.
### 전처리기
쿼리 파서가 만든 문장트리를 검사하여 실행을 위한 정보를 추출하는 역할을 한다.
### 쿼리 옵티마이저
전처리기가 넘겨준 정보를 바탕으로 가장 적은 비용으로 해당 작업을 진행하기 위한 수행계획을 세운다.
### 실행 엔진
수립된 수행계획에 맞춰 헨들러에 요청을 수행하며 실질적인 작업을 진행한다.
### 핸들러
실행 엔진의 요청에 따라 스토리지에서 데이터를 읽어오거나 쓰는 역할을 수행한다.