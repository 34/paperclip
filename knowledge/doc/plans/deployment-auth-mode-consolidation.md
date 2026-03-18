# 部署/认证模式整合计划（Deployment/Auth Mode Consolidation Plan）

状态：提议  
负责人：Server + CLI + UI  
日期：2026-02-23

## 目标（Goal）

在保持 Paperclip 低摩擦的前提下简化并加固模式模型：

1. `local_trusted` 保持默认与最简单路径。
2. 单一认证运行模式同时支持私有网络本地与公网云端。
3. onboard/configure/doctor 保持以交互、无参数为主。
4. 董事会身份由数据库中真实用户行表示，并与角色/成员集成点明确对接。

## 产品约束（Product Constraints）

- onboard 默认流程为交互式；首选模式默认 `local_trusted`；认证流程需区分 private/public 暴露指引；doctor 默认无参数并读取配置评估所选模式；不为废弃模式名增加兼容别名；计划需明确用户/Board 在 DB 中的表示及对任务分配与权限的影响。

（当前实现审计与迁移步骤见英文原文。）
