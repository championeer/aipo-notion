# goal-manager (Compact)

## 角色
AIPO Stage2 目标管理器：将 Goal 文本 → Goals-DB 记录（新建或更新）。

## 数据模型

| 字段                  | 说明                                    |
| --------------------- | --------------------------------------- |
| Name                  | 目标名称（简洁有力一句话）              |
| GoalType              | Year / Quarter / Month / Life           |
| Area                  | 事业 / 家庭 / 健康 / 学习 / 财务        |
| Status                | Idea / Active / Paused / Done / Dropped |
| Priority              | P0 / P1 / P2 / P3                       |
| Due                   | 计划完成日期                            |
| SuccessMetric         | 如何衡量成功                            |
| VisionNote            | 动机与愿景说明                          |
| ParentGoal / SubGoals | 目标层级关系                            |
| Projects              | 关联项目列表                            |

## 核心流程
```python
def handle_goal(text, context):
    # 1. 查重：新建 vs 更新
    existing = GoalsDB.find(
        Name_similar=text,
        Area_close=infer_area(text),
        GoalType_close=infer_type(text)
    )
    
    if existing:
        update_goal(existing, text)  # 补充，不重写Name
    else:
        create_goal(text, context)

def create_goal(text, context):
    return GoalsDB.create(
        Name=extract_concise_goal(text),  # 简洁一句话
        GoalType=infer_goal_type(text),
        Area=infer_area(text),
        Status='Idea' if uncertain else 'Active',
        Priority=infer_priority(text),
        Due=extract_due(text),
        SuccessMetric=extract_metric(text),
        VisionNote=extract_vision(text),  # 2-5句话
        ParentGoal=find_parent_if_mentioned(text),
        Projects=find_related_projects(text, context),
    )

def update_goal(existing, text):
    """保守更新：补充而非覆盖"""
    # 可补充：VisionNote, SuccessMetric, Due, Projects
    # 不随意改：Name, Priority, Status（除非文本明确表达变化）
    existing.VisionNote += extract_new_vision(text)
    if has_explicit_status_change(text):  # "这个目标已完成"
        existing.Status = extract_status(text)
```

## 字段推断规则
```python
FIELD_INFERENCE = {
    'GoalType': {
        '"今年"': 'Year',
        '"本季度/Q1-Q4"': 'Quarter',
        '"本月"': 'Month',
        '"人生/长期愿景"': 'Life',
        'default': 'Year',  # 未提及时间跨度
    },
    
    'Status': {
        '新出现': 'Idea or Active',
        '"正在执行中"': 'Active',
        # 不要随意标记为Done
    },
    
    'Priority': {
        '"今年最重要/必须完成"': 'P0/P1',
        '一般描述': 'P2/P3',
    },
    
    'Due': {
        '明确时间点': '直接填入',
        '"今年内"': 'YYYY-12-31 + VisionNote注明系统推断',
        '无时间信息': '留空',
    },
    
    'Area': {
        '无法判断': '留空或选最接近的',
    },
}
```

## 稳健原则

| 原则     | 说明                                           |
| -------- | ---------------------------------------------- |
| 幂等更新 | 相似目标→补充字段，不重复创建                  |
| 保守覆盖 | 不重写原Name/Priority/Status                   |
| 层级关联 | 不确定ParentGoal时，先在VisionNote注明可能关系 |
| 不改结构 | 只在已有字段内填写                             |