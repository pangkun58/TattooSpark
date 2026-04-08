# 纹身 AI 设计生成网站 — MVP 产品需求文档

**文档版本：** v1.2  
**日期：** 2026-04-08  
**项目代号：** InkGen  
**预算上限：** ¥1000 人民币  

> **v1.1 更新说明：** 将「用户账号系统 / 登录注册」和「付费 / 充值功能」从 Out of Scope 提升为 MVP 核心功能。  
> **v1.2 更新说明：** 登录方式新增 Google OAuth（面向国际用户），移除手机号/验证码方式；支付从微信/支付宝切换为 PayPal（面向海外用户）；Prompt 处理新增英文输入检测，英文描述跳过翻译直接使用。

---

## 一、项目背景与目标

### 1.1 背景

纹身行业痛点：用户想要定制纹身图案，但：
- 找纹身师沟通成本高，效果不可预期
- 网上找的参考图往往"纹不出来"（过于复杂、细节太多、线条太细）
- 通用 AI 图（Midjourney 等）生成的纹身图不符合实际纹身工艺要求

### 1.2 MVP 目标

上线一个可用的 Web 应用，实现：

> **用户登录（Google OAuth / 邮箱） → 输入文字描述 + 选择纹身风格 → AI 生成"能纹得出来"的纹身设计图 → 下载使用**  
> **免费额度用完后，通过 PayPal 充值继续生成**

### 1.3 成功指标（MVP 阶段）

| 指标 | 目标值 |
|------|--------|
| 上线时间 | 4 周内 |
| 单张生图成本 | ≤ ¥0.10 |
| 首批用户生图数 | 100 张 |
| 注册用户数 | ≥ 50 |
| 付费转化率 | ≥ 10% |
| 用户满意度（自评） | ≥ 4/5 |

---

## 二、目标用户

### 主要用户

- **纹身爱好者**：有纹身意向，想先看到效果再跟纹身师沟通（以海外用户为主）
- **纹身师**：需要快速出草稿/参考图给客户确认

### 用户核心诉求

1. 快速看到"接近真实纹身效果"的图
2. 图能直接给纹身师使用，无需大改
3. 操作简单，不需要懂 AI prompt

---

## 三、功能需求

### 3.1 核心功能（Must Have）

#### F1：纹身风格选择

用户从以下预设风格中选择一种：

| 风格 | 英文名 | 说明 |
|------|--------|------|
| 线条风 | Linework / Fine Line | 单针细线，无填充，极简 |
| 黑灰风 | Black & Grey | 黑白渐变，写实感强 |
| 传统美式 | Old School / Traditional | 粗线条，色块填充，水手风 |
| 日式 | Japanese | 波浪、鲤鱼、樱花等传统元素 |
| 几何 | Geometric | 点线面构成，对称图形 |
| 新传统 | Neo-Traditional | 传统基础上加细节装饰 |

#### F2：文字描述输入

- 支持中文和英文输入
- **语言自动检测**：
  - 若输入为**英文**：直接使用，跳过翻译，降低延迟和成本
  - 若输入为**中文**：调用翻译 API 转为英文 prompt
  - 判断逻辑：检测文本中 ASCII 字母占比，> 80% 视为英文
- 字数限制：≤ 200 字符（英文）/ ≤ 100 汉字（中文）
- 提供双语示例引导：
  - "一只展翅的老鹰，霸气" / "A fierce eagle with wings spread"
  - "玫瑰花与骷髅组合，暗黑风" / "Rose and skull combination, dark style"
  - "极简的山脉轮廓线" / "Minimalist mountain ridge outline"

#### F3：身体部位选择（影响构图比例）

| 部位 | 构图比例 |
|------|----------|
| 手臂（上臂/前臂） | 竖向长方形 |
| 小腿 | 竖向长方形 |
| 背部 | 正方形/横向 |
| 胸口 | 横向 |
| 手腕/脚踝 | 环形/细长 |
| 不限 | 默认正方形 |

#### F4：AI 生图（核心）

- **API：** Replicate
- **推荐模型：**
  - 主力：`black-forest-labs/flux-schnell`（速度快，约 $0.003/张 ≈ ¥0.02）
  - 备用：`stability-ai/sdxl`（效果更可控）
- **输出规格：**
  - 分辨率：1024×1024（可选 512×768 竖版）
  - 格式：PNG，白色背景
  - 高对比度，线条清晰

#### F5：图片下载

- 提供原始 PNG 下载
- 提供**黑白线稿版**（灰度处理 + 对比度增强，方便纹身师转印）
- 文件命名格式：`inkgen_[风格]_[时间戳].png`

---

#### F6：用户账号系统 / 登录注册 🆕

**功能说明：**

用户需登录后方可使用生图功能（访客可浏览首页和风格示例）。主要面向海外用户，采用 OAuth 和邮箱两种方式，无需手机号。

**登录 / 注册方式：**

| 方式 | 说明 | 优先级 |
|------|------|--------|
| **Google OAuth** | 一键登录，国际用户首选，无需填写任何信息 | 主推 ⭐ |
| **邮箱 + 密码** | 备选方式，覆盖无 Google 账号用户 | 备选 |

> **注：** 手机号/验证码方式不做，微信登录放二期。

**用户信息：**

| 字段 | 说明 |
|------|------|
| user_id | 系统唯一 ID（UUID） |
| email | 登录凭证（来自 Google OAuth 或手动填写） |
| name | 显示名称（Google 授权自动获取） |
| avatar_url | 头像（Google 授权自动获取） |
| oauth_provider | 登录来源（google / email） |
| credits | 生图积分余额，初始 30 |
| created_at | 注册时间 |
| total_generations | 累计生图数 |

**权限控制：**

| 用户类型 | 每日免费生图次数 | 历史记录 | 高清下载 |
|----------|----------------|----------|----------|
| 未登录 | 0（需登录） | ❌ | ❌ |
| 免费用户 | 3 次 | ✅（最近 20 张） | ❌ |
| 付费用户 | 不限（按积分扣） | ✅（全部） | ✅ |

**安全要求：**
- NextAuth.js 管理 OAuth 会话，JWT Token 有效期 7 天（支持刷新）
- 邮箱密码用 bcrypt 加密存储
- HTTPS 全程加密

---

#### F7：付费 / 充值功能 🆕

**商业模式：** 免费体验 + 积分制付费

**积分规则：**

| 操作 | 积分变动 |
|------|---------|
| 新用户首次登录 | +30 积分（可生成 3 张） |
| 每次生图（标准） | -10 积分 |
| 每次生图（高清 / 变体） | -20 积分 |
| 充值（见下方套餐） | 按套餐增加 |

**充值套餐（USD 定价，面向国际用户）：**

| 套餐 | 价格 | 积分 | 单张成本 | 备注 |
|------|------|------|----------|------|
| Starter | $1.99 | 100 积分 | ≈$0.20/张 | 首充推荐 |
| Standard | $4.99 | 300 积分 | ≈$0.17/张 | 最受欢迎 |
| Pro | $14.99 | 1200 积分 | ≈$0.12/张 | 高频用户 |
| Artist | $39.99 | 4000 积分 | ≈$0.10/张 | 纹身师专用 |

**支付方式：**

| 方式 | 说明 |
|------|------|
| **PayPal** | 主推，国际主流，支持信用卡 + PayPal 余额 |

> **注：** 微信支付、支付宝不做。PayPal 支持个人开发者快速接入，无需营业执照。

**PayPal 接入方案：**
- 使用 **PayPal Checkout SDK**（前端）+ **PayPal Orders API**（后端）
- 支持 PayPal 余额付款和信用卡直付（Debit/Credit Card）
- 支付成功后 webhook 回调更新积分（幂等处理防重复）
- 沙盒环境测试完成后切换生产环境

**前端页面：**
- 个人中心 → 积分余额 + 充值入口
- 充值页面 → 套餐选择 → PayPal 支付按钮 → 支付成功提示
- 消费明细（最近 20 条记录）

---

### 3.2 增强功能（Should Have，有时间再做）

#### F8："能纹性"提示

生成后自动分析图片，给出简单提示：

- ✅ **可以纹** — 线条清晰，对比度好，细节适中
- ⚠️ **建议调整** — 部分区域细节过密，建议放大尺寸
- ❌ **不建议纹** — 线条过细或细节过多，实际纹身效果差

实现方式：调用 GPT-4o Vision API 做简单判断，或规则引擎（图像对比度/边缘密度检测）

#### F9：变体生成

- 用户对某张图满意后，点击"生成 4 个变体"
- 基于同一 prompt + seed 微调生成

---

### 3.3 不做（Out of Scope，MVP 阶段）

- ❌ 手机号 / 短信验证码登录
- ❌ 微信一键登录（二期实现）
- ❌ 微信支付 / 支付宝
- ❌ 纹身师入驻/对接
- ❌ 社区展示墙
- ❌ 图片编辑功能
- ❌ 移动 App

---

## 四、技术方案

### 4.1 技术栈

| 层级 | 技术选型 | 理由 |
|------|----------|------|
| 前端框架 | Next.js 14（App Router） | 全栈，部署简单 |
| UI 样式 | Tailwind CSS + shadcn/ui | 开发快，风格统一 |
| AI 生图 | Replicate API | 按量付费，无月费 |
| 语言检测 | 前端/后端正则检测 | 英文跳过翻译，中文调用翻译 API |
| 中文翻译 | 百度翻译 API 免费版 | 免费 50万字/月，仅中文输入时调用 |
| 部署 | Vercel 免费套餐 | 免费，全球 CDN |
| 图片存储 | Cloudflare R2 | 免费 10GB |
| 数据库 | PostgreSQL（Supabase 免费版） | 存用户/订单/积分数据 |
| 用户认证 | NextAuth.js | 支持 Google OAuth + 邮箱密码 |
| 支付 | PayPal Checkout SDK + Orders API | 国际主流，个人开发者可快速接入 |

### 4.2 数据库表设计

#### users 表

```sql
CREATE TABLE users (
  id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  email           VARCHAR(255) UNIQUE NOT NULL,
  name            VARCHAR(100),
  avatar_url      TEXT,
  password        VARCHAR(255),          -- bcrypt hash，OAuth 登录可为空
  oauth_provider  VARCHAR(20),           -- 'google' | 'email'
  oauth_id        VARCHAR(255),          -- Google sub ID
  credits         INTEGER DEFAULT 30,    -- 初始赠送 30 积分
  total_generations INTEGER DEFAULT 0,
  created_at      TIMESTAMP DEFAULT NOW(),
  updated_at      TIMESTAMP DEFAULT NOW()
);
```

#### generations 表

```sql
CREATE TABLE generations (
  id            UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id       UUID REFERENCES users(id),
  style         VARCHAR(50),
  input_lang    VARCHAR(10),             -- 'en' | 'zh'
  prompt_input  TEXT,                    -- 用户原始输入
  prompt_en     TEXT,                    -- 最终英文 prompt
  image_url     TEXT,
  credits_used  INTEGER DEFAULT 10,
  created_at    TIMESTAMP DEFAULT NOW()
);
```

#### orders 表

```sql
CREATE TABLE orders (
  id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id         UUID REFERENCES users(id),
  amount_usd      DECIMAL(10,2),         -- 支付金额（USD）
  credits         INTEGER,               -- 到账积分
  status          VARCHAR(20) DEFAULT 'pending',  -- pending/paid/failed
  pay_method      VARCHAR(20) DEFAULT 'paypal',
  paypal_order_id VARCHAR(100) UNIQUE,   -- PayPal 订单 ID（幂等键）
  paid_at         TIMESTAMP,
  created_at      TIMESTAMP DEFAULT NOW()
);
```

### 4.3 Prompt 处理逻辑

```
用户输入描述
    ↓
语言检测（ASCII字母占比 > 80%？）
    ├── YES（英文）→ 直接使用原文作为 prompt_en，跳过翻译
    └── NO（中文/混合）→ 调用百度翻译 API → 得到 prompt_en
    ↓
组合：[风格基础词] + [prompt_en] + [纹身约束词] + [构图词]
    ↓
调用 Replicate API 生图
```

### 4.4 核心 Prompt 模板

每种风格对应专属 prompt 结构：

```
[风格基础词] + [用户描述(英文)] + [纹身约束词] + [构图词(按部位)]

纹身约束词（固定）：
tattoo design, on white background, high contrast, 
bold clean lines, no photorealistic skin texture, 
stencil-ready, professional tattoo flash art style

线条风追加：
fine line, single needle, minimalist, no shading, ultra thin lines

黑灰风追加：
black and grey realism, smooth gradient shading, no color

Old School 追加：
traditional american tattoo, bold black outlines, 
flat color fills, sailor jerry style, limited palette

日式追加：
japanese irezumi style, flowing composition, 
bold outlines, traditional japanese motifs

几何追加：
geometric tattoo, sacred geometry, precise lines, 
symmetrical, dotwork elements

新传统追加：
neo traditional tattoo, decorative flourishes, 
jewel tones, detailed but wearable
```

### 4.5 系统主流程

```
用户访问首页
    ↓
点击"Get Started" → Google OAuth 一键登录 / 邮箱登录
    ↓
检查积分余额（< 10 则提示充值）
    ↓
用户填写表单（风格 + 描述 + 部位）
    ↓
前端发送请求到 Next.js API Route（携带 JWT）
    ↓
验证用户身份 + 积分是否充足
    ↓
语言检测 → 英文直接用 / 中文调用百度翻译
    ↓
组合 Prompt → 调用 Replicate API 生图（flux-schnell）
    ↓
生图成功 → 扣除积分 + 保存记录到数据库
    ↓
返回图片 URL → 前端显示
    ↓
用户预览 → 下载原图 / 线稿版
```

### 4.6 PayPal 支付流程

```
用户选择充值套餐
    ↓
后端创建 PayPal Order（返回 order_id）
    ↓
前端渲染 PayPal 支付按钮（PayPal JS SDK）
    ↓
用户点击按钮 → 弹出 PayPal 支付窗口
    ↓
用户完成支付（PayPal 余额 / 信用卡）
    ↓
PayPal Webhook 回调后端（验签）
    ↓
幂等检查（paypal_order_id 是否已处理）
    ↓
更新订单状态 paid + 原子增加用户积分
    ↓
前端轮询确认，刷新积分余额 + 成功提示
```

### 4.7 API 密钥管理

所有密钥通过 Vercel 环境变量管理，不硬编码：

```env
# AI 生图
REPLICATE_API_TOKEN=xxx

# 翻译（仅中文输入时使用）
BAIDU_TRANSLATE_APPID=xxx
BAIDU_TRANSLATE_SECRET=xxx

# 数据库
DATABASE_URL=xxx                          # Supabase PostgreSQL

# 用户认证
NEXTAUTH_SECRET=xxx
NEXTAUTH_URL=https://yourdomain.com
GOOGLE_CLIENT_ID=xxx                      # Google OAuth
GOOGLE_CLIENT_SECRET=xxx

# 支付
PAYPAL_CLIENT_ID=xxx
PAYPAL_CLIENT_SECRET=xxx
PAYPAL_WEBHOOK_ID=xxx
```

---

## 五、页面设计

### 5.1 页面结构

```
首页（未登录可见）
├── Header：Logo + "Sign in with Google" 按钮
├── Hero：产品介绍 + CTA "Try for Free"
└── 风格展示（6种风格示例图）

登录页
├── "Continue with Google"（主按钮，带 Google 图标）
├── 分割线 "or"
├── 邮箱 + 密码登录/注册（折叠表单）
└── 新用户提示：Free 30 credits on signup

主功能页（登录后）
├── Header：Logo + 积分余额 + 用户头像（Google 头像）
├── 左侧：输入区
│   ├── 风格选择（图标卡片，6选1）
│   ├── 文字描述输入框（支持中/英文）
│   ├── 身体部位选择（下拉）
│   └── "Generate" 按钮（显示消耗积分数）
│
└── 右侧：结果区
    ├── 生成中状态（骨架屏 + 进度提示）
    ├── 结果图展示
    ├── "能纹性" 提示标签
    ├── Download PNG / Download Stencil 按钮
    └── Generate Variants 按钮（预留）

个人中心页（Account）
├── 头像 + 名称 + 邮箱（来自 Google）
├── Credits 余额 + "Top Up" 按钮
├── 消费明细（最近 20 条）
└── 历史生图记录

充值页（Top Up Credits）
├── 套餐选择（4档，USD 定价）
├── PayPal 支付按钮（PayPal JS SDK 渲染）
└── 支付成功/失败提示
```

### 5.2 视觉风格

- 整体色调：深色背景（#0f0f0f）+ 白色文字 + 金色/红色点缀
- 字体：标题用衬线字体，正文用无衬线字体
- 风格参考：暗黑、艺术、专业纹身工作室感
- UI 语言：英文为主（国际用户导向）

---

## 六、预算规划

| 项目 | 费用 | 备注 |
|------|------|------|
| 域名（.com） | $10-15/年 | 建议 tattoospark.com 或 inkgen.io |
| Replicate API（测试期） | $30-50 | 约 2000-3000 张生图量 |
| 百度翻译 API | ¥0 | 免费额度够用，仅中文输入时调用 |
| Vercel 托管 | $0 | 免费套餐 |
| Cloudflare R2 存储 | $0 | 免费 10GB |
| Supabase 数据库 | $0 | 免费套餐（500MB） |
| Google OAuth | $0 | 免费，需在 Google Console 创建应用 |
| PayPal 手续费 | 交易额 3.49% + $0.49 | 按成交收取，无月费 |
| UI 素材/图标 | $0 | 用免费资源（Lucide、Unsplash） |
| **合计（启动成本）** | **≈ $40-65** | **PayPal 手续费按实际交易收取** |

---

## 七、开发计划

### Week 1：基础框架 + 生图核心

- [ ] 初始化 Next.js 项目，部署到 Vercel
- [ ] 完成首页 UI（风格选择 + 输入表单）
- [ ] 接入 Replicate API（flux-schnell 模型）
- [ ] 实现语言检测逻辑（英文直接用，中文调百度翻译）
- [ ] 实现 Prompt 拼装逻辑（6种风格）
- [ ] 图片展示 + 原图下载

### Week 2：用户系统

- [ ] 接入 Supabase PostgreSQL，建表（users / generations / orders）
- [ ] 配置 Google OAuth（Google Cloud Console）
- [ ] NextAuth.js 接入（Google + 邮箱密码）
- [ ] JWT 鉴权中间件
- [ ] 个人中心页面（积分余额 + 历史记录）
- [ ] 积分扣减逻辑（生图前检查 + 生图后扣除）

### Week 3：支付系统 + 体验优化

- [ ] 注册 PayPal 开发者账号，创建应用（获取 Client ID/Secret）
- [ ] 接入 PayPal Checkout SDK + Orders API
- [ ] 充值套餐页面 + PayPal 按钮渲染
- [ ] PayPal Webhook 回调处理 + 积分到账（幂等）
- [ ] 消费明细页面
- [ ] 黑白线稿版下载（Canvas 处理）
- [ ] 移动端响应式适配
- [ ] 错误处理（API 超时、积分不足、支付失败）

### Week 4：测试 + 上线

- [ ] "能纹性"简单判断逻辑
- [ ] 端到端测试（登录→生图→充值→再生图）
- [ ] PayPal 沙盒测试 → 切换生产环境
- [ ] 性能优化（图片懒加载、CDN 缓存）
- [ ] SEO 基础配置（title、description、OG 图）
- [ ] 正式域名绑定，上线发布

---

## 八、风险与应对

| 风险 | 概率 | 应对方案 |
|------|------|----------|
| Replicate API 访问慢 | 中 | Vercel 部署在海外，由服务端调用，用户无感知 |
| 生成图质量不符合纹身要求 | 中 | 持续优化 Prompt，增加 negative prompt |
| Google OAuth 配置错误 | 低 | 提前在沙盒环境测试；配置 Authorized redirect URIs |
| PayPal Webhook 延迟/丢失 | 低 | 前端主动轮询支付状态作为补充；设置重试机制 |
| 积分重复到账（回调重放） | 低 | 幂等处理（paypal_order_id UNIQUE 索引） |
| 百度翻译结果偏差（中文输入） | 低 | 允许用户直接输入英文绕过翻译 |
| 预算超支 | 低 | 限制免费用户每日生成次数（每账号每天 3 张） |

---

## 九、后续迭代方向（Post-MVP）

1. **微信一键登录** — 覆盖国内用户
2. **Stripe 支付** — 更低手续费，更好的开发体验
3. **纹身师社区** — 入驻纹身师，用户可直接预约
4. **展示墙 + SEO** — UGC 内容沉淀，带来自然流量
5. **尺寸模拟** — 将设计图叠加到手臂/小腿照片上预览
6. **会员订阅制** — 月付/年付，适合高频用户

---

*文档结束 | v1.2 更新于 2026-04-08*
