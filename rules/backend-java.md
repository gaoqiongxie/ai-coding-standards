# Java 后端编码规范（AI 协作版）

> 面向 AI 编程工具（Claude Code / Trae / Cursor / Codex 等）的 Java/Spring 后端规范。
> 每条红线尽量标注 **AI 为什么爱犯**，让规范不止于"是什么"，还有"为什么要防"。
> 规则仅对新开发代码生效，历史代码保持不动。

---

## 1. 命名规范

### 包结构（按层划分，可按项目调整根包名）

```
com.company.project
├── controller          # 控制器层
├── service             # 服务层
│   └── impl            # 实现类
├── mapper              # 数据访问层
├── entity              # 实体类
├── dto                 # 数据传输对象
├── vo                  # 视图对象
├── query               # 查询参数对象
├── proxy               # 外部服务适配层（见 3.4）
├── config              # 配置类
├── utils               # 工具类
├── enums               # 枚举类
├── constant            # 常量类
├── interceptor         # 拦截器
├── handler             # 处理器
└── annotation          # 自定义注解
```

### 类命名

| 类型 | 规范 | 示例 |
|------|------|------|
| 实体类 | 名词 | `Customer`、`Order` |
| DTO | 名词 + Dto | `CustomerSaveDto` |
| VO | 名词 + Vo | `CustomerListVo` |
| Query | 名词 + Query | `CustomerQuery` |
| Service / Impl | 名词 + Service(/Impl) | `CustomerServiceImpl` |
| Mapper | 名词 + Mapper | `CustomerMapper` |
| Controller | 名词 + Controller | `CustomerController` |
| Utils | 名词 + Utils | `DateUtils` |
| 常量类 | 名词 + Constants | `CacheConstants` |
| 枚举 | 名词 + Enum | `CustomerStatusEnum` |
| 异常 | 名词 + Exception | `ServiceException` |

### 方法命名

| 操作 | 前缀 | 示例 |
|------|------|------|
| 获取单个 | `get` | `getById()`、`getByCode()` |
| 获取列表 | `list` | `listByDept()` |
| 分页查询 | `page` | `pageList()` |
| 新增 | `save` / `add` / `create` | `save()` |
| 修改 | `update` / `modify` | `updateById()` |
| 删除 | `remove` / `delete` | `removeById()` |
| 逻辑删除 | `logicDelete` | `logicDeleteById()` |
| 校验 | `check` / `validate` | `checkUnique()` |
| 转换 | `convert` / `to` | `convertToVo()` |
| 处理 | `process` / `handle` | `processData()` |
| 发送 | `send` | `sendMessage()` |
| 导出 / 导入 | `export` / `import` | `exportExcel()` |

### 变量命名

```java
// 常量：全大写，下划线分隔
public static final String CACHE_KEY_PREFIX = "biz:customer:";

// 成员/局部变量：驼峰
private String customerName;

// 布尔变量：避免 is 开头（避免与 getter 冲突）
private boolean deleted;  // 而不是 isDeleted

// 集合变量：复数或 list/map 后缀
private List<Customer> customers;
private Map<Long, String> customerNameMap;
```

- **禁止**拼音/中英文混合命名；**禁止**魔法值，统一用常量或枚举。

---

## 2. 代码风格

- 缩进 **4 空格**（不用 Tab）；大括号 **K&R 风格**；行宽 ≤ **120 字符**
- 方法尽量 ≤ **80 行**，复杂业务拆方法；重复逻辑必须抽取，禁止流水账代码
- 空行：包声明/导入/类声明之间空一行；成员变量按逻辑分组；方法之间空一行
- 注释：类与方法必须有 Javadoc（`@param`/`@return`/`@throws`）；复杂逻辑分步加行注释

```java
/**
 * 根据ID获取客户详情
 *
 * @param id 客户ID
 * @return 客户实体，不存在返回 null
 * @throws ServiceException 参数不合法时抛出
 */
public Customer getById(Long id) { ... }
```

### 禁止项

| 禁止项 | 说明 | 正确做法 |
|--------|------|---------|
| 手动创建线程 | `new Thread()` 不可控 | 统一用线程池 |
| `System.out.println` | 无法分级收集 | SLF4J 日志 |
| `@Deprecated` 依赖 | 未来版本移除 | 升级依赖或换替代方案 |
| 裸 `RuntimeException` | 丢失业务语义 | 抛 `ServiceException` |
| 魔法值 | 裸字符串/数字散落各处 | 枚举或常量类 |

---

## 3. 接口设计

### 3.1 RESTful 风格与统一入参

```java
@RestController
@RequestMapping("/api/customer")
public class CustomerController {

    @PostMapping("/list")
    public Result<List<CustomerVo>> list(@RequestBody @Validated CustomerQuery query) { ... }

    @PostMapping("/page")
    public Result<PageResult<CustomerVo>> page(@RequestBody @Validated CustomerQuery query) { ... }

    @PostMapping("/detail")
    public Result<CustomerVo> getById(@RequestBody @Validated IdReq req) { ... }

    @PostMapping
    public Result<Long> save(@RequestBody @Validated CustomerSaveDto dto) { ... }

    @PostMapping("/update")
    public Result<Void> update(@RequestBody @Validated CustomerSaveDto dto) { ... }

    @PostMapping("/delete")
    public Result<Void> delete(@RequestBody @Validated IdReq req) { ... }
}
```

- 入参统一 `@RequestBody` + `@Validated`，**禁止**用 `@PathVariable` 传业务参数（与项目约定不一致时以项目为准，但全项目必须统一）
- 统一返回 `Result<T>`（`code: 200` 成功 + `msg` + `data`），由全局异常处理器兜底

### 3.2 路径前缀按调用方分层（模板，按项目网关实际调整）

| 接口类型 | 前缀 | 示例 |
|---------|------|------|
| 前端业务接口 | `/api/` | `/api/customer/list` |
| 微服务内部调用 | `/feign/`（或 `/internal/`） | `/feign/customer/getById` |
| 第三方开放接口 | `/open-api/` | `/open-api/customer/sync` |
| 定时任务 | `/job/` | `/job/customer/refresh` |

网关按前缀做访问控制：内部前缀禁止外部访问。

### 3.3 URL 命名

- URL 全小写；双斜杠间尽量单个有意义的单词；多单词按项目存量惯例统一（驼峰或中划线二选一，**新接口一律遵循现行约定，存量不批量迁移**）

| 不推荐 | 推荐 |
|--------|------|
| `/api/customer/getCustomerList` | `/api/customer/list` |
| `/api/customer/get_by_id` | `/api/customer/detail` |

### 3.4 外部服务调用必须走 Proxy 适配层

> **AI 为什么爱犯**：AI 生成跨服务调用时习惯直接在 ServiceImpl 注入 Feign/HTTP Client，导致业务代码与外部契约硬耦合，外部接口一变满地返工。

- **ServiceImpl 禁止直接注入外部服务 Client**，统一 `ServiceImpl → Proxy → Client`
- Proxy 职责：参数校验、请求构建（禁止透传大对象）、出入参日志、错误处理

```java
@Service
public class ExternalServiceProxy {

    @Autowired
    private ExternalServiceClient client;

    /**
     * 获取外部业务数据
     */
    public ExternalDataVo getExternalData(String bizId) {
        // 1. 参数校验
        if (StringUtils.isBlank(bizId)) {
            throw new ServiceException("bizId 不能为空"); // 实际项目走 i18n key
        }

        // 2. 构建请求对象（提取必要字段，禁止透传大对象）
        ExternalDataReq req = new ExternalDataReq();
        req.setBizId(bizId);

        // 3. 调用 + 出入参日志
        log.info("获取外部业务数据 req: {}", JSON.toJSONString(req));
        ExternalResp<ExternalDataVo> result = client.getData(req);
        log.info("获取外部业务数据 req: {}, result: {}", JSON.toJSONString(req), JSON.toJSONString(result));

        // 4. 错误处理：禁止吞异常返回 null
        if (result == null || !result.isSuccess()) {
            log.error("获取外部业务数据失败，req: {}", JSON.toJSONString(req));
            throw new ServiceException("外部数据获取失败");
        }
        return result.getData();
    }
}
```

**Proxy 层 Checklist**：
- [ ] ServiceImpl 只依赖 Proxy，不直接注入外部 Client
- [ ] 参数校验覆盖所有必填字段
- [ ] 请求/响应均有 JSON 化日志
- [ ] 失败抛业务异常，不返回 null 吞异常
- [ ] 返回集合时失败返回空集合而非 null

---

## 4. 异常处理

- 统一全局异常处理器（`@RestControllerAdvice`）：业务异常 `warn` 日志 + 业务码返回；未知异常 `error` 日志 + 通用兜底文案
- 业务异常统一用项目自定义异常（如 `ServiceException`），携带业务码

```java
@RestControllerAdvice
@Slf4j
public class GlobalExceptionHandler {

    @ExceptionHandler(ServiceException.class)
    public Result<Void> handleServiceException(ServiceException e) {
        log.warn("业务异常: {}", e.getMessage());
        return Result.fail(e.getCode(), e.getMessage());
    }

    @ExceptionHandler(Exception.class)
    public Result<Void> handleException(Exception e) {
        log.error("系统异常", e);
        return Result.fail(500, "系统繁忙，请稍后重试");
    }
}
```

| 禁止项 | 正确做法 |
|--------|---------|
| `e.printStackTrace()` | `log.error("...", e)` |
| 空 `catch` 块 | 至少记录日志或抛自定义异常 |
| 裸 `RuntimeException` | `ServiceException` |

---

## 5. 事务规范

> **AI 为什么爱犯**：AI 常把 `@Transactional` 标在 Controller 或内部私有方法上，或在事务方法里写远程调用导致长事务。

- `@Transactional` **只在 Service 层**，且**必须** `rollbackFor = Exception.class`（默认不回滚受检异常）
- 同类方法内部调用**不生效**（自调用绕过代理）：需注入自身或拆到另一个 Bean
- 事务方法内禁止远程调用 / 长耗时操作（事务粒度最小化）

```java
@Transactional(rollbackFor = Exception.class)
public void saveCustomer(CustomerSaveDto dto) { ... }
```

---

## 6. 数据库规范

### 6.1 SQL 编写

```xml
<select id="selectByCondition" resultType="Customer">
    SELECT id, customer_no, customer_name, customer_type, create_time
    FROM biz_customer
    WHERE delete_flag = 0
      <if test="customerName != null and customerName != ''">
      AND customer_name LIKE CONCAT('%', #{customerName}, '%')
      </if>
    ORDER BY create_time DESC
</select>

<insert id="batchInsert">
    INSERT INTO biz_customer (customer_no, customer_name, create_time)
    VALUES
    <foreach collection="list" item="item" separator=",">
        (#{item.customerNo}, #{item.customerName}, NOW())
    </foreach>
</insert>
```

| 规则 | 说明 |
|------|------|
| **禁止 `SELECT *`** | 显式列字段，避免网卡压力和模型不匹配 |
| `INSERT` 必须指定列名 | 禁止 `insert into t values(...)` |
| DML 必须有 `WHERE` 且走索引 | 静态小表除外 |
| **禁止全模糊 `LIKE '%x%'` 裸奔** | 必须配合其他等值/范围条件，否则全表扫 |
| 禁止生产环境使用 `hint` | 相信优化器 |
| `IN` 列表 ≤ 500 个 | 过多则分批 |
| 单条批量 `INSERT` ≤ 5000 行 | 大批次按事务和包大小拆批 |
| 优先 `UNION ALL` 替代 `UNION` | 且子句不超过 5 个 |
| 索引列不用函数 | `WHERE DATE(create_time)=?` 无法走索引；改为范围条件 |
| 减少 `or` | 可优化为 `UNION` + 各条件建索引 |
| WHERE 等号两侧类型一致 | 避免隐式转换导致索引失效 |

### 6.2 索引设计

| 规则 | 说明 |
|------|------|
| 主键固定为 `id`（`bigint unsigned auto_increment`） | 禁止更新主键值 |
| 唯一索引 `uniq_` 前缀，普通索引 `idx_` 前缀 | |
| 单表索引 ≤ 7 个 | 避免维护成本过高 |
| 联合索引区分度最高字段在前 | |
| 避免冗余索引 | 已有 `key(a,b)` 则 `key(a)` 冗余 |
| 被驱动表连接列必须有索引 | |
| VARCHAR 超长列用前缀索引 | |

### 6.3 表结构设计

| 级别 | 规则 |
|------|------|
| 强制 | 表名/列名 ≤ 32 字符，字母数字下划线，全小写；表名带模块前缀（如 `biz_`） |
| 强制 | 字符集统一 `utf8mb4`（库/表/列/程序连接字符集一致） |
| 强制 | 表和每个字段都必须有 `COMMENT` |
| 强制 | 中间表 `tmp_` 开头、备份表 `bak_` 开头，定期清理 |
| 建议 | 大表 `ALTER TABLE` 需审核并在低峰执行；`blob`/`text` 大字段垂直拆分；高频 join 字段可冗余 |

> **注意**：部分团队规范禁止在 DDL 中显式写 `ENGINE=InnoDB`（由数据库默认或审核平台处理），按你团队的 SQL 审核平台要求执行。

### 6.4 核心字段约定：NOT NULL + DEFAULT

> **AI 为什么爱犯**：AI 生成 DDL 时默认写 `DEFAULT NULL`，导致业务层 NPE 和三值逻辑歧义。

**所有字段禁止 `NULL`，必须显式默认值**（极少数强依赖 NULL 语义的字段需评审并记录原因）：

| 字段类型 | MySQL 定义 | 默认值 |
|----------|-----------|--------|
| 枚举/状态 | `TINYINT` | `DEFAULT '0'` |
| 整型 | `INT` / `BIGINT` | `DEFAULT '0'` |
| 字符串 | `VARCHAR` | `DEFAULT ''`（空串代替 NULL） |
| 金额 | `DECIMAL(19,4)`（精度按项目约定） | `DEFAULT '0.0000'`，**必须带币种字段** |
| 日期时间 | `DATETIME` | `DEFAULT CURRENT_TIMESTAMP`；业务时间戳可用 `bigint` 毫秒，字段名以 `_timestamp` 结尾 |
| 布尔/标记 | `TINYINT` | `DEFAULT '0'` |

- 状态/类型字段用 `TINYINT`，**禁止 `enum`/`set`**；Java 侧用包装类型 `Integer` 映射，不用基本类型
- Mapper XML 动态 SQL 用 `<if test="... != null">`：insert 传 null 不插入（DB 填默认），update 传 null 不更新（保留原值）；要显式重置必须传值而不是 null

### 6.5 逻辑删除与审计字段

- **禁止物理删除**，统一逻辑删除：字段 `delete_flag`（0-正常，1-已删除），**只能 0→1，禁止重置回 0**
- 审计字段 `create_time` / `edit_time` 由数据库自动管理（`ON UPDATE CURRENT_TIMESTAMP`），**代码和 SQL 禁止手动赋值**

```java
@Data
public class BaseEntity {
    @Schema(description = "创建时间")
    private LocalDateTime createTime;

    @Schema(description = "修改时间")
    private LocalDateTime editTime;

    @Schema(description = "删除标记:0正常 1已删除")
    @TableLogic(value = "0", delval = "1")
    private Integer deleteFlag;
}
```

### 6.6 高危操作清单（SQL 审核平台常见拦截项）

| 高危操作 | 原因 | 替代方案 |
|----------|------|----------|
| `update/delete ... limit N` | 无主键排序导致主从不一致 | 加 `ORDER BY 主键` |
| 关联子查询（`update ... where in (select...)`） | 效率极低 | 拆成 join 或多次查询 |
| `insert ... on duplicate key update` | 高并发主从不一致 | 程序端判断后 insert 或 update |
| 联表更新（`update t1,t2 where...`） | 不可控 | 程序端分步处理 |
| 存储过程/函数/触发器/视图/事件/外键 | 消耗 DB 资源、降低扩展性 | 一律程序端实现 |
| `ALTER TABLE ... AFTER/FIRST` | 大表重建风险 | 直接 `ADD COLUMN` |
| `RIGHT JOIN` | 可读性差 | 统一 `LEFT JOIN` |

### 6.7 更新与查询纪律

> **AI 为什么爱犯**：AI 最喜欢"查出来全量实体 → 改两个字段 → 整体 update"，把别人刚改的字段覆盖掉；也喜欢在 for 循环里逐条查库。

- **通过 ID 更新时，禁止直接使用查询出的全量实体执行 update**：新建实体对象，仅复制 ID + 需变更字段（避免并发覆盖）
- **禁止在循环中执行数据库操作**：批量处理、预聚合，减少 DB 交互
- 多表 JOIN 不超过 3 张，复杂查询拆代码
- 分页查询必须带排序或索引字段；分页统一用分页插件

### 6.8 规范建表示例

```sql
CREATE TABLE `biz_customer` (
  `id` bigint unsigned NOT NULL AUTO_INCREMENT COMMENT '主键ID',
  `customer_no` varchar(50) NOT NULL DEFAULT '' COMMENT '客户编号',
  `customer_name` varchar(200) NOT NULL DEFAULT '' COMMENT '客户名称',
  `customer_level` tinyint NOT NULL DEFAULT '0' COMMENT '客户分层：0-未分类 1-价值客户 2-基石客户',
  `amount` decimal(19,4) NOT NULL DEFAULT '0.0000' COMMENT '金额',
  `currency` varchar(10) NOT NULL DEFAULT '' COMMENT '币种',
  `status` tinyint NOT NULL DEFAULT '0' COMMENT '状态：0-启用 1-禁用',
  `create_time` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '创建时间',
  `edit_time` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP COMMENT '更新时间',
  `create_user` varchar(50) NOT NULL DEFAULT '' COMMENT '创建人',
  `update_user` varchar(50) NOT NULL DEFAULT '' COMMENT '修改人',
  `delete_flag` tinyint NOT NULL DEFAULT '0' COMMENT '删除标记(0.未删除,1.已删除)',
  `remark` varchar(200) NOT NULL DEFAULT '' COMMENT '备注',
  PRIMARY KEY (`id`),
  UNIQUE KEY `uniq_customer_no` (`customer_no`),
  KEY `idx_customer_name` (`customer_name`),
  KEY `idx_create_time` (`create_time`,`status`)
) DEFAULT CHARSET=utf8mb4 COMMENT='客户信息表';
```

---

## 7. Maven 规范

- **所有依赖版本统一在父 POM `<properties>` 定义，`<dependencyManagement>` 声明**；子模块引用不指定版本号
- 禁止覆盖父 POM 已定义的版本；禁止使用 `<scope>system</scope>` + `<systemPath>` 引本地 jar

---

## 8. 日志规范

```java
// 级别：ERROR=需立即处理 / WARN=业务异常或潜在问题 / INFO=重要业务信息 / DEBUG=仅开发环境
log.error("保存客户失败, customerId={}", customerId, e);   // 异常必须带异常对象
log.warn("客户数量超过阈值, count={}", customerCount);
log.info("客户创建成功, customerId={}", customer.getId());
if (log.isDebugEnabled()) {                                 // 大对象有条件打印
    log.debug("查询结果: {}", JsonUtils.toJson(resultList));
}
```

- **入参、出参、异常必须打印日志**；使用占位符 `{}`，**禁止字符串拼接**
- **禁止日志级别混乱**：错误场景不能用 `info`，正常流程不能用 `error`
- **禁止打印敏感数据**（密码、证件号、手机号——需脱敏：`138****8888`）
- 禁止 `toString()` 直接打印大对象

---

## 9. 安全规范

| 项 | 要求 |
|----|------|
| SQL 注入 | MyBatis 用 `#{}` 参数绑定，**避免 `${}` 拼接** |
| XSS | 输入转义、输出过滤 |
| 敏感信息 | 日志/响应中手机号、证件号等脱敏 |
| 接口鉴权 | 敏感接口加 `@PreAuthorize("hasAuthority('xxx')")` |
| 入参校验 | `@Validated` + `@NotNull` / `@NotBlank` / `@Pattern`，在 DTO 上做，不在业务代码里手工 if |

---

## 10. 多语言（i18n）合规内核

> 各团队的 i18n 机制不同（资源文件、DB 下发、平台托管），但原则通用：

- **禁止中文硬编码**：接口响应、异常消息、枚举描述中禁止出现中文字符串字面量，一律走 i18n key
- **语义精确**：每个业务场景独立 key，禁止用"操作失败"这类通用 key 覆盖多种语义
- **key 命名**：按项目约定格式（如 `{模块}_static_{序号}` / `enum_{枚举类名}`），并遵守该项目的语言包生成/发布流程
- 抛异常示例：`throw new ServiceException(I18nTool.get(I18nCode.customerNotExist));` 而不是 `throw new ServiceException("客户不存在");`

---

## 11. 其他强制规范

- 金额用 `BigDecimal`，日期用 `LocalDateTime`（不用 `Date`）
- MQ 消费遵循项目现有模式；新增方法必须有 Javadoc
- 凭证（密码、Token、密钥）统一放配置中心或环境变量，**禁止写进代码、注释、日志**

---

## 12. 代码审查清单（提交前自查）

- [ ] 命名符合规范，无拼音/魔法值
- [ ] 新方法有 Javadoc，复杂逻辑有行注释
- [ ] 无空指针风险；流/连接等资源已关闭
- [ ] SQL 无注入风险（无 `${}` 拼接）
- [ ] 日志用占位符，无敏感明文
- [ ] 异常被正确处理，无空 catch / printStackTrace
- [ ] 无 `SELECT *`，无循环查库，无全量实体更新
- [ ] 事务注解在 Service 层且 `rollbackFor = Exception.class`
- [ ] `@Transactional` 方法内无远程调用
- [ ] 重复代码已抽取；单元测试通过
- [ ] 接口走统一 `Result<T>` 与正确路径前缀
- [ ] 方法 ≤ 80 行、单行 ≤ 120 字符
