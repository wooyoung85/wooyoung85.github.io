# Prerequsite

- Postgresql 설치 (https://www.postgresql.org/)
- path 설정 추가 (`C:\Program Files\PostgreSQL\13\bin\`)
- PowerShell 기반 명령어
- Azure Database for PostgreSQL 사용
- 🚀 <u>**원본 DB 데이터 백업 > Owner 변경 > 새로운 DB에 복원**</u>

# (원본 DB) 백업

> ⭐ 스키마와 데이터 따로 백업

```
$date = (Get-Date).ToString("yyyyMMdd")
$version = 1
$env:PGPASSWORD = "Pass123!@#"
pg_dump -h "testapp-db.postgres.database.azure.com" -p "5432" -d "testapp" -U "testapp" -f "c:\\postgresql_backup\\testapp-schema-$date-$version.sql" -v -s
pg_dump -h "testapp-db.postgres.database.azure.com" -p "5432" -d "testapp" -U "testapp" -f "c:\\postgresql_backup\\testapp-data-$date-$version.sql" -v -a
```

# Schema 백업 파일에서 Owner 변경 (`testapp` 👉 `testappnew`)

- `c:\\postgresql_backup\\testapp-schema-$date-$version.sql` 파일 vscode 로 열어서 Text Replace (`testapp` 👉 `testappnew`)
- `testappnew-schema-$date-$version.sql` 로 저장

# 복원

```
$date = (Get-Date).ToString("yyyyMMdd")
$version = 1
$env:PGPASSWORD = "Pass123!@#"
psql -h "testapp-db.postgres.database.azure.com" -U "testappnew" -d "testappnew" -v sslmode="require" -f "c:\\postgresql_backup\\testappnew-schema-$date-$version.sql"
psql -h "testapp-db.postgres.database.azure.com" -U "testappnew" -d "testappnew" -v sslmode="require" -f "c:\\postgresql_backup\\testapp-data-$date-$version.sql"
```

# 추가 작업

## DB 정보 확인

```
psql "host=testapp-db.postgres.database.azure.com port=5432 user=testapp password=Pass123\!\@\# sslmode=require"
postgres=> \l+
```

## DB 생성

### 일반적인 방법

```
$env:PGPASSWORD = "Pass123!@#"
psql -h "testapp-db.postgres.database.azure.com" -p "5432" -U testapp
```

```sql
CREATE USER testappnew WITH ENCRYPTED PASSWORD 'testappnew!!';
GRANT testappnew TO testapp;
CREATE DATABASE testappnew OWNER testappnew ENCODING 'utf-8';
```

## DB 삭제 후 생성

```
$env:PGPASSWORD = "Tlwps123!@#"
psql -h "testapp-db.postgres.database.azure.com" -p "5432" -U testapp -c "DROP DATABASE testappnew;"
psql -h "testapp-db.postgres.database.azure.com" -p "5432" -U testapp -c "CREATE DATABASE testappnew OWNER testappnew ENCODING 'utf-8';"
```

### 기존 계정으로 생성 (DB 생성 여부도 체크)

```
$env:PGPASSWORD = "Pass123!@#"
if (psql -h "testapp-db.postgres.database.azure.com" -p "5432" -U testapp -tc "SELECT CASE WHEN COUNT(*) = 1 THEN 'EXIST' ELSE 'NOT EXIST' END FROM pg_database WHERE datname = 'testappnew'" | select-string -q "NOT EXIST") {
  psql -h "testapp-db.postgres.database.azure.com" -p "5432" -U testapp -c "CREATE DATABASE testappnew OWNER testapp ENCODING 'utf-8';"
}
```
