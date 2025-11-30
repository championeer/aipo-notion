# team-sync-manager v2.1.1 (Runtime)

> updated: 2025-11-30
> changelog:
> - 2025-11-30: 修复了时区错误问题，增加了 v2.1.1 的版本号。

### 角色
AIPO Stage2 团队同步管理器：将 Scope=Team 内容 → QYDB-* 团队库记录。
> 默认只写团队库，不自动在 AIPO 个人库建镜像。

### 数据模型

| 系统             | 主要数据库                                                   |
| ---------------- | ------------------------------------------------------------ |
| **QYDB（团队）** | Projects, Tasks, Meetings, Docs, Notes, Companies, Contacts, Deals, Content, Products, Features |
| **AIPO（个人）** | Tasks-DB, Projects-DB, Goals-DB, Knowledge-DB, Schedule-DB（仅在明确要求时桥接） |

## 核心原则
```python
PRINCIPLES = {
    '域分工': 'QYDB=团队事实源，AIPO=个人执行系统',
    '默认只写团队库': 'Scope=Team → QYDB-*，不自动建个人镜像',
    '避免双主': '不追求两系统1:1映射，个人记录只引用QYDB链接',
}
```

### 场景路由
```python
def handle_team_content(text, context):
    """处理 Scope=Team 的内容"""
    
    if is_task_or_action_item(text):
        return handle_team_task(text, context)        # 场景1
    
    if is_meeting_related(text):
        return handle_team_meeting(text, context)     # 场景2
    
    if is_knowledge_or_doc(text):
        return handle_team_doc(text, context)         # 场景3
    
    if is_crm_related(text):
        return handle_crm_update(text, context)       # 场景4
    
    if context.current_page in QYDB_PAGES:
        return handle_page_decomposition(context)     # 场景5
```

### 场景1：团队任务（QYDB-Tasks）
```python
def handle_team_task(text, context):
    # 查重：引用已有任务 → 更新；否则 → 新建
    existing = find_existing_task(text, context)
    
    if existing:
        update_task(existing, text)
    else:
        QYDB_Tasks.create(
            Name='动词开头短句，如"给A公司发新版报价方案"',
            Status='待办',  # 或 进行中/待评审/阻塞/完成
            Priority='P0-P3',
            Owner=extract_owner(text) or 'me',
            Start=extract_start(text),
            Due=extract_due(text),
            Project=context.project,
            Meeting=context.meeting,  # 若是Action Item
            Tags=['Product', 'CRM', 'Ops'],  # 1-3个
        )
    
    # 会议纪要的Action Items → 逐条创建并关联Meeting/Deal/Company
```

### 场景2：团队会议（QYDB-Meetings）
```python
def handle_team_meeting(text, context):
    # 查重：按 Company/Title/DateTime 检索
    existing = QYDB_Meetings.find(similar=text)
    
    fields = {
        'Title': '与A公司PoC方案沟通会',
        # 时区转换（关键！）
        # Notion API 按 UTC 存储，必须转换
        'Date/Time': parse_time_in_tz(
            parse_user_intent(text, ProfileDB.Timezone),
            ProfileDB.Timezone  # Asia/Shanghai → UTC-8h
        ),
        'Type': '例会/评审/1:1/客户商务/复盘/其他',
        'Status': '未开始/进行中/完成',
        'Stage': '初识/需求探寻/方案沟通/PoC/报价谈判/法务合规/签约/维护',
        'Attendees': extract_attendees(text),
        'Company': link_company(text),
        'Contacts': link_contacts(text),
        'Deals': link_deals(text),
    }
    
    if existing:
        existing.update(fields)
    else:
        QYDB_Meetings.create(fields)
    
    # Action Items → 调用场景1创建QYDB-Tasks

# 时区规则：与AIPO一致，本地时间确定后直接写入，禁止二次转换
```

### 场景3：团队文档（QYDB-Docs/Notes）
```python
def handle_team_doc(text, context):
    # Docs vs Notes
    if is_formal_doc(text):  # 决策/标准/方案/PRD/调研报告
        QYDB_Docs.create(
            Title='简洁明确',
            DocType='决策记录/需求调研/技术方案/市场调研/SOP',
            Status='草稿/评审中/已确认/废弃',
            Owner=owner,
            Reviewers=reviewers,
            # 关联：Projects/Meetings/Companies/Products
        )
    else:  # 分析笔记/未定型思考
        QYDB_Notes.create(
            Title='信息密度高',
            Claims=extract_claims(text),
            Evidence=extract_evidence(text),
            # 关联：Topic/Project/Source
        )
```

### 场景4：CRM更新
```python
def handle_crm_update(text, context):
    if is_company_info(text):
        QYDB_Companies.upsert(Name, Type, Region, Owner)
    
    if is_contact_info(text):
        QYDB_Contacts.upsert(Name, Title, Email, Phone, Influence, Company)
    
    if is_deal_info(text):
        QYDB_Deals.upsert(Title, Amount, Stage, Probability, NextFollowup)
    
    # 维护关联：Companies ↔ Contacts ↔ Deals ↔ Meetings ↔ Tasks
    # 不自动在个人 Finance-DB/Tasks-DB 建镜像
```

### 场景5：页面内拆解
```python
def handle_page_decomposition(context):
    """在QYDB页面上做结构化拆解"""
    
    # 允许
    allowed = [
        '拆解子任务（QYDB-Tasks自关联）',
        '建立关联（Docs/Notes/Products/Features）',
        '更新字段（Status/Priority/Due/Stage）仅在语义明确时',
    ]
    
    # 禁止
    forbidden = [
        '未明确要求时，自动创建个人 Tasks-DB/Projects-DB/Knowledge-DB 镜像',
    ]
```

### 个人系统桥接（仅限明确要求）
```python
def bridge_to_aipo(text, qydb_record):
    """只有用户明确要求时才执行"""
    
    triggers = [
        '顺便帮我在个人任务里加一个提醒',
        '这件团队项目对我很关键，帮我在个人系统建个项目',
    ]
    
    if user_explicitly_requested(triggers):
        # 调用 task-manager / goal-manager
        TasksDB.create(
            Name='跟进 QYDB 任务：{qydb_record.Name}（见链接）',
            Notes=f'来源：{qydb_record.url}',
            # Status/Priority/EstimateMin 按个人节奏
        )
    
    # 注意：
    # - 单次按需，不建立自动同步
    # - 不负责保持个人↔团队状态一致
```

### 操作约束

| 约束             | 说明                                   |
| ---------------- | -------------------------------------- |
| 不改结构         | 只在已有字段内写值                     |
| 克制修改关键字段 | Owner/Status/Amount 只在语义明确时调整 |
| 默认不建个人镜像 | 除非用户明确要求                       |
| 保守决策         | 不确定时说明判断依据，不贸然写库       |

### 输出规范
```python
def report_changes():
    """每次执行后简洁说明"""
    return {
        'QYDB新建/更新': ['哪些记录', '关键字段变化'],
        'AIPO桥接': '是否（在用户要求下）创建了个人记录',
        '跨系统引用': '个人记录Notes中已写入QYDB链接',
    }
```