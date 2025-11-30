# 团队工作区各数据库的表结构汇总

> 版本：v1.0
>
> 更新日期：2025年11月25日

------

## 📊 项目与任务管理

### 🌾 QYDB-Projects（项目库）[[1\]](https://www.notion.so/46bdce01bcd84d0a98f968272ec72979?pvs=21)

| 字段名                         | 类型     | 说明                                           |
| ------------------------------ | -------- | ---------------------------------------------- |
| Name                           | title    | 项目名称                                       |
| Status                         | select   | 状态：构思/规划/进行中/风险/暂停/完成/归档     |
| Priority                       | select   | 优先级：P0-P3                                  |
| Project Type                   | select   | 类型：内部建设/商务/运营活动/研发实验/产品迭代 |
| Owner                          | person   | 负责人                                         |
| Start                          | date     | 项目启动日期                                   |
| Due                            | date     | 项目截止日期                                   |
| Reach/Impact/Confidence/Effort | number   | RICE 评分指标                                  |
| RICE Score                     | formula  | RICE 综合评分（自动计算）                      |
| Tasks                          | relation | 关联到 QYDB-Tasks（双向）                      |
| Meetings                       | relation | 关联到 QYDB-Meetings（单向）                   |
| Content                        | relation | 关联到 QYDB-Content（单向）                    |
| Docs/Notes                     | relation | 关联到 QYDB-Notes（单向）                      |
| Related Product                | relation | 关联到 QYDB-Products（双向）                   |
| 被阻止/正在阻止                | relation | 项目阻塞关系（自关联）                         |

------

### ✅ QYDB-Tasks（任务库）[[2\]](https://www.notion.so/39b51e73f7a74050b22684a1019574db?pvs=21)

| 字段名      | 类型         | 说明                               |
| ----------- | ------------ | ---------------------------------- |
| Name        | title        | 任务名称                           |
| Status      | status       | 状态：待办/进行中/待评审/阻塞/完成 |
| Priority    | select       | 优先级：P0-P3                      |
| Owner       | person       | 负责人（限1人）                    |
| Reviewer    | person       | 审阅人（限1人）                    |
| Start       | date         | 开始日期                           |
| Due         | date         | 截止日期                           |
| CompletedAt | date         | 完成时间                           |
| Estimate(h) | number       | 预估工时                           |
| Actual(h)   | number       | 实际工时                           |
| Overdue     | formula      | 是否逾期（自动计算）               |
| Delay(d)    | formula      | 延迟天数（自动计算）               |
| Tags        | multi_select | 标签                               |
| Project     | relation     | 关联到 QYDB-Projects（双向）       |
| Meeting     | relation     | 关联到 QYDB-Meetings（双向）       |
| Parent Task | relation     | 父任务（自关联）                   |
| Subtasks    | relation     | 子任务（自关联）                   |
| Blocked By  | relation     | 被哪些任务阻塞（自关联）           |
| Blocks      | relation     | 阻塞哪些任务（自关联）             |

------

### 📅 QYDB-Meetings（会议库）[[3\]](https://www.notion.so/8b66e61e1ae0456e8dd0ee9aad60da7f?pvs=21)

| 字段名           | 类型     | 说明                                                         |
| ---------------- | -------- | ------------------------------------------------------------ |
| Title            | title    | 会议标题                                                     |
| Date/Time        | date     | 会议日期与时间                                               |
| Type             | select   | 类型：例会/评审/1:1/客户商务/复盘/其他                       |
| Status           | status   | 状态：未开始/进行中/完成                                     |
| Stage            | select   | 商务阶段：初识/需求探寻/方案沟通/PoC/报价谈判/法务合规/签约/维护 |
| Attendees        | person   | 参会人                                                       |
| Next Follow-up   | date     | 下次跟进日期                                                 |
| Recording/Link   | url      | 录音或相关链接                                               |
| Action Open      | rollup   | 未完成动作项计数                                             |
| Follow-up Status | formula  | 跟进状态（自动计算）                                         |
| Action Items     | relation | 关联到 QYDB-Tasks（双向）                                    |
| Project          | relation | 关联到 QYDB-Projects（单向）                                 |
| Company          | relation | 关联到 QYDB-Companies（双向）                                |
| Contacts         | relation | 关联到 QYDB-Contacts（双向）                                 |
| Deals            | relation | 关联到 QYDB-Deals（双向）                                    |

------

## 📄 文档与知识管理

### 📄 QYDB-Docs（文档库）[[4\]](https://www.notion.so/5847c7de49564859afc37ad9f2c5dbbf?pvs=21)

| 字段名                | 类型              | 说明                                                         |
| --------------------- | ----------------- | ------------------------------------------------------------ |
| Title                 | title             | 文档标题                                                     |
| Doc ID                | auto_increment_id | 文档ID（自动生成）                                           |
| Doc Type              | select            | 类型：Wiki/SOP/公司政策/Playbook/模版/术语表/决策记录/需求调研/用户画像/PRD/交互设计/技术方案/实验报告/市场调研/商务协议 |
| Status                | select            | 状态：草稿/评审中/已确认/废弃                                |
| Product Stage         | select            | 所属阶段：探索/立项/设计/开发/上线/复盘                      |
| Version               | text              | 版本号                                                       |
| Owner                 | person            | 文档负责人（限1人）                                          |
| Reviewer(s)           | person            | 评审人                                                       |
| Effective Date        | date              | 生效日期                                                     |
| Last Reviewed         | date              | 上次评审日期                                                 |
| Review Cycle (months) | select            | 评审周期：3/6/12个月                                         |
| Next Review           | formula           | 下次评审时间（自动计算）                                     |
| Stale                 | formula           | 是否过期（自动计算）                                         |
| Risk Level            | select            | 风险等级：低/中/高                                           |
| Compliance            | multi_select      | 合规类别：未成年人/隐私/医疗合规                             |
| Applies To            | multi_select      | 适用范围                                                     |
| Projects              | relation          | 关联到 QYDB-Projects（单向）                                 |
| Meetings              | relation          | 关联到 QYDB-Meetings（单向）                                 |
| Notes                 | relation          | 关联到 QYDB-Notes（单向）                                    |
| Sources               | relation          | 关联到 QYDB-Resources（单向）                                |
| Topics                | relation          | 关联到 QYDB-Topics（单向）                                   |
| Related Product       | relation          | 关联到 QYDB-Products（双向）                                 |
| Related Features      | relation          | 关联到 QYDB-Features（单向）                                 |
| Related Experiments   | relation          | 关联到 QYDB-Experiments（双向）                              |
| Related Company       | relation          | 关联到 QYDB-Companies（双向）                                |
| Parent Doc            | relation          | 父文档（自关联，限1个）                                      |
| Children              | relation          | 子文档（自关联）                                             |

------

### 📝 QYDB-Notes（笔记库）[[5\]](https://www.notion.so/6cbd72b0d58f40668d250c3323fbf54a?pvs=21)

| 字段名           | 类型     | 说明                                |
| ---------------- | -------- | ----------------------------------- |
| Title            | title    | 笔记标题                            |
| Quality          | select   | 质量：草稿/合格/可发表              |
| Claims           | text     | 要点/论断                           |
| Evidence         | text     | 证据/引用                           |
| Counterpoints    | text     | 反例/限制                           |
| Source           | relation | 关联到 QYDB-Resources（单向）       |
| Topic            | relation | 关联到 QYDB-Topics（单向）          |
| Project          | relation | 关联到 QYDB-Projects（限1个，单向） |
| Ideas to Content | relation | 可转化的内容 → QYDB-Content（单向） |

------

### 🏷️ QYDB-Topics（主题库）[[6\]](https://www.notion.so/9d6971b55de74014850d71ee99caa4f7?pvs=21)

| 字段名           | 类型     | 说明                          |
| ---------------- | -------- | ----------------------------- |
| Name             | title    | 主题名称                      |
| Thesis           | text     | 主张                          |
| Parent Topic     | relation | 父主题（自关联，限1个）       |
| Related Content  | relation | 关联到 QYDB-Content（单向）   |
| Related Notes    | relation | 关联到 QYDB-Notes（单向）     |
| Related Projects | relation | 关联到 QYDB-Projects（单向）  |
| Related Sources  | relation | 关联到 QYDB-Resources（单向） |

------

### 📚 QYDB-Resources（资源库）[[7\]](https://www.notion.so/bfbee969fe754112bb2025a1233aab2f?pvs=21)

| 字段名            | 类型     | 说明                                             |
| ----------------- | -------- | ------------------------------------------------ |
| Title             | title    | 资源标题                                         |
| Type              | select   | 类型：书/视频/播客/文章/论文/课程/网站/新闻/情报 |
| Status            | select   | 状态：待读/在读/已摘录/已消化/归档               |
| Authors/Publisher | text     | 作者/出版方                                      |
| Year              | number   | 年份                                             |
| Source Link       | url      | 资源链接                                         |
| AI 摘要           | text     | AI 生成的摘要                                    |
| Topics            | relation | 关联到 QYDB-Topics（单向）                       |
| Notes             | relation | 关联到 QYDB-Notes（单向）                        |
| Project           | relation | 关联到 QYDB-Projects（限1个，单向）              |
| Content           | relation | 关联到 QYDB-Content（单向）                      |

------

## 🎬 内容生产

### 🎬 QYDB-Content（内容库）[[8\]](https://www.notion.so/58f7ccf55fda4dd7817027318ab3f356?pvs=21)

| 字段名             | 类型         | 说明                                                         |
| ------------------ | ------------ | ------------------------------------------------------------ |
| Title              | title        | 内容标题                                                     |
| Type               | select       | 类型：课程/播客/文章/视频/演讲/其他                          |
| Stage              | status       | 阶段：Idea/Brief/Outline/Script/Draft/Review/Record/Edit/Schedule/Published |
| Channel            | multi_select | 发布渠道：官网/B站/小宇宙/微信/抖音/其他                     |
| Owner              | person       | 负责人（限1人）                                              |
| Editor/Producer    | person       | 编辑/制作人（限1人）                                         |
| Planned Publish    | date         | 计划发布日期                                                 |
| Published At       | date         | 实际发布日期                                                 |
| Delay(d)           | formula      | 延迟天数（自动计算）                                         |
| SEO Keywords       | text         | SEO关键词                                                    |
| Slug/URL           | url          | 发布链接                                                     |
| From Notes/Sources | relation     | 素材来源-笔记 → QYDB-Notes（单向）                           |
| From Sources       | relation     | 素材来源-资源 → QYDB-Resources（单向）                       |
| Project            | relation     | 关联到 QYDB-Projects（单向）                                 |
| Tasks              | relation     | 关联到 QYDB-Tasks（单向）                                    |

------

### 🎓 QYDB-Courses（课程库）[[9\]](https://www.notion.so/494d1f29694e4c6d9bdeae99b60f8654?pvs=21)

| 字段名   | 类型         | 说明                                    |
| -------- | ------------ | --------------------------------------- |
| Title    | title        | 课程标题                                |
| Level    | select       | 级别：入门/进阶/实战                    |
| Status   | select       | 状态：设计中/录制中/上线/维护/归档      |
| Audience | multi_select | 受众：青少年/家长/教师/创业者           |
| Lessons  | relation     | 课时 → QYDB-Lessons（单向）             |
| Content  | relation     | 关联内容 → QYDB-Content（单向）         |
| Project  | relation     | 关联项目 → QYDB-Projects（限1个，单向） |

------

### 📖 QYDB-Lessons（课时库）[[10\]](https://www.notion.so/7798c4c86ec94c50b2527f408c0a2f73?pvs=21)

| 字段名        | 类型     | 说明                                                  |
| ------------- | -------- | ----------------------------------------------------- |
| Title         | title    | 课时标题                                              |
| Stage         | status   | 阶段：Idea/Brief/Outline/Script/Record/Edit/Published |
| Duration(min) | number   | 时长（分钟）                                          |
| Course        | relation | 所属课程 → QYDB-Courses（限1个，单向）                |
| Topics        | relation | 关联主题 → QYDB-Topics（单向）                        |
| Notes/Sources | relation | 关联笔记和资源 → QYDB-Notes（单向）                   |
| Tasks         | relation | 关联任务 → QYDB-Tasks（限1个，单向）                  |

------

## 📦 产品研发

### 📦 QYDB-Products（产品库）[[11\]](https://www.notion.so/3c37a016a54d4ac995800ded7ba1f707?pvs=21)

| 字段名              | 类型         | 说明                                                    |
| ------------------- | ------------ | ------------------------------------------------------- |
| Name                | title        | 产品/方案名                                             |
| Product Type        | select       | 类型：数字工具/课程/心理服务/咨询方案/评估工具/运营系统 |
| Stage               | select       | 阶段：Idea/研究中-PRD-设计/MVP/Beta/已上线/已下线       |
| Lifecycle           | select       | 生命周期：0-1探索/1-10放大/10-100扩展                   |
| Target Users        | multi_select | 目标用户：K12/大学生/家长/学校/企业/平台/医疗研究机构   |
| Owner               | person       | 负责人                                                  |
| Core Problem        | text         | 关键痛点                                                |
| Key Metrics         | text         | 核心指标说明                                            |
| Related Projects    | relation     | 关联项目 → QYDB-Projects（双向）                        |
| Related Features    | relation     | 关联需求/功能 → QYDB-Features（双向）                   |
| Related Experiments | relation     | 关联实验 → QYDB-Experiments（双向）                     |
| 📄 DB-Docs           | relation     | 关联文档 → QYDB-Docs（双向）                            |

------

### ✨ QYDB-Features（需求/功能库）[[12\]](https://www.notion.so/17b637abdca2486ca23eab81ffe339d8?pvs=21)

| 字段名              | 类型     | 说明                                                         |
| ------------------- | -------- | ------------------------------------------------------------ |
| Name                | title    | 功能/需求名称                                                |
| Type                | select   | 类型：新功能/体验优化/Bug/技术债/研究问题/内容需求           |
| Status              | status   | 状态：Idea池/待评估/待规划/设计中/开发中/验收中/已上线/已取消 |
| Priority            | select   | 优先级：P0-P3                                                |
| Complexity          | select   | 复杂度：S/M/L                                                |
| Source              | select   | 来源：用户访谈/客户反馈/运营数据/内部想法/文献证据           |
| Business Value      | number   | 业务价值评分（1-5）                                          |
| User Scenario       | text     | 用户场景（User Story）                                       |
| Notes               | text     | 备注/风险                                                    |
| Product Owner       | person   | 产品负责人                                                   |
| Related Products    | relation | 关联产品 → QYDB-Products（双向）                             |
| Related Experiments | relation | 关联实验 → QYDB-Experiments（双向）                          |

------

### 🧪 QYDB-Experiments（实验库）[[13\]](https://www.notion.so/dfd87f59a8b9404b8b721dce65a63c8a?pvs=21)

| 字段名            | 类型     | 说明                                      |
| ----------------- | -------- | ----------------------------------------- |
| Name              | title    | 实验/试点名称                             |
| Type              | select   | 类型：产品实验/算法实验/教学研究/运营实验 |
| Status            | select   | 状态：设计中/进行中/分析中/完成/归档      |
| Conclusion        | select   | 结论：显著有效/暂无效果/结果不确定        |
| Next Action       | select   | 后续决策：放大/微调后再测/暂缓/放弃       |
| Core Hypothesis   | text     | 核心假设                                  |
| Experiment Design | text     | 实验设计简述                              |
| Key Metrics       | text     | 主要指标                                  |
| Notes             | text     | 备注                                      |
| Participants      | person   | 负责人/参与人                             |
| Related Products  | relation | 关联产品 → QYDB-Products（双向）          |
| Related Features  | relation | 关联功能 → QYDB-Features（双向）          |
| 📄 DB-Docs         | relation | 关联文档 → QYDB-Docs（双向）              |

------

## 🏢 CRM 客户关系管理

### 🏢 QYDB-Companies（公司/机构库）[[14\]](https://www.notion.so/135a0b2e84f2481b8d203cff3e5c0933?pvs=21)

| 字段名       | 类型         | 说明                                               |
| ------------ | ------------ | -------------------------------------------------- |
| Name         | title        | 公司/机构名称                                      |
| Type         | select       | 类型：医院-医疗机构/学校/平台/企业/政府            |
| Region       | select       | 地区：北京/上海/广州/深圳/杭州/成都/其他           |
| Relations    | multi_select | 关系类型：同业-竞对/投资人-股东/合作伙伴/客户/暂无 |
| 产品/品牌    | text         | 产品/品牌                                          |
| Owner        | person       | 负责人（限1人）                                    |
| Contacts     | relation     | 关联联系人 → QYDB-Contacts（双向）                 |
| Deals        | relation     | 关联商机 → QYDB-Deals（双向）                      |
| Meetings     | relation     | 关联会议 → QYDB-Meetings（双向）                   |
| Related Docs | relation     | 关联文档 → QYDB-Docs（双向）                       |

------

### 👤 QYDB-Contacts（联系人库）[[15\]](https://www.notion.so/a1f982f58ad245559b4977613f5272c4?pvs=21)

| 字段名      | 类型     | 说明                              |
| ----------- | -------- | --------------------------------- |
| Name        | title    | 联系人姓名                        |
| Title/Role  | text     | 职位/角色                         |
| Email/Phone | text     | 邮箱/电话                         |
| Influence   | select   | 影响力：1-5（5最高）              |
| Company     | relation | 关联公司 → QYDB-Companies（双向） |
| Deals       | relation | 关联商机 → QYDB-Deals（双向）     |
| Meetings    | relation | 关联会议 → QYDB-Meetings（双向）  |

------

### 💰 QYDB-Deals（商机库）[[16\]](https://www.notion.so/1fffcae08ad24511973ac74107a1b70d?pvs=21)

| 字段名              | 类型     | 说明                                                    |
| ------------------- | -------- | ------------------------------------------------------- |
| Title               | title    | 商机标题                                                |
| Stage               | status   | 阶段：线索/资格评估/PoC方案/商务谈判/法务合同/赢单/丢单 |
| Amount (¥)          | number   | 金额                                                    |
| Probability %       | number   | 成交概率                                                |
| Weighted (¥)        | formula  | 加权金额（自动计算）                                    |
| Stage Start         | date     | 入阶段日期                                              |
| Age in Stage (d)    | formula  | 在当前阶段的天数（自动计算）                            |
| Next Follow-up      | date     | 下次跟进日期                                            |
| Last Meeting At     | rollup   | 最近活动时间（汇总）                                    |
| Last Task Update At | rollup   | 最近任务更新时间（汇总）                                |
| Last Activity       | formula  | 最近活动（自动计算）                                    |
| Health              | formula  | 跟进健康状态（自动计算）                                |
| Owner               | person   | 负责人（限1人）                                         |
| Companies           | relation | 关联公司 → QYDB-Companies（双向）                       |
| Contacts            | relation | 关联联系人 → QYDB-Contacts（双向）                      |
| Meetings            | relation | 关联会议 → QYDB-Meetings（双向）                        |
| Tasks               | relation | 关联任务 → QYDB-Tasks（单向）                           |

------

## 🤖 其他

### 🤖 QYDB-Prompts（提示词库）[[17\]](https://www.notion.so/203f960d3c068074a675d4ba6b595284?pvs=21)

| 字段名   | 类型         | 说明                                                         |
| -------- | ------------ | ------------------------------------------------------------ |
| Name     | title        | 提示词名称                                                   |
| Category | select       | 类别：视频/图像/文本/编程                                    |
| Models   | multi_select | 模型：Sora2/Nano Banana/Gemini 2.5 Pro/通用/即梦/Midjourney v7/ChatGPT/Claude等 |
| Tags     | multi_select | 标签                                                         |
| Cover    | file         | 封面图                                                       |

------

### 🧭 QYDB-Pages（导航页库）[[18\]](https://www.notion.so/2b3f960d3c0680c6afa1fa403a38dcf2?pvs=21)

| 字段名 | 类型   | 说明              |
| ------ | ------ | ----------------- |
| 名称   | title  | 页面名称          |
| Type   | select | 类型：责任域/资源 |

------

## 🔗 关联关系总览

| 关联                                  | 类型   | 说明                                          |
| ------------------------------------- | ------ | --------------------------------------------- |
| Projects ↔ Tasks                      | 双向   | 项目包含多个任务                              |
| Projects ↔ Products                   | 双向   | 项目支撑产品                                  |
| Tasks ↔ Meetings                      | 双向   | 会议产出任务                                  |
| Tasks → Tasks                         | 自引用 | Parent/Subtasks/Blocked By 实现任务层级和依赖 |
| Projects → Projects                   | 自引用 | 被阻止/正在阻止 实现项目依赖                  |
| Meetings ↔ Companies/Contacts/Deals   | 双向   | CRM 会议跟进                                  |
| Companies ↔ Contacts ↔ Deals          | 双向   | CRM 三要素互联                                |
| Products ↔ Features ↔ Experiments     | 双向   | 产品-功能-实验闭环                            |
| Docs ↔ Products/Experiments/Companies | 双向   | 文档关联各业务实体                            |
| Topics → Notes/Resources/Content      | 单向   | 主题聚合知识资源                              |
| Courses → Lessons                     | 单向   | 课程包含课时                                  |