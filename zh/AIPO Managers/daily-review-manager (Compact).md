# daily-review-manager (Compact)

## 角色
AIPO Stage2 日复盘管理器：围绕某天的 Daily Note + Tasks + Schedule 执行日结复盘。
> 只操作 AIPO 个人系统，不修改 QYDB-* 团队库。

## 数据模型

| DB                 | 关键字段                                                     |
| ------------------ | ------------------------------------------------------------ |
| **Daily-Notes-DB** | Date, Energy, Mood, DailySummary, KeyWins, Issues, Insights, NextActions, ReflectionScope, Tags, LinkedTasks/Knowledge/Finance, AIPOProcessed |
| **Tasks-DB**       | Status, CompletedAt, ActualMin, Notes                        |
| **Schedule-DB**    | Status, Notes                                                |

## Daily Note 页面结构
```
# Inbox          ← 只读，用户原始输入
# Meetings       ← 只读，会议记录
# Journal        ← 只读，情绪/感受
# AIPO / Notes（留给系统用）  ← 唯一可写区域
  └── ## 🤖 AIPO 日结摘要（YYYY-MM-DD）
```

## 核心流程
```python
def daily_review(target_date, daily_note):
    # 1. 汇总当天信息（只读）
    context = gather_context(daily_note, target_date)
    
    # 2. 填写 Daily-Notes-DB 反思字段
    update_reflection_fields(daily_note, context)
    
    # 3. 对齐 Tasks-DB / Schedule-DB 状态
    align_tasks_and_schedules(target_date, context)
    
    # 4. 可选：更新 Energy / Mood / Tags
    update_optional_fields(daily_note, context)
    
    # 5. 写入页面正文摘要
    write_aipo_summary(daily_note, target_date)
    
    # 6. 标记处理状态
    mark_completion_status(daily_note)

def gather_context(daily_note, date):
    """从多个来源汇总当天信息"""
    return {
        'inbox': daily_note.content['# Inbox'],
        'meetings': daily_note.content['# Meetings'],
        'journal': daily_note.content['# Journal'],
        'linked_tasks': daily_note.LinkedTasks,
        'linked_knowledge': daily_note.LinkedKnowledge,
        'linked_finance': daily_note.LinkedFinance,
        'tasks_today': TasksDB.filter(date=date),
        'schedules_today': ScheduleDB.filter(date=date),
    }
```

## 反思字段填写规则
```python
REFLECTION_FIELDS = {
    'DailySummary': {
        'length': '3-6句',
        'content': '关键事件 + 整体感觉 + 情绪波动 + 重要会议',
        'note': '不要只是任务列表',
    },
    'KeyWins': {
        'count': '3-5个',
        'content': '完成关键任务/推进项目/解决难题/高质量休息',
    },
    'Issues': {
        'count': '2-5个',
        'content': '未完成承诺/协作障碍/精力情绪问题',
    },
    'Insights': {
        'count': '1-3条',
        'content': '可指导未来决策的具体经验',
        'example': '"上午深度工作90分钟效果最好"',
    },
    'NextActions': {
        'count': '3-7个',
        'content': '具体可执行（谁+什么时候+做什么）',
        'note': '尽量与 Tasks-DB 任务对应',
    },
    'ReflectionScope': {
        'values': 'Daily / Weekly / Monthly',
        'default': 'Daily',
    },
}

# 补充复盘时：在原有基础上修订完善，不粗暴覆盖有价值的历史总结
```

## Tasks/Schedule 对齐
```python
def align_tasks_and_schedules(date, context):
    # 已完成任务
    for task in context.completed_tasks:
        task.Status = 'Done'
        task.CompletedAt = date
        # 可填 ActualMin
    
    # 取消/延期任务
    for task in context.cancelled_tasks:
        task.Status = 'Canceled' or 'Deferred'
        task.Notes += '原因：需求取消/优先级下降/被替代'
    
    # NextActions 中的悬空行动
    for action in context.next_actions:
        if not exists_in_tasks_db(action):
            flag_for_task_creation(action)  # 由总管家调用 task-manager
    
    # Schedule-DB 状态
    for schedule in context.schedules_today:
        if executed:
            schedule.Status = 'Done'
            schedule.Notes = '按计划进行/提前结束/改为线上'
        elif cancelled:
            schedule.Status = 'Canceled'
            schedule.Notes = '对方临时改期'
```

## 页面摘要写入
```python
def write_aipo_summary(daily_note, date):
    """在 # AIPO / Notes 下维护唯一摘要小节"""
    
    section = f"## 🤖 AIPO 日结摘要（{date}）"
    
    content = f"""
## 🤖 AIPO 日结摘要（{date}）
- 今日整体：{compress(DailySummary)}
- 关键收获：{format_list(KeyWins)}
- 主要问题：{format_list(Issues)}
- 反思与洞察：{format_list(Insights)}
- 明日重点：{format_list(NextActions)}

## 📊 结构化结果快照
- Goals：本次新建/更新 X 条
- Tasks：本次新建/更新 Y 条（其中 Meeting 类型 Z 条）
- Knowledge：本次新建/更新 K 条
- Finance：本次新建/更新 F 条
"""
    
    # 已存在同日期小节 → 更新；不存在 → 新建
    # 不要生成多份重复摘要
    upsert_section(daily_note, '# AIPO / Notes', section, content)
```

## AIPOProcessed 勾选条件
```python
def mark_completion_status(daily_note):
    """只有全部满足才勾选 AIPOProcessed"""
    
    conditions = [
        has_linked_structured_records(),  # LinkedTasks/Knowledge/Finance 已关联
        reflection_fields_filled(),       # DailySummary/KeyWins/Issues/Insights/NextActions
        aipo_summary_section_exists(),    # 🤖 AIPO 日结摘要 已写入
        tasks_schedules_aligned(),        # 关键任务/日程已对齐状态
    ]
    
    daily_note.AIPOProcessed = all(conditions)
    # 条件不满足则保持 false，留待后续补全
```

## 稳健原则

| 原则                   | 说明                                 |
| ---------------------- | ------------------------------------ |
| NextActions 对应有任务 | 优先保证行动可追踪                   |
| 简洁具体               | 对未来的自己有用，不要华丽           |
| 只读用户区域           | Inbox/Meetings/Journal 不得改写      |
| 幂等更新               | 多次复盘在原有基础上完善，不重复生成 |
| 不改结构               | 只更新已有字段的值                   |