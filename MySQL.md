# 1. 데이터 베이스의 종류

- Key-value DB
  - 오로지 키와 밸류 잘 사용하지않음.
  - redis(레디스) 는 많이 쓴다. 왜냐믄 데이터를 일차로 램에 저장해서 빠름. (하드에 백업)
  - 메인 DB를 두고 redis에 일부 데이터를 캐싱해둠 (서브 디비로 활용)
- Relational DB
  - 엑셀 표 생각하면 쉬움. relation 행렬에서 착안.
  - 정규화(중복은 다른테이블로 쪼갬) /트랜젝션(중요거래 안전)이란 장점
  - 정규화때문에 입출력 문법 복잡해진다. 속도 단점
  - 데이터 정확도 높음. 안정성 높음.
  - DB 점유권 가장 높음 ex) 오라클, MySQL 등
- Graph DB
  - 관계를 나타내기 위해 사용
  - 노드에 데이터 저장. 노드별 관계 저장. ex) neo4j(네오포제이)
  - 비행기 노선, SNS 친구관계, 코로나 전염 맵 등
- Document DB
  - 일반적인 상황에서는 Relational or Document DB 사용 ex) mongo DB
  - Collection 이라는 폴더에 document 라는 파일을 만들어서 파일 안에 json 형태로 저장
  - 구조 변경되어도 오류안남. 중복제거 / 정규화 안함. 입출력 간단.
  - 분산처리에 좋아서 입출력이 많은 서비스에 많이 사용.
  - DB 간 정확도/일관성 떨어질 수 있음.
- Colum-family DB
  - ex) cassandra, google bigtable
  - 테이블을 안에 로우를 여러개 만들어서 로우 마다 자유롭게 컬럼을 만들어서 데이터 기입.
  - 정규화 안함. 분산처리 잘함.
  - 입출력이 많은 경우 사용. 시계열 데이터 저장에 용이 (넷플릭스에서 쓴다고함)
- Search engine
  - ex) elastic search, Amazon CloudSearch
  - index 보관에 특화됨

# 2. MySQL & DBeaver 설치

- DBMS (Database Management System) 설치
  - 데이터입출력 쉬워짐
  - DB접속 계정발급 가능
  - 백업도 쉬워짐
  - ex) MySQL, PostgreSQL, ORACLE, SQL Server
- MySQL 서버 띄워놔야 DB 조작가능
  - brew install mysql //설치
  - brew services start mysql // MySQL 서버 띄우기
  - mysql -u root -p //실행
- DBeaber(디비버) 이용하여 조작해보자 (워크밴치랑 동일하지만 깔끔/쉬운 GUI 툴)

# 3. DML (Data Manipulation Language)

- **문법 개념을 명확하게 정리하는 습관을 가지자. 어려운 문제 해결에 도움이 된다.**
- **코드 짜기 전에 한글로 정리해보면 쉬워진다. 복잡한 문제 해결할 때 도움이 된다.**
- 문법은 용도를 잘 기억하자.

1. WHERE
   - 필터링을 위해 사용. 조건식을 주려고 사용.
   - NOT 과 != 는 완전히 동일하다. NOT 은 조건식 앞에 붙인다. '조건식에 해당되지 않는 것' 말할때 사용
   - IN 은 같은 '카테고리'에서 만 사용 가능
   - 조건식 간 연결에는 OR 나 AND 를 사용
   - LIKE를 사용할땐
     - 성능 주의 index 사용 못한다.
     - CHAR 주의 '%소파' 로 찾으면 CHAR(6) '가죽소파\_\_' 검색 안된다..
2. ORDER BY
3. 집계함수
   - AVG(DISTINCT 연체횟수) 가능
   - 일반 컬럼 과 함께 사용할때는 GROUP BY 가 반드시 필요. 레벨이 다름.
   - 당연히 ORDER BY 로도 최대, 최소를 구할 수 있음. 어떨땐 정렬 + LIMIT 1 이 MAX, MIN 보다 빠름.
     - 참고로 LIMIT 은 SELECT 결과를 자르는 역할이라 가장 마지막에 사용한다.
4. 컬럼에 사칙연산 넣기 가능
   - ex) SELECT 사용금액 / 결제횟수 FROM ~
5. 문자 다루는 함수 존재
   - CONCAT(컬럼명1, **'(공백)'**, 컬럼명2, '메롱'): 문자끼리 이어붙여 출력 가능
     (숫자를 넣어도 붙여서 문자로 만들어준다.)
   - TRIM(문자): 앞뒤 공백제거
     - 모든 공백을 제거하려면 REPLACE 를 써야겠지용
   - REPLACE(컬럼명, '패','훼') : '패'를 '훼'로 변경
   - SUBSTR(컬럼명, 몇째글자부터, 몇자): 문자의 일부만 출력 가능
     - RIGHT(번호, N): 번호 뒤에서 N자리
       (번호의 자리수가 고정일거란 법 없으니깐.. 참고로 오라클엔 RIGHT 없음)
     - SUBSTR(번호, LENGTH(번호) - (N-1)): 마찬가지로 번호 뒤에서 N 자리
   - INSERT(컬럼명, 몇째글자부터, 몇자, 바꿀단어): 문자의 일부만 변경해서 출력 가능
     - ex) SELECT INSERT('test@naver.com',1,4,'hello')
6. 숫자 다루는 함수 존재
   - GREATEST: 최댓값
   - LEAST: 최솟값
   - FLOOR: 소수부분 내림 정수로 변환
   - CEIL: 소수부분 올림 정수로 변환
   - ROUND(숫자, 자릿수): 소수부분 반올림
   - TRUNCATE(숫자, 자릿수): 소수부분 내림
     - Oracle, Postgres에선 TRUNCATE() 말고 TRUNC()
   - POWER: 거듭제곱
   - ABS: 절댓값
7. 서브 쿼리
   - 문자나 숫자가 들어가는 자리에만 서브쿼리를 사용할 수 있다.
     - 문자나 숫자의 배열인 IN 안에서도 서브쿼리를 사용할 수 있다.
   - 데이터 대신 서브쿼리 넣기 가능
   - 한 개의 데이터만 뱉는 쿼리문만 서브쿼리 역할 가능 (IN 은 예외.. ㅎ)
   - 소괄호 반드시 사용해야함 !!
8. GROUP BY / HAVING
   - WHERE 를 여러번 써서 통계를 내고 싶지 않을 때 GROUP BY 를 사용한다.
   - 카테고리 컬럼에 주로 사용. 분류를 위해서.
     - ex) SELECT AVG(사용금액) FROM card WHERE 고객등급 = 'vip';
     - SELECT AVG(사용금액) FROM card WHERE 고객등급 = '패밀리';
     - SELECT AVG(사용금액) FROM card WHERE 고객등급 = '로열';
     - > SELECT 고객등급, AVG(사용금액) FROM card GROUP BY 고객등급 ;
   - GROUP BY 결과를 필터링 하고싶으면 HAVING
     - WHERE 는 전체 테이블의 필터. HAVING 은 GROUP BY 테이블의 필터.

> [!note] SQL 실행 순서

    1) FROM
    2) WHERE
    3) GROUP BY
    4) HAVING
    5) SELECT
    6) ORDER BY
    7) LIMIT
    - 이 순서대로 이전 순서의 실행에서 정한 Alias 를 그대로 사용할 수 있다.
    - 예외적으로 MySQL만 SELECT 에서 정한 Alias HAVING에서 사용 가능
    - SELECT의 Alias는 “SELECT 단계 끝나고” 만들어짐
    - 따라서 SELECT MAX(사용금액) AS 최대, MIN(사용금액) AS 최소, 최대/최소 FROM card 안됨

9. IF/CASE
   - IF(조건식, 조건식이 맞을 떄 남길 값, 조건식이 틀릴 때 남길 값)
     - 양자택일밖에 못함
   - CASE
     - N자 택일 까쥐

```sql
CASE
WHEN 조건식1 THEN 남길값1 -- 윗줄부터 읽다가 조건식이 참이면 오른쪽 값을 남기고 종료
WHEN 조건식2 THEN 남길값2
WHEN 조건식3 THEN 남길값3
ELSE 남길값4
END
```

# 4. DDL (Data Definition Language)

```sql
CREATE DATABASE test -- 데이터베이스 생성
DROP DATABASE test -- 데이터베이스 삭제
CREATE TABLE test.new_table( -- 테이블 생성
id int AUTO_INCREMENT PRIMARY KEY,
-- AUTO_INCREMENT: 1씩 자동 증가
-- PRIMARY KEY 제약 (NOT NULL, UNIQUE 자동부여)
주민등록번호 varchar(20) NOT NULL UNIQUE, -- NOT NULL, UNIQUE 제약
이름 varchar(100) DEFAULT '홍길동', -- DEFAULT 옵션
나이 int CHECK(나이 > 0) -- 나이 0보다 큰 데이터만 저장 가능 제약
CONSTRAINT 제약조건작명 CHECK(LENGTH(주민등록번호) = 13) -- 별도 제약 추가 가능
);
DROP TABLE test; -- 테이블 삭제
ALTER TABLE new_table ADD 주소 varchar(1000); --테이블 행 추가
ALTER TABLE new_table MODIFY COLUMN 이름 varchar(100) DEFAULT '홍길동'; -- 테이블 수정
```

# 5. 정규형

1. 제1정규화 (1NF)
   - 한 칸에는 하나의 데이터만 보관
   - 제1정규형 테이블: 제1정규화를 만족하는 테이블
   - 왜쓰냐면, 한 칸에 두개 이상의 데이터를 넣어야 한다면 찾기도 수정하기도 어려워지므로.
     물론 JSON, Array 사용 시 저장은 가능하지만 수정이 어렵고 느리다.
2. 제2정규화 (2NF)
   - partial dependency 를 제거한 테이블. 하나의 composite primary key 에 종속되면 테이블 분리. <small>\*composite primary key: 기본키(PK)를 컬럼 2개 이상 조합해서 만드는 것</small>
   - 현재 테이블의 주제와 관련 없는 컬럼을 다른 테이블로 빼는 작업
   - 왜쓰냐면, 중복 데이터가 적어지므로 수정이 쉬워진다.
   - 테이블이 쪼개지므로 조회가 다소 불편해진다.
   - **관계형은 주로 제2정규화를 만족하는게 일반적, 비관계형은 안하는 경우도 많음**
3. 제3정규화 (3NF)
   - 제2정규형에서 transitive dependency 제거한 버전 <br><small>\* transitive dependency:
     PK 컬럼 A에 종속하는 일반 컬럼 B가 있고,<br>
     B에 종속하는 또 다른 일반 컬럼 C가 있다면, C는 A에 transitive dependency 가 있다고 표현한다. </small>
   - composite primary key가 없을 경우, 일반 컬럼에만 종속된 컬럼을 다른 테이블로 분리
   - **관계형은 주로 제3정규화 까지 만족하는 편** 그럼 대부분의 중복 문제는 해결된다.
4. 제 4, 5, 6정규화 등 더 있다.
5. 유의사항
   - PK는 있는게 좋다. 왜? 중복 데이터가 있을때 참조를 어떻게 할 것인가 생각해보면 쉬움.
   - 따라서 다른 테이블의 데이터를 사용할 땐 PK 를 적는게 좋다.
   - Foreign key: 다른 테이블의 PK 사용하는 컬럼
     - 등록해두면 조회가 쉽고, 참조중인 테이블이 실수로 삭제되는걸 방지할 수 있다.

# 6. JOIN

1. INNER JOIN : 교집합

```sql
-- 1) 그냥 ,로도 붙일 수 있다.
SELECT * FROM program, teacher
WHERE 강사id = teacher.id;
-- 2) 아래 코드나 위 코드나 같은 결과가 나온다. 아래가 가독성이 더 좋다. (필터링은 ON)
SELECT * FROM program INNER JOIN teacher
ON 강사id = teacher.id;
-- 3) CROSS JOIN 은 가능한 모든조합을 뽑아준다.
-- SELECT * FROM program, teacher 와 아래 코드는 동일하다.
SELECT * FROM program CROSS JOIN teacher;
```

2. LEFT JOIN / RIGHT JOIN : 접점이 없는 행들 + 교집합

- 빵꾸 찾기에 좋다
- 참고로 `LEFT JOIN` 와 `LEFT OUTER JOIN` 은 **완전히 같은 문법**

3. FULL JOIN : LEFT JOIN + RIGHT JOIN

- MYSQL, MariaDB, SQLite 에선 사용할 수 없다.

4. 참고용 이미지 ![[Pasted image 20260507234342.png|599]]

# 7. INSERT

```sql
CREATE TABLE stock (
id int AUTO_INCREMENT PRIMARY KEY,
상품명 varchar(100),
가격 int
)

INSERT INTO stock (id, 상품명, 가격) VALUES (1, '김치', 500);
INSERT INTO stock VALUES (1, '김치', 500); -- 모든 컬럼을 넣을때는 컬럼명 생략가능
INSERT INTO stock (상품명, 가격) VALUES ('김치', 500); -- AUTO_INCREMENT 자동 1씩증가
INSERT INTO stock SELECT * FROM stock2; -- VALUES 대신 테이블 복사도 가능

/* 기존테이블명 복사해서 새로운테이블명 생성 */
SELECT INTO 새로운테이블명 FROM 기존테이블명; -- mysql 외
CREATE TABLE 새로운테이블명 SELECT * FROM 기존테이블명 -- mysql
CREATE TEMPORARY TABLE 새로운테이블명 SELECT * FROM 기존테이블명 -- 재접속시 사라짐
```

- MySQL Workbench 프로그램 상단메뉴 Server - Users and Privileges 누르면 특정 DB 접속용 계정생성 & 권한부여 가능

# 8. UPDATE / DELETE

```sql
UPDATE stock -- 테이블명
SET 가격 = 가격 + 1000, 상품명 = '석박지' -- 수정할 컬럼
WHERE id = 1 -- 수정할 행

UPDATE stock
SET 가격 = 가격 + 1000; -- WHERE 조건이 없으므로 모든행에 적용

DELETE
FROM stock -- 테이블명
WHERE id = 1

/*조인 테이블 가능*/
UPDATE A INNER JOIN B ON 조건식 SET ...
DELETE FROM A INNER JOIN B ON 조건식 WHERE ...
```

- Foreign key 로 사용중인 행은 삭제 불가능

# 9. UNION

- `SELECT` 두번 하기 싫어요. 한번에 보고싶어요.
- 컬럼 갯수 맞아야 함 주의. 명은 달라도 됨 신기.
- JOIN 은 테이블을 양옆으로 붙임. UNION 은 위아래로 붙임.

```sql
SELECT * FROM stock
UNION
SELECT * FROM stock2;
```

# 10. VIEW

- 가상의 테이블 (실제로 테이블이 생성되는게 아니라 SELECT 문법을 테이블형태로 눈속임해주는 것일 뿐)
- 실제 테이블이 아니라서 테이블만큼 하드용량을 많이 차지하지 않는다. 재사용하기 좋다.
- 삭제 전까지 유지된다. 왜 ? DB 서버 내부 메타데이터에 저장된다. 로컬 MySQL 쓰면 PC 특정 경로에 저장.

```sql
SELECT * FROM stock
UNION
SELECT * FROM stock2;

DROP VIEW view2;
```

# 11. procedure

- 긴 쿼리문을 재사용하고 싶다면 ? stored procedure를 사용해보자
- ==DB 접속하는 사람들끼리 공유 가능==

```sql
-- 이미 존재한다면 삭제
DROP PROCEDURE IF EXISTS mart.go_pro;

-- 생성
DELIMITER $$ -- 딜리미터 잠시 변경해두고
$$
CREATE PROCEDURE mart.go_pro()
BEGIN
	SELECT * FROM mart.product ;
END
$$
DELIMITER ;

--호출
call mart.go_pro();
```

1. 속도 향상 효과 (아주 조금)
   ![[Pasted image 20260508224106.png|509]]

2. 변수

- SET @변수명 : 전역변수 역할. DB 접속 종료 시 사라진다.
- DECLARE 변수명 : 지역변수 역할. 프로시저 실행 종료 시 사라진다.

```sql
SET @hello = '안녕하세요 반갑습니다'; -- 우측에 서브쿼리를 넣어도 상관없다.
SELECT @hello;
SET @older = 31;
SET @older = @older + 1; -- 수정도 된다.

DROP PROCEDURE IF EXISTS mart.var_test;
CREATE PROCEDURE mart.var_test()
BEGIN
DECLARE var1 int DEFAULT 123; -- procedure 안에서 선언하려면 DECLARE
DECLARE var2 varchar(100) DEFAULT 'Hi';
SET var1 = 123; -- procedure 안에서 변경하려면 SET
SELECT var1, var2;
END;
```

3. 파라미터

- 프로시저 안에서 변수를 사용할 수 있다. 마치 메소드 처럼

```sql
-- 이미 존재한다면 삭제
DROP PROCEDURE IF EXISTS mart.go_pro;

-- 생성
DELIMITER $$ -- 딜리미터 잠시 변경해두고
$$
CREATE PROCEDURE mart.go_pro(param int, param2 varchar(100))
BEGIN
SELECT * FROM mart.product
WHERE 가격 = param AND 상품명 = param2;
END
$$
DELIMITER ;

--호출
CALL mart.go_pro(5000,'셔츠');
```

- 프로시저 밖으로 변수를 꺼낼 수도 있다. 함수처럼 return 도 가능

```sql
CREATE PROCEDURE mart.get_age(OUT param int)
BEGIN
SET param = 20;
END

CALL mart.get_age(@param2);
SELECT @param2;
```

# 12. 날짜 & 시간

##### 1) Mysql

| Data type   | 형식                   |
| ----------- | ---------------------- |
| DATE        | YYYY-MM-DD             |
| TIME        | hh:mm:ss               |
| TIMESTAMP   | YYYY-MM-DD hh:mm:ss    |
| TIMESTAMPTZ | YYYY-MM-DD hh:mm:ss+hh |

##### 2) PostgreSQL

| Data type   | 형식                   |
| ----------- | ---------------------- |
| DATE        | YYYY-MM-DD             |
| TIME        | hh:mm:ss               |
| TIMESTAMP   | YYYY-MM-DD hh:mm:ss    |
| TIMESTAMPTZ | YYYY-MM-DD hh:mm:ss+hh |

```sql
-- 기입은 문자처럼
INSERT INTO newTable values('2020-01-01 12:10:00');

-- 특정 날짜만 조회
SELECT * FROM blog
WHERE 발행일 > '2022-03-10 00:00:00'
AND 발행일 < '2022-03-11 00:00:00';

-- BETWEEN 은 이상 ~ 이하 이므로 사용하지 않는 게 좋다. 23.59.59.9 는 누락됨.
WHERE 발행일 BETWEEN '2022-03-10 00:00:00'
AND '2022-03-10 23:59:59';

-- 현재날짜까지 출력하고 싶을땐
AND 발행일 < now();

-- 포맷팅 가능
SELECT date_format('2022-03-09 23:24:25', '%Y %m %d %H %i %s');
```

# 13. function

- 사실 function 도 있다. 문법도 procedure 와 거의 동일.
- call 없이 사용 가능하며 파라미터 넣을 수 있고, return 반드시 써야한다. 파라미터는 function 이 더 직관적
- ==**그럼 왜쓰지 ?** == - 긴 쿼리문 재사용에는 `procedure` - 계산기능 등 유닛 생성에는 `function`
  > [!warning]
  > MySQL 안에선 SELECT 사용금지
  > SELECT 로 찾은 결과를 변수에 집어넣어주는 `SELECT 어쩌구 INTO 변수명` 은 사용 가능

```sql
DROP FUNCTION IF EXISTS data_test.yen;
DELIMITER $$
$$
CREATE FUNCTION data_test.yen(won INT )
RETURNS INT
DETERMINISTIC
BEGIN
  RETURN won * 0.1 + 50;
END
$$
DELIMITER ;

--사용
SELECT yen(가격) FROM mart.product;
```

> [!info]
> `DETERMINISTIC` 는 성능때문에 사용한다. DB한테 함수의 성격을 알려주는 용도의 옵션.
> DB가 최적화/복제 처리할 때 참고할 수 있어서, 같은 입력에 항상 같은 결과가 나오면 써주는 게 좋다.
> 함수가 항상 같은 값을 RETURN 하면 `DETERMINISTIC` 랜덤하면 `NOT DETERMINISTIC`.
>
> 같은식으로
> `SQL` 문법 안쓰면 `NO SQL`
> `SELECT` 쓰면 `READS SQL DATA`
> `INSERT` `DELETE` 쓰면 `MODIFIES SQL DATA`

1. IF 안의 SQL

- procedure, function 안에서만 사용 가능

```sql
DROP PROCEDURE IF EXISTS mart.test;
DELIMITER $$
CREATE PROCEDURE mart.test()
BEGIN
	--사용법
	IF 조건식1 THEN
		실행할 SQL 쿼리문;
	ELSEIF 조건식2 THEN
		실행할 SQL 쿼리문;
	ELSE
		실행할 SQL 쿼리문;
	END IF;

	-- 실습
	IF (SELECT SUM(사용금액) FROM mart.card) > 5000000 THEN
	    SELECT '잘했어요';
    ELSE
	    SELECT '분발하세요';
	END IF;
END $$
DELIMITER ;
```

# 14. index

조회를 빨리 하려면 ? 이진탐색. 이진탐색 하려면? 미리 정렬을 해야한다.
검색을 빠르게 만들기 위해 =="미리 정렬해둔 컬럼복사본"== 을 index라고 한다.

1. **Binary search tree** : 이진탐색 원리를 트리 구조에 적용한 자료구조. 데이터를 Binary tree 형태로 미리 배치해놓아야 빠르게 찾을 수 있다. index도 tree 형태로 배치해두는 경우가 많다.
2. **B-tree** : 노드에 값을 여러개 둔 이진탐색트리 . 50%가 아닌 66% or 75%씩 소거가능
   ![[Pasted image 20260510015720.png|437]]
3. **B+tree** : B-tree와 정렬방식이 똑같은데 데이터를 가장 밑에만 저장. 데이터 탐색 가이드라인만 윗부분에 저장. 추가로 내 옆 데이터들도 화살표로 몰래 연결. 따라서 범위검색이 빠름. **따라서 관계형 DB들은 일반적으로 B+tree로 index를 정렬한다.**
   ![[Pasted image 20260510015812.png]]
4. 추가로 생각해볼 부분

- LIKE 사용시엔 시작이 %기호가 아닐 때만 (LIKE 어쩌구% 일 때만) index를 사용한다.
  생각해보면 당연함. 인덱스는 앞글자 기준으로 정렬되어 있으므로 Full Scan 가능성이 크다.
- 전체대비 출력할 행이 20%를 넘으면 index를 사용하지 않을수도 ? (때마다 옵티마이저가 판단)
  찾는 비용이 줄어드는 것 보다, index 에서 찾고 원래 테이블의 행을 다시 찾아가는 비용이 커져서 오히려 손해일 때가 있으므로 Full Scan 이 이득이라고 판단할 경우가 있다.
- composite index 도 가능
  `SELECT * FROM 어쩌구 WHERE name = 'park' AND age = 20` 처럼 컬럼 2개 이상에서 검색을 수행하고 있는 경우, 각각을 index 로 만드는 것 보다 묶어서 index 로 만들어두면 성능이 향상될 수 있다.

  > [!Tip]
  > 다중컬럼 index 를 만들 때, **cardinality**(구분명확도) 가 높은 컬럼을 먼저 즉, 왼쪽에 넣는게 좋다.
  > 복합 인덱스는 **B-Tree 안에 `(name, age)` 라는 하나의 복합 키를 하나의 노드로 저장하는 구조** 란걸 생각하자. 이걸 이해하면 왜 컬럼 순서가 중요한지도 자연스럽게 이해할 수 있다.
  > 컬럼 순서가 정렬 기준이 되므로 `(name, age)` 인덱스는 `(name)`, `(name, age)` 두 역할을 어느 정도 동시에 수행하지만, `(age)`는 수행할 수 없다.

- index는 자동생성되진 않는다. 아래 단점이 존재하기 때문이겠지? 하지만 PK 컬럼은 자동으로 적용된다.

5.  ==**index 의 단점**==

- index를 만들 때 마다 추가 하드용량을 차지한다. (사실 컬럼 한두개 복사하는 거라 별문제는 아님)
- index가 있는 테이블은 나중에 삽입, 수정, 삭제 문법으로 데이터 넣을 때 내부적으로 추가연산이 필요해 느려질 수 있다.

6. execution plan 분석

- cost: 소요시간 (10만이 넘어가면 PC에 부담이 될 수 있다.)
- access_type: ndex, range, ref, const 등 가능. ALL만 피하면 된다. (전체 행 full scan)
- filtered: 컴퓨터가 읽은 행을 출력결과에 넣는 비율. 100% 에 가까울 수록 좋다.'

# 15. Full Text search

게시판 검색 기능 구현하려면?
우선 긴 글도 text 타입을 사용하면(6만 5천자까지 보관가능) 컬럼 하나로 관리가 가능하다.

##### 1. like

%를 맨앞에 사용하면 인덱스 활용을 못한다. 문장이 길거나 행이 많아지면 매우 느리게 동작한다.

##### 2. full text index

| id  | 글                   |
| --- | -------------------- |
| 1   | I run regularly.     |
| 2   | I eat breakfast.     |
| 3   | I like running.      |
| 4   | I like eating pizza. |
| 5   | I swim in the sea.   |

###### ▲ 이런 테이블이 있다고 치자. 이 테이블에 full text index를 만들라고 시키면 아래와 같이 출몰중인 행을 적으면 된다. 이게 full text index

| 단어 | 어떤 행에 나오냐면 |
| ---- | ------------------ |
| eat  | 2, 4               |
| run  | 1, 3               |
| swim | 5                  |

문장 안에서 stopwords라고 부르는 "is the a are and I" 등 내용과 상관없는 쓸데없는 단어들을 제거하고 index를 만드는 경우가 많다. (웹의 검색엔진들이 is the a are 이런 내용 대부분 무시하는 이유)

```sql
-- 생성할라믄
CREATE FULLTEXT INDEX 인덱스이름작명 ON 테이블명(컬럼명);

-- 검색할라믄
-- MATCH() AGAINST()가 필요하며, WHERE 뒤에 조건식 형태로 넣을 수 있다.
-- 한두글자는 검색이 안된다. (따로 설정 바꾸면 가능)
SELECT * FROM library WHERE MATCH(서명) AGAINST('부동산');
```

##### 3. IN BOOLEAN MODE

사실 위처럼 모드 작성을 생략하고 `MATCH() AGAINST()` 를 사용하면 `IN NATURAL LANGUAGE MODE` 라는 모드로 검색해주는 중이다.
더 다양한 기능을 지원하는 `IN BOOLEAN MODE` 가 있다.

```sql
SELECT * FROM library WHERE MATCH(서명) AGAINST('부동산' IN BOOLEAN MODE);

-- * 기호를 넣어주면 %기호와 유사하게 동작 (부동산을, 부동산이 이런거 검색됨)
SELECT * FROM library WHERE MATCH(서명) AGAINST('부동산*' IN BOOLEAN MODE);
-- 여러 단어를 넣으면 OR 연산 (부동산 또는 종이접기가 포함된 결과 출력)
SELECT * FROM library WHERE MATCH(서명) AGAINST('부동산 종이접기' IN BOOLEAN MODE);
-- + 기호를 앞에 붙이면 AND 연산 (해당단어가 꼭 들어가있는 것만 찾아줌)
SELECT * FROM library WHERE MATCH(서명) AGAINST('+부동산 +빅데이터' IN BOOLEAN MODE);
-- - 기호를 앞에 붙이면 NOT 연산 (해당단어 걸러줌)
SELECT * FROM library WHERE MATCH(서명) AGAINST('-부동산 +빅데이터' IN BOOLEAN MODE);
```

##### 4. **n-gram parser**

일본어, 중국어라서 띄어쓰기가 없다면 ? 한국어여도 띄어쓰기를 잘못 입력했다면 ? '부 동 산' 입력했으면 ?
몇 글자단위로 잘라서 index를 생성하는 방식이 있는데 이걸 `n-gram parser`라고 한다.

###### '주식 투자' 라는 문자를 n-gram으로 index를 생성하면 아래처럼 띄어쓰기 무시하고 2자 단위로 생성

| 단어 | 어떤 행에 나오냐면 |
| ---- | ------------------ |
| 주식 | (생략)             |
| 식투 | (생략)             |
| 투자 | (생략)             |

```sql
CREATE FULLTEXT INDEX 인덱스이름작명 ON 테이블명(컬럼명) WITH PARSER ngram;
```

==장단점==

- 지엽적인 책 제목까지 싸그리 가져올 수 있다
- 하지만 하드용량을 많이 차지할 수 있고
- 쓸데없는 결과도 많이 출력된다 ('철학을' 로 찾으면 '철학' 내용 '학을' 내용 같이나옴)

##### 5. elastic search

형태소 분석 + 역색인 + 점수 계산 기반으로 빠르고 똑똑한 검색을 제공
DBMS 마다 문법이 다른 경우가 많아 따로 파악해보자.

# 16. Transaction

작업을 하나의 논리적인 단위로 묶어서 처리하는 것. 가장 쉬운 필요 예시가 계좌이체.

- 추가로 UPDATE, DELETE를 직접 해야하는데 잘못될까봐 손발이 바들바들 떨리면 transaction으로 먼저 테스트부터 해볼 수도 있다.
- START TRANSACTION / COMMIT / ROLLBACK 문법 사용

```sql
START TRANSACTION; -- TRANSACTION 모드 ON

INSERT INTO mart.product VALUES (100, 'a', 'a', 1, 1);
INSERT INTO mart.product VALUES (101, 'b', 'b', 2, 2);

COMMIT; -- 변경 보류사항 전부 반영
ROLLBACK; -- 변경 보류사항 전부 취소
```

```java
// 웹서버에서는 이런식으로 작성하면 된다.
try {
    START TRANSACTION;
    // A 고객 잔고 -1000
    // B 고객 잔고 +1000
    COMMIT;
} catch (Exception e) {
    ROLLBACK;
}
```

- **auto commit**
  테이블 만들 때 엔진을 InnoDB로 설정해놓으면 (기본이 InnoDB임) 모든 일반 쿼리문은 하나의 transaction으로 처리해준다. (모든 쿼리문을 한 줄 마다 start transaction / commit으로 몰래 감싸놓고 실행해준다는 뜻)
  이걸 ==auto commit== 기능이라고 한다.
  START TRANSACTION 코드를 실행하면 auto commit 은 자동으로 꺼진다. 혹은 예전 MySQL 5.x 버전에선 auto commit 옵션을 끄는 명령어도 있는데 그걸 써야 가능
- **ACID 속성**
  transaction 기능을 ACID transaction 이렇게 부르는 경우가 있는데,
  Atomicity, Consistency, Isolation, Durability 속성을 가져야 transaction이라고 부를 수 있다는 transaction 기능의 교과서적 정의이다.
  - **Atomicity -** 1개의 transaction 안에 있는 코드들은 전부 실행되거나 실행되지 않거나 택1 해야한다
  - **Consistency** - transaction은 조작하려는 테이블의 제약조건과 룰 같은걸 잘 따라야한다 
  - **Isolation** - transaction 끼리는 서로 간섭하지 않아야한다 
  - **Durability** - transaction이 commit 되면 데이터베이스에 영구적으로 기록되어야한다 (심지어 도중에 컴퓨터 전원을 껐다 켜도) 

# 17. hosting

MySQL을 컴퓨터에 설치하면 내 컴퓨터에서 데이터베이스를 이용할 수 있다.
하지만 많은 사람들이 이용하게 하고 싶으면 데이터베이스를 직접 컴퓨터에 설치하는 것 보다 AWS, GCP 같은 클라우드서비스에서 하나 호스팅받는게 훨씬 안전하다.
유료긴 하지만, 아래기능 지원해준다.

- 자동 백업
- 자동 스케일업
- 모니터링 
- 로그작성 
- DB 접속 허용 지역설정, 접속 유저 관리
  **AWS RDS 또는 Aurora** 를 많이 사용한다. AWS Aurora는 RDS와 비슷한데 내부적으로 컴퓨터와 데이터베이스가 분리되어 있다. 대부분의 상황에서 Aurora가 더 빠르고 유연하다.
  당근마켓도 RDS 쓰다가 Aurora로 이전했다고 함

**1. aws.amazon.com 가입**
**2. 우측상단 지역을 서울로 바꾸고**
**3. DB 접속용 아이디 비번** 만들어주고
**4. 퍼블릭 엑세스 가능으로 변경해주고 접속가능한 IP 설정해 주면 끝**
**5. 이제 아래와 같이 디비버에서 접속 / 사용 할 수 있다.**

- Host : AWS 콘솔의 '엔드포인트' 적으면 되고
- Database : AWS 콘솔의 구성 메뉴에서 볼 수 있는 'DB 이름' 적으면 되는데 설정한 기억이 없으면 비워두면 되고
- Username, Password : 아까 기입했던 DB 접속용 아이디 비번 입력하면 된다.
  ![[Pasted image 20260511010718.png|635]]

# 18. Trigger

INSERT, UPDATE, DELETE 실행 시 자동으로 다른 코드 실행되게 만들려면 사용
아래처럼 ==통계작업, 검증작업 에 유용==

- 데이터를 다른 테이블에 반영하고 싶을 때
- 데이터 넣기 전에 데이터를 깔끔하게 정제하고 싶을 때 
- 테이블 변경기록 (로그)를 다른 테이블에 저장해두고 싶을 때
- 테이블 변동사항 생길 때 마다 통계를 다른 테이블에 기록해두고 싶을 때

```sql
CREATE TABLE trigger_test.product( 상품명 varchar(100), 가격 int );
CREATE TABLE trigger_test.counter( 자료 varchar(100), 값 int );
-- product 테이블에 데이터 넣을 때 마다
-- counter 테이블에 있던 상품갯수 숫자에 +1을 해주고 싶다면 ?

-- trigger 생성
DROP TRIGGER IF EXISTS db명.트리거이름;
DELIMITER $$
CREATE TRIGGER db명.트리거이름
AFTER INSERT ON db명.테이블명 -- AFTER 대신 BEFORE, INSERT 대신 UPDATE, DELETE
FOR EACH ROW
BEGIN
	실행할코드
	-- OLD: 변경전 데이터
	-- NEW: 변경후 데이터
	-- INSERT 될 값이 음수면 0원으로 바꾸고 싶을 경우 아래처럼 작성
	IF NEW.가격 < 0 THEN
		SET NEW.가격 = 0;
	END IF;
END $$
DELIMITER ;

-- 아래와같이 -1000 넣으면 0이 들어간다.
INSERT INTO trigger_test.product VALUES('상품명', -1000);
```

# 19. ERD

테이블의 정보, 테이블과의 관계를 도식화한 관계도. 테이블간의 연관관계를 파악하기 쉽다.
보통 툴에서 자동으로 그려준다. 그치만 기획단계에서 ERD 를 요구할 수 있으므로 그릴줄 알아야..
