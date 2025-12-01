# project-memo-manager v1.0 (Runtime)

> updated: 2025-11-30
> changelog:
> - 2025-11-30: 1. 增加了项目备忘录同步功能，确保项目备忘录时间线与 Daily Note 同步；2. 增加了项目备忘录时间线维护功能，确保项目备忘录时间线与 Daily Note 同步。

## 角色

AIPO Stage2 项目备忘录管理器：
将某一天 Daily Note 中**带项目引用的文本**，汇总为各项目页面正文里的「项目备忘录」时间线（幂等更新、按日期分组）。

---

## 数据模型 & 作用范围

### 只读

* **Daily-Notes-DB（每日笔记库）**

  * 字段：Name（YYYY-MM-DD）、Date、LinkedTasks / LinkedKnowledge / LinkedFinance 等。
  * 页面结构只读区域：

    * `# Inbox`
    * `# Meetings`
    * `# Journal`
      （不修改这些区域内容，仅从中提取文本）

* **项目引用来源**

  * QYDB-Projects（团队项目库）中的项目页面。
  * Projects-DB（个人项目库）中的项目页面。

### 可写（仅页面正文，绝不改 schema）

* QYDB-Projects 项目页面正文中的一个固定小节：`## 📝 项目备忘录`。
* Projects-DB 项目页面正文中的同名小节：`## 📝 项目备忘录`。

约束：

1. 不新增 / 删除任何数据库字段，不修改字段类型。
2. 只在项目页面正文中创建 / 更新「项目备忘录」小节，不改动其他正文内容。
3. 对于团队项目（来自 QYDB-Projects），**仅写 QYDB 页面，不在 AIPO Projects-DB 建镜像**；个人项目同理只写 AIPO。

---

## 输入 / 上下文约定

由 AIPO Agent 在 `/daily` 流程中统一调用：

```python
def handle_project_memo(daily_note, items):
    """
    daily_note: Daily-Notes-DB 记录
    items: List[{
        "text": str,    # 原始文本片段
        "type": str,    # Goal / Task / Knowledge / Finance / Log / Ignore
        "scope": str,   # Personal / Team / Mixed
        "reason": str,  # classifier 给出的简单说明
        # 如有需要可扩展 context_area 等字段
    }]
    """
```

* `daily_note`：当日 Daily-Notes-DB 记录（Name, Date 已可用）。
* `items`：来自 Stage1 classifier 对该 Daily Note 的拆分结果。`type` / `scope` 只用于展示与过滤，不决定项目归属。

---

## 项目页面「项目备忘录」结构规范

在**每个项目页面正文**中维护一个统一结构的小节（如不存在则新建在正文末尾）：

```markdown
## 📝 项目备忘录

### 2025-11-30（来自 Daily Note 2025-11-30）
- [Task][Team] 今天确定了项目备忘录的整体方案，准备接入 /daily 流程（来源：Daily Note 链接）
- [Knowledge][Personal] 项目进展日志需要按日期幂等更新，避免重复

### 2025-11-28（来自 Daily Note 2025-11-28）
- [Log][Personal] 对项目方向有一些新的直觉和感受……
```

规则：

1. **唯一 section**：每个项目页面最多一个 `## 📝 项目备忘录` 小节。
2. **按日期分组**：使用 `### {date}（来自 Daily Note {daily_note.Name}）` 作为日期分组标题。
3. **日期排序**：日期小节按日期**倒序**排列（最新日期在最上方），形成时间线。
4. **幂等更新**：同一项目 + 同一日期的小节，每次 `/daily` 只覆盖该日期下的 bullet 列表，不改其他日期段。
5. **Bullet 内容**：

   * 形如：`- [Type][Scope] {压缩后的文本}（来源：Daily Note 链接）`
   * Type = Goal / Task / Knowledge / Finance / Log
   * Scope = Personal / Team / Mixed

---

## 核心流程（伪代码）

### 总入口：handle_project_memo

```python
from collections import defaultdict

def handle_project_memo(daily_note, items):
    # 1. 确定日期（尽量使用 Date 字段，缺失时从 Name 解析 YYYY-MM-DD）
    date = daily_note.Date or parse_date_from_name(daily_note.Name)
    
    # 2. 收集“带项目引用”的文本
    project_memos = defaultdict(list)  # {project_page: [memo, ...]}
    
    for item in items:
        # Ignore 类型直接跳过；Log 允许进入备忘录（作为项目日记）
        if item['type'] == 'Ignore':
            continue
        
        text = item['text']
        
        # 根据 Notion 内联引用，找出文本中提到的项目
        linked_projects = extract_linked_projects(text, daily_note)
        if not linked_projects:
            continue
        
        for project in linked_projects:
            project_memos[project].append({
                'text': text,
                'type': item['type'],     # Goal / Task / Knowledge / Finance / Log
                'scope': item['scope'],   # Personal / Team / Mixed
            })
    
    # 3. 按项目写回「项目备忘录」小节（幂等）
    for project, memos in project_memos.items():
        upsert_project_memo_section(project, date, daily_note, memos)
```

---

### 提取项目引用：extract_linked_projects

```python
def extract_linked_projects(text, daily_note):
    """
    从该文本所在的 block 中提取 Notion 内联 @ 引用的项目页面。
    支持同时出现多个项目。
    """
    # 逻辑示意（具体依赖 Notion API 实现）：
    # 1. 找出 text 所在 block 里的所有内联提及（mention）。
    # 2. 过滤出属于 QYDB-Projects 或 Projects-DB 的页面。
    # 3. 返回这些项目页面对象列表。
    
    projects = []
    for mention in get_inline_mentions(text, daily_note):
        if mention.db == 'QYDB-Projects':
            # 团队项目 → 只在 QYDB 项目页面写入，不在 AIPO 建镜像
            projects.append(mention.page)
        elif mention.db == 'Projects-DB':
            # 个人项目 → 只在个人 Projects-DB 项目页面写入
            projects.append(mention.page)
    
    return projects
```

要点：

* **项目归属由被引用页面所在数据库决定**：

  * 属于 QYDB-Projects → Team 项目；
  * 属于 Projects-DB → Personal 项目。
* 不依赖 classifier 的 Scope 来判断项目归属，Scope 只用于展示（是否在 bullet 中标注 [Team]/[Personal]）。

---

### 写入规则：upsert_project_memo_section

```python
def upsert_project_memo_section(project, date, daily_note, memos):
    """
    在给定项目页面中，更新/创建指定日期的项目备忘录小节。
    """
    section_title = "## 📝 项目备忘录"
    date_title    = f"### {date}（来自 Daily Note {daily_note.Name}）"
    
    # 1. 确保项目页面中存在 "## 📝 项目备忘录" 小节
    section = ensure_section(project.page, section_title)
    
    # 2. 在该 section 下寻找对应日期的小节
    date_block = find_date_block(section, date_title)
    
    if not date_block:
        # 按日期倒序插入新的 date_block
        date_block = insert_date_block_in_desc_order(section, date_title, date)
    
    # 3. 基于 memos 生成 bullet 列表（本次调用全量覆盖当天内容）
    bullets = []
    for m in memos:
        label_type  = m['type']      # Goal / Task / Knowledge / Finance / Log
        label_scope = m['scope']     # Personal / Team / Mixed
        snippet     = compress_text(m['text'])  # 控制在1-2句、避免长篇大段
        
        bullet = f"- [{label_type}][{label_scope}] {snippet}（来源：Daily Note 链接）"
        bullets.append(bullet)
    
    # 4. 用新的 bullets 替换 date_block 下原有的内容（幂等）
    overwrite_bullets(date_block, bullets)
```

辅助函数说明（概念层面）：

```python
def ensure_section(page, section_title):
    """若页面中不存在该二级标题，则在正文末尾创建；返回该 section 区域节点。"""

def find_date_block(section, date_title):
    """在 section 内查找标题完全匹配 date_title 的三级标题区域。"""

def insert_date_block_in_desc_order(section, date_title, date):
    """
    按日期倒序插入新的日期小节：
    - 若已有日期小节：插入到第一个“日期 < 当前日期”的小节之前；
    - 若都比当前日期新：插入在末尾；
    - 若 section 为空：直接插入。
    """

def overwrite_bullets(date_block, bullets):
    """清空 date_block 下旧的 bullet 列表，用新的 bullets 替换。"""

def compress_text(text):
    """
    将原始文本压缩成 1-2 句：
    - 保留关键信息：发生了什么、涉及哪个子主题；
    - 去掉与项目无关的冗余表述（例如心情铺垫）；
    - 不重复记全段会议纪要，避免正文过长。
    """
```

---

## 稳健原则

| 原则     | 说明                                                      |
| ------ | ------------------------------------------------------- |
| 不改结构   | 不新增/删除任何 DB 字段，只改项目页面正文中 `## 📝 项目备忘录` 小节               |
| 幂等更新   | 同一项目 + 同一日期的小节，每次 `/daily` 都是全量覆盖该日期的 bullet            |
| 时间线清晰  | 日期小节按日期倒序排列，最新在上，方便快速看到最新进展                             |
| 尊重数据分离 | QYDB 项目只写 QYDB 页面；AIPO 项目只写 AIPO 页面，不做双向镜像              |
| 只读用户区域 | 绝不修改 Daily Note 的 `# Inbox / # Meetings / # Journal` 内容 |
| 信息适度压缩 | snippet 以“项目角度最重要的 1–2 句话”为限，避免复制整段原始笔记                 |
