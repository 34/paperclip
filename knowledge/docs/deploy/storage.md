 ---
 title: 存储（Storage）
 summary: 本地磁盘 vs S3 兼容存储
 ---

 Paperclip 会通过可配置的存储提供方来保存上传的文件（issue 附件、图片等）。

 ## 本地磁盘（Local Disk，默认）

 文件默认存储在：

 ```
 ~/.paperclip/instances/default/data/storage
 ```

 无需额外配置，适用于本地开发和单机部署。

 ## S3 兼容存储（S3-Compatible Storage）

 生产或多节点部署推荐使用 S3 兼容对象存储（AWS S3、MinIO、Cloudflare R2 等）。

 通过 CLI 配置：

 ```sh
 pnpm paperclipai configure --section storage
 ```

 ## 配置选项（Configuration）

 | Provider | 适用场景 |
 |----------|----------|
 | `local_disk` | 本地开发与单机部署 |
 | `s3` | 生产、多节点、云部署 |

 存储配置保存在实例配置文件中：

 ```
 ~/.paperclip/instances/default/config.json
 ```
