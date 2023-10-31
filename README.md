# LuoBI

## 项目介绍

**BI（商业智能）：**数据可视化，报表可视化系统

**主流BI：**

- 帆软BI https://www.finebi.com
- 小马BI https://bi.zhls.qq.com
- PowerBI https://powerbi.microsoft.com/zh-cn 

**传统BI：**

https://chartcube.alipay.com

1. 需要人工上传数据
2. 需要人工选择数据的行和列进行分析（数据分析师）
3. 需要人工选择图表类型（数据分析师）
4. 生成图表（保存配置）

**智能BI平台：**

1. 用户（数据分析者）只需要导入原始数据，输入想要分析的目标（比如”帮我分析以下网站的用户增长趋势“），就能利用AI自动生成一个符合要求的图表以及结论
2. 不会数据分析的用户，也能通过输入目标快速完成数据分析，大幅节约人力成本

## 需求分析

1. 智能分析：用户输入目标，原始数据和其他参数（如图表类型），可以自动生成图表和结论
2. 图表管理
3. 图表生成的异步化（消息队列 ）
4. 对接 AI 

## 架构图

![image-20231030214212882](assets/image-20231030214212882.png)

## 技术选择

**前端：**

- React
- Ant Design Pro
- Umi
- ECharts 可视化开发库
- umi openapi 代码生成（自动生成后端调用代码）

**后端：**

- Spring Boot（万用 Java 后端项目模板）
- MySQL
- MyBatis Plus
- RabbitMQ
- AI 能力（Open AI 接口开发/ 使用现成的 AI 接口）
- EasyExcel（Excel 的上传和数据的解析）
- Swagger + Knife4j
- Hutool

## 计划

**初始化**

- 前端
- 后端

**后端**

- 数据库表设计
- 自动生成增删改查代码
- 图表管理
- 文件上传

**前端**

- 自动生成后端调用代码
- 修改requestErrorConfig

- 登录功能
- 图表分析页面
- 图表管理页面

## 初始化

### 前端

https://pro.ant.design/

1. 创建项目

```sh
# 使用 npm，
npm i @ant-design/pro-cli -g
pro create luobi-frontend
```

![image-20231031101557449](assets/image-20231031101557449.png)

2. 安装依赖

```
yarn
```

![image-20231031102623399](assets/image-20231031102623399.png)

3. 去除国际化

![image-20231031105510481](assets/image-20231031105510481.png)

[🐛 [BUG\] 执行删除国际化命令报错 · Issue #10452 · ant-design/ant-design-pro (github.com)](https://github.com/ant-design/ant-design-pro/issues/10452)

![image-20231031105637111](assets/image-20231031105637111.png)

```sh
yarn add eslint-config-prettier --dev yarn add eslint-plugin-unicorn --dev 
```

![image-20231031105714817](assets/image-20231031105714817.png)

![image-20231031105729743](assets/image-20231031105729743.png)

再次执行去除国际化就能成功了

![image-20231031105849430](assets/image-20231031105849430.png)



### 后端

1. 使用后端项目模板

复制模板，修改目录名为项目名

2. 连接数据库

   ![image-20231031113006757](assets/image-20231031113006757.png)

3. 修改数据库名

![image-20231031113312177](assets/image-20231031113312177.png)

4. 修改项目名

![image-20231031113526512](assets/image-20231031113526512.png)

![image-20231031113636977](assets/image-20231031113636977.png)

5. 开启分布式session

![image-20231031114027856](assets/image-20231031114027856.png)

![image-20231031114048256](assets/image-20231031114048256.png)

6. 修改端口

![image-20231031114227436](assets/image-20231031114227436.png)

7. 启动并访问接口文档

![image-20231031114401762](assets/image-20231031114401762.png)

## 后端

### 数据库表设计

**用户表：**

```sql
create table if not exists user
(
    id           bigint auto_increment comment 'id' primary key,
    userAccount  varchar(256)                           not null comment '账号',
    userPassword varchar(512)                           not null comment '密码',
    userName     varchar(256)                           null comment '用户昵称',
    userAvatar   varchar(1024)                          null comment '用户头像',
    userProfile  varchar(512)                           null comment '用户简介',
    userRole     varchar(256) default 'user'            not null comment '用户角色：user/admin/ban',
    gender       tinyint                                null comment '性别',
    createTime   datetime     default CURRENT_TIMESTAMP not null comment '创建时间',
    updateTime   datetime     default CURRENT_TIMESTAMP not null on update CURRENT_TIMESTAMP comment '更新时间',
    isDelete     tinyint      default 0                 not null comment '是否删除',
    index idx_userAccount (userAccount)
) comment '用户' collate = utf8mb4_unicode_ci;
```

**图表信息表：**

```sql
-- 图表信息表
create table if not exists chart
(
    id         bigint auto_increment comment 'id' primary key,
    goal       text                               null comment '分析目标',
    rawData    text                               null comment '原始数据',
    chartType  varchar(128)                       null comment '图表类型',
    genChart   text                               null comment 'AI生成的图表数据',
    genSummary text                               null comment 'AI生成的分析总结',
    userId     bigint                             null comment '创建人id',
    createTime datetime default CURRENT_TIMESTAMP not null comment '创建时间',
    updateTime datetime default CURRENT_TIMESTAMP not null on update CURRENT_TIMESTAMP comment '更新时间',
    isDelete   tinyint  default 0                 not null comment '是否删除'
) comment '图表信息' collate = utf8mb4_unicode_ci;
```

### 自动生成增删改查代码

![image-20231031141404931](assets/image-20231031141404931.png)

![image-20231031141439155](assets/image-20231031141439155.png)

![image-20231031141503920](assets/image-20231031141503920.png)

1. 生成的UserService和UserServiceImpl不用替换原来的

2. 修改字段的注解

   ![image-20231031142728852](assets/image-20231031142728852.png)

   ![image-20231031142744734](assets/image-20231031142744734.png)

   ![image-20231031142801435](assets/image-20231031142801435.png)

   ![image-20231031142813281](assets/image-20231031142813281.png)

3. 删掉UserService和UserServiceImpl和UserController中用不到的方法

   ![image-20231031143327862](assets/image-20231031143327862.png)

   ![image-20231031143356282](assets/image-20231031143356282.png)

   ![image-20231031143538773](assets/image-20231031143538773.png)

4. 删掉微信公众号开发包

   ![image-20231031143643260](assets/image-20231031143643260.png)

5. 删掉WxMPController

   ![image-20231031143826847](assets/image-20231031143826847.png)

### 图表管理

1. 复制PostController修改成ChartController

   ![image-20231031145120748](assets/image-20231031145120748.png)

   ![image-20231031145230166](assets/image-20231031145230166.png)

   ![image-20231031145445986](assets/image-20231031145445986.png)  

2. 创建关于Chart的DTO类

   ![image-20231031153057014](assets/image-20231031153057014.png)

3. 剪裁ChartController并为ChartService提供getQueryWrapper方法

   ```java
   /**
   * @author 落樱的悔恨
   * @description 针对表【chart(图表信息表)】的数据库操作Service实现
   * @createDate 2023-10-31 14:16:29
   */
   @Service
   public class ChartServiceImpl extends ServiceImpl<ChartMapper, Chart>
       implements ChartService{
   
       /**
        * 获取查询包装类
        *
        * @param chartQueryRequest
        * @return
        */
       @Override
       public QueryWrapper<Chart> getQueryWrapper(ChartQueryRequest chartQueryRequest) {
           QueryWrapper<Chart> queryWrapper = new QueryWrapper<>();
           if (chartQueryRequest == null) {
               return queryWrapper;
           }
           Long id = chartQueryRequest.getId();
           Long userId = chartQueryRequest.getUserId();
           String goal = chartQueryRequest.getGoal();
           String chartType = chartQueryRequest.getChartType();
           String sortField = chartQueryRequest.getSortField();
           String sortOrder = chartQueryRequest.getSortOrder();
           // 拼接查询条件
           queryWrapper.eq(id != null && id > 0, "id", id);
           queryWrapper.eq(ObjectUtils.isNotEmpty(userId), "userId", userId);
           queryWrapper.like(StringUtils.isNotEmpty(goal),"goal",goal);
           queryWrapper.like(StringUtils.isNotEmpty(chartType),"chartType",chartType);
           queryWrapper.eq("isDelete", false);
           queryWrapper.orderBy(SqlUtils.validSortField(sortField), sortOrder.equals(CommonConstant.SORT_ORDER_ASC),
                   sortField);
           return queryWrapper;
       }
   }
   ```

   

   

## 前端

### 自动生成后端调用代码

![image-20231031161114725](assets/image-20231031161114725.png)

![image-20231031161044255](assets/image-20231031161044255.png)

![image-20231031161225991](assets/image-20231031161225991.png)

![image-20231031160739020](assets/image-20231031160739020.png)

### 修改requestErrorConfig

![image-20231005160127336](assets/image-20231005160127336.png)

![image-20231005160035130](assets/image-20231005160035130.png)

![image-20231005170916555](assets/image-20231005170916555.png)

![image-20231007102657764](assets/image-20231007102657764.png)

![image-20231007102743482](assets/image-20231007102743482.png)

![image-20231005160150723](assets/image-20231005160150723.png)

![image-20231005160213602](assets/image-20231005160213602.png)

