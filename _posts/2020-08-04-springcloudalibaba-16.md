---
layout:     post
title:      16.Spring Cloud Alibaba学习笔记
subtitle:   Seata案例--原理
date:       2020-08-04
author:     Chen Xingxu
header-img: img/post-bg-road.jpg
catalog:    true
tags:
    - SpringCloudAlibaba
---
# 16.Spring Cloud Alibaba学习笔记--Seata案例--原理

## TC/TM/RM三个组件的近似理解

![](/img-post/2020-08-04-springcloudalibaba/16-01-no.png)

- TC（Transaction Coordinator） —— Seata 服务器
- TM（Transaction Manager）—— 带有 `@GlobalTransactional` 的方法，事务的发起方
- RM（Resource Manager）—— 订单 / 库存 / 账户这三个数据库，事务的参与方

## 分布式事务的执行流程

1. TM 开启分布式事务（TM 向 TC 注册全局事务记录）；
2. 按业务场景，编排数据库、服务等事务内资源（RM 向 TC 汇报资源准备状态）；
3. TM 结束分布式事务，事务一阶段结束（TM 通知 TC 提交 / 回滚分布式事务）；
4. TC 汇报事务信息，决定分布式事务是提交还是回滚；
5. TC 通知所有 RM 提交 / 回滚资源，事务二阶段结束。

## AT模式如何做到对业务的无侵入

### AT模式简介

>## 前提
>
>- 基于支持本地 ACID 事务的关系型数据库。
>- Java 应用，通过 JDBC 访问数据库。
>
>## 整体机制
>
>两阶段提交协议的演变：
>
>- 一阶段：业务数据和回滚日志记录在同一个本地事务中提交，释放本地锁和连接资源。
>- 二阶段：
>  - 提交异步化，非常快速地完成。
>  - 回滚通过一阶段的回滚日志进行反向补偿。
>
>

参考：

http://seata.io/zh-cn/docs/overview/what-is-seata.html

AT 模式对应于阿里云的全局事务服务（Global Transaction Service，简称 GTS）。

### 一阶段加载

![](/img-post/2020-08-04-springcloudalibaba/16-02-no.png)

在一阶段，Seata 拦截“业务 SQL”：

1. 解析 SQL 语义，找到“业务 SQL”要更新的业务数据，在业务数据被更新前，将其保存成“before image”（前置镜像）；
2. 执行“业务 SQL”更新业务数据；
3. 在业务数据更新之后，其保存成"after image” （后置镜像），最后生成行锁。

以上操作全部在一个数据库事务内完成，这样保证了一阶段操作的原子性。

官方的过程可参考：

http://seata.io/zh-cn/docs/dev/mode/at-mode.html

### 二阶段提交

![](/img-post/2020-08-04-springcloudalibaba/16-03-no.png)

二阶段如果顺利提交的话，因为“业务 SQL”在一阶段已经提交至数据库，所以 Seata 框架只需将一阶段保存的快照数据和行锁删掉，完成数据清理即可。

### 二阶段回滚

![](/img-post/2020-08-04-springcloudalibaba/16-04-no.png)

二阶段如果是回滚的话，Seata 就需要回滚一阶段已经执行的“业务 SQL”，还原业务数据。回滚方式便是用“before image”还原业务数据；但在**还原前要首先要校验脏写** ，对比”数据库当前业务数据”和"after image”，如果两份数据完全一致就说明没有脏写， 可以还原业务数据，如果**不一致就说明有脏写，出现脏写就需要转人工处理** 。

## 通过 DEBUG 了解 Seata 原理

### 打断点

### 1、以 DEBUG 模式启动

seata-order-service2001，seata-storage-service2002，seata-account-service2003

### 2、打断点

在 seata-account-service2003 的 AccountServiceImpl 中打断点

![](/img-post/2020-08-04-springcloudalibaba/16-05.png)

### 3、刷新网址

 `http://localhost:2001/order/create?userId=1&productId=1&count=10&money=100`

### 4、查看数据库

#### seata -> branch_table

![](/img-post/2020-08-04-springcloudalibaba/16-06.png)

| branch_id  | xid                            | transaction_id | resource_group_id | resource_id                                    | lock_key    | branch_type | status | client_id                                  | application_data | gmt_create         | gmt_modified       |
| ---------- | ------------------------------ | -------------- | ----------------- | ---------------------------------------------- | ----------- | ----------- | ------ | ------------------------------------------ | ---------------- | ------------------ | ------------------ |
| 2051276927 | 192.168.25.141:8091:2051276925 | 2051276925     |                   | jdbc:mysql://192.168.25.158:3306/seata_order   | t_order:17  | AT          | 2      | seata-order-service:192.168.25.141:61317   |                  | 2020-8-16 12:14:23 | 2020-8-16 12:14:23 |
| 2051276931 | 192.168.25.141:8091:2051276925 | 2051276925     |                   | jdbc:mysql://192.168.25.158:3306/seata_storage | t_storage:1 | AT          | 2      | seata-storage-service:192.168.25.141:61184 |                  | 2020-8-16 12:14:23 | 2020-8-16 12:14:23 |
| 2051276934 | 192.168.25.141:8091:2051276925 | 2051276925     |                   | jdbc:mysql://192.168.25.158:3306/seata_account | t_account:1 | AT          | 2      | seata-account-service:192.168.25.141:61232 |                  | 2020-8-16 12:14:23 | 2020-8-16 12:14:23 |

#### seata -> global_table

| xid                            | transaction_id | status | application_id      | transaction_service_group | transaction_name | timeout | begin_time    | application_data | gmt_create         | gmt_modified       |
| ------------------------------ | -------------- | ------ | ------------------- | ------------------------- | ---------------- | ------- | ------------- | ---------------- | ------------------ | ------------------ |
| 192.168.25.141:8091:2051276925 | 2051276925     | 5      | seata-order-service | fsp_tx_group              | fsp-create-order | 60000   | 1597551263304 |                  | 2020-8-16 12:14:23 | 2020-8-16 12:14:54 |

#### seata -> lock_table

| row_key                                                      | xid                            | transaction_id | branch_id  | resource_id                                    | table_name | pk   | gmt_create         | gmt_modified       |
| ------------------------------------------------------------ | ------------------------------ | -------------- | ---------- | ---------------------------------------------- | ---------- | ---- | ------------------ | ------------------ |
| jdbc:mysql://192.168.25.158:3306/seata_account^^^t_account^^^1 | 192.168.25.141:8091:2051276925 | 2051276925     | 2051276934 | jdbc:mysql://192.168.25.158:3306/seata_account | t_account  | 1    | 2020-8-16 12:14:23 | 2020-8-16 12:14:23 |
| jdbc:mysql://192.168.25.158:3306/seata_order^^^t_order^^^17  | 192.168.25.141:8091:2051276925 | 2051276925     | 2051276927 | jdbc:mysql://192.168.25.158:3306/seata_order   | t_order    | 17   | 2020-8-16 12:14:23 | 2020-8-16 12:14:23 |
| jdbc:mysql://192.168.25.158:3306/seata_storage^^^t_storage^^^1 | 192.168.25.141:8091:2051276925 | 2051276925     | 2051276931 | jdbc:mysql://192.168.25.158:3306/seata_storage | t_storage  | 1    | 2020-8-16 12:14:23 | 2020-8-16 12:14:23 |

#### seata_order -> undo_log

![](/img-post/2020-08-04-springcloudalibaba/16-07.png)

| id   | branch_id  | xid                            | context            | rollback_info | log_status | log_created        | log_modified       | ext  |
| ---- | ---------- | ------------------------------ | ------------------ | ------------- | ---------- | ------------------ | ------------------ | ---- |
| 11   | 2051276927 | 192.168.25.141:8091:2051276925 | serializer=jackson | 见下方        | 0          | 2020-8-16 12:14:23 | 2020-8-16 12:14:23 |      |

seata_order -> undo_log -> rollback_info

```json
{
  "@class": "io.seata.rm.datasource.undo.BranchUndoLog",
  "xid": "192.168.25.141:8091:2051276925",
  "branchId": 2051276927,
  "sqlUndoLogs": [
    "java.util.ArrayList",
    [
      {
        "@class": "io.seata.rm.datasource.undo.SQLUndoLog",
        "sqlType": "INSERT",
        "tableName": "t_order",
        "beforeImage": {
          "@class": "io.seata.rm.datasource.sql.struct.TableRecords$EmptyTableRecords",
          "tableName": "t_order",
          "rows": [
            "java.util.ArrayList",
            []
          ]
        },
        "afterImage": {
          "@class": "io.seata.rm.datasource.sql.struct.TableRecords",
          "tableName": "t_order",
          "rows": [
            "java.util.ArrayList",
            [
              {
                "@class": "io.seata.rm.datasource.sql.struct.Row",
                "fields": [
                  "java.util.ArrayList",
                  [
                    {
                      "@class": "io.seata.rm.datasource.sql.struct.Field",
                      "name": "id",
                      "keyType": "PrimaryKey",
                      "type": -5,
                      "value": [
                        "java.lang.Long",
                        17
                      ]
                    },
                    {
                      "@class": "io.seata.rm.datasource.sql.struct.Field",
                      "name": "user_id",
                      "keyType": "NULL",
                      "type": -5,
                      "value": [
                        "java.lang.Long",
                        1
                      ]
                    },
                    {
                      "@class": "io.seata.rm.datasource.sql.struct.Field",
                      "name": "product_id",
                      "keyType": "NULL",
                      "type": -5,
                      "value": [
                        "java.lang.Long",
                        1
                      ]
                    },
                    {
                      "@class": "io.seata.rm.datasource.sql.struct.Field",
                      "name": "count",
                      "keyType": "NULL",
                      "type": 4,
                      "value": 10
                    },
                    {
                      "@class": "io.seata.rm.datasource.sql.struct.Field",
                      "name": "money",
                      "keyType": "NULL",
                      "type": 3,
                      "value": [
                        "java.math.BigDecimal",
                        100
                      ]
                    },
                    {
                      "@class": "io.seata.rm.datasource.sql.struct.Field",
                      "name": "status",
                      "keyType": "NULL",
                      "type": 4,
                      "value": 0
                    }
                  ]
                ]
              }
            ]
          ]
        }
      }
    ]
  ]
}
```

#### seata_storage -> undo_log

| id   | branch_id  | xid                            | context            | rollback_info | log_status | log_created        | log_modified       | ext  |
| ---- | ---------- | ------------------------------ | ------------------ | ------------- | ---------- | ------------------ | ------------------ | ---- |
| 11   | 2051276931 | 192.168.25.141:8091:2051276925 | serializer=jackson | 见下方        | 0          | 2020-8-16 12:14:23 | 2020-8-16 12:14:23 |      |

seata_storage -> undo_log -> rollback_info

```json
{
  "@class": "io.seata.rm.datasource.undo.BranchUndoLog",
  "xid": "192.168.25.141:8091:2051276925",
  "branchId": 2051276931,
  "sqlUndoLogs": [
    "java.util.ArrayList",
    [
      {
        "@class": "io.seata.rm.datasource.undo.SQLUndoLog",
        "sqlType": "UPDATE",
        "tableName": "t_storage",
        "beforeImage": {
          "@class": "io.seata.rm.datasource.sql.struct.TableRecords",
          "tableName": "t_storage",
          "rows": [
            "java.util.ArrayList",
            [
              {
                "@class": "io.seata.rm.datasource.sql.struct.Row",
                "fields": [
                  "java.util.ArrayList",
                  [
                    {
                      "@class": "io.seata.rm.datasource.sql.struct.Field",
                      "name": "id",
                      "keyType": "PrimaryKey",
                      "type": -5,
                      "value": [
                        "java.lang.Long",
                        1
                      ]
                    },
                    {
                      "@class": "io.seata.rm.datasource.sql.struct.Field",
                      "name": "used",
                      "keyType": "NULL",
                      "type": 4,
                      "value": 20
                    },
                    {
                      "@class": "io.seata.rm.datasource.sql.struct.Field",
                      "name": "residue",
                      "keyType": "NULL",
                      "type": 4,
                      "value": 80
                    }
                  ]
                ]
              }
            ]
          ]
        },
        "afterImage": {
          "@class": "io.seata.rm.datasource.sql.struct.TableRecords",
          "tableName": "t_storage",
          "rows": [
            "java.util.ArrayList",
            [
              {
                "@class": "io.seata.rm.datasource.sql.struct.Row",
                "fields": [
                  "java.util.ArrayList",
                  [
                    {
                      "@class": "io.seata.rm.datasource.sql.struct.Field",
                      "name": "id",
                      "keyType": "PrimaryKey",
                      "type": -5,
                      "value": [
                        "java.lang.Long",
                        1
                      ]
                    },
                    {
                      "@class": "io.seata.rm.datasource.sql.struct.Field",
                      "name": "used",
                      "keyType": "NULL",
                      "type": 4,
                      "value": 30
                    },
                    {
                      "@class": "io.seata.rm.datasource.sql.struct.Field",
                      "name": "residue",
                      "keyType": "NULL",
                      "type": 4,
                      "value": 70
                    }
                  ]
                ]
              }
            ]
          ]
        }
      }
    ]
  ]
}
```

#### seata_account -> undo_log

| id   | branch_id  | xid                            | context            | rollback_info | log_status | log_created        | log_modified       | ext  |
| ---- | ---------- | ------------------------------ | ------------------ | ------------- | ---------- | ------------------ | ------------------ | ---- |
| 2    | 2051276842 | 192.168.25.141:8091:2051276834 | serializer=jackson | {}            | 1          | 2020-8-16 12:06:26 | 2020-8-16 12:06:26 |      |
| 7    | 2051276934 | 192.168.25.141:8091:2051276925 | serializer=jackson | 见下方        | 0          | 2020-8-16 12:14:23 | 2020-8-16 12:14:23 |      |

seata_account -> undo_log -> rollback_info

```json
{
  "@class": "io.seata.rm.datasource.undo.BranchUndoLog",
  "xid": "192.168.25.141:8091:2051276925",
  "branchId": 2051276934,
  "sqlUndoLogs": [
    "java.util.ArrayList",
    [
      {
        "@class": "io.seata.rm.datasource.undo.SQLUndoLog",
        "sqlType": "UPDATE",
        "tableName": "t_account",
        "beforeImage": {
          "@class": "io.seata.rm.datasource.sql.struct.TableRecords",
          "tableName": "t_account",
          "rows": [
            "java.util.ArrayList",
            [
              {
                "@class": "io.seata.rm.datasource.sql.struct.Row",
                "fields": [
                  "java.util.ArrayList",
                  [
                    {
                      "@class": "io.seata.rm.datasource.sql.struct.Field",
                      "name": "id",
                      "keyType": "PrimaryKey",
                      "type": -5,
                      "value": [
                        "java.lang.Long",
                        1
                      ]
                    },
                    {
                      "@class": "io.seata.rm.datasource.sql.struct.Field",
                      "name": "residue",
                      "keyType": "NULL",
                      "type": 3,
                      "value": [
                        "java.math.BigDecimal",
                        800
                      ]
                    },
                    {
                      "@class": "io.seata.rm.datasource.sql.struct.Field",
                      "name": "used",
                      "keyType": "NULL",
                      "type": 3,
                      "value": [
                        "java.math.BigDecimal",
                        200
                      ]
                    }
                  ]
                ]
              }
            ]
          ]
        },
        "afterImage": {
          "@class": "io.seata.rm.datasource.sql.struct.TableRecords",
          "tableName": "t_account",
          "rows": [
            "java.util.ArrayList",
            [
              {
                "@class": "io.seata.rm.datasource.sql.struct.Row",
                "fields": [
                  "java.util.ArrayList",
                  [
                    {
                      "@class": "io.seata.rm.datasource.sql.struct.Field",
                      "name": "id",
                      "keyType": "PrimaryKey",
                      "type": -5,
                      "value": [
                        "java.lang.Long",
                        1
                      ]
                    },
                    {
                      "@class": "io.seata.rm.datasource.sql.struct.Field",
                      "name": "residue",
                      "keyType": "NULL",
                      "type": 3,
                      "value": [
                        "java.math.BigDecimal",
                        700
                      ]
                    },
                    {
                      "@class": "io.seata.rm.datasource.sql.struct.Field",
                      "name": "used",
                      "keyType": "NULL",
                      "type": 3,
                      "value": [
                        "java.math.BigDecimal",
                        300
                      ]
                    }
                  ]
                ]
              }
            ]
          ]
        }
      }
    ]
  ]
}
```

### 放行后继续执行

异步任务阶段的分支提交请求将异步和批量地删除相应 UNDO LOG 记录。

![](/img-post/2020-08-04-springcloudalibaba/16-04-no.png)

- seata -> branch_table
- seata -> global_table
- seata -> lock_table
- seata_order -> undo_log
- seata_storage -> undo_log
- seata_account -> undo_log 

内容被全部删除。

## 总体执行流程

![](/img-post/2020-08-04-springcloudalibaba/16-08-no.png)