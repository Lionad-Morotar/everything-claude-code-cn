# 安全指南

## 强制安全检查

提交前必须执行以下操作：
- [ ] 无硬编码密钥（API 密钥、密码、令牌）
- [ ] 所有用户输入均经过验证
- [ ] 防止 SQL 注入（使用参数化查询）
- [ ] 防止 XSS 攻击（经过清理的 HTML）
- [ ] 启用 CSRF 保护
- [ ] 验证身份认证与授权
- [ ] 对所有接口启用速率限制
- [ ] 错误信息不得泄露敏感数据

## 密钥管理

```typescript
// 绝不允许：硬编码密钥
const apiKey = "sk-proj-xxxxx"

// 始终使用：环境变量
const apiKey = process.env.OPENAI_API_KEY

if (!apiKey) {
  throw new Error('OPENAI_API_KEY 未配置')
}
```

## 安全响应流程

如发现安全问题：
1. 立即停止操作
2. 使用 **security-reviewer** 代理
3. 在继续之前修复所有严重问题
4. 更换任何已泄露的密钥
5. 检查整个代码库是否存在类似问题
