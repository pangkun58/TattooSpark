# 纹身 AI 设计生成网站 — MVP 产品需求文档

**文档版本：** v1.1  
**日期：** 2026-04-08  
**项目代号：** InkGen  
**预算上限：** ¥1000 人民币  

> **v1.1 更新说明：** 将「用户账号系统 / 登录注册」和「付费 / 充值功能」从 Out of Scope 提升为 MVP 核心功能，同步更新技术方案、预算、开发计划。

---

## 一、项目背景与目标

### 1.1 背景

纹身行业痛点：用户想要定制纹身图案，但：
- 找纹身师沟通成本高，效果不可预期
- 网上找的参考图往往"纹不出来"（过于复杂、细节太多、线条太细）
- 通用 AI 图（Midjourney 等）生成的纹身图不符合实际纹身工艺要求

### 1.2 MVP 目标

上线一个可用的 Web 应用，实现：

> **用户注册登录 → 输入文字描述 + 选择纹身风格 → AI 生成"能纹得出来"的纹身设计图 → 下载使用**  
> **免费额度用完后，通过充值继续生成**

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

- **纹身爱好者**：有纹身意向，想先看到效果再跟纹身师沟通
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

- 支持中文输入（后端自动翻译为英文 prompt）
- 字数限制：≤ 100 字
- 提供示例引导文字，如：
  - "一只展翅的老鹰，霸气"
  - "玫瑰花与骷髅组合，暗黑风"
  - "极简的山脉轮廓线"

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

用户需注册账号后方可使用生图功能（访客可浏览首页和风格示例）。

**注册 / 登录方式：**

| 方式 | 说明 |
|------|------|
| 手机号 + 验证码 | 国内主流，低门槛 |
| 微信一键登录 | （二期，MVP 可不做） |
| 邮箱 + 密码 | 备选，供非国内用户使用 |

**用户信息：**

| 字段 | 说明 |
|------|------|
| user_id | 系统唯一 ID |
| 手机号 / 邮箱 | 登录凭证 |
| 昵称 | 可选，默认"用户XXXX" |
| 注册时间 | 系统记录 |
| 积分余额 | 生图消耗积分，充值可增加 |
| 累计生图数 | 用于数据统计 |

**权限控制：**

| 用户类型 | 每日免费生图次数 | 历史记录 | 高清下载 |
|----------|----------------|----------|----------|
| 未登录 | 0（需注册） | ❌ | ❌ |
| 免费用户 | 3 次 | ✅（最近20张） | ❌ |
| 付费用户 | 不限（按积分扣） | ✅（全部） | ✅ |

**安全要求：**
- 验证码有效期 5 分钟，同一手机号每分钟限发 1 次
- JWT Token 鉴权，有效期 7 天（支持刷新）
- 密码加密存储（bcrypt）

---

#### F7：付费 / 充值功能 🆕

**商业模式：** 免费体验 + 积分制付费

**积分规则：**

| 操作 | 积分变动 |
|------|---------|
| 新用户注册 | +30 积分（可生成 3 张） |
| 每次生图（标准） | -10 积分 |
| 每次生图（高清 / 变体） | -20 积分 |
| 充值（见下方套餐） | 按套餐增加 |

**充值套餐：**

| 套餐 | 价格 | 积分 | 单张成本 | 备注 |
|------|------|------|----------|------|
| 体验包 | ¥9.9 | 100 积分 | ≈¥0.99/张 | 首充推荐 |
| 标准包 | ¥29.9 | 400 积分 | ≈¥0.75/张 | 最受欢迎 |
| 专业包 | ¥99 | 1500 积分 | ≈¥0.66/张 | 高频用户 |
| 纹身师包 | ¥299 | 5000 积分 | ≈¥0.60/张 | B 端用户 |

**支付方式：**

| 方式 | 说明 |
|------|------|
| 微信支付 | 主推，国内用户首选 |
| 支付宝 | 备选 |

**实现方案：**
- 接入**微信支付商户号**（需营业执照，或使用第三方聚合支付如 Ping++/收钱吧）
- 支付回调更新用户积分余额（原子操作，防止重复到账）
- 订单记录存储（order_id、金额、积分、支付时间、状态）

**前端页面：**
- 个人中心 → 积分余额 + 充值入口
- 充值页面 → 套餐选择 → 微信/支付宝二维码 → 支付成功提示
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

- ❌ 微信一键登录（二期实现）
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
| 中文翻译 | 百度翻译 API 免费版 | 免费 50万字/月 |
| 部署 | Vercel 免费套餐 | 免费，全球 CDN |
| 图片存储 | Cloudflare R2 | 免费 10GB |
| 数据库 | PostgreSQL（Supabase 免费版） | 存用户/订单/积分数据 |
| 用户认证 | NextAuth.js + JWT | 支持手机号/邮箱登录 |
| 短信验证码 | 阿里云短信服务 | 约 ¥0.045/条，稳定 |
| 支付 | 微信支付 / Ping++ 聚合支付 | 快速接入，支持个人开发者 |

### 4.2 数据库表设计

#### users 表

```sql
CREATE TABLE users (
  id          UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  phone       VARCHAR(20) UNIQUE,
  email       VARCHAR(255) UNIQUE,
  nickname    VARCHAR(50),
  password    VARCHAR(255),          -- bcrypt hash，手机号登录可为空
  credits     INTEGER DEFAULT 30,   -- 初始赠送30积分
  created_at  TIMESTAMP DEFAULT NOW(),
  updated_at  TIMESTAMP DEFAULT NOW()
);
```

#### generations 表

```sql
CREATE TABLE generations (
  id          UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id     UUID REFERENCES users(id),
  style       VARCHAR(50),
  prompt_cn   TEXT,
  prompt_en   TEXT,
  image_url   TEXT,
  credits_used INTEGER DEFAULT 10,
  created_at  TIMESTAMP DEFAULT NOW()
);
```

#### orders 表

```sql
CREATE TABLE orders (
  id           UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id      UUID REFERENCES users(id),
  amount       DECIMAL(10,2),        -- 支付金额（元）
  credits      INTEGER,              -- 到账积分
  status       VARCHAR(20) DEFAULT 'pending',  -- pending/paid/failed
  pay_method   VARCHAR(20),          -- wechat/alipay
  pay_order_id VARCHAR(100),         -- 第三方支付单号
  paid_at      TIMESTAMP,
  created_at   TIMESTAMP DEFAULT NOW()
);
```

### 4.3 核心 Prompt 模板

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

### 4.4 系统流程

```
用户注册/登录
    ↓
检查积分余额（< 10 则提示充值）
    ↓
用户填写表单（风格 + 描述 + 部位）
    ↓
前端发送请求到 Next.js API Route（携带 JWT）
    ↓
验证用户身份 + 积分是否充足
    ↓
百度翻译 API 将中文描述翻译为英文
    ↓
组合风格 Prompt 模板 + 翻译结果 + 约束词
    ↓
调用 Replicate API 生图（flux-schnell）
    ↓
生图成功 → 扣除积分 + 保存记录到数据库
    ↓
返回图片 URL → 前端显示
    ↓
用户预览 → 下载原图 / 线稿版
```

### 4.5 支付流程

```
用户选择充值套餐
    ↓
后端创建订单（status: pending）
    ↓
调用微信支付 API 生成二维码
    ↓
前端展示二维码（轮询支付状态）
    ↓
用户扫码支付
    ↓
微信回调通知后端（验签）
    ↓
更新订单状态 + 原子增加用户积分
    ↓
前端收到成功通知，刷新积分余额
```

### 4.6 API 密钥管理

所有密钥通过 Vercel 环境变量管理，不硬编码：

```env
REPLICATE_API_TOKEN=xxx
BAIDU_TRANSLATE_APPID=xxx
BAIDU_TRANSLATE_SECRET=xxx
DATABASE_URL=xxx                    # Supabase PostgreSQL
NEXTAUTH_SECRET=xxx
NEXTAUTH_URL=https://yourdomain.com
ALIYUN_SMS_ACCESS_KEY=xxx
ALIYUN_SMS_SECRET=xxx
WECHAT_PAY_MCH_ID=xxx
WECHAT_PAY_API_KEY=xxx
```

---

## 五、页面设计

### 5.1 页面结构

```
首页（未登录可见）
├── Header：Logo + 登录/注册按钮
├── Hero：产品介绍 + CTA "立即体验"
└── 风格展示（6种风格示例图）

登录/注册页
├── 手机号 + 验证码登录
├── 邮箱 + 密码登录（切换）
└── 新用户注册送 30 积分提示

主功能页（登录后）
├── Header：Logo + 积分余额 + 用户头像
├── 左侧：输入区
│   ├── 风格选择（图标卡片，6选1）
│   ├── 文字描述输入框
│   ├── 身体部位选择（下拉）
│   └── "生成设计图" 按钮（显示消耗积分数）
│
└── 右侧：结果区
    ├── 生成中状态（骨架屏 + 进度提示）
    ├── 结果图展示
    ├── "能纹性" 提示标签
    ├── 下载原图 / 下载线稿 按钮
    └── 生成变体 按钮（预留）

个人中心页
├── 基本信息（昵称、手机号）
├── 积分余额 + 充值按钮
├── 消费明细（最近 20 条）
└── 历史生图记录

充值页
├── 套餐选择（4档）
├── 支付方式（微信/支付宝）
├── 二维码展示
└── 支付成功/失败提示
```

### 5.2 视觉风格

- 整体色调：深色背景（#0f0f0f）+ 白色文字 + 金色/红色点缀
- 字体：标题用衬线字体，正文用无衬线字体
- 风格参考：暗黑、艺术、专业纹身工作室感

---

## 六、预算规划

| 项目 | 费用 | 备注 |
|------|------|------|
| 域名（.cn 或 .com） | ¥60-150/年 | 建议买 tattoospark.cn 或类似 |
| Replicate API（测试期） | ¥200-300 | 约 2000-3000 张生图量 |
| 百度翻译 API | ¥0 | 免费额度够用 |
| Vercel 托管 | ¥0 | 免费套餐 |
| Cloudflare R2 存储 | ¥0 | 免费 10GB |
| Supabase 数据库 | ¥0 | 免费套餐（500MB） |
| 阿里云短信（测试期） | ¥50 | 约 1000 条验证码 |
| 微信支付开通费 | ¥0 | 企业主体免费；个人可用 Ping++ |
| UI 素材/图标 | ¥0 | 用免费资源（Lucide、Unsplash） |
| **合计** | **≈ ¥310-500** | **剩余预算可用于推广或应急** |

---

## 七、开发计划

### Week 1：基础框架 + 生图核心

- [ ] 初始化 Next.js 项目，部署到 Vercel
- [ ] 完成首页 UI（风格选择 + 输入表单）
- [ ] 接入 Replicate API（flux-schnell 模型）
- [ ] 接入百度翻译 API
- [ ] 实现 Prompt 拼装逻辑（6种风格）
- [ ] 图片展示 + 原图下载

### Week 2：用户系统

- [ ] 接入 Supabase PostgreSQL，建表（users / generations / orders）
- [ ] 实现手机号 + 验证码注册/登录（阿里云短信）
- [ ] 实现邮箱 + 密码注册/登录
- [ ] JWT 鉴权中间件
- [ ] 个人中心页面（积分余额 + 历史记录）
- [ ] 积分扣减逻辑（生图前检查 + 生图后扣除）

### Week 3：支付系统 + 体验优化

- [ ] 接入微信支付（或 Ping++ 聚合支付）
- [ ] 充值套餐页面 + 二维码展示
- [ ] 支付回调处理 + 积分到账
- [ ] 消费明细页面
- [ ] 黑白线稿版下载（Canvas 处理）
- [ ] 移动端响应式适配
- [ ] 错误处理（API 超时、积分不足、支付失败）

### Week 4：测试 + 上线

- [ ] "能纹性"简单判断逻辑
- [ ] localStorage 历史记录（已登录用户用数据库）
- [ ] 端到端测试（注册→生图→充值→再生图）
- [ ] 性能优化（图片懒加载、CDN 缓存）
- [ ] SEO 基础配置（title、description、OG 图）
- [ ] 正式域名绑定，上线发布

---

## 八、风险与应对

| 风险 | 概率 | 应对方案 |
|------|------|----------|
| Replicate API 在中国访问慢 | 中 | Vercel 部署在海外，由服务端调用，用户无感知 |
| 生成图质量不符合纹身要求 | 中 | 持续优化 Prompt，增加 negative prompt |
| 微信支付审核慢/无营业执照 | 中 | 使用 Ping++ 或收钱吧（支持个人开发者快速接入） |
| 短信验证码被刷（薅羊毛） | 中 | 图形验证码前置 + 同 IP 频率限制 |
| 积分重复到账（支付回调重放） | 低 | 幂等处理（pay_order_id 唯一索引） |
| 预算超支 | 低 | 限制免费用户每日生成次数（每 IP 每天 5 张） |
| 百度翻译结果偏差 | 低 | 提供英文直接输入选项作为备选 |

---

## 九、后续迭代方向（Post-MVP）

1. **微信一键登录** — 降低注册门槛
2. **纹身师社区** — 入驻纹身师，用户可直接预约
3. **展示墙 + SEO** — UGC 内容沉淀，带来自然流量
4. **尺寸模拟** — 将设计图叠加到手臂/小腿照片上预览
5. **会员订阅制** — 月付/年付，适合高频用户

---

*文档结束 | v1.1 更新于 2026-04-08*
