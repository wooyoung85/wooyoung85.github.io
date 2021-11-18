> PostgreSQL 을 처음 접하면서 유용했던 부분을 기록해두는 목적으로 작성한 내용입니다 ^^

## Material View

### Material View 생성

> user 와 관련된 테이블을 조인하여 View 생성하는 예제

```sql
CREATE MATERIALIZED VIEW mv_user_set as
SELECT
  row_number() OVER ()                              AS user_set_no,
  u1.user_info_no,
  ...
FROM user_info       u1
JOIN user_item       u2
ON u1.user_no = u2.user_no
WHERE 1 = 1
  AND u1.use_yn::text = 'Y'::text
  AND u2.use_yn::text = 'Y'::text;

ALTER MATERIALIZED VIEW mv_user_set OWNER TO myuser;

CREATE UNIQUE INDEX ix_user_set_no ON mv_user_set(user_set_no);

CREATE INDEX ix_user_set_01 ON mv_user_set(expt_rslt_no, well_id);
```

- `mv_` 와 같이 Material View 임을 식별할 수 있는 Prefix 권장
- `row_number()` 를 활용하여 View 에 `UNIQUE INDEX` 를 생성한 이유
  - 👉 MV 업데이트 시 `CONCURRENTLY` 파라미터를 사용하기 위해
  - View 데이터 동기화를 위해 업데이트를 시도할 경우 관련 테이블에 Lock 이 걸릴 수 있음
  - Postgresql 9.3 이후 버전부터 Concurrecy 기능을 제공

### Material View 업데이트

```sql
REFRESH MATERIALIZED VIEW CONCURRENTLY mv_user_set
```

### Material View 삭제

```sql
DROP MATERIALIZED VIEW mv_user_set;
```

## 반복문을 활용한 데이터 입력

> 생각보다 맘대로 잘 되지 않아서 고생했음 <g-emoji>🤦‍♂️</g-emoji>

```sql
do
$$
  declare
    row       record;
    item_num  int;
  begin
    for row in SELECT user_info_no, reg_id
               FROM user_info
               WHERE user_no
               AND use_yn = 'Y'
               ORDER BY user_no
      loop
        RAISE NOTICE E'INSERT INTO user_item (user_item_no, user_info_no, use_yn, reg_id, reg_dt) VALUES ((select max(user_item_no) + 1 from user_item), %, \'Y\', \'%\', now());', row.user_info_no, row.reg_id;

        INSERT INTO user_item (user_item_no, user_info_no, use_yn, reg_id, reg_dt)
        VALUES ((select max(user_item_no) + 1 from user_item), row.user_info_no, 'Y', row.reg_id, now());

        item_num := (select max(user_item_no) from user_item);

        INSERT INTO user_attrbt (user_attrbt_no, user_item_no, attrbt_nm, attrbt_val, sort_seq, reg_id, reg_dt)
        VALUES ((select max(user_attrbt_no) + 1 from user_attrbt), item_num, 'Test', 'Test', 0, row.reg_id, now());
      end loop;
  end
$$
```

- `declare` 를 사용하여 반복문을 순회하면서 사용할 데이터와 필요한 변수를 정의
- `RAISE NOTICE` 구문으로 로그를 찍을 수 있음

## 임시테이블 생성

```postgresql
DROP TABLE IF EXISTS temp_table;
CREATE TEMP TABLE temp_table as
SELECT * FROM origin_table
```

## 참고자료

[PostgreSQL Materialized Views](https://www.postgresqltutorial.com/postgresql-materialized-views/)  
[PostgreSQL – For Loops](https://www.geeksforgeeks.org/postgresql-for-loops/)
