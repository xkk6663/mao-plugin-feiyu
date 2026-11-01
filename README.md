# mao-plugin-feiyu · 大肥鱼插件包

> 大肥鱼 = 桌宠对接云端 AstrBot persona 的多模态对话能力（图像双向 / 点歌 / 跨设备记忆）。
> 从 `Mao-dsh-pet` 拆出的**独立插件**：桌宠主体与 dsh 插件的发布链路不含本包。
> 注：主体代码内部标识符沿用历史前缀 `__mio*`（kernel/registry/loader），但**项目名称是 Mao**。

## 定位（一句话）

桌宠主体的「可卸载增强」：不装 = 桌宠没有任何云端大肥鱼功能；装了 = 对话页、气泡识图、点歌、跨设备记忆全部点亮，且可在 Mao 插件市场一键停用。

## 架构铁律

- **界面随插件走**：左侧「大肥鱼」对话页 UI 属于本插件，不依赖主体 Vue 应用内嵌页面；
- **插件之间零依赖**：本插件只声明自己、只调主体公开接口，不引用任何其它插件（与主体清单契约「插件零相互依赖」一致）。

## 插件分层总览（与主仓库 README 同口径）

| 层 | 是什么 | 代码位置 | 发布位 |
|---|---|---|---|
| 桌宠主体 | Electron 壳 + 渲染端 + 后端 + 内置插件市场 | `Mao-dsh-pet` 仓库 | 桌宠安装包 |
| dsh 插件 | `dsh-live2d-mao` 连接器（只推事件） | `Mao-dsh-pet/packages/dsh-live2d-mao` | npm + dsh-market（**不含大肥鱼**） |
| 内置插件 | 随主体预装、清单化加载 | `Mao-dsh-pet` 仓库 `dist/plugins/*` | 只随桌宠安装包 |
| **大肥鱼插件** | 云端 AstrBot 多模态对话（界面 + 后端） | **本仓库** | Mao 插件市场按需安装 |

## 仓库结构

```
frontend/            Mao 插件市场前端件（落盘到 澪Agent应用/dist/plugins/feiyu/）
  manifest.js          插件声明（agentId=cloud_feiyu / nav host 模式 / settingsTab）
  feiyu-panel.js       设置页面板
  src/                 宿主对话页 Vue 源码（FeiyuChatPage / CompanionPage / useCompanionChat 等）
  dist-assets/         主仓库 Vue 构建的对应编译产物（hash 命名，供对照/回填）
backend/cloud_feiyu/ 后端模块（落盘到 backend/app/cloud_feiyu/）
  client.py            AstrBot Open API 客户端（对话/附件/文件，1462 行）
  device.py            设备上下文
  devices.py           设备上线通知
  mcp_routes.py        MCP 路由
  relay_service.py     中转（relay）服务
  transfer_receiver.py 跨设备传输投递（crosssend → 桌宠落库）
backend/companion/
  cover_service.py     翻唱服务（每日翻唱 / 点歌翻唱管线，主仓 app/companion/cover_service.py）
assets/
  da-feiyu/            大肥鱼 Live2D 桌宠模型（33MB，主仓 dist/live2d-pet/models/da-feiyu）
  da-feiyu.json        桌宠台词包
docs/                拆分方案、耦合地图、功能总览与协议文档
```

> **2026-11-01 v1.1.0 同步**：主仓库（dsh-live2d-mao）v1.4.0 起对大肥鱼做 git 级移出，
> 本仓收编全部大肥鱼源码（含 10-29/10-30 的 transfer_receiver 落库、200 错误体防呆、
> 每日翻唱进度通路等修复）。主仓库工作区保留一份**不入库的本地运行副本**，
> 本地桌宠功能不受影响；阶段 2/3 落地后改由市场安装本包。

## 与桌宠主体的关系

| 链路 | 内容 | 大肥鱼在不在 |
|---|---|---|
| dsh npm 包（dsh-market 上架） | `dsh-live2d-mao` 连接器 | ❌ 从来不含 |
| 桌宠安装包（打包发布.ps1） | 桌宠主体 + Mao 市场 builtin 插件 | 拆分完成后不含（改为市场按需安装） |
| Mao 插件市场「安装」 | 本包（远期） | ✅ 唯一分发位 |

## 分期路线（与 Mao-dsh-pet 仓库的解耦进度）

- **阶段 1（当前）**：源码在本仓快照；主仓库仍为 builtin 预装，行为不变。
- **阶段 2**：Mao 市场 `POST /api/plugin/market/install` —— 拉 npm tarball → 解压 `dist/plugins/<id>/` → 复用现有上架登记。前端件先走远程分发。
- **阶段 3**：「后端插件协议」落地（importlib 按清单挂路由 + 生命周期 hook），`cloud_feiyu` 从主体 8 处耦合中解耦为纯外挂。见 [`docs/耦合地图.md`](./docs/耦合地图.md)。

## 已裁决事项（2026-10-26 用户拍板）

- **FeiyuChatPage.vue 随插件走**：主体 Vue 应用里的宿主对话页最终移出 dist/assets，由本插件自带界面（阶段 2/3 实施；过渡期主体页面继续服务，但归属已定）。
- **插件之间无依赖**：本插件不引用其它任何插件。

## 配置约定（安全红线）

- `ASTRBOT_BASE_URL` / `ASTRBOT_API_KEY` 一律走环境变量或 backend `.env`，**绝不写进源码与文档**（真实服务器 IP 曾因此差点入库，2026-10-26 已全仓清理）。
