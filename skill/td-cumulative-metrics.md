---
name: "cumulative-metric-config"
description: "基于客户需求提供累计指标配置的完整指导方案，涵盖场景识别、实现路径推荐、SQL 代码生成和配置步骤说明"
---

# cumulative-metric-config
## 触发条件

当用户提及以下任何场景时触发：
- **意图动词**：「配置累计指标」「累计值怎么算」「累计指标设置」「累计数据统计」「累计计算」
- **场景关键词**：
  - 小时展示累计（hourly cumulative display）
  - 付费累计指标（cumulative payment metrics）
  - 留存模型阶段累计（stage cumulative in retention model）
  - 开服累计 N 天数据（server opening N-day cumulative）
  - 当月累计日均（monthly cumulative daily average）
  - 过去 N 日累计（past N-day cumulative）
- **问题类型**：
  - "如何统计累计充值金额"
  - "怎么看小时累计数据"
  - "开服后累计 7 天的付费人数怎么算"
  - "当月累计日均 DAU 如何配置"
  - "过去 30 天累计新增用户"

## 绝对不触发的情形

- **场景 1**：非累计类指标（如单日指标、环比同比、分布分析）
- **场景 2**：纯数据查询需求（无配置/计算逻辑需求）
- **场景 4**：累计指标的数据质量问题排查（应归数据质量 skill）

---

## 核心能力

1. **场景识别与分类**：准确识别用户需求属于 6 大累计场景中的哪一类
2. **实现路径推荐**：根据场景特点推荐最优实现方式（优先配置方案，必要时提供 SQL）
3. **配置步骤指导**：给出 ThinkingData 平台的详细配置步骤（模型、标签、虚拟属性）
4. **SQL 代码生成**（仅在配置方案无法满足时）：提供可直接使用的 SQL 查询代码
5. **最佳实践建议**：提供性能优化和常见陷阱提示

---

## 工作流程

### 第一步：场景识别

分析用户需求，识别属于以下哪种累计场景：

#### 场景 1：过去 N 日累计
**特征**：统计过去 N 天的累计值
**典型需求**：
- "过去 7 天累计新增用户"
- "过去 30 天累计活跃用户数"

#### 场景 2：按小时或分钟展示累计
**特征**：需要在小时或分钟粒度下展示累计值（如当日 0 点到当前小时的累计）
**典型需求**：
- "今天 0-10 点的累计充值金额"
- "当日截至当前小时的累计新增用户"

#### 场景 3：自某个关键时间节点后 N 日累计
**特征**：自某个关键时间节点开始，统计累计 N 天的累计值
**典型需求**：
- "新增用户累计充值金额"
- "开服累计 30 天的流水"

#### 场景 4：当月累计日均
**特征**：统计当月截至当前日期的日均值
**典型需求**：
- "当月累计日均 DAU"
- "当月累计日均充值金额"

#### 场景 5：留存模型阶段累计
**特征**：在留存分析中统计某阶段的累计行为次数
**典型需求**：
- "注册后 7 天内累计登录次数"
- "首次付费后 30 天内累计充值金额"

---

### 第二步：推荐实现路径

**优先原则**：始终优先尝试使用平台配置功能（模型、标签、虚拟属性等），仅在配置方案无法满足需求或用户反馈配置结果不合理时，才提供 SQL 方案。

根据场景特点，推荐以下实现方式之一或组合：

#### 路径 A：事件分析 + 累计配置（优先推荐）
**适用场景**：
- 场景 2（付费场景累计）
- 场景 6（过去 N 日累计）
- 大部分标准累计统计需求

**优势**：配置简单，无需写 SQL，易于维护和复用
**劣势**：灵活性受限于平台功能
**何时使用**：这是默认首选方案，适用于绝大多数累计指标需求

#### 路径 B：留存分析 + 阶段累计（优先推荐）
**适用场景**：
- 场景 3（留存模型阶段累计）
- 所有涉及用户生命周期阶段的累计行为分析

**优势**：专为留存场景设计，配置直观，无需 SQL
**劣势**：仅适用于留存分析场景
**何时使用**：当需求涉及"注册后 N 天"、"首次付费后 N 天"等阶段性累计时，优先使用此方案

#### 路径 C：用户标签 + 虚拟属性（优先推荐）
**适用场景**：
- 场景 2（付费场景累计）- 用户维度的累计值
- 需要在多个分析中复用的累计指标
- 需要实时查询的累计数据

**优势**：配置后可在所有分析模型中复用，支持实时查询
**劣势**：需要定期更新标签数据
**何时使用**：当累计指标需要频繁使用或跨多个分析场景时，优先考虑此方案

#### 路径 D：SQL 自定义查询（仅作为备选方案）
**适用场景**：
- 场景 1（小时展示累计）- 平台暂不支持小时粒度累计配置
- 场景 4（开服累计 N 天）- 需要动态计算开服日期
- 场景 5（当月累计日均）- 需要动态计算已过天数
- 用户尝试配置方案后反馈结果不符合预期
- 需求过于复杂或特殊，无法通过配置实现

**优势**：灵活性高，可实现任意复杂逻辑
**劣势**：需要 SQL 能力，维护成本较高，不易复用
**何时使用**：仅在以下两种情况下使用：
1. 用户需求无法匹配到合适的配置场景
2. 用户反馈根据配置方案无法得到正确结果

---

### 第三步：提供解决方案

根据识别的场景和推荐路径，提供以下内容：

#### 1. 方案概述
- 场景说明
- 推荐实现路径
- 预期效果

#### 2. SQL 代码（如适用）
```sql
-- 提供完整的 SQL 查询代码
-- 包含详细注释说明
-- 标注可配置参数
```

#### 3. 配置步骤（如适用）
1. 进入 ThinkingData 平台
2. 选择对应分析模型
3. 详细配置步骤（带截图说明位置）
4. 参数设置说明

#### 4. 最佳实践建议
- 性能优化建议
- 常见错误提示
- 数据准确性检查点

---

## 场景解决方案模板

### 场景 1：小时展示累计

**方案概述**：
使用 SQL 自定义查询，通过窗口函数实现小时粒度的累计计算。

**SQL 代码**：
```sql
-- 小时展示累计模板
SELECT
    date_format(#time, 'yyyy-MM-dd HH') as hour_time,
    -- 累计指标（使用窗口函数）
    SUM(指标字段) OVER (
        PARTITION BY date_format(#time, 'yyyy-MM-dd')
        ORDER BY date_format(#time, 'yyyy-MM-dd HH')
        ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW
    ) as cumulative_value
FROM
    事件表
WHERE
    #time >= '起始日期'
    AND #time < '结束日期'
GROUP BY
    date_format(#time, 'yyyy-MM-dd HH')
ORDER BY
    hour_time
```

**配置步骤**：
1. 进入「事件分析」→「SQL 查询」
2. 粘贴上述 SQL 代码
3. 替换以下参数：
   - `指标字段`：要累计的指标（如充值金额、用户数）
   - `事件表`：数据来源表
   - `起始日期`、`结束日期`：查询时间范围
4. 点击「运行查询」验证结果
5. 保存为「自定义查询」供后续使用

**最佳实践**：
- 建议按天分区查询，避免跨天累计导致的数据错误
- 对于大数据量场景，考虑增加时间范围限制
- 验证首小时数据是否正确（应等于该小时的实际值）

---

### 场景 2：付费场景累计

**方案概述**：
使用事件分析的「累计值」功能，或通过用户属性存储累计值。

**方法 A：事件分析配置**
1. 进入「事件分析」
2. 选择付费事件（如 `charge_success`）
3. 指标选择「事件属性总和」→ 选择金额字段
4. 分组维度选择「用户 ID」
5. 在「高级设置」中勾选「显示累计值」
6. 时间范围选择「全部时间」或指定范围

**方法 B：用户属性方案**（推荐用于实时查询）
```sql
-- 计算用户累计付费金额并更新用户属性
SELECT
    #account_id as user_id,
    SUM(charge_amount) as total_charge_amount
FROM
    charge_success
WHERE
    #time >= '起始日期'
GROUP BY
    #account_id
```
然后通过数据导入功能更新用户属性 `total_charge_amount`。

**最佳实践**：
- 方法 A 适合临时分析，方法 B 适合需要实时查询的场景
- 定期校验累计值与实际充值记录是否一致
- 注意退款场景的处理逻辑

---

### 场景 3：留存模型阶段累计

**方案概述**：
使用留存分析的「回访行为」功能，统计阶段内的累计次数。

**配置步骤**：
1. 进入「留存分析」
2. 设置「初始行为」（如注册、首次付费）
3. 设置「回访行为」（如登录、充值）
4. 在「回访行为」设置中：
   - 选择「累计次数」或「累计金额」
   - 设置统计窗口（如 7 天、30 天）
5. 选择留存周期（如次日、7 日、30 日）
6. 运行分析查看结果

**SQL 实现**（高级场景）：
```sql
-- 留存阶段累计模板
WITH first_event AS (
    -- 获取初始事件时间
    SELECT
        #account_id,
        MIN(#time) as first_time
    FROM
        初始事件表
    GROUP BY
        #account_id
),
stage_cumulative AS (
    -- 计算阶段内累计值
    SELECT
        a.#account_id,
        COUNT(b.event_id) as cumulative_count,
        SUM(b.amount) as cumulative_amount
    FROM
        first_event a
    LEFT JOIN
        回访事件表 b
    ON
        a.#account_id = b.#account_id
        AND b.#time >= a.first_time
        AND b.#time < date_add(a.first_time, N)  -- N 为阶段天数
    GROUP BY
        a.#account_id
)
SELECT * FROM stage_cumulative
```

**最佳实践**：
- 明确定义「阶段」的起止时间（含当天还是不含）
- 注意时区问题，确保时间计算准确
- 对于大用户量场景，考虑分批计算

---

### 场景 4：开服累计 N 天

**方案概述**：
通过 SQL 查询，以服务器开服日期为基准，计算累计 N 天的数据。

**SQL 代码**：
```sql
-- 开服累计 N 天模板
WITH server_open_date AS (
    -- 定义开服日期（可从配置表或固定值获取）
    SELECT '2024-01-01' as open_date
),
cumulative_data AS (
    SELECT
        date_format(#time, 'yyyy-MM-dd') as date,
        COUNT(DISTINCT #account_id) as daily_users,
        SUM(charge_amount) as daily_revenue
    FROM
        事件表
    WHERE
        #time >= (SELECT open_date FROM server_open_date)
        AND #time < date_add((SELECT open_date FROM server_open_date), N)  -- N 为累计天数
    GROUP BY
        date_format(#time, 'yyyy-MM-dd')
)
SELECT
    date,
    daily_users,
    daily_revenue,
    -- 累计指标
    SUM(daily_users) OVER (ORDER BY date ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW) as cumulative_users,
    SUM(daily_revenue) OVER (ORDER BY date ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW) as cumulative_revenue
FROM
    cumulative_data
ORDER BY
    date
```

**配置步骤**：
1. 确认服务器开服日期（从运营配置或数据库获取）
2. 进入「SQL 查询」
3. 替换以下参数：
   - `open_date`：开服日期
   - `N`：累计天数（如 7、30）
   - `事件表`：数据来源
   - 指标字段：根据需求调整
4. 运行查询并验证结果
5. 可保存为「看板」供日常监控

**最佳实践**：
- 开服日期应从权威数据源获取，避免硬编码
- 注意跨服场景，需按服务器 ID 分组计算
- 验证累计天数的计算逻辑（是否含开服当天）

---

### 场景 5：当月累计日均

**方案概述**：
计算当月截至当前日期的日均值，需要动态计算已过天数。

**SQL 代码**：
```sql
-- 当月累计日均模板
WITH monthly_data AS (
    SELECT
        date_format(#time, 'yyyy-MM-dd') as date,
        COUNT(DISTINCT #account_id) as daily_value
    FROM
        事件表
    WHERE
        date_format(#time, 'yyyy-MM') = date_format(CURRENT_DATE, 'yyyy-MM')  -- 当月
    GROUP BY
        date_format(#time, 'yyyy-MM-dd')
),
cumulative_avg AS (
    SELECT
        date,
        daily_value,
        -- 当月累计总和
        SUM(daily_value) OVER (ORDER BY date ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW) as cumulative_sum,
        -- 当月已过天数
        ROW_NUMBER() OVER (ORDER BY date) as days_passed,
        -- 当月累计日均
        SUM(daily_value) OVER (ORDER BY date ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW) /
        ROW_NUMBER() OVER (ORDER BY date) as cumulative_daily_avg
    FROM
        monthly_data
)
SELECT * FROM cumulative_avg
ORDER BY date
```

**配置步骤**：
1. 进入「SQL 查询」
2. 粘贴上述 SQL 代码
3. 替换参数：
   - `事件表`：数据来源
   - `daily_value`：要计算日均的指标
4. 运行查询验证结果
5. 可设置为「定时查询」每日自动更新

**最佳实践**：
- 确认「当月」的定义（自然月还是业务月）
- 注意月初数据的准确性（第 1 天日均 = 当天值）
- 对于跨月查询，需要按月分组计算

---

### 场景 6：过去 N 日累计

**方案概述**：
统计过去 N 天（含今天）的累计值，可通过事件分析或 SQL 实现。

**方法 A：事件分析配置**
1. 进入「事件分析」
2. 选择目标事件
3. 时间范围选择「过去 N 天」
4. 指标选择「总和」或「去重数」
5. 查看累计结果

**方法 B：SQL 查询**（更灵活）
```sql
-- 过去 N 日累计模板
SELECT
    date_format(#time, 'yyyy-MM-dd') as date,
    COUNT(DISTINCT #account_id) as daily_users,
    -- 过去 N 日累计（滚动窗口）
    COUNT(DISTINCT #account_id) OVER (
        ORDER BY date_format(#time, 'yyyy-MM-dd')
        ROWS BETWEEN N-1 PRECEDING AND CURRENT ROW
    ) as rolling_N_day_users
FROM
    事件表
WHERE
    #time >= date_sub(CURRENT_DATE, N-1)  -- 过去 N 天
GROUP BY
    date_format(#time, 'yyyy-MM-dd')
ORDER BY
    date
```

**配置步骤**：
1. 确定 N 的值（如 7、30）
2. 选择实现方法（配置或 SQL）
3. 如使用 SQL：
   - 进入「SQL 查询」
   - 替换 `N` 和相关参数
   - 运行查询
4. 验证结果（最后一天的累计值应为过去 N 天总和）

**最佳实践**：
- 明确「过去 N 天」是否包含今天
- 对于去重指标（如活跃用户），注意窗口函数的使用
- 滚动窗口适合趋势分析，固定窗口适合周期对比

---

## 通用最佳实践

### 性能优化
1. **时间范围限制**：避免查询过长时间范围，建议单次查询不超过 90 天
2. **分区字段利用**：WHERE 条件中优先使用分区字段（如 `#time`）
3. **索引优化**：对于高频查询字段，建议添加索引
4. **结果缓存**：对于固定时间范围的累计查询，可使用结果缓存

### 数据准确性
1. **时区一致性**：确保所有时间字段使用统一时区
2. **去重逻辑**：明确去重维度（用户、设备、订单等）
3. **边界条件**：验证起始日期、结束日期的包含关系
4. **数据延迟**：考虑数据上报延迟，建议查询时排除最近 1-2 小时

### 常见错误
1. **窗口函数误用**：PARTITION BY 和 ORDER BY 的字段选择错误
2. **累计重置**：未按正确维度分组导致累计值异常重置
3. **时间粒度不匹配**：小时、天、月粒度混用导致计算错误
4. **NULL 值处理**：未处理 NULL 值导致累计计算中断

---

## 输出格式

每次响应应包含以下结构：

### 1. 场景识别
- 识别的场景类型
- 关键特征说明

### 2. 推荐方案
- 实现路径（SQL / 配置 / 组合）
- 方案优劣势对比

### 3. 详细方案
- SQL 代码（如适用）
- 配置步骤（如适用）
- 参数说明

### 4. 最佳实践
- 性能优化建议
- 数据准确性检查
- 常见错误提示

### 5. 验证方法
- 结果验证步骤
- 预期输出示例

---

## 示例对话

**用户**：如何统计开服后累计 7 天的付费人数？

**助手**：

### 场景识别
您的需求属于**场景 4：开服累计 N 天**，需要以服务器开服日期为基准，统计累计 7 天的付费人数。

### 推荐方案
**实现路径**：SQL 自定义查询
**原因**：需要动态计算开服日期，并按天累计，SQL 方式最灵活。

### 详细方案

**SQL 代码**：
```sql
-- 开服累计 7 天付费人数
WITH server_open_date AS (
    SELECT '2024-01-01' as open_date  -- 替换为实际开服日期
),
daily_payers AS (
    SELECT
        date_format(#time, 'yyyy-MM-dd') as date,
        COUNT(DISTINCT #account_id) as daily_payer_count
    FROM
        charge_success  -- 付费成功事件
    WHERE
        #time >= (SELECT open_date FROM server_open_date)
        AND #time < date_add((SELECT open_date FROM server_open_date), 7)
    GROUP BY
        date_format(#time, 'yyyy-MM-dd')
)
SELECT
    date,
    daily_payer_count,
    SUM(daily_payer_count) OVER (
        ORDER BY date
        ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW
    ) as cumulative_payer_count
FROM
    daily_payers
ORDER BY
    date
```

**配置步骤**：
1. 确认服务器开服日期（从运营配置获取）
2. 进入 ThinkingData「事件分析」→「SQL 查询」
3. 将上述 SQL 粘贴到查询框
4. 替换 `'2024-01-01'` 为实际开服日期
5. 点击「运行查询」
6. 查看结果表，最后一行的 `cumulative_payer_count` 即为累计 7 天付费人数

### 最佳实践
- **开服日期来源**：建议从服务器配置表读取，避免硬编码
- **跨服场景**：如有多个服务器，需在 WHERE 条件中增加 `server_id` 过滤
- **去重逻辑**：`COUNT(DISTINCT #account_id)` 确保同一用户多次付费只计数一次
- **验证方法**：第 1 天的累计值应等于当天付费人数

### 验证方法
1. 检查第 1 天数据：`cumulative_payer_count` 应等于 `daily_payer_count`
2. 检查第 7 天数据：应为 7 天内所有去重付费用户总数
3. 对比原始数据：手动统计 7 天内付费用户数，与查询结果对比

---

## 注意事项

1. **平台版本差异**：不同版本的 ThinkingData 功能可能有差异，以实际平台为准
2. **权限要求**：SQL 查询功能需要相应权限，如无权限请联系管理员
3. **数据安全**：涉及敏感数据时，注意脱敏处理
4. **持续优化**：根据实际使用情况，持续优化 SQL 性能和配置方案

---

## 相关资源

- ThinkingData 官方文档：[累计指标配置指南]
- SQL 窗口函数教程：[OVER 子句详解]
- 最佳实践案例库：[累计指标典型案例]

---

**版本**：v1.0
**更新日期**：2025-01-XX
**维护者**：ThinkingData 数据分析团队
