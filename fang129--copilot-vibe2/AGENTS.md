
# 项目开发指令（Java API 服务）

## 1. 项目基本信息
- 开发语言：Java 8
- 框架：Spring Boot 2.7.14
- 构建工具：Maven
- 基础包名：com.productpackage.emgcallservice
- 开发规约：严格遵循《阿里巴巴 Java 开发手册》泰山版 2.0

## 2. 包结构规范
采用标准后端分层架构：
- controller：对外接口层
- service：业务逻辑层
    - service.impl：实现类
- mapper / dao：数据访问层
- entity / model：实体对象
    - dto：入参、出参
    - vo：视图返回对象
    - bo：业务对象
- common / util：工具类
- config：配置类
- exception：异常处理
- enums：枚举类

## 3. 命名规范（严格阿里规约）
- 包名：小写，点分隔
- 类名：PascalCase（UpperCamelCase）
- 方法名、变量名：camelCase（lowerCamelCase）
- 常量：UPPER_SNAKE_CASE
- 集合/数组：使用复数形式
- Service 实现类：以 Impl 结尾，如 UserServiceImpl
- Controller 方法：语义化，如 list、get、create、update、delete
- DTO/VO 后缀明确：XxxDTO、XxxVO、XxxQuery、XxxRequest

## 4. 代码风格与格式
- 缩进：4 空格，不使用 Tab
- 大括号：不换行（K&R 风格）
- 行宽控制在 120 字符内
- 方法行数尽量不超过 80 行
- 禁止魔法数值，必须抽为常量或枚举
- 所有 POJO 必须使用 Lombok（@Data/@Getter/@Setter）
- 禁止使用 System.out.println，统一使用日志框架

## 5. Spring Boot 开发规范
- 控制层使用 @RestController
- 接口统一返回封装结果对象（Result<T>）
- 入参使用 @RequestBody / @RequestParam / @PathVariable
- 事务使用 @Transactional(rollbackFor = Exception.class)
- 配置优先使用 application.yml
- 禁止硬编码，配置统一放配置文件
- 异步使用 @Async，线程池自定义
- 接口版本控制统一放在路径或请求头

## 6. 异常与日志规范
- 统一全局异常处理器 @RestControllerAdvice
- 自定义业务异常 BusinessException / SystemException
- 日志使用 SLF4J + LoggerFactory
- 关键入参、出参、异常必须打日志
- 禁止捕获 Exception 后空处理
- 禁止 e.printStackTrace()

## 7. 接口规范
- RESTful 风格 API
- 请求方法：GET 查询、POST 新增、PUT 修改、DELETE 删除
- 分页统一使用 pageNum、pageSize
- 返回结构统一：code、msg、data
- 接口必须写 Javadoc 注释，包含功能、入参、出参、异常说明

## 8. 编码禁忌（阿里规约强制）
- 禁止使用 BeanUtils 工具类跨层拷贝（禁止模糊拷贝）
- 禁止在 for 循环中新开线程、查询数据库
- 禁止 SQL 拼接，使用 #{} 而非 ${}
- 集合初始化指定容量
- 包装类比较用 equals，禁止 ==
- 所有 POJO 必须实现序列化（implements Serializable）
- 日期使用 LocalDateTime，禁止 Date
- 字符串判空使用工具类，禁止 "".equals(str)
- 禁止使用枚举的 ordinal() 做持久化

## 9. 生成代码要求
- 生成的代码必须可直接编译运行
- 结构清晰，符合分层
- 自动补充必要注释
- 遵循阿里规约，不出现规约中明令禁止的写法
- 变量、方法命名必须见名知意
- 
## 10. 项目测试规范
- 所有单元测试使用 JUnit 5 (Jupiter) + AssertJ + Mockito
- 测试类命名：{原类名}Test，放在src/test/java对应包下
- 测试方法命名：test{方法名}_{场景}_{预期结果}
- 必须添加@DisplayName标注测试场景
- controller层使用@WebMvcTest + MockMvc
- service层使用@ExtendWith(MockitoExtension.class) + Mockito
- util类必须覆盖空值、边界、异常场景
- 每个测试方法只测一个逻辑，独立可运行

---
> Source: [fang129/copilot-vibe2](https://github.com/fang129/copilot-vibe2) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-04-29 -->
