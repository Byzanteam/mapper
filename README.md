### 映射服务一共有三个版本

1. [table_mapper](https://github.com/Byzanteam/slp_table_mapper)
2. [trucker](https://github.com/Byzanteam/trucker)
3. [skylark 内置 Mapping 模块](https://github.com/Byzanteam/skylark/tree/develop/app/services/mapping)。
   通过配置 sink 库来启用，[example](https://github.com/Byzanteam/skylark/blob/develop/config/database.example.yml)
   - 其中 search_path 可以让数据映射到不同的 schema 下面。

- 前两版已经放弃更新，目前主要维护第三版。

### sync 针对区别:

如果你熟悉trucker 的字段，可以直接读这一张，如果不熟悉，也可以读 [全字段](#全字段)
由于不同服务器的前缀不同，以下都以西园服务器来举例

1. 用 schema 替换表名前缀 ，取而代之的是将不同的服务器映射的表， 放在不同的 schema 当中
   比如 trucker 会把 西园的表单隐射为 `xy_forms_1_260`
   但是 mapper 会将该表当放入名为 `xy` 的 schema 中 ， 表名会是 `xy.forms_260`

   - 词缀中的 `form` 替换成了复数形式的 `forms`
   - 表名中的 namespace_id 被去掉了

2. 新增用户信息的映射，表名为 users 其中包含了，用户名，手机号，openid, 等信息，如果在 journey 中需要查看用户信息,需要通过外键进行联表查询.
   | 表名 | 外键 |
   | -- | --|
   | moment | user_id |
   | journey | user_id | 
   | form | slp_user_id |
   | assignment| slp_user_id |
   | flow | user_id | 

3. 取消删除回传字段的功能, 比如某个动态字段，原映射名为 "test", 修改为"demo" 以后。新映射库中会增加一个名为 "demo"的字段。 “test“ 会保存在映射库中，如有需要可以手动删除

4. 新增 flows 表来存放表名以及 external_settings 功能，其中 id 字段和 skylark 的流程 id 一一对应
5. 部分表不再分表, 主要是以下三张表
    journeys
    moments
   vertices
   其中需要 `jounryes, moments, vertices` 查询的时候通过添加 flow_id 来过滤不同流程
   比如 查询 流程为 274 的 journyes
   在 trucker 当中可能是

```sql
  select * from xy_journey_1_275
```
在新映射库中需要调整为

```sql
select * from xy.journeys where xy.journeys.flow_id = 275
```
6. journeys  中 stashed 的数据不会在页面显示 需要跳过
7. assignment 中 stashed 和 skipped 不会再skylark 页面中显示，需要跳过去

### [全字段]
> 这里是针对 sync 服务的字段说明

##### Form
###### "forms_123"
| 字段名             | 含义 | 备注 |
|-----------------| -- | -- |
| slp_response_id | 表单记录 | |
| slp_created_at  | 表单记录创建时间 | |
| slp_updated_at  | 表单记录更新时间 | |
| slp_user_id     | 用户 id | |


##### Flow


###### flows
| 字段名 | 含义   | 备注 |
| -- |------| -- |
| flow_id | 流程id | |
| title | 流程名称 | |
| external_settings | 流程设置 | |

###### vertices 
| 字段名 | 含义 | 备注 |
| -- | -- | -- |
| id | 节点 id | |
| name | 节点名称 | |
| type | 节点类型 | 发起节点，结束节点，机器节点 |
| alias_name | 节点别名 | |
| flow_id| 流程id

###### journeys
| 字段名                        | 含义 | 备注 |
|----------------------------| -- | -- |
| id                         | 流程任务 id | |
| user_id                    | 发起人 id | |
| status                     | 流程状态 | |
| created_at                 | 发起时间 | |
| updated_at                 | 更新时间 | |
| current_duration_threshold | 当前逾期时间 | |
| user_name                  | 发起人名字 | |
| user_identifier            | 发起人识别码 | |
| current_vertex_id          | 当前节点 |  |
| flow_id | 流程id | |

###### moments
| 字段名                 | 含义 | 备注 |
|---------------------| -- | -- |
| id                  | moment id | 主要是记录流程通过，转交，回退的一些信息 |
| journey_id          | 流程任务 id | |
| status              | 处理状态 | |
| comment             | 处理意见 | |
| esignature          | 电子签字 | |
| created_at          | 处理时间 | |
| updated_at          | 更新时间 | |
| user_id             | 处理人 id | |
| flow_id | 流程id | |

###### assignment_123
| 字段名                            | 含义 | 备注 |
|--------------------------------| -- | -- |
| slp_assignemnt_id              | assignment id | 节点处理人和抄送者会分配一个 assignment |
| slp_journey_id                 | 关联 journey | |
| slp_status                     | 状态 | processing, processed, skipped |
| slp_category                   | 类型 | 描述关联信息是发起者，处理者还是抄送者 |
| slp_read                       | 是否查看 | |
| slp_current_duration_threshold | 当前任务逾期时间 | |
| slp_vertex_id                  | 流程节点 | |
| slp_user_id                    | 任务人 id | |
| created_at                 | 创建时间 | |
| updated_at                 | 更新时间 | |
| flow_id                    | 流程id | |

###### 备注：
- `assignment` 和 `form` 只对基本字段进行了说明