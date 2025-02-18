## 1. 할인 정책 설정 테이블 생성 SQL
```sql
create table acd_pms_parkinglot_discount_policy
(
    pklp_park_seq      int(11) unsigned not null comment '주차장_순번',
    stor_seq           int(10)          default 0 null,
    pklp_limit_group   int(11) unsigned default 0 null comment '할인 정책 그룹 같은 그룹끼리 별도 정책확인 후 합산',
    pklp_charge_type   tinyint unsigned not null comment '징수유형 (0:전체 1:유료 2:무료)',
    pklp_limit_type    tinyint unsigned not null comment '최대제한유형 (1:시간(분단위) 2:금액 3:비율(퍼센트) 4:횟수)',
    pklp_limit_value   int(11) unsigned not null comment '제한 값',
    pklp_vehicle_type  char default 'A' not null comment '차종유형 : A 기본정책 우선순위',
    is_over            tinyint(1) default 0 null comment '초과값허용 여부 (1:허용, 0:미허용)',
    pklp_del_ny        tinyint unsigned default 0 not null comment '사용여부',
    pklp_reg_ip        char(15) null comment '등록_IP',
    pklp_reg_seq       int unsigned null comment '등록_순번',
    pklp_reg_device_cd tinyint unsigned null comment '등록_디바이스_코드',
    pklp_reg_datetime  datetime(6) not null comment '등록_날짜',
    pklp_mod_ip        char(15) null comment '수정_IP',
    pklp_mod_seq       int unsigned null comment '수정_순번',
    pklp_mod_device_cd tinyint unsigned null comment '수정_디바이스_코드',
    pklp_mod_datetime  datetime null comment '수정_날짜'
)
comment '주차장 할인정책';

```

## 2. 할인 정책 설정 테이블

|할인 정책 설정 |
| --- |
|acd_pms_parkinglot_discount_policy  |

| 컬럼명            | 설명                                                         |
| :---------------- | :----------------------------------------------------------- |
| pklp_park_seq  | 주차장_순번 (필수값)                                       |
| stor_seq        | 스토어_순번 (0: 전체 정책, 스토어순번 지정 시 지정된 스토어 정책 설정) |
| pklp_limit_group  | 할인 정책 그룹 같은 그룹끼리 별도 정책확인 후 합산         |
| pklp_charge_type  | 징수유형 (0:전체 1:유료 2:무료)                              |
| pklp_limit_type   | 최대제한유형 (1:시간(분단위) 2:금액 3:비율(퍼센트) 4:횟수)   |
| pklp_limit_value  | 제한 값 (최대제한유형에 따른 설정 단위의 값)                 |
| pklp_vehicle_type | 차종유형 : A 기본정책 우선순위                               |
| is_over           | 초과값허용 여부 (1:허용, 0:미허용)                           |
| pklp_del_ny       | 사용여부                                                     |

## 3. 할인 정책 기본설정 설명

### 1.기본 정책 기능
주차장 또는 스토어에 대한 징수유형 (0:전체 1:유료 2:무료)별로 
최대제한유형 (1:시간(분단위) 2:금액 3:비율(퍼센트) 4:횟수)을 설정하여 정책 제한 설정을 할 수 있음

### 2. 할인 공통 정책 설정
스토어순번 **[stor_seq]** 을 0 으로 설정하면 주차장 전체 스토어를 대상으로 한 공통 설정이 설정된다

### 3. 스토어 정책 설정(주차장 공통 정책을 무시하고 스토어 정책을 우선으로 함)
주차장의 특정 스토어에 대해서만 정책 설정을 하고 싶다면 특정 스토어의 순번을 스토어순번 **[stor_seq]** 에 설정하면 된다
**<span style="color:#e11d21">주의사항은 스토어 정책 설정이 된 스토어는 동일한 징수유형, 최대제한유형의 주차장 공통 정책을 무시한다</span>**

### 4. 초과값허용 여부를 허용으로 설정하면 제한값의 여유가 있다면 초과되는 할인권을 사용할 수 있다
ex) 현재 총 사용시간이 3시간 30분이고 제한 시간이 4시간이면 초과값허용 여부를 허용으로 하면 30분 이상의 할인권을 적용 할 수 있다

## 4. 할인 정책 그룹 설정
    그룹 설정이 된 스토어들은 해당 스토어들 끼리의 사용 값에 대해서만 합산하여 제한한다
### 1. 할인 정책 그룹 설정을 위한 조건
할인 정책 그룹 **[pklp_limit_group]** 설정을 위해서는 반드시 
스토어순번 **[stor_seq]** 이 0이 아니고 스토어순번 **[stor_seq]** 이 있어야 한다

같은 할인 정책 그룹 **[pklp_limit_group]** 의
같은 징수유형, 최대제한유형에 대해서 제한값은 동일하게 설정해야 한다 

### 2. 할인 정책 그룹 설정 설명
동일한 그룹으로 설정할 스토어에 대해

할인 정책 그룹 **[pklp_limit_group]** 에 0이 아닌 동일한 번호를 설정하고 

스토어순번 **[stor_seq]** 을 0이 아닌 스토어순번으로 설정 하고 제한 설정을 하면

동일한 그룹에 대해서 그룹 정책이 설정된다



ex) 1그룹에 2시간, 2그룹에 3시간, 1,2그룹에 포함된 스토어들은 3시간으로 하고 싶다면

- 1그룹 설정 - 할인 정책 그룹 **[pklp_limit_group]** 을 1로 설정하고 
  1그룹에 설정할 스토어들에 대해서 2시간 설정한 데이터를 추가한다
- 2그룹 설정 - 할인 정책 그룹 **[pklp_limit_group]** 을 
  2로 설정하고 2그룹에 설정할 스토어들에 대해서 3시간 설정한 데이터를 추가한다
- 3그룹 설정 - 할인 정책 그룹 **[pklp_limit_group]** 을 
  3으로 설정하고 1,2그룹에 설정한 스토어들에 대해서 3시간 설정한 데이터를 추가한다

## 5. 할인 정책 처리 FLOW
![discount](./할인%20정책%20설정%20FLOW.png)

6. 할인 정책 확인 SQL
```sql
## 할인 정책 확인
select 
        CASE 
            WHEN c.park_name IS NULL THEN CONCAT('주차장없음','(',a.pklp_park_seq,')')
            ELSE CONCAT(c.park_name,'(',a.pklp_park_seq,')')
        END AS 주차장,
        a.pklp_park_seq AS 주차장순번,
        CASE a.stor_seq
            WHEN 0 THEN CONCAT('주차장전체','(',a.stor_seq,')')
            ELSE CONCAT(b.stor_name,'(',a.stor_seq,')')
       END AS 스토어,
       a.stor_seq AS 스토어순번,
       a.pklp_vehicle_type AS 차종유형,
       CASE a.pklp_limit_group
           WHEN 0 THEN '0:없음'
           ELSE CONCAT(a.pklp_limit_group,'그룹')
       END AS 주차장_스토어_그룹,
       CASE a.pklp_charge_type
           WHEN 0 THEN '0:전체'
           WHEN 1 THEN '1:유료'
           WHEN 2 THEN '2:무료'
       END AS 징수유형,
       CASE a.pklp_limit_type
           WHEN 1 THEN '1:시간'
           WHEN 2 THEN '2:금액'
           WHEN 3 THEN '3:비율'
           WHEN 4 THEN '4:횟수'
       END AS 제한유형,
       a.pklp_limit_value AS 제한값,
       CASE a.is_over
           WHEN 0 THEN '미허용'
           WHEN 1 THEN '허용'
       END AS 초과값허용,
       CASE a.pklp_del_ny
           WHEN 0 THEN '미삭제'
           WHEN 1 THEN '삭제'
       END AS 삭제여부,
       d.cmpy_new_id AS 멤버스ID
from acd_pms_parkinglot_discount_policy a
LEFT JOIN acd_rpms_parkinglot_store e ON e.park_seq = a.pklp_park_seq
LEFT JOIN acd_rpms_store b ON b.stor_seq = a.stor_seq
LEFT JOIN acd_pms_parkinglot c ON c.park_seq = a.pklp_park_seq
LEFT JOIN arf_b2bcore_company d ON d.cmpy_seq = b.cmpy_seq
where 1=1
where a.pklp_park_seq = #주차장순번
and a.pklp_del_ny = 0
#   and a.pklp_limit_group = #할인정책그룹 번호
group by a.stor_seq, a.pklp_park_seq, a.pklp_limit_group ,a.pklp_charge_type, a.pklp_limit_type, a.pklp_vehicle_type, a.is_over, a.pklp_del_ny
order by c.park_seq, a.pklp_vehicle_type, a.pklp_limit_group, a.stor_seq, a.pklp_charge_type, a.pklp_limit_type, a.stor_seq
;
```