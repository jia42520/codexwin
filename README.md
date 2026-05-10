# Codex Win 免登录配置工具

用于配置 Codex 使用 API Key 方式登录的 Windows 桌面工具。

## 功能特点

- **免登录模式**：使用 API Key 替代 Codex 网页登录
- **自动配置**：一键生成正确的 `config.toml` 配置文件
- **环境变量**：自动写入 Windows 用户环境变量 `CODEX_API_KEY`
- **安全清理**：自动备份并移除网页登录认证文件 `auth.json`
- **配置预览**：实时预览生成的配置内容
- **快速启动**：v1.0 采用文件夹打包模式，启动更快

## 系统要求

- Windows 10 / Windows 11
- 需要管理员权限（写入环境变量时）

## 使用方法

### 快速启动

双击运行：

```
一键运行Codex免登录配置工具.bat
```

### 配置步骤

1. **填写 API Key**：在输入框中输入您的 API Key
2. **确认配置**：默认已配置好以下参数：
   - Provider: `lightconeapi`
   - API 地址: `https://api.lightcone.hk/v1/chat/completions`
   - 模型: `gpt-5.5`
3. **选项设置**：
   - 勾选「写入环境变量」：将 API Key 写入 Windows 用户环境变量
   - 勾选「移除网页登录」：备份并移除旧的网页登录认证文件
4. **点击保存**：配置将自动写入 `C:\Users\<用户名>\.codex\config.toml`

## 配置说明

### 默认配置

```toml
model = "gpt-5.5"
model_provider = "lightconeapi"
preferred_auth_method = "apikey"

[model_providers.lightconeapi]
name = "lightconeapi"
base_url = "https://api.lightcone.hk/v1"
env_key = "CODEX_API_KEY"
wire_api = "responses"

[env]
CODEX_API_KEY = "<您的API密钥>"
```

### 文件路径

- 配置文件：`C:\Users\<用户名>\.codex\config.toml`
- 认证文件：`C:\Users\<用户名>\.codex\auth.json`（备份后移除）

## 注意事项

1. **不要单独复制 exe 文件**：此工具需要保留完整的 `CodexConfigTool` 文件夹才能运行
2. **管理员权限**：写入环境变量时需要管理员权限
3. **重启生效**：环境变量设置后，需要重启 Codex 才能生效
4. **配置备份**：保存配置时会自动备份旧配置文件（后缀 `.bak_时间戳`）

## 版本历史

| 版本 | 更新内容 |
|------|----------|
| v1.0 beta | 初始版本 |
| v1.0 beta | 修复高 DPI 文字模糊问题 |
| v1.0 beta | 优化 UI 布局 |
| v1.0 beta | 修复选项区显示不全 |
| v1.0 beta | 修复按钮显示不全 |
| v1.0 beta | 修复配置预览显示不全 |
| v1.0 beta | 修复 provider 覆盖内置 openai 问题 |
| v1.0 | **优化启动速度**（改用文件夹打包模式） |

## 技术支持

如有问题，请检查：

1. 是否保留了完整的 `CodexConfigTool` 文件夹
2. 是否以管理员身份运行
3. API Key 是否正确
4. 网络连接是否正常
