## 1. 스토어별 할인권 베타적 사용 속성 테이블 생성 SQL

```sql

create table store_discount_ticket_exclusive_attribute
(
    park_seq int unsigned not null comment '주차장_순번',
    stor_seq int unsigned not null comment '스토어_순번',
    discount_key varchar(20)  not null comment '할인_키',
    is_exclusive char default 'Y' not null comment 'N : 비배타적으로 사용. 타 할인권과 동시 사용 가능
 Y : 배타적으로 사용. (해당 할인권 사용시 타 할인권 사용 불가. 마찬가지로 타 할인권 사용중일시 해당 할인권 사용 불가)
 ',
    primary key (park_seq, stor_seq, discount_key)
)
comment '스토어별_할인권_베타적사용_속성';
```

## 2.  베타사용 연관 테이블
### 1. 할인권 일반 속성
`discount_ticket_attribute`: 주차장 할인키에 베타적사용 속성 설정
```sql
create table discount_ticket_attribute
(
    park_seq                 int unsigned     not null comment '주차장_순번',
    discount_key             varchar(20)      not null comment '할인_키',
    validate_expire_datetime datetime         not null comment '유효_만료_일자',
    validate_start_datetime  datetime         not null comment '유효_시작_일자',
    is_auto_recharge         char             null comment '자동충전_여부',
    recharge_day             int(2)           null comment '충전_일자',
    recharge_type            varchar(10)      null comment 'ACCU : (Accumulation) 누적 — 이월분에서 충전개수를 합산한다. (예 - 이월분 : 3개, 충전 : 30개 = 33개)
	REFL : (Refill) 보충 — 이월분 상관없이 정해진 할인권 개수로 초기화 한다. (예 - 이월분 :3개, 충전 :30개 = 30개)',
    recharge_quantity        int unsigned     null comment '충전_수량',
    is_exclusive             char default 'N' null comment 'N : 비배타적으로 사용. 타 할인권과 동시 사용 가능
	Y : 배타적으로 사용. (해당 할인권 사용시 타 할인권 사용 불가. 마찬가지로 타 할인권 사용중일시 해당 할인권 사용 불가)
',
    max_usable_count         int unsigned     null comment '최대_사용가능_개수',
    sort_order               tinyint(3)       null comment '정열_순서',
    is_holiday               char default 'Y' null comment '공휴일_사용_가능',
    is_extraduty             char default 'Y' null comment '특근일_사용_가능',
    sales_price              int unsigned     null comment '1장당 가격',
    is_showing               char default 'Y' null comment '노출_여부',
    primary key (park_seq, discount_key, validate_expire_datetime)
)
    comment '할인권_일반_속성';

create index idx_discount_ticket_attribute_01
    on discount_ticket_attribute (park_seq, discount_key);
```

### 2. 스토어별 할인권 베타적사용 속성
`store_discount_ticket_exclusive_attribute`: 스토어별 할인키에 베타적사용 속성 설정

## 3. 베타사용 정책 기능 설명 
해당 주차장내, 스토어내의 베타사용여부가 **'Y'** 인 할인키 끼리는 **동시에 적용할 수 없음**

1번 할인키가 적용되면 2번 할인키가 적용되지 않게 하고 싶으면

1번 할인키의 베타사용여부를 **'Y'** 로 설정하고

2번 할인키의 베타사용여부도 **'Y'** 로 설정하면 됨 

## 4. 베타사용 정책 우선순위
`discount_ticket_attribute` 의 주차장 할인키 베타사용 설정이 있고
`store_discount_ticket_exclusive_attribute` 의 스토어 할인키 베타사용 설정이 있으면
스토어 할인키 베타사용 설정이 우선순위를 가지며 주차장 할인키 베타사용 설정을 덮어씀

스토어 할인키의 베타사용 설정을 삭제하면 다시 주차장 할인키의 베타사용 설정이 적용됨

## 5. 참고문서
[Members_베타사용여부_시뮬레이션_20190404.xlsx](https://parkingcloud.dooray.com/page-files/2544191220932583055?disposition=attachment)


## 6. 베타사용확인 처리 FLOW
![beta](./베타사용%20정책%20FLOW.png)

## 7. 베타사용정책 확인 SQL
```sql
## 주차장 베타사용 여부 이름과 같이 보기
SELECT discount_key, discount_name, is_exclusive
FROM (
        SELECT a.discount_key, b.discount_name, a.is_exclusive
        FROM store_discount_ticket_exclusive_attribute a
        INNER JOIN parkinglot_discount_ticket_list b
                ON b.park_seq = a.park_seq AND b.discount_key = a.discount_key
        AND b.is_deleted = 'N'
        AND b.disc_seq IS NOT NULL
        WHERE a.park_seq = #주차장순번
        AND a.stor_seq = #스토어순번
        AND a.is_exclusive IN ('Y','N')
        GROUP BY a.discount_key

        UNION ALL

        SELECT a.discount_key, b.discount_name, a.is_exclusive
        FROM discount_ticket_attribute a
        INNER JOIN parkinglot_discount_ticket_list b
                ON b.park_seq = a.park_seq AND b.discount_key = a.discount_key
        AND b.is_deleted = 'N'
        AND b.disc_seq IS NOT NULL
        WHERE a.park_seq = #주차장순번
        AND a.validate_expire_datetime >= now()
        AND a.validate_start_datetime <= now()
        AND a.is_exclusive IN ('Y','N')
        GROUP BY a.discount_key
) a
# WHERE discount_key IN (#할인키)
group by discount_key
```