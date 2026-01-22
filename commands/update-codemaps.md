# 更新代码图谱

分析代码库结构并更新架构文档：

1. 扫描所有源文件中的导入、导出和依赖项
2. 以以下格式生成精简的代码图谱：
   - codemaps/architecture.md - 整体架构
   - codemaps/backend.md - 后端结构
   - codemaps/frontend.md - 前端结构
   - codemaps/data.md - 数据模型与模式

3. 计算与上一版本的差异百分比
4. 若变更超过 30%，则在更新前请求用户确认
5. 为每个代码图谱添加时效性时间戳
6. 将报告保存至 .reports/codemap-diff.txt

使用 TypeScript/Node.js 进行分析。重点关注高层结构，而非实现细节。
