<font color= "red" >常见Stream</font>

```java
Map<String, WindPowerEquipmentInfoDO> studentIdToStudent=
        windPowerEquipmentInfoDOList.stream().collect(Collectors.toMap(WindPowerEquipmentInfoDO::getMachineNum,
                Function.identity()));

将list中的值转换成Map

 // 第一个参数是构造key,第二参数是构造value，第三方解决冲突key时 都是Function<T,V>类型，T是入参，V是出参   
Map<String, DeviceEmployeeInfoPO> newInfoPOS = deviceEmployeeInfoDOS.stream()
                .collect(Collectors.toMap(infoDO -> buildKey(infoDO.getObjectId(), String.valueOf(infoDO.getObjectType())),
                        DeviceEmployeeInfoDO::convertPO, (v1, v2) -> v2));
    
```



**在对象集合中根据对象的属性进行排序**

```java
kpiRelationshipProcessDOList.sort(Comparator.comparing(KpiRelationshipProcessDO::getOrderId));
```

```java
Consumer<T> : 消费型接口:一个入参，⽆无返回值
// 执行方法
void accept(T t);


Supplier<T> : 供给型接口:⽆入参，有返回值
T get();


Function<T, R> : 函数型接⼝:一个入参，一个返回值
R apply(T t);


Predicate<T> : 断⾔型接口:一个入参，返回值类型确定是boolean
boolean test(T t);
```

从集合对象中获取每一个对象的一个属性

```
List<Integer> list1 = list.stream().map(A::getId).collect(Collectors.toList());
```



将集合转换成Map对象，key是对象主键，value是对象本身

```java
Map<String, List<KpiRuleLevelSettingDO>> levelMap =evelDoList.stream()
					.collect(Collectors.groupingBy(KpiRuleLevelSettingDO::getRuleId));
```





flatmap: 汇总子集合

```java
 List<KpiProcessChildNodeTO> completeEntryFlowList = flowData.stream()
                .filter(v -> Objects.equals(v.getInspectionStatus(), EInspectionStatus.COMPLETE_VALUE_ENTRY.getType()))
                .flatMap(f->{
                    List<KpiProcessChildNodeTO> childNodes = f.getChildNodes();
                    return childNodes.stream();
                }).collect(Collectors.toList());
```

