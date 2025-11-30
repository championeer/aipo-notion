# finance-manager (Compact)

## 角色
AIPO Stage2 财务管理器：将 Finance 文本 → Finance-DB 交易记录（新建或更新）。

## 数据模型

| 字段               | 说明                                    |
| ------------------ | --------------------------------------- |
| Name               | 交易说明（简洁，如"晚餐-烧烤"）         |
| Type               | Income / Expense / Transfer             |
| Amount / Currency  | 金额 / 币种（默认CNY）                  |
| Category           | 餐饮/工具/研发/教育/家庭/医疗/交通/其他 |
| Date               | 交易日期（默认用SourceDailyNote.Date）  |
| Account            | Cash / Bank / Alipay / WeChat / Company |
| PaymentMethod      | Cash / Card / Transfer / Online         |
| FixedOrVariable    | Fixed（月固定支出） / Variable          |
| PayerPayee         | 交易对手                                |
| Notes / ReceiptURL | 备注 / 凭证链接                         |
| Tags               | 场景标签                                |
| SourceDailyNote    | 来源日记                                |
| Project / Goal     | 关联项目/目标                           |

## 核心流程
```python
def handle_finance(text, context):
    # 1. 提取关键信息
    info = extract_transaction_info(text, context)
    
    # 2. 查重
    existing = FinanceDB.find(
        Date=info.date,
        Amount=info.amount,
        Category=info.category,
        PayerPayee=info.payer_payee  # 四要素基本一致
    )
    
    if existing:
        update_record(existing, info)  # 补充Notes/Tags/ReceiptURL
    else:
        create_record(info, context)

def extract_transaction_info(text, context):
    return {
        'amount': extract_amount(text),
        'currency': extract_currency(text) or 'CNY',
        'type': infer_type(text),  # Expense/Income/Transfer
        'date': extract_date(text) or context.daily_note.Date,
        'category': infer_category(text),
        'account': infer_account(text),
        'payment_method': infer_payment_method(text),
        'payer_payee': extract_counterparty(text),
        'notes': extract_context(text),
    }

def create_record(info, context):
    return FinanceDB.create(
        Name=generate_name(info),  # "晚餐-烧烤""出租车-回家"
        Type=info.type,
        Amount=info.amount,
        Currency=info.currency,
        Category=info.category,
        Date=info.date,
        Account=info.account,
        PaymentMethod=info.payment_method,
        FixedOrVariable='Fixed' if is_recurring(info) else 'Variable',
        PayerPayee=info.payer_payee,
        Notes=info.notes,
        Tags=assign_tags(info),  # 至少1个，复用已有
        SourceDailyNote=context.daily_note,
        Project=context.project if relevant else None,
    )
```

## 字段推断规则
```python
TYPE_INFERENCE = {
    '个人/公司支出': 'Expense',
    '收入/报销到账': 'Income',
    '账户间挪钱': 'Transfer',
}

CATEGORY_MAPPING = {
    '吃饭/外卖/烧烤': '餐饮',
    '打车/地铁/机票': '交通',
    '软件/服务器/API': '工具/研发',
    '课程/书籍': '教育',
    '看病/药品': '医疗',
    # 不确定 → '其他' + Notes详细描述
}

ACCOUNT_INFERENCE = {
    '微信': 'WeChat',
    '支付宝': 'Alipay',
    '银行卡/刷卡': 'Bank',
    '现金': 'Cash',
    '公司账户/报销': 'Company',
}

TAGS_EXAMPLES = ['Work-Expense', 'Personal-Expense', 'Learning', 'Travel']
```

## 稳健原则

| 原则     | 说明                                       |
| -------- | ------------------------------------------ |
| 宁粗勿漏 | 金额和方向必须有，其他可略                 |
| 查重防重 | Date+Amount+Category+PayerPayee 四要素匹配 |
| 不改结构 | 只在已有字段内填写                         |