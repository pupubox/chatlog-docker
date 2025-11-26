# Chatlog Docker 部署项目

微信聊天记录管理和查询工具 [Chatlog](https://github.com/sjzar/chatlog) 的 Docker 部署配置。

## 项目简介

本项目提供了将 Chatlog 部署到 Docker 容器的完整配置，用于：
- 解密和查询微信聊天记录
- 提供 HTTP API 接口
- 支持 MCP 协议与 AI 工具集成
- 本地化数据处理，保护隐私安全

## 项目背景

这是一个用于研究微信聊天记录合并、去重和搜索方案的实验项目。探索了以下技术方案：
- 微信官方迁移功能的限制
- 第三方工具（WeChatMsg、Chatlog）的能力边界
- SQLite 数据库合并去重的技术实现
- Docker 容器化部署的可行性

## 项目结构

```
chatlog-docker/
├── docker-compose.yml    # Docker Compose 配置文件
├── .gitignore           # Git 忽略规则
├── config/              # 配置文件目录（被 gitignore）
└── README.md            # 项目文档
```

## 快速开始

### 前置要求

- Docker Desktop 已安装并运行
- macOS / Windows / Linux 系统
- 已安装微信 PC 客户端

### 配置说明

1. **克隆仓库**
   ```bash
   git clone https://github.com/pupubox/chatlog-docker.git
   cd chatlog-docker
   ```

2. **修改 docker-compose.yml**

   找到微信数据目录，替换配置文件中的路径：

   **macOS**：
   ```yaml
   volumes:
     - ~/Library/Containers/com.tencent.xinWeChat/Data/Library/Application Support/com.tencent.xinWeChat/2.0b4.0.9/[你的微信ID]:/app/wechat-data:ro
   ```

   **Windows**：
   ```yaml
   volumes:
     - C:/Users/[用户名]/Documents/WeChat Files/[微信ID]:/app/wechat-data:ro
   ```

3. **启动容器**
   ```bash
   docker-compose up -d
   ```

4. **访问服务**
   ```
   http://localhost:5030
   ```

## 重要说明

### ⚠️ 已知限制

1. **密钥获取问题**
   - Chatlog 的 `chatlog key` 命令需要访问运行中的微信进程
   - Docker 容器内无法访问宿主机进程
   - 建议在宿主机直接运行 Chatlog，而非 Docker

2. **没有 Web 前端**
   - Chatlog 只提供 HTTP API 和 TUI 终端界面
   - 没有图形化的 Web 管理界面
   - 需要通过 API 调用或终端命令使用

3. **不支持多设备合并**
   - Chatlog 无法自动合并多个微信数据库
   - 需要手动合并数据或使用其他方案

### 推荐的替代方案

如果你需要：

**1. 图形界面浏览聊天记录**
- 使用 [WeChatMsg](https://github.com/LC044/WeChatMsg)
- 支持导出 HTML、Word、CSV
- 有可视化界面

**2. 完美合并去重**
- 手动编写 Python 脚本
- 基于 SQLite UNION 或 INSERT OR IGNORE
- 使用 `MsgSvrId` 字段去重

**3. AI 分析聊天记录**
- 在宿主机直接运行 Chatlog
- 配置 MCP 协议
- 连接 Claude、ChatGPT 等 AI 工具

## Chatlog 功能说明

### 核心能力

- 🔐 **数据解密**：自动解密微信 SQLite 数据库
- 🔍 **智能搜索**：按时间、关键词、联系人查询
- 📡 **HTTP API**：提供 RESTful API 接口
- 🤖 **AI 集成**：支持 MCP 协议，可接入 AI 助手
- 🎯 **多媒体处理**：实时解密图片、转码语音（SILK→MP3）

### API 端点

```bash
GET /api/v1/chatlog   # 查询聊天记录
GET /api/v1/contact   # 获取联系人列表
GET /api/v1/chatroom  # 获取群聊列表
GET /api/v1/session   # 获取会话列表
```

### 查询示例

```bash
# 查询指定时间段的聊天记录
curl "http://localhost:5030/api/v1/chatlog?time=2024-01-01~2024-12-31&talker=张三&limit=100"

# 导出为 CSV 格式
curl "http://localhost:5030/api/v1/chatlog?format=csv"
```

## 实际应用场景

1. **产品反馈分析** - 自动总结用户群的反馈意见
2. **会议纪要生成** - AI 自动整理群聊讨论内容
3. **数据可视化** - 导出结构化数据进行分析
4. **智能搜索** - 模糊查找历史聊天内容
5. **个性化分析** - 分析特定联系人的聊天特征

## 技术调研总结

### 微信数据库结构

- **数据库**：SQLite（加密）
- **核心表**：MSG 表
- **关键字段**：
  - `MsgSvrId`：服务器消息ID（唯一，用于去重）
  - `CreateTime`：时间戳
  - `Talker`：聊天对象
  - `Content`：消息内容

### 合并去重方案

```sql
-- 方法1：UNION（自动去重）
SELECT * FROM db1.MSG
UNION
SELECT * FROM db2.MSG
ORDER BY CreateTime;

-- 方法2：INSERT OR IGNORE
INSERT OR IGNORE INTO MSG
SELECT * FROM source.MSG;
```

## 故障排查

### 容器无法启动

```bash
# 查看容器日志
docker logs chatlog

# 检查容器状态
docker ps -a

# 重启容器
docker-compose restart
```

### 服务无响应

```bash
# 检查端口是否被占用
lsof -i :5030

# 查看容器健康状态
docker inspect chatlog --format='{{json .State.Health}}'
```

## 开发进度

- [x] Docker 环境配置
- [x] docker-compose.yml 编写
- [x] 微信数据目录定位
- [x] Chatlog 镜像拉取
- [ ] 密钥获取自动化（受限于 Docker）
- [ ] 服务正常启动（待解决）
- [ ] API 功能测试
- [ ] MCP 协议集成

## 参考资源

- [Chatlog 官方仓库](https://github.com/sjzar/chatlog)
- [WeChatMsg 官方仓库](https://github.com/LC044/WeChatMsg)
- [微信数据库结构分析](https://zhuanlan.zhihu.com/p/552876079)
- [SQLite 数据库合并教程](https://stackoverflow.com/questions/80801/how-can-i-merge-many-sqlite-databases)

## 许可证

本项目遵循 MIT 许可证。

## 免责声明

本项目仅供学习、研究和个人合法使用，禁止用于任何非法目的或未授权访问他人数据。使用本工具时请遵守相关法律法规和微信用户协议。

## 贡献

欢迎提交 Issue 和 Pull Request！

## 联系方式

如有问题或建议，请通过 GitHub Issues 联系。

---

⚠️ **重要提示**：此项目为技术研究项目，当前 Docker 部署方案存在已知限制，建议在宿主机直接运行 Chatlog。
