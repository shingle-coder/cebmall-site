# CEBMall B2B 询盘站

> 面向海外采购商的英文 B2B 询盘站：无线鼠标与 AC/DC 电源适配器，43 种语言、构建期静态生成、SEO / GEO 就绪。

CEBMall 是 Sanxing Plastics (Dongguan) Co., Ltd 的独立站项目。站点负责产品展示、规格核对、OEM/ODM 咨询与报价询盘，不做在线支付、购物车、订单履约与会员体系，也不在页面上承诺价格、交期、质保与认证结果。

- 公开站点：<https://cebmall.com>
- 管理后台：`https://admin.cebmall.com`
- 后端 API：`https://api.cebmall.com`
- 媒体 CDN：`https://media.cebmall.com`

---

## 一、项目做什么

| 能力 | 说明 |
| ---- | ---- |
| 产品目录 | 4 个分类（无线鼠标、有线鼠标、电源适配器、线材配件），当前 32 款已发布产品 |
| 产品详情 | 型号、规格表、卖点、认证、FAQ、图片/视频媒体与规格书下载 |
| 询盘闭环 | 前台表单 → API 校验与防刷 → 后台跟进；提交后立即返回固定模板的 AI 受控回执，不含报价与承诺 |
| 内容中心 | Resources 文章、分类页与 Markdown 镜像，供搜索引擎与 AI 抓取 |
| 多语言 | 43 个语种独立 URL 与文案，按语种生成站点快照 |
| 管理后台 | 产品、分类、文章、认证、询盘、友情链接、客服知识库与语种文案管理 |
| AI 客服 | 基于知识库的问答与 SSE 流式响应，答案受业务边界约束 |
| SEO / GEO | 预渲染、canonical、hreflang、结构化数据、Markdown 镜像、llms.txt、分语种 sitemap、社交分享图 |

---

## 二、技术架构

### 2.1 仓库形态

pnpm workspace 单仓多包，依赖方向固定为 `apps/* → packages/*`，`packages/*` 不允许反向依赖 `apps/*`，也不允许循环依赖。

```text
cebmall-site/
├── apps/
│   ├── api/                 # NestJS + Prisma + MySQL，端口 3000
│   ├── web/                 # 公开站点，Vue 3 + Vite + vite-ssg，端口 3010
│   └── admin/               # 管理后台，Vue 3 + Vite SPA，端口 3020
├── packages/
│   ├── shared/              # 品牌、语言、产品、分类、认证、询盘类型与纯工具
│   ├── api-client/          # 基于 fetch 的公开接口与后台接口客户端
│   ├── ui/                  # 无 TDesign 依赖的共享导出与设计令牌
│   ├── config/              # ESLint、Prettier、Tailwind、TypeScript 预设
│   └── i18n/                # 43 种语言注册、语言包与已发布语种定义
├── docs/                    # 方案、Spec、实施计划与诊断报告
├── scripts/                 # 快照构建、发布、校验与内容处理脚本
├── prototypes/              # 静态交互原型
└── .cnb.yml                 # CNB 流水线配置
```

### 2.2 应用与包

| 目录 | 包名 | 端口 | 说明 |
| ---- | ---- | ---- | ---- |
| `apps/api` | `@cebmall/api` | 3000 | NestJS，全局前缀 `/api`，公开接口 `/public`，后台接口 `/admin` |
| `apps/web` | `@cebmall/web` | 3010 | 公开站点，构建期 SSG，产物为静态文件 |
| `apps/admin` | `@cebmall/admin` | 3020 | 管理后台 SPA，JWT 登录 |
| `packages/shared` | `@cebmall/shared` | — | 跨端类型、错误码与纯函数，不依赖 DOM |
| `packages/api-client` | `@cebmall/api-client` | — | `createCebmallApiClient()` 返回公开与后台两组接口 |
| `packages/i18n` | `@cebmall/i18n` | — | 语言注册、语言包与已发布语种清单 |
| `packages/ui` | `@cebmall/ui` | — | 设计令牌 `UI_TOKENS`、断点与 CSS 变量生成 |
| `packages/config` | `@cebmall/config` | — | 工程预设 |

### 2.3 数据流

公开站点不在运行时直连数据库或后台接口，而是在构建期读取站点快照：

1. `apps/web/scripts/build-snapshot.mjs` 从 API 或静态数据生成英文快照；
2. `scripts/generate-locale-snapshots.mjs` 按语种生成 43 份快照；
3. 页面只读 `apps/web/src/data/snapshot.ts` 暴露的只读接口；
4. `vite-ssg` 逐页预渲染，输出带完整 `head` 的静态 HTML；
5. 构建产物连同 `index.md` 镜像、`llms.txt`、`sitemap*.xml`、`robots.txt` 一起发布到对象存储。

这样页面渲染与抓取不依赖运行时 API 可用性，首屏和搜索引擎都能拿到完整 HTML；询盘提交等交互仍调用 `api.cebmall.com`。

---

## 三、技术栈

| 层 | 选型 |
| ---- | ---- |
| 运行时 | Node.js 24、pnpm 12.21 |
| 后端 | NestJS 12.5、Prisma 7.10.1、MySQL 8、JWT、bcryptjs、nodemailer |
| 前端 | Vue 3.5、Vite 8、vite-ssg 28、Vue Router 5、Pinia 4 |
| UI | TDesign Vue Next 1.20、TDesign Chat、TailwindCSS 4.3 |
| 测试 | Vitest 5 + `@vue/test-utils`（web/admin）、Jest 30（api）、Playwright 1.65（e2e） |
| 部署 | 腾讯云 COS + CDN（`ap-hongkong`）、Nginx、Let's Encrypt |
| 流水线 | CNB（`.cnb.yml`），Node 24 + pnpm 12 |

---

## 四、核心实现要点

### 4.1 多语言

- 43 个语种各有独立 URL：英文在根路径，其余语种使用 `/de/`、`/ar/` 这类前缀；页面、导航、产品与文章文案都按语种输出。
- `packages/i18n/src/locale-registry.ts` 是语种唯一注册表，`apps/web/src/i18n/published-locales.ts` 由脚本生成，禁止手改。
- 每个页面输出 43 条 `hreflang` 互指，并带 `x-default` 指向英文。

### 4.2 SEO / GEO

- 预渲染 HTML、canonical 自指、分类与面包屑结构。
- 结构化数据覆盖 `Organization`、`WebSite`、`Product`、`Article`、`FAQPage`、`BreadcrumbList`。
- 每个可索引页面同时生成 `index.md` Markdown 镜像，并输出 `llms.txt` 供大模型抓取。
- 分语种 `sitemap`、`robots.txt`，对 AI 爬虫使用具名 UA 白名单放行。
- 社交分享输出 `og:image`、`twitter:image` 与 `summary_large_image` 卡片。
- 媒体统一走 `media.cebmall.com`，避免结构化数据出现重复域名。

### 4.3 询盘与合规

- 询盘提交有频率限制与重复窗口控制，后台可查看与跟进。
- 自动回执使用固定模板，不承诺报价、交期、质保与认证。
- 所有密钥、Token、数据库密码只从环境变量读取，`.env*` 不入库、不写入文档。

### 4.4 质量门禁

`.cnb.yml` 中 `push` 与 `pull_request` 执行同一套门禁：

```text
install → typecheck → test + coverage → build → verify-build
        → 客服知识库离线校验 → Playwright E2E → lint + format:check
```

- API 覆盖率阈值：行 70%、分支 60%。
- 构建后由 `scripts/verify-build.mjs` 校验产物完整性。
- 任一环节失败即视为未完成，不允许跳过。

---

## 五、本地开发

### 5.1 环境要求

- Node.js ≥ 24
- pnpm ≥ 12（仓库声明 `pnpm@12.21.0`）
- MySQL 8（本地开发）

### 5.2 首次启动

```bash
pnpm install

# 准备后端环境变量（复制后填写数据库、JWT、管理员初始密码）
cp apps/api/.env.example apps/api/.env.development

pnpm --filter @cebmall/api prisma:migrate
pnpm --filter @cebmall/api prisma:seed

pnpm dev
```

启动后：

- Web：<http://localhost:3010>
- Admin：<http://localhost:3020>
- API：<http://localhost:3000/api>

初始管理员为 `admin@cebmall.com`，密码取自 `ADMIN_INITIAL_PASSWORD`，首次登录后应立即修改。

### 5.3 常用命令

| 命令 | 作用 |
| ---- | ---- |
| `pnpm dev` | 并行启动 api / web / admin |
| `pnpm build` | 递归构建全部包与应用 |
| `pnpm typecheck` | 递归执行 TypeScript 类型检查 |
| `pnpm test` | 递归执行单元测试 |
| `pnpm test:coverage` | 递归执行覆盖率统计 |
| `pnpm test:e2e` | 执行 Playwright E2E |
| `pnpm lint` | ESLint 检查 |
| `pnpm format` / `pnpm format:check` | Prettier 格式化 / 只检查 |
| `pnpm --filter @cebmall/api prisma:migrate` | 应用数据库迁移 |
| `pnpm --filter @cebmall/api prisma:seed` | 写入种子数据 |

---

## 六、环境变量

环境变量按应用隔离，真实值不入库：

| 位置 | 用途 |
| ---- | ---- |
| `apps/api/.env.development` / `.env.production` | 数据库、JWT、管理员初始密码、邮件、规格书与限流配置 |
| `apps/web/.env.development` / `.env.production` | 前端 `VITE_` 变量，随产物打进前端 |
| `apps/admin/.env.development` / `.env.production` | 后台 `VITE_` 变量 |

约定：

- 只有 `VITE_` 前缀变量会暴露给浏览器，禁止存放任何密钥。
- `SecretId`、`SecretKey`、数据库密码、JWT 密钥不得出现在代码、日志、文档与提交记录中。
- 仓库只提供 `.env.example` 占位说明，复制后填写真实值。

---

## 七、部署与发布

| 部分 | 承载方式 |
| ---- | ---- |
| Web | 腾讯云 COS 静态网站 + CDN（`cebmall.com`、`www`） |
| Admin | 独立 COS 桶 + CDN（`admin.cebmall.com`） |
| 媒体 | 独立 COS 桶 + CDN（`media.cebmall.com`），页面统一引用媒体域名 |
| API | 服务器 + Nginx（`api.cebmall.com`） |
| 证书 | Let's Encrypt ECC，DNS-01 续期，覆盖主域、`www`、`api`、`admin` |

发布流程为「构建 → 校验产物 → 上传 COS → 刷新 CDN → 冒烟检查」，脚本位于 `scripts/`，细节见部署手册。

---

## 八、文档索引

`docs/` 是方案与决策的权威来源，按编号阅读：

| 范围 | 文档 |
| ---- | ---- |
| 方案总览 | `00-产品方案总览.md`、`02-网站建设方案.md`、`03-网站建设需求文档.md` |
| 产品与内容 | `01-产品与内容资料整理.md`、`04-产品与竞品研究报告.md`、`11-用户研究与需求分析.md` |
| SEO / GEO | `07-SEO方案.md`、`08-GEO方案.md`、`26-SEO与GEO检查及改进优化方案.md`、`27-SEO与GEO生产环境诊断报告.md` |
| 架构与工程 | `13-技术架构与技术选型.md`、`17-前端工程与性能规范.md`、`18-UI设计规范.md` |
| 后端与数据 | `20-后端API方案.md`、`21-数据库与Prisma方案.md` |
| 多语言与 AI | `22-多语言语言包方案.md`、`23-AI智能客服方案.md`、`24-知识库建设方案.md` |
| 测试与发布 | `16-测试方案.md`、`19-部署与发布手册.md` |
| 执行口径 | `docs/superpowers/specs/`、`docs/superpowers/plans/` |

---

## 九、项目边界

- 业务范围：无线鼠标与 AC/DC 电源适配器，面向 B 端采购与 OEM/ODM 咨询。
- 不做：在线支付、购物车、订单履约、会员体系。
- 不承诺：价格、交期、质保与认证结果；询盘后由业务人工确认。
- 工程约束：依赖单向、密钥不入库、测试就近放置、门禁失败不得声明完成。

---

## 十、许可

本仓库与文档为 CEBMall 项目资料，仅供项目展示与协作参考，未经授权不得用于商业再分发。
