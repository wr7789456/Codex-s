# 世界OL 本地源码 MCP 接入规范

目标：让 ChatGPT/Codex 稳定读取本地世界OL源码、APK逆向分析数据和项目状态，不需要反复上传完整压缩包。

## 推荐目录

```text
~/WorldOL/
├─ source/
├─ apk/
│  └─ 1046045.apk
├─ apk_extract/
├─ analysis/
├─ checkpoints/
│  └─ WORLDOL_PROJECT_STATE.md
└─ mcp-server/
```

第一版只读，根目录固定为 `~/WorldOL`，所有路径 realpath 后必须仍位于根目录内。

建议 tools：
- worldol_status
- list_files
- search_code
- read_file
- read_json
- file_info
- git_status
- git_diff
- get_checkpoint
- inspect_spr
- inspect_fr
- inspect_pl
- inspect_protected_png

第二阶段再开放：
- write_checkpoint（只允许 checkpoints/）
- apply_patch（只允许 source/）
- run_tests（只允许预定义测试命令）

不要提供任意 shell 执行器作为第一版写权限。

大文件/APK/ZIP/PNG默认不返回raw bytes，只返回路径、大小、SHA256、magic和结构摘要。read_file建议单次限制256KB；search结果限制50~100条。

推荐 MCP endpoint：
`http://127.0.0.1:4173/mcp`

健康检查：
`http://127.0.0.1:4173/health`

如果通过Tailscale访问，优先绑定本机/Tailscale接口并做认证，不要0.0.0.0裸奔。

## 给 Codex 的任务

请在本机为“世界OL单机版完善”建立 Streamable HTTP MCP Server。

根目录固定为 ~/WorldOL，禁止访问根目录之外。
第一版只读，不提供任意shell执行。
实现：worldol_status、list_files、search_code、read_file、read_json、file_info、git_status、git_diff、get_checkpoint、inspect_spr、inspect_fr、inspect_pl、inspect_protected_png。
MCP endpoint 为 http://127.0.0.1:4173/mcp，health 为 http://127.0.0.1:4173/health。
read_file支持start_line/end_line；search_code支持path/glob/max_results；所有路径必须realpath后验证仍位于~/WorldOL；单次文本最大256KB；二进制禁止直接返回raw bytes。
get_checkpoint默认读取 ~/WorldOL/checkpoints/WORLDOL_PROJECT_STATE.md。
优先复用当前世界OL源码中的 legacy-resource-codec.mjs 和现有SPR/FR/PL/protected PNG解析逻辑，不要重复写两套解析器。
为每个tool写自动测试，最后用MCP Inspector验证。
