# 存储系统实现计划（V1）（Storage System Implementation Plan）

状态：草稿  
负责人：Backend + UI  
日期：2026-02-20

## 目标（Goal）

为 Paperclip 增加统一存储子系统，支持：

- 单用户本地部署的本地磁盘存储
- 云端部署的 S3 兼容对象存储
- 与提供方无关的接口（issue 图片及后续文件附件）

## V1 范围（V1 Scope）

- 首个消费者：issue 附件/图片。
- 存储适配器：`local_disk` 与 `s3`。
- 文件均以公司为作用域并做访问控制。
- API 通过已认证的 Paperclip 端点提供附件字节。

## 关键决策（Key Decisions）

- 本地默认路径：`~/.paperclip/instances/<instanceId>/data/storage`。
- 对象字节存于存储提供方，元数据存 Postgres。
- `assets` 为通用元数据表；`issue_attachments` 将 assets 关联到 issues/comments。
- S3 凭证来自运行时环境/默认 AWS 链，不存 DB。
- 所有对象 key 含公司前缀以保持租户边界。

（阶段与清单见英文原文。）
