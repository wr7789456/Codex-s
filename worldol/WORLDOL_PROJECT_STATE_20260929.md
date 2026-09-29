# 世界OL单机版完善 · 当前技术状态

更新时间：2026-09-29

当前主线：官方 APK 旧资源逆向 → 资源恢复 → GM 控制台批量迁移。

关键状态：
- APK `.spr`：3153
- 当前 v305 manifest model：4193
- 完全一致：2004
- 同ID版本差异：754
- 当前真正缺失：384
- 其中11个现在即可安全迁移，373个仍被受保护fragment阻塞
- 缺失fragment：140（139主保护组 + 5389特殊）
- 结构分层：87 IHDR→IDAT；25 IHDR→tEXt→IDAT；18 IHDR→tEXt→iTXt；8 PLTE；1 pHYs+iCCP；1 special 5389

已确认格式：
- SPR：基本解完，可转当前v305 model JSON
- FR：1730个全部归入compact 5-byte或wide 9-byte
- PL：1830/1830验证为PLTE payload + CRC32("PLTE"+payload)
- Protected PNG：8-byte wrapper + protected PNG[0:1000] + clear PNG[1000:]
- 跨资源全局确认可逆区间：PNG offset 0-127（128 bytes）

重要纠正：
- 709只是recorded-expansion-import子集，不是整个v305模型总数；完整manifest为4193
- 真正缺失模型是384，不是早期按709子集得到的2994
- 全局确认保护前缀是128字节，不是132
- AF1BB1FA不应单独定义为“易盾格式”；只能称保护封装/保护层

GM控制台现有能力：
- SPR/FR/PL解析
- 受保护PNG识别与结构审计
- 模型/fragment依赖查询与PNG预览
- APK 3153 SPR与v305 4193模型全量对比
- SPR→v305 model JSON
- fragment依赖检查
- 安全模型导入、manifest备份、哈希资源写入、运行时刷新
- 批量检查/导入、事务记录、批量回滚、孤儿哈希清理
- 保护族/结构分层审计

已验证11个可直接迁移模型：
792, 1359, 1360, 1361, 1362, 1363, 1364, 1365, 6011, 6012, 50001

下一步：
1. 专攻87张IHDR→IDAT简单RGBA目标图
2. 用19087–19104的同前128明文样本研究offset128后的状态派生
3. 继续追libnesec.so中的128-byte/EOR函数调用链
4. 新恢复fragment后自动重算373个被阻塞模型中可迁移数量

新对话恢复指令：
读取本文件和当前源码，继续世界OL资源恢复主线；不要回到装备MOD；从protected PNG offset 128+ / libnesec 128-byte函数调用链继续。
