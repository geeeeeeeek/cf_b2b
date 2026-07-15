# B2B 产品展示网站

[English](README.md) | [简体中文](readme-cn.md)

基于 Cloudflare Workers 构建的专业 B2B 产品展示网站，采用 D1 数据库、R2 存储和基于角色的管理系统。

## 功能特性

### ✅ 前台网站
1. **首页** - 企业简介和精选产品展示
2. **产品页** - 产品列表，支持分类筛选和搜索
3. **产品详情页** - 产品详细信息和询盘表单
4. **关于我们页** - 公司介绍、发展历程和资质证书
5. **联系我们页** - 联系信息和在线询盘表单
6. **响应式设计** - 自适应移动端、平板和桌面设备

### ✅ 后台管理系统
1. **基于角色的访问控制**
   - **超级管理员** - 完整的增删改查权限
   - **普通管理员** - 只读权限（仅查看）
2. **管理后台首页** - 统计数据概览和数据洞察
3. **产品管理** - 产品列表、编辑、删除和创建
4. **图片上传** - 支持上传图片到 Cloudflare R2
5. **询盘管理** - 查看、处理和删除客户询盘
6. **网站设置** - 配置网站信息、联系方式和社交媒体链接（仅超级管理员）

### ✅ 后端 API
1. **产品 API** - 完整的 CRUD 操作（GET/POST/PUT/DELETE）
2. **询盘 API** - 提交询盘、查询和状态更新
3. **管理员 API** - 登录认证、Token 验证、角色管理
4. **图片上传 API** - 上传图片到 R2 存储
5. **设置 API** - 通过 KV 存储管理网站配置

### ✅ 数据库设计
- **Products 表** - 产品信息存储
- **Inquiries 表** - 客户询盘记录
- **Admins 表** - 管理员账户及角色权限

## 技术栈

- **后端**: Cloudflare Workers
- **数据库**: Cloudflare D1 (SQLite)
- **存储**: Cloudflare R2 (图片)、Cloudflare KV (设置)
- **前端**: HTML5、CSS3、JavaScript (原生)
- **身份验证**: JWT tokens 配合 SHA-256 密码哈希
- **权限管理**: 基于角色的访问控制（RBAC）

## 准备域名

Namesilo域名申请入口（Affiliate）：

https://www.namesilo.com/?rid=0ac5797ay

折扣码（Tim2026）

## 部署步骤

### 1. 安装依赖

```bash
npm install
```

### 2. 创建 D1 数据库

```bash
# 创建数据库
wrangler d1 create b2b_database

# 记录输出的 database_id，并更新到 wrangler.toml 文件中
```

### 3. 更新配置

编辑 `wrangler.toml`，将 `database_id` 替换为步骤 2 中创建的实际 ID：

```toml
[[d1_databases]]
binding = "DB"
database_name = "b2b_database"
database_id = "your-database-id"  # 替换这里
```

### 4. 创建 R2 存储桶（用于图片上传）

```bash
# 创建 R2 bucket
wrangler r2 bucket create b2b-product-images
```

确认 `wrangler.toml` 中已配置 R2：

```toml
[[r2_buckets]]
binding = "IMAGES"
bucket_name = "b2b-product-images"
```

### 5. 创建 KV 命名空间（用于网站设置）

```bash
# 为生产环境创建 KV 命名空间
wrangler kv namespace create "STATIC_ASSETS"

# 为开发环境创建 KV 命名空间（可选）
wrangler kv namespace create "STATIC_ASSETS" --preview
```

更新 `wrangler.toml` 中的 KV 命名空间 ID：

```toml
[[kv_namespaces]]
binding = "STATIC_ASSETS"
id = "your-kv-namespace-id"  # 替换这里
preview_id = "your-preview-kv-namespace-id"  # 可选，用于本地开发
```

### 6. 初始化数据库

```bash
# 执行数据库 schema
wrangler d1 execute b2b_database --file=./schema/schema.sql
```

这将会创建：
- 数据库表（products、inquiries、admins）
- 两个默认管理员账户：
  - **超级管理员**：用户名 `admin123`，密码 `admin123`
  - **普通管理员**：用户名 `staff`，密码 `staff123`
- 用于测试的示例产品数据

### 7. 本地开发

```bash
npm run dev
```

访问 http://localhost:8787 测试网站功能。

**管理员登录**：访问 http://localhost:8787/admin 进行登录。

### 8. 部署到 Cloudflare

```bash
npm run deploy
```



## 管理员账户管理

### 默认凭据

初始设置后，您将拥有两个管理员账户：

| 用户名 | 密码 | 角色 | 权限 |
|--------|------|------|------|
| admin123 | admin123 | 超级管理员 | 完整的增删改查权限 |
| staff | staff123 | 普通管理员 | 只读权限 |

### 安全最佳实践

🔐 **重要提示**：首次部署后请立即更改默认密码！

#### 1. 生成新的密码哈希

在浏览器控制台中使用以下代码生成新的密码哈希：

```javascript
async function hashPassword(password) {
  const encoder = new TextEncoder();
  const data = encoder.encode(password);
  const hashBuffer = await crypto.subtle.digest('SHA-256', data);
  const hashArray = Array.from(new Uint8Array(hashBuffer));
  return hashArray.map(b => b.toString(16).padStart(2, '0')).join('');
}

// 为您的新密码生成哈希
hashPassword('your-new-secure-password').then(hash => console.log(hash));
```

#### 2. 在数据库中更新密码

```sql
-- 更新管理员密码
UPDATE admins
SET password_hash = 'your-generated-hash'
WHERE username = 'admin';
```

通过 Wrangler CLI 执行：
```bash
wrangler d1 execute b2b_database --command="UPDATE admins SET password_hash = 'your-hash' WHERE username = 'admin';"
```

或通过 Cloudflare Dashboard 控制台执行。

#### 3. 配置 JWT Secret

**方式 A**：使用 wrangler.toml（生产环境不推荐）
```toml
[vars]
JWT_SECRET = "your-very-secure-secret-key-change-this"
ENVIRONMENT = "production"
```

**方式 B**：使用 Cloudflare Dashboard Secrets（推荐）
```bash
wrangler secret put JWT_SECRET
# 提示时输入您的密钥
```

## 使用指南

### 前台功能

1. **浏览产品**
   - 访问首页查看精选产品
   - 进入产品页面浏览所有产品
   - 使用分类筛选和搜索查找特定产品

2. **提交产品询盘**
   - 在产品详情页点击"发送询盘"
   - 填写并提交询盘表单
   - 或通过联系我们页面提交通用询盘

### 后台管理

1. **登录**
   - 访问 `/admin` 或 `/admin/login`
   - 使用您的管理员凭据登录

2. **管理产品**（仅超级管理员）
   - 查看所有产品
   - 编辑产品信息
   - 上传产品图片（支持 JPEG、PNG、GIF、WebP，最大 5MB）
   - 删除不需要的产品
   - 添加新产品

3. **管理询盘**
   - 查看所有客户询盘（所有管理员）
   - 查看询盘详情（所有管理员）
   - 更新询盘状态（所有管理员）
   - 删除已处理的询盘（仅超级管理员）

4. **配置设置**（仅超级管理员）
   - 更新网站名称和描述
   - 编辑公司介绍
   - 修改联系信息
   - 设置社交媒体链接

## 项目结构

```
cf_b2b/
├── src/
│   ├── index.js              # Workers 入口文件
│   ├── api/
│   │   ├── router.js         # API 路由
│   │   └── handlers/         # API 处理器
│   │       ├── products.js   # 产品 API
│   │       ├── inquiries.js  # 询盘 API
│   │       ├── admin.js      # 管理员 API
│   │       ├── upload.js     # 图片上传 API
│   │       └── settings.js   # 设置 API
│   ├── pages/
│   │   ├── router.js         # 页面路由
│   │   ├── layout.js         # 页面布局模板
│   │   ├── home.js           # 首页
│   │   ├── products.js       # 产品列表页
│   │   ├── product-detail.js # 产品详情页
│   │   ├── about.js          # 关于我们页
│   │   ├── contact.js        # 联系我们页
│   │   ├── admin-login.js    # 管理员登录页
│   │   ├── admin-dashboard.js# 管理后台
│   │   └── static.js         # 静态资源处理器
│   └── utils/
│       └── auth.js           # 认证工具
├── public/
│   ├── css/
│   │   └── main.css          # 主样式表
│   └── js/
│       ├── main.js           # 主 JS 文件
│       └── admin.js          # 管理后台 JS
├── schema/
│   ├── schema.sql            # 数据库结构
│   ├── init.sql              # 初始化脚本
├── package.json
├── wrangler.toml             # Cloudflare Workers 配置
├── README.md
└── DEPLOY.md
```

## 故障排除

### 常见问题

1. **数据库连接错误**
   - 验证 wrangler.toml 中的 `database_id` 与您的 D1 数据库匹配
   - 确保数据库 schema 已初始化

2. **图片上传失败**
   - 确认 R2 bucket 存在：`wrangler r2 bucket list`
   - 检查 wrangler.toml 中的 R2 绑定名称是否为 "IMAGES"

3. **设置无法保存**
   - 确保 KV 命名空间已创建并绑定
   - 验证您是以超级管理员身份登录
   - 检查浏览器控制台是否有错误

4. **登录问题**
   - 验证密码哈希是否正确
   - 检查 JWT_SECRET 是否已配置
   - 清除浏览器 localStorage 后重试

5. **权限被拒绝错误**
   - 确认您的管理员角色（super_admin vs admin）
   - 只有超级管理员可以创建/更新/删除
   - 普通管理员只有只读权限

## 后续优化建议

1. 📧 **邮件通知** - 为新询盘发送提醒
2. 🔍 **增强搜索** - 全文搜索功能
3. 📊 **分析仪表板** - 更详细的统计数据和图表
4. 🌐 **多语言支持** - 面向全球受众的国际化
5. 💾 **数据导出** - 导出数据到 Excel/CSV
6. 🔔 **实时通知** - 基于 WebSocket 的通知系统
7. 🖼️ **图片优化** - 自动调整大小和压缩图片
8. 📱 **移动端管理应用** - 原生移动端管理界面

## 技术支持

遇到问题时请检查：
1. 检查 Wrangler CLI 版本：`wrangler --version`
2. 验证 D1 数据库初始化
3. 检查 wrangler.toml 配置
4. 查看浏览器控制台错误信息
5. 查看 Cloudflare Workers 日志

## 许可证

MIT
