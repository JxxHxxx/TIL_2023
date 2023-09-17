
개인적으로 테이블 결합은 매우 중요합니다. 어떤 결과를 보기 위해 다른 테이블과 협력해야 할 때가 많기 때문입니다.


#### 내부결합

앞서 설명하진 않았지만 `SELECT * FROM table1, table2` 이렇게 `FROM` 뒤에 복수 개의 테이블을 나열하면 교차 결합이 일어납니다. 교차 결합의 단점은 지나치게 결과가 방대진다는 것입니다. 내부 결합은 수학에서 말하는 **교집합**을 구하는 것이라고 생각하면 됩니다.

*내부결합 문법*

```
SELECT * FROM {table1 name} 
INNER JOIN {table2 bane}
ON {결합 조건}
```


내부 결합은 매우 중요하니까! 제가 하고 있는 프로젝트 테이블에서 예시를 가져와보겠습니다.

*예시*

*study_group 테이블 스키마*

![[Pasted image 20230910130359.png]](https://github.com/JxxHxxx/TIL_2023/blob/master/SQL%20%EC%B2%AB%EA%B1%B8%EC%9D%8C/7%EC%9E%A5%20%EB%B3%B5%EC%88%98%EC%9D%98%20%ED%85%8C%EC%9D%B4%EB%B8%94%20%EB%8B%A4%EB%A3%A8%EA%B8%B0/Pasted%20image%2020230910130359.png)

*study_product 테이블 스키마*

![[Pasted image 20230910130443.png]](https://github.com/JxxHxxx/TIL_2023/blob/master/SQL%20%EC%B2%AB%EA%B1%B8%EC%9D%8C/7%EC%9E%A5%20%EB%B3%B5%EC%88%98%EC%9D%98%20%ED%85%8C%EC%9D%B4%EB%B8%94%20%EB%8B%A4%EB%A3%A8%EA%B8%B0/Pasted%20image%2020230910130443.png)


만약 특정 스터디 그룹의 남은 인원(`left capactiy`), 상태(`group_status`) 그리고 해당 스터디 그룹에서 진행할 스터디할 내용의 이름과 썸네일을 가져온다고 해봅시다. `study_group` 테이블에서는 스터디 내용 이름과 썸네일을 알지 못합니다. 대신 스터디 상품의 기본키(`pk`)를 들고 있습니다. (`study_product_id`) 이를 통해 내부 결합을 진행할 수 있습니다.

```
SELECT sp.study_product_name, sp.thumbnail, sg.group_status, sg.left_capacity, sp.category, 
FROM study_group AS sg  
    INNER JOIN study_product AS sp 
    ON sg.study_product_id = sp.study_product_id  
WHERE sg.study_product_id = '6dcb2927-574b-49c6-a414-48b0f1ecdfc0';
```


- 외부키
``study_product_id`` 컬럼은 `study_product` 테이블의 기본키입니다. 한 편 `study_product_id` 은 `study_group` 에도 존재하는데요. 이를 외부키라고 부릅니다. 즉, 다른 테이블의 기본키를 참조하는 열이 외부키가 됩니다.

