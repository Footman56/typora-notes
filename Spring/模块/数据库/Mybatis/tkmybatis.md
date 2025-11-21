Criteria  用于绑定一些查询条件

```java
 Example example = new Example(KpiDraftTempDataPO.class);
        example.createCriteria().andEqualTo("companyId", companyId);
        example.createCriteria().andEqualTo("planId", planId);
        example.createCriteria().andEqualTo("assesseeEmpId", assessEmpId);
        example.createCriteria().andIn("todoEmpId", todoEmpIds);
```

在使用createCriteria 的时候。

```java
public Criteria createCriteria() {
        Criteria criteria = createCriteriaInternal();
        if (oredCriteria.size() == 0) {
            criteria.setAndOr("and");
            oredCriteria.add(criteria);
        }
        return criteria;
}

之后再执行createCriteria（）的时候不会加入到example.oredCriteria ,导致查询条件没有被使用。
```

