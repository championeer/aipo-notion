# AIPO 系统中 8 个核心数据库表结构

> 版本：v1.0
>
> 更新日期：2025年11月25日

------

## 🎯 Goals-DB（目标库）

| 字段名             | 类型                   | 说明                                          |
| ------------------ | ---------------------- | --------------------------------------------- |
| **Name**           | title                  | 目标名称                                      |
| **ID**             | auto_increment_id      | 目标ID（自动生成）                            |
| **GoalType**       | select                 | 目标层级：Year / Quarter / Month / Life       |
| **Area**           | select                 | 所属领域：事业 / 家庭 / 健康 / 学习 / 财务    |
| **Status**         | status                 | 状态：Idea / Active / Paused / Done / Dropped |
| **Priority**       | select                 | 优先级：P0 / P1 / P2 / P3                     |
| **Due**            | date                   | 计划完成日期                                  |
| **SuccessMetric**  | text                   | 成功衡量标准                                  |
| **VisionNote**     | text                   | 目标的动机、愿景说明                          |
| **ParentGoal**     | relation → Goals-DB    | 上级目标（自引用）                            |
| **SubGoals**       | rollup                 | 子目标（从 ParentGoal 汇总）                  |
| **Projects**       | relation ↔ Projects-DB | 关联的项目（双向）                            |
| **CreatedTime**    | created_time           | 创建时间                                      |
| **LastEditedTime** | last_edited_time       | 最近编辑时间                                  |

------

## 📔 Daily-Notes-DB（每日笔记库）

| 字段名              | 类型                    | 说明                                      |
| ------------------- | ----------------------- | ----------------------------------------- |
| **Name**            | title                   | 当天标识，格式为 YYYY-MM-DD               |
| **Date**            | date                    | 当天日期                                  |
| **Week**            | formula                 | 周次（自动计算）                          |
| **Energy**          | select                  | 当日能量自评：High / Medium / Low         |
| **Mood**            | select                  | 当日心情：😊 / 😐 / 😔 / 😤 / 🤗               |
| **DailySummary**    | text                    | 当日摘要                                  |
| **KeyWins**         | text                    | 今日收获/亮点                             |
| **Issues**          | text                    | 遇到的问题/阻碍                           |
| **Insights**        | text                    | 反思与洞察                                |
| **NextActions**     | text                    | 下一步要做的事情                          |
| **ReflectionScope** | select                  | 复盘范围：Daily / Weekly / Monthly / None |
| **Tags**            | multi_select            | 标签                                      |
| **AIPOProcessed**   | checkbox                | 标记是否已被 AIPO 处理过                  |
| **LinkedTasks**     | relation ↔ Tasks-DB     | 关联的任务（双向）                        |
| **LinkedKnowledge** | relation ↔ Knowledge-DB | 关联的知识卡片（双向）                    |
| **LinkedFinance**   | relation ↔ Finance-DB   | 关联的财务记录（双向）                    |
| **CreatedTime**     | created_time            | 创建时间                                  |
| **LastEditedTime**  | last_edited_time        | 最近编辑时间                              |

------

## ✅ Tasks-DB（任务库）

| 字段名              | 类型                      | 说明                                                         |
| ------------------- | ------------------------- | ------------------------------------------------------------ |
| **Name**            | title                     | 任务名称                                                     |
| **TaskID**          | auto_increment_id         | 任务ID（自动生成）                                           |
| **Status**          | status                    | 状态：Inbox / Open / Scheduled / Doing / Done / Deferred / Canceled |
| **Priority**        | select                    | 优先级：P0 / P1 / P2 / P3                                    |
| **TaskType**        | select                    | 任务类型：Deep Work / Shallow / Admin / Meeting              |
| **EnergyNeed**      | select                    | 能量需求：High / Medium / Low / Any                          |
| **EstimateMin**     | number                    | 预估时长（分钟）                                             |
| **ActualMin**       | number                    | 实际花费时长（分钟）                                         |
| **CompletedAt**     | date                      | 完成时间                                                     |
| **Notes**           | text                      | 补充说明、上下文信息                                         |
| **Tags**            | multi_select              | 标签                                                         |
| **Owner**           | person                    | 执行任务的负责人                                             |
| **Project**         | relation ↔ Projects-DB    | 所属项目（双向）                                             |
| **Goal**            | rollup                    | 从 Project 属性汇总得到的关联目标                            |
| **SourceDailyNote** | relation ↔ Daily-Notes-DB | 来源日记（双向）                                             |
| **ScheduleBlocks**  | relation ↔ Schedule-DB    | 关联的日程块（双向）                                         |
| **NextSchedule**    | rollup                    | 从 ScheduleBlocks 筛选最近未来的一次日程                     |
| **BlockedBy**       | relation → Tasks-DB       | 阻塞本任务的任务（自引用）                                   |
| **Blocking**        | rollup                    | 被本任务阻塞的其他任务                                       |
| **CapturedAt**      | created_time              | 任务首次被捕获的时间                                         |
| **CreatedTime**     | created_time              | 创建时间                                                     |
| **LastEditedTime**  | last_edited_time          | 最近编辑时间                                                 |

------

## 📊 Projects-DB（项目库）

| 字段名             | 类型                | 说明                                                    |
| ------------------ | ------------------- | ------------------------------------------------------- |
| **Name**           | title               | 项目名称                                                |
| **ProjectID**      | auto_increment_id   | 项目ID（自动生成）                                      |
| **Status**         | status              | 状态：Planning / In Progress / On Hold / Done / Dropped |
| **Priority**       | select              | 优先级：P0 / P1 / P2 / P3                               |
| **Area**           | select              | 所属领域：事业 / 家庭 / 健康 / 学习 / 财务              |
| **Start**          | date                | 项目启动日期                                            |
| **Due**            | date                | 项目截止日期                                            |
| **Notes**          | text                | 项目备注、背景、补充信息                                |
| **Owner**          | person              | 项目负责人                                              |
| **Goal**           | relation ↔ Goals-DB | 关联目标（双向）                                        |
| **Tasks**          | relation ↔ Tasks-DB | 关联的任务（双向）                                      |
| **CreatedTime**    | created_time        | 创建时间                                                |
| **LastEditedTime** | last_edited_time    | 最近编辑时间                                            |

------

## 💡 Knowledge-DB（知识库）

| 字段名              | 类型                      | 说明                                                         |
| ------------------- | ------------------------- | ------------------------------------------------------------ |
| **Name**            | title                     | 知识卡片标题                                                 |
| **KnowledgeID**     | auto_increment_id         | 知识ID（自动生成）                                           |
| **Type**            | select                    | 知识类型：Idea / Concept / Article / Book / Meeting / Snippet / Reference |
| **Status**          | status                    | 状态：Inbox / Draft / Canonical / Archived                   |
| **Importance**      | select                    | 重要性：High / Medium / Low                                  |
| **SourceType**      | select                    | 来源类型：DailyNote / Web / Book / Video / Email / Other     |
| **Summary**         | text                      | 2-3句摘要                                                    |
| **Keywords**        | text                      | 关键词                                                       |
| **SourceLink**      | url                       | 原始链接                                                     |
| **Tags**            | multi_select              | 标签                                                         |
| **ReviewDate**      | date                      | 最近一次回顾时间                                             |
| **NextReview**      | formula                   | 下次回顾时间（自动计算）                                     |
| **SourceDailyNote** | relation ↔ Daily-Notes-DB | 来源日记（双向）                                             |
| **Project**         | relation → Projects-DB    | 关联项目（单向）                                             |
| **Goal**            | relation → Goals-DB       | 关联目标（单向）                                             |
| **CreatedTime**     | created_time              | 创建时间                                                     |
| **LastEditedTime**  | last_edited_time          | 最近编辑时间                                                 |

------

## 💰 Finance-DB（财务库）

| 字段名              | 类型                      | 说明                                                        |
| ------------------- | ------------------------- | ----------------------------------------------------------- |
| **Name**            | title                     | 交易说明                                                    |
| **TransactionID**   | auto_increment_id         | 交易ID（自动生成）                                          |
| **Type**            | select                    | 类型：Income / Expense / Transfer                           |
| **Amount**          | number                    | 金额                                                        |
| **Currency**        | select                    | 币种：CNY / USD / EUR / GBP                                 |
| **Category**        | select                    | 类别：餐饮 / 工具 / 研发 / 教育 / 家庭 / 医疗 / 交通 / 其他 |
| **Date**            | date                      | 交易日期                                                    |
| **Month**           | formula                   | 交易月份（自动计算）                                        |
| **Account**         | select                    | 账户：Cash / Bank / Alipay / WeChat / Company               |
| **PaymentMethod**   | select                    | 支付方式：Cash / Card / Transfer / Online                   |
| **FixedOrVariable** | select                    | 固定或可变：Fixed / Variable                                |
| **PayerPayee**      | text                      | 交易对手                                                    |
| **Notes**           | text                      | 备注                                                        |
| **ReceiptURL**      | url                       | 凭证链接                                                    |
| **Tags**            | multi_select              | 标签                                                        |
| **SourceDailyNote** | relation ↔ Daily-Notes-DB | 来源日记（双向）                                            |
| **Project**         | relation → Projects-DB    | 关联项目（单向）                                            |
| **Goal**            | relation → Goals-DB       | 关联目标（单向）                                            |
| **CreatedTime**     | created_time              | 创建时间                                                    |
| **LastEditedTime**  | last_edited_time          | 最近编辑时间                                                |

------

## 📅 Schedule-DB（日程库）

| 字段名              | 类型              | 说明                                              |
| ------------------- | ----------------- | ------------------------------------------------- |
| **Name**            | title             | 日程名称                                          |
| **ScheduleID**      | auto_increment_id | 自动编号                                          |
| **When**            | date              | 日程时间安排，用于和 Notion Calendar 同步         |
| **Start**           | formula           | 开始时间（公式计算）                              |
| **End**             | formula           | 结束时间（公式计算）                              |
| **DurationMin**     | formula           | 时长（分钟，公式计算）                            |
| **Status**          | status            | 日程状态：Scheduled / Committed / Done / Canceled |
| **Calendar**        | select            | 日历类型：Personal / Work / Family                |
| **Location**        | text              | 地点                                              |
| **AllDay**          | checkbox          | 是否为全天事件                                    |
| **EnergyZone**      | select            | 能量区间：Peak / Stable / Low / Unknown           |
| **CalendarEventID** | text              | 外部日历的事件ID                                  |
| **Notes**           | text              | 日程补充说明                                      |
| **Task**            | relation          | 关联到 Tasks-DB                                   |
| **TeamMeeting**     | relation          | 关联的团队会议                                    |
| **CreatedTime**     | created_time      | 创建时间                                          |
| **LastEditedTime**  | last_edited_time  | 最近编辑时间                                      |



## 👤 Profile-DB（个人配置库）

| 字段名                      | 类型              | 说明                                                         |
| --------------------------- | ----------------- | ------------------------------------------------------------ |
| **Name**                    | title             | 配置名称                                                     |
| **ProfileID**               | auto_increment_id | 配置ID（自动生成）                                           |
| **Active**                  | checkbox          | 是否为当前有效配置                                           |
| **Owner**                   | person            | 所有人                                                       |
| **Timezone**                | select            | 时区：Asia/Shanghai / UTC / America/New_York / Europe/London |
| **WorkDays**                | multi_select      | 工作日：Mon / Tue / Wed / Thu / Fri / Sat / Sun              |
| **MorningEnergy**           | number            | 上午能量评分 (0-10)                                          |
| **AfternoonEnergy**         | number            | 下午能量评分 (0-10)                                          |
| **EveningEnergy**           | number            | 晚上能量评分 (0-10)                                          |
| **PreferredDeepWorkSlots**  | text              | 深度工作时间段                                               |
| **PreferredLightWorkSlots** | text              | 浅层工作时间段说明                                           |
| **NoWorkTime**              | text              | 禁用时间段说明                                               |
| **MaxDeepWorkPerDayMin**    | number            | 每日最大深度工作时长（分钟）                                 |
| **MaxMeetingsPerDay**       | number            | 每日允许的最多会议数                                         |
| **DefaultTaskDurationMin**  | number            | 默认任务时长（分钟）                                         |
| **DefaultCalendar**         | select            | 默认同步日历：Personal / Work / Family                       |
| **SyncRules**               | text              | AIPO 与外部系统同步策略的文字说明                            |
| **CreatedTime**             | created_time      | 创建时间                                                     |
| **LastEditedTime**          | last_edited_time  | 最近编辑时间                                                 |

------

### 关联关系总览

| 关联                       | 类型   | 说明                    |
| -------------------------- | ------ | ----------------------- |
| Goals ↔ Projects           | 双向   | 目标与项目互相关联      |
| Goals → Goals              | 自引用 | ParentGoal 实现目标层级 |
| Projects ↔ Tasks           | 双向   | 项目包含多个任务        |
| Tasks ↔ Schedule           | 双向   | 任务可分配到日程块      |
| Tasks → Tasks              | 自引用 | BlockedBy 实现任务依赖  |
| Daily-Notes ↔ Tasks        | 双向   | 日记是任务的来源        |
| Daily-Notes ↔ Knowledge    | 双向   | 日记是知识卡片的来源    |
| Daily-Notes ↔ Finance      | 双向   | 日记是财务记录的来源    |
| Knowledge → Projects/Goals | 单向   | 知识卡片可关联项目/目标 |
| Finance → Projects/Goals   | 单向   | 财务记录可关联项目/目标 |