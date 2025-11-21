# 一、角色说明

你是一个资深的java专家，请在开发中遵循如下规则：

- 采用 **分层架构设计**，确保职责分离
- 代码变更需通过 **单元测试覆盖**（测试覆盖率 ≥ 80%）
- 变量命名需要符合阿里巴巴规范，函数功能简短，需要生成必要的注释，打印对应的日志
- 严格遵循 **SOLID、DRY、KISS、YAGNI** 原则
- 遵循 **OWASP 安全最佳实践**（如输入验证、SQL注入防护）

------

# 二、技术栈规范

## 技术栈要求

- **框架**：Spring Boot 2.2.13.RELEASE + Java 8
- 依赖
  - 核心：Spring Web, Lombok
  - 数据库：mysql 8.0 、elasticsearch、mongo、redis
  - 定时任务采用 xxl-job
  - 消息中台采用 RabbitMQ

------

# 三、应用逻辑设计规范

## 1. 项目模块

+ 项目分成两个模块
  + appraisal-webapp  是基础配置模块，所有与配置项目的功能都在这里
  + appraisal-webfront 是业务功能模块，功能模块采用DDD 领域分层设计

# 四、核心代码规范

## 1. 分层架构原则

| 层级           | 职责                                   | 约束条件                                                     |
| -------------- | -------------------------------------- | ------------------------------------------------------------ |
| **Controller** | 处理 HTTP 请求与响应，采用RestFull风格 | - 禁止直接操作数据库 - 必须通过 Service 层或者Business调用   |
| **Business**   | 复杂业务逻辑实现，数据校验             | - 禁止直接操作数据库 - 必须通过 Service层  - 返回 TO实体类(除非必要) |
| **Service**    | 单一业务逻辑实现，事务管理，数据校验   | - 必须通过Manager层访问数据库 - 返回 DO实体类                |
| **Manager**    | 数据持久化操作，定义数据库查询逻辑     | - 必须通过Mapper来直接操作数据库，-复杂逻辑通过xml形式实现，简单逻辑通过Example来实现  - 返回 PO数据传输对象 |
| **DO**         | 绩效业务对象                           |                                                              |
| **PO**         | 数据库传输对象                         | 通过javax.persistence.* 包下的@Table、@Column 等表明与mysql表中间的映射关系 |
| **TO、Model**  | 模型对象，用于展示给前端               |                                                              |
|                |                                        |                                                              |
|                |                                        |                                                              |

### 1. 实体类（DO）规范

```java
/**
 * @author
 * @date 2023/7/3 15:46
 * @description
 */
@Data
public class KpiCalibrationRoundGroupDO {

    /**
     * 主键
     */
    private Long id;

    /**
     * 总公司id
     */
    private String headId;

    /**
     * 公司id
     */
    private String companyId;

    /**
     * 方案id
     */
    private String planId;

    /**
     * 轮次id
     */
    private String roundId;
```

### 2. 数据传输对象（PO） 规范

```java

@Table(name = "kpi.kpi_calibration_round_group")
public class KpiCalibrationRoundGroupPO {
    /**
     * 主键
     */
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    /**
     * 总公司id
     */
    @Column(name = "head_id")
    private String headId;

    /**
     * 公司id
     */
    @Column(name = "company_id")
    private String companyId;

    /**
     * 方案id
     */
    @Column(name = "plan_id")
    private String planId;

    /**
     * 轮次id
     */
    @Column(name = "round_id")
    private String roundId;
```

### 3. 数据访问层（Manager）规范

```java
/**
 * @author zdl
 * @date 2023/7/5 10:45
 * @description
 */
@Repository
public class KpiCalibrationGroupManagerImpl implements IKpiCalibrationGroupManager {

    @Resource
    private KpiCalibrationRoundGroupMapper roundGroupMapper;

    @Resource
    private KpiCalibrationGroupScopeMapper groupScopeMapper;


    @Override
    public void batchAdd(List<KpiCalibrationRoundGroupPO> kpiCalibrationRoundGroupPOS, List<KpiCalibrationGroupScopePO> kpiCalibrationScopes) {
        int unixTimestamp = DateUtil.getUnixTimestamp();

        if (CollectionUtils.isNotEmpty(kpiCalibrationRoundGroupPOS)) {
            StreamUtil.doEach(kpiCalibrationRoundGroupPOS, v->{
                v.setIsDel(AppraisalConstant.STATUS_NORMAL);
                v.setAddtime(unixTimestamp);
                v.setModtime(unixTimestamp);
            });
            roundGroupMapper.insertList(kpiCalibrationRoundGroupPOS);
        }

        if (CollectionUtils.isNotEmpty(kpiCalibrationScopes)) {
            StreamUtil.doEach(kpiCalibrationScopes, v->{
                v.setIsDel(AppraisalConstant.STATUS_NORMAL);
                v.setAddtime(unixTimestamp);
                v.setModtime(unixTimestamp);
            });

            groupScopeMapper.insertList(kpiCalibrationScopes);
        }

    }


    @Override
    public List<KpiCalibrationRoundGroupPO> listCalibrationGroup(String companyId, String planId, List<String> roundIds) {
        if (StringUtil.isAnyBlank(companyId, planId)) {
            return Lists.newArrayList();
        }
        Example example = new Example(KpiCalibrationRoundGroupPO.class, true, true);
        Example.Criteria criteria = example.createCriteria();
        criteria.andEqualTo("companyId", companyId);
        criteria.andEqualTo("planId", planId);
        if (CollectionUtils.isNotEmpty(roundIds)) {
            criteria.andIn("roundId", roundIds);
        }
        criteria.andEqualTo("isDel", AppraisalConstant.STATUS_NORMAL);

        return roundGroupMapper.selectByExample(example);
    }
```

### 4. 服务层（Service）规范

```java
@Slf4j
@Service
public class KpiCalibrationGroupService implements IKpiCalibrationGroupService {

    @Resource
    private IKpiCalibrationGroupManager kpiCalibrationGroupManager;
    @Resource
    private IOrganizationService organizationService;
    @Resource
    private IEmployeeService employeeService;

    @Override
    public void batchAdd(List<KpiCalibrationRoundGroupDO> calibrationRoundGroupDOList, List<KpiCalibrationGroupScopeDO> kpiCalibrationScopes) {

        List<KpiCalibrationRoundGroupPO> kpiCalibrationRoundGroupPOS = StreamUtil.mapToList(calibrationRoundGroupDOList, v -> v.convertTOPO());

        List<KpiCalibrationGroupScopePO> kpiCalibrationScopesPOS =  StreamUtil.mapToList(kpiCalibrationScopes, v -> v.convertTOPO());

        kpiCalibrationGroupManager.batchAdd(kpiCalibrationRoundGroupPOS, kpiCalibrationScopesPOS);
    }
}
```

### 5. busniess规范

```java
@Slf4j
@Service
public class KpiCalibrationBusiness {

    @Resource
    private PlatformTransactionManager transactionManager;
    @Resource
    private IKpiCalibrationGroupService kpiCalibrationGroupService;

    /**
     * 开启方案，生成方案的校准分组数据
     */
    public void generateCalibrationGroupDataForStartPlan(KpiPlanInfoDO kpiPlanInfoDO,
                                                         List<KpiAssesseeInfoDO> assesseeInfoDOList,
                                                         Map<String, DepartmentDO> departmentDOMap) {
        if (kpiPlanInfoDO == null || CollectionUtils.isEmpty(assesseeInfoDOList)) {
            return;
        }
        log.info("generateCalibrationGroupDataForStartPlan planId:{}, assesseeInfoDOList:{} ", kpiPlanInfoDO.getPlanId(), assesseeInfoDOList.size());
        String companyId = kpiPlanInfoDO.getCompanyId();
        String planId = kpiPlanInfoDO.getPlanId();

        // 查询方案校准的process
        List<KpiProcessInfoDO> calibrationProcessList = kpiProcessInfoService.listPlanProcess(companyId, Lists.newArrayList(planId), EInspectionStatus.PERFORMANCE_CALIBRATION.getType());
        if (CollectionUtils.isEmpty(calibrationProcessList)) {
            log.info("generateCalibrationGroupDataForStartPlan 没有校准环节 planId:{} ", kpiPlanInfoDO.getPlanId());
            return;
        }

        // 被考核人基本信息
        List<String> assesseeEmpIds = StreamUtil.mapToList(assesseeInfoDOList, KpiAssesseeInfoDO::getEmployeeId);
        Map<String, AppraisalEmployeeBasicDO> employeeBasicDOMap = datacenterAppraisalService.getOnAndOffEmpBasicInfo(companyId, assesseeEmpIds);

        // 查询使用的绩效角色
        Set<String> allKpiRoleIds = calibrationProcessList.stream()
                .filter(v -> Objects.equals(EReviewPersonType.CUSTOM_KPI_ROLE.getReviewPersonType(), v.getReviewPersonType()) && StringUtil.isNotBlank(v.getReviewPersonTypeId()))
                .map(KpiProcessInfoDO::getReviewPersonTypeId)
                .collect(Collectors.toSet());
        Map<String, KpiRoleDataDO> roleDataDOMap = kpiRoleService.listKpiRoleData(companyId, allKpiRoleIds);

        // 生成的校准数据
        List<KpiCalibrationRoundGroupDO> calibrationRoundGroupDOList = new ArrayList<>();
        List<KpiCalibrationGroupScopeDO> calibrationGroupScopeDOList = new ArrayList<>();
        List<KpiCalibrationAssesseeRelationDO> calibrationAssesseeRelationDOList = new ArrayList<>();

        // 循环校准process
        for (KpiProcessInfoDO processInfoDO : calibrationProcessList) {
            // 该轮次的所有待办
            List<KpiTempProcessTodoDO> currentProcessTempTodoList = new ArrayList<>(assesseeInfoDOList.size());
            for (KpiAssesseeInfoDO kpiAssesseeInfoDO : assesseeInfoDOList) {
                String assesseeEmpId = kpiAssesseeInfoDO.getEmployeeId();
                KpiReviewPersonContext reviewPersonContext = new KpiReviewPersonContext(employeeBasicDOMap.get(assesseeEmpId), processInfoDO, departmentDOMap, roleDataDOMap, null);
                String todoEmpId = reviewPersonHelper.getCurrentReviewPersonWithProcessEmp(reviewPersonContext, null);
                KpiTempProcessTodoDO kpiTempProcessTodoDO = KpiTempProcessTodoDO.generateTempTodo(processInfoDO, todoEmpId, assesseeEmpId, employeeBasicDOMap.get(assesseeEmpId), kpiPlanInfoDO.getCuringPersonSwitch(), departmentDOMap);
                currentProcessTempTodoList.add(kpiTempProcessTodoDO);
            }
            List<KpiCalibrationTodoListGroup> todoListGroups = getGroupList(processInfoDO, currentProcessTempTodoList, kpiPlanInfoDO.getCuringPersonSwitch());
            generateRoundGroupData(kpiPlanInfoDO, processInfoDO, processInfoDO.getRoundOrder(), todoListGroups, calibrationRoundGroupDOList, calibrationGroupScopeDOList, calibrationAssesseeRelationDOList);
        }

        // 插入数据
        kpiCalibrationGroupService.batchAdd(calibrationRoundGroupDOList, calibrationGroupScopeDOList);
        kpiCalibrationAssesseeService.batchAdd(calibrationAssesseeRelationDOList);
    }

```

------

### 6、controller 规范

```java
// 使用 record 或 @Data 注解
public record UserDTO(
    @NotBlank String username,
    @Email String email/**
 * 方案管理
 *
 * @author liuchenhui
 * @date 2021/5/19 2:53 下午
 */
@RestController
@RequestMapping("/appraisal/service/kpi/plan")
@Slf4j
public class KpiPlanController {
    @Resource
    private TranslationHelper translationHelper;
    @Resource
    private KpiCalibrationBusiness kpiCalibrationBusiness;
    /**
     * 刷新方案校准数据
     * @param planId
     * @return
     */
    @PostMapping("/ajax-refresh-plan-calibration")
    public AjaxResult ajaxRefreshPlanCalibration(String planId) {
        CommonGlobalParam globalParam = GlobalParamHelper.getCommonGlobalParam();
        String companyId = globalParam.getCompanyId();
        kpiCalibrationBusiness.refreshPlanCalibration(companyId, planId, globalParam.getAccId());
        return AjaxResult.success();
    }
}
```

------

# 五、全局异常处理规范

### 1. 统一异常错误码类（EAppraisalErrorCode）

```java
public enum  EAppraisalErrorCode implements ErrorCode {

    /*
     * 通用异常Code 1/325001
     */
    INTER_SERVICE_ERROR(125005000, "服务内部错误"),
    CONTENT_NOT_FOUND(325001001, "对象不存在"),
    FIELD_NOT_VALID(325001002, "参数校验错误"),
    AUTH_NOT_VALID_FAILED(325001003, "权限操作不允许"),
    OPERATION_VALIED_ERROR_CODE(325001005, "操作错误"),
    BIZ_FAILED_CODE(325001006, "业务异常"),
    PARAM_NOT_VALID(325001007, "参数不可用"),
    OPERATION_FAILED(325001008, "操作失败（操作过程遇到了不可知的问题）"),
    DATA_NOT_VALID(325001009, "数据异常");
      
    private final int code;

    private final String message;

    EAppraisalErrorCode(Integer code, String message) {
        this.code = code;
        this.message = message;
    }

    @Override
    public int getCode() {
        return this.code;
    }

    @Override
    public String getMessage() {
        return this.message;
    }
}
```

## 2.异常示例

```java
 // 批量操作最大人数校验
        if (assesseeEmpIds.size() > BATCH_OPT_MAX_EMPLOYEE) {
            throw new ParamNotValidException(EAppraisalErrorCode.BATCH_OPERATE_SUPPORT_MAX_EMPLOYEE.getCode(),
                    EAppraisalErrorCode.BATCH_OPERATE_SUPPORT_MAX_EMPLOYEE.getMessage(),
                    DicVar.of(EBatchOperateType.BATCH_REJECT_TYPE.getMessage()),
                    DicVar.of(String.valueOf(BATCH_OPT_MAX_EMPLOYEE)));
        }
```

------

# 六、代码风格规范

1. 命名规范
   - 类名：`UpperCamelCase`（如 `UserServiceImpl`）
   - 方法/变量名：`lowerCamelCase`（如 `saveUser`）
   - 常量：`UPPER_SNAKE_CASE`（如 `MAX_LOGIN_ATTEMPTS`）
2. 注释规范
   - 方法必须添加注释且方法级注释使用 Javadoc 格式
   - 计划待完成的任务需要添加 `// TODO` 标记
   - 存在潜在缺陷的逻辑需要添加 `// FIXME` 标记

------

# 七、扩展性设计规范

1. 接口优先:

   - 服务层接口（`IKpiCalibrationGroupService`）与实现（`KpiCalibrationGroupService`）分离
   - 服务层接口（`IKpiCalibrationGroupManager`）与实现（KpiCalibrationGroupManager`Impl`）分离

2. 日志规范

   ：

   - 使用 `SLF4J` 记录日志（禁止直接使用 `System.out.println`）
   - 核心操作需记录 `INFO` 级别日志，异常记录 `ERROR` 级别