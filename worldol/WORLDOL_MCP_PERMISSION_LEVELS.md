# 世界OL MCP 权限分级与多目录暴露

## 推荐原则

不要把整个用户目录或磁盘作为一个 unrestricted root 暴露。采用多个命名 root + 每个 root 独立权限。

示例：

```json
{
  "roots": {
    "source": {
      "path": "/Users/YOU/WorldOL/source",
      "read": true,
      "write": true
    },
    "analysis": {
      "path": "/Users/YOU/WorldOL/analysis",
      "read": true,
      "write": true
    },
    "apk": {
      "path": "/Users/YOU/WorldOL/apk",
      "read": true,
      "write": false
    },
    "apk_extract": {
      "path": "/Users/YOU/WorldOL/apk_extract",
      "read": true,
      "write": false
    },
    "checkpoints": {
      "path": "/Users/YOU/WorldOL/checkpoints",
      "read": true,
      "write": true
    }
  }
}
```

每次收到路径后：
1. 解析对应 root
2. realpath
3. 验证 realpath 仍在 root 内
4. 拒绝 ../ 越界
5. 默认拒绝跨 root symlink

## Level 1：只读

允许：
- list_files
- read_file
- read_json
- search_code
- file_info
- git_status
- git_diff
- inspect_spr/fr/pl/protected_png

适合首次连接 ChatGPT。

## Level 2：受限写入

新增：
- write_file
- apply_patch
- mkdir
- delete_generated_file
- write_checkpoint

限制：
- source/ 可修改源码
- analysis/ 可写分析报告
- checkpoints/ 可写状态文件
- apk/、apk_extract/ 仍只读
- 单文件写入大小设置上限
- 写入前做git diff或备份

## Level 3：受限命令执行

不要开放任意 `shell(command)`。

实现命令 allowlist：
- npm test
- npm run test
- node --test ...
- git status
- git diff
- git log
- python3 scripts/指定脚本.py
- node scripts/指定脚本.mjs

可以增加：
`run_task(task_name, args)`

例如：
- run_task("resource_tests")
- run_task("audit_spr")
- run_task("audit_missing_fragments")
- run_task("start_worldol_dev_server")

服务端把 task_name 映射为固定命令。

## Level 4：Git 写权限

新增：
- git_create_branch
- git_commit
- git_restore_file

默认不提供：
- git push --force
- git reset --hard
- 删除远程分支

如果本机已经配置GitHub，可让 push 仍由人手动执行。

## Level 5：高权限开发模式（仅你明确需要时）

如果确实希望AI可以完成完整本地开发闭环，可增加受控 exec：

`exec(argv, cwd_root, timeout, env_allowlist)`

安全约束：
- argv 数组，不经 shell 展开
- cwd 必须属于命名 root
- 禁止 sudo
- 禁止访问 ~/.ssh、钥匙串、浏览器profile
- 环境变量只允许显式 allowlist
- timeout
- stdout/stderr 大小限制
- 命令日志

即使Level 5，也不要直接实现：
`bash -c <任意字符串>`

## 二进制和大文件

APK/ZIP/PNG/SO等不要通过普通 read_file 整个返回。

提供：
- binary_info
- hexdump_range(offset,length)
- strings_search
- sha256
- extract_zip_member
- inspect_elf
- inspect_png
- inspect_spr/fr/pl

hexdump_range 建议每次最大 64KB。

## 多目录暴露

若世界OL文件散落在多个位置，可以增加 roots，不必把共同父目录整个暴露：

```json
{
  "roots": {
    "worldol_source": "/Volumes/Software/WorldOL/source",
    "worldol_apk": "/Volumes/Software/WorldOL/apk",
    "worldol_analysis": "/Users/YOU/Documents/WorldOL-analysis"
  }
}
```

tool 输入使用：
`{ "root": "worldol_source", "path": "lib/legacy-resource-codec.mjs" }`

而不是接受任意绝对路径。

## Tailscale

Tailscale适合让你自己的其他设备/Codex访问MCP，但ChatGPT云端通常不能直接加入你的Tailnet。

建议：
- MCP本机仍监听 127.0.0.1 或 Tailscale IP
- 你自己的设备通过 Tailscale 使用
- ChatGPT 使用 OpenAI 支持的私有MCP连接方式/安全隧道；若界面暂不支持，再单独使用带认证的HTTPS反向隧道

不要因为用了Tailscale就把MCP绑定 `0.0.0.0` 且无认证。

## 推荐最终权限

对于世界OL项目，建议长期使用：
- source: read/write
- analysis: read/write
- checkpoints: read/write
- apk: read-only
- apk_extract: read-only
- predefined test/audit tasks: executable
- git diff/status/commit: enabled
- unrestricted shell: disabled

这已经足够让AI完成绝大多数源码修改和逆向分析，同时不会把整台电脑暴露出去。
