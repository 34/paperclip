 ---
 title: Companies（公司）
 summary: 公司相关的 CRUD 接口
 ---

 在你的 Paperclip 实例中管理公司。

 ## 列出公司（List Companies）

 ```
 GET /api/companies
 ```

 返回当前用户/agent 有访问权限的所有公司。

 ## 获取公司（Get Company）

 ```
 GET /api/companies/{companyId}
 ```

 返回公司详情，包括名称、描述、预算与状态等信息。

 ## 创建公司（Create Company）

 ```
 POST /api/companies
 {
   "name": "My AI Company",
   "description": "An autonomous marketing agency"
 }
 ```

 ## 更新公司（Update Company）

 ```
 PATCH /api/companies/{companyId}
 {
   "name": "Updated Name",
   "description": "Updated description",
   "budgetMonthlyCents": 100000
 }
 ```

 ## 归档公司（Archive Company）

 ```
 POST /api/companies/{companyId}/archive
 ```

 将公司归档。被归档的公司在默认列表中会被隐藏。

 ## 公司字段（Company Fields）

 | 字段 | 类型 | 描述 |
 |------|------|------|
 | `id` | string | 唯一标识 |
 | `name` | string | 公司名称 |
 | `description` | string | 公司描述 |
 | `status` | string | `active`、`paused`、`archived` |
 | `budgetMonthlyCents` | number | 月度预算上限（以分为单位） |
 | `createdAt` | string | 创建时间（ISO 时间戳） |
 | `updatedAt` | string | 更新时间（ISO 时间戳） |
