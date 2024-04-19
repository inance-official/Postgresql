## DUAL의 존재

DUAL은 Oracle의 문법. postgresql은 DUAL이 존재하지 않음.
```postgresql
SELECT 'X' AS DUMMY;
```
다음과 같이 가상의 DUAL을 만들어서 사용해야함. 혹은 테이블을 만들거나...

## NEXTVAL

Oracle의 경우
```sql
SELECT TERM_ID_SQ.NEXTVAL 
INTO :o_term_id_sq 
FROM DUAL ;
```

Postgresql의 경우
```postgresql
SELECT NEXTVAL('TERM_ID_SQ') 
INTO :o_term_id_sq;
```

DUAL이 없기 때문에...

## sysdate

sysdate도 oracle의 문법. limit를 사용
```sql
SELECT UPT_SQ 
INTO :o_upt_sq 
FROM ( 
	SELECT * 
	FROM A 
	ORDER BY UPT_SQ DESC ) AS SUB_A
WHERE ROWNUM <= 1 ;
```

```postgresql
SELECT UPT_SQ 
INTO :o_upt_sq 
FROM ( 
	SELECT * 
	FROM A
	ORDER BY UPT_SQ DESC ) AS SUB_A 
LIMIT 1;
```

## NVL -> COALESCE

NULL값을 치환하려고 하면 NVL이 아닌 COALESCE를 사용
```sql
SELECT NVL(MAX(BLCK_ID), 0) + 1 
INTO :i_a.blck_id 
FROM A 
WHERE FIELD_TBL_ID = :i_b.field_tbl_id ;
```

```postgresql
SELECT COALESCE(MAX(BLCK_ID), 0) + 1 
INTO :i_a.blck_id 
FROM A 
WHERE FIELD_TBL_ID = :i_b.field_tbl_id;
```

## 초기화

postgresql은 'ECPG - Embedded SQL in C' 에서 배열 초기화가 불가능.
EXEC SQL 밖에서 메모리를 초기화 해주어야 함.
```c
EXEC SQL BEGIN DECLARE SECTION;
char        i_usr_id     [20+1];
XPM001      o_xpm001;
EXEC SQL END DECLARE SECTION;

memset(i_usr_id,0x00,sizeof(i_usr_id));
```
초기화 하지 않으면, i_user_id 에는 쓰레기 값이 들어가게 되어서 EUC_KR에 어긋나는 글자 값을 뱉어냄.

## indicator

DB에 NULL이 존재하게 되면 FETCH때 indicator가 없을 경우, 에러를 발생.
호스트 변수 선언 시, indicator를 따로 선언 한 뒤, indicator를 사용, 구조체 통째로가 아닌 각각 따로 인디케이터를 설정해줘야함.

기존 코드
```C
    EXEC SQL BEGIN DECLARE SECTION;
    long        i_key               ;

    char        i_inpt_vl   [200+1] ;

    XPM103      o_xpm103;

    EXEC SQL END DECLARE SECTION;

...

EXEC SQL
FETCH cur_XPM11001_XPM103_TLT
INTO :o_xpm103;
```
수정 코드
```c
    EXEC SQL BEGIN DECLARE SECTION;
    long        i_key               ;

    char        i_inpt_vl   [200+1] ;

    XPM103      o_xpm103;

	/* indicator xpm103*/
	int field_id_ind;
	...
	int rgst_tm_ind;

    EXEC SQL END DECLARE SECTION;

...

EXEC SQL
FETCH cur_XPM11001_XPM103_TLT
INTO :o_xpm103.field_id INDICATOR :field_id_ind,
		...
     :o_xpm103.rgst_tm INDICATOR :rgst_tm_ind;
```

