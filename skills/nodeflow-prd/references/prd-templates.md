# PRD Templates

Use these templates only when drafting or writing a PRD. Keep `SKILL.md` focused on routing, gates, and execution rules.

## Human-Guided Draft

```markdown
## 产品定义草案

## For Human

### 核心结论

### 解决的问题

### 目标用户

### 项目范围

### 明确不做

### 需要确认

## For Agent

### 开发必读

#### 项目范围

#### 核心功能

#### 产品边界 / 暂不做什么

#### 需要确认

#### 不确定点

### 背景参考

#### 设计灵感

#### 痛点定义

#### 用户画像

#### 竞品拆解

#### 假设

## 建议下一步
```

## Final `docs/PRD.md`

```markdown
# PRD

## 产品目标

## 目标用户

## 核心场景

## 核心需求

## 非目标

## 数据边界

## 交互设计与页面流向 (Interaction Flow & App Surface)
- 主要页面/视图与核心控件结构
- 关键状态反馈（如 Loading / Empty / Error / Success / Submitting）
- 核心交互行为与跳转逻辑
- 高风险/敏感动作的二次确认关口（Confirmation Layers）

## 输出要求

## 成功标准

## 需要确认

## 不确定点

## 背景参考
```

Rules:

- Final `docs/PRD.md` is flat and development-oriented.
- Do not use `For Human / For Agent` in the final file.
- Keep `背景参考` last and short.
- Omit `背景参考` only if there is no useful background context.
- Put competitor-derived `可借鉴点` into `核心需求` or `成功标准` only when it fits the user's brief.
- Put competitor-derived `不跟随点` into `非目标`.
- Use `交互预期` for CLI commands, page flow, API input, agent instruction, file input/output, or another product-specific interface.
- Do not place implementation architecture, task breakdown, technology stack, or database design in `PRD.md`.

## One-Shot Input Template

```markdown
请确认或补充下面产品定义模板：

1. 设计灵感：
2. 痛点定义：
3. 用户画像：
4. 核心功能：
5. 产品边界 / 暂不做什么：
6. 竞品拆解：

确认后回复 `确认` 或 `1`，我会整理产品定义草案。
```

## Step-By-Step Module Draft

~~~markdown
## 当前模块：{模块名}

## 我对当前信息的理解

## 可补充的问题 / 方向

## 本模块草案

```markdown
### {模块名}

- ...
```

请确认这个模块是否满意。回复 `确认` 或 `1` 后进入下一个模块。
~~~
