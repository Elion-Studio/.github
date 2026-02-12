# Elion 项目总览与 Routemap

更新时间：2026-02-12

本文档用于统一回答三个问题：
- 整个项目目前有哪些仓库与职责
- 哪些能力已经实现并可运行
- 下一步规划与阶段性路线图（Routemap）

## 1. 项目目标（一句话）

Elion 是一个面向 1v1 长期陪伴场景的实时语音对话系统，强调稳定人格、持续关系与低延迟流式交互。

## 2. 全局架构总览（Repo Map）

当前工作区采用多仓库组织，核心链路为：

`Client -> elion-gateway -> elion-conversation-service -> {asr/llm/tts/memory/avatar}`

仓库分层如下：

- 核心运行链路
  - `elion-gateway`：唯一对外入口（HTTP/WS）
  - `elion-conversation-service`：对话编排与 turn 管理
  - `elion-asr-service`：语音识别
  - `elion-llm-service`：大模型生成
  - `elion-tts-service`：语音合成
  - `elion-memory-service`：记忆写入与检索
  - `elion-avatar-service`：情绪到角色事件映射
- 契约与基础
  - `elion-contracts`：proto 契约定义
  - `elion-go-kit`：Go 通用库（基础能力仓库）
  - `elion-infra`：基础设施与部署仓库
- 领域扩展（当前多为占位）
  - `elion-relationship-service`
  - `elion-skill-service`
  - `elion-content-policy-service`
  - `elion-game-extension-service`

## 3. 已经实现的（可运行状态）

- 已实现仓库
  - `elion-gateway`
  - `elion-conversation-service`
  - `elion-memory-service`
  - `elion-llm-service`
  - `elion-asr-service`
  - `elion-tts-service`
  - `elion-avatar-service`
  - `elion-contracts`
- 已支持输入模式
  - `audio.chunk + audio.end`（语音）
  - `text.input`（文本）
- 运行与运维脚本
  - 一键启动/停止/重启：`scripts/start_all_services.sh` / `stop_all_services.sh` / `restart_all_services.sh`
  - 健康检查：`GET /healthz`
  - Demo 入口：`http://127.0.0.1:8091/`

## 4. 规划中的（待完善或待实装）

### 4.1 仓库层面

- `elion-relationship-service`：关系状态、长期偏好、里程碑管理（当前为通用模板，待业务实装）
- `elion-skill-service`：工具/技能编排与执行（当前为通用模板，待业务实装）
- `elion-content-policy-service`：内容安全与策略审核（当前为通用模板，待业务实装）
- `elion-game-extension-service`：游戏化扩展域（当前为通用模板，待业务实装）
- `elion-go-kit`：基础库能力沉淀尚未形成明确模块清单
- `elion-infra`：部署编排、环境模板、runbook 需要系统化落地

### 4.2 架构与契约层面

- 事件契约需从当前 `internal.proto` 继续演进为更稳定的流式事件模型
- session/turn 的幂等、过期丢弃、取消语义需要进一步固化到 contracts 版本策略
- Memory 分层策略（短期/长期/Profile/Event）已有方案文档，需与线上实现持续对齐

## 5. Routemap（分阶段路线图）

## 阶段 P0：稳定现有主链路（当前优先）

目标：把“语音输入 -> 对话生成 -> 语音输出”链路稳定在可持续迭代状态。

- 统一并冻结现有对外事件字段（先做到兼容优先）
- 完成核心服务健康检查、日志字段、错误码最小标准化
- 补齐主链路端到端回归脚本（语音与文本双路径）
- 固化本地与测试环境启动规范（env、端口、依赖）

交付标准：
- 全链路可在一套脚本下稳定启动，且能重复通过 e2e smoke test

## 阶段 P1：契约治理与可维护性提升

目标：从“可运行”升级到“可协作、可演进”。

- 强化 `elion-contracts`：版本化、兼容性约束、错误码统一
- 明确 gateway 与 conversation 的职责边界并文档化
- 将关键行为（取消、中断、过期 turn 丢弃）形成可测试规范
- 建立跨仓库最小 CI 基线（lint/test/build + 合约检查）

交付标准：
- 契约变更有版本规则，跨仓库升级有明确流程

## 阶段 P2：记忆系统与关系能力深化

目标：提升长期连续性与个性化体验。

- 按“工作记忆/短期记忆/长期语义/Profile/Event”推进分层实现
- 完善 Memory 写入幂等、去重、重排与回忆注入策略
- 逐步实装 `elion-relationship-service`（先读模型，再写模型）
- 增强中断事件与用户交互策略联动

交付标准：
- 跨会话记忆召回可观测、可解释，误记率与检索延迟可量化

## 阶段 P3：能力扩展与产品化

目标：为后续复杂场景和商业化预留扩展性。

- 实装 `elion-skill-service`，引入工具执行闭环
- 实装 `elion-content-policy-service`，在关键节点接入策略审查
- 按需推进 `elion-game-extension-service` 的独立域能力
- 完成 `elion-infra` 的部署模板（Compose/K8s）与运维手册
- 推进 TTS 音色训练（前置：`5090` 购买、`evil` 音频获取）
- 推进模型训练（前置：`5090` 购买、`neuro-sama` 数据获取）

交付标准：
- 新能力可独立演进，不破坏主对话链路的稳定性

## 6. 当前建议的执行顺序（无具体时间约束）

- 优先级 1：稳定主链路与契约基线
  - 梳理并冻结现有事件字段与错误码最小集
  - 完成主链路 smoke test 清单（语音 + 文本）
  - 补齐 contracts 变更流程与最小 CI
- 优先级 2：可维护性与记忆能力增强
  - 完成 gateway/conversation 的职责边界文档与用例
  - 推进 memory 分层能力与可观测指标
  - 选择一个扩展仓库做首个实装试点（建议 relationship 或 skill）
- 优先级 3：训练与产品化扩展
  - 推进 TTS 音色训练
  - 推进模型训练
  - 持续完善扩展域能力与运维模板

## 7. 文档导航

- 项目定位：`项目说明.md`
- 架构拆分：`仓库架构说明.md`
- 交互与编排：`实时语音对话系统架构说明.md`
- 能力服务与 proto 设计：`服务划分与 Proto.md`
- 记忆系统方案：`记忆系统技术方案.md`
- 已实现服务清单：`docs/IMPLEMENTED_SERVICES.md`
- 运维操作：`docs/OPERATIONS.md`
