# 职业测试系统 - 部署和使用文档

## 📋 系统架构

```
用户访问 → nginx (80 端口) → 前端静态文件 + API 反向代理
                              ↓
                         Flask API (5001 端口)
                              ↓
                    文件系统存储结果和订单
```

---

## 🚀 访问地址

| 页面 | 地址 |
|------|------|
| 测试页 | http://www.myjobtest.cn/ |
| 报告页 | http://www.myjobtest.cn/report.html |
| API 状态 | http://www.myjobtest.cn/api/status |

---

## 📦 完整用户流程

### 1. 免费测试
```
用户访问 www.myjobtest.cn
  ↓
完成 10 道测试题
  ↓
系统自动保存结果到后端
  ↓
显示简要结果（职业名称 + 描述）
```

### 2. 购买完整报告
```
用户点击「去闲鱼购买 ¥1.1」
  ↓
跳转闲鱼商品页面
  ↓
用户付款 ¥1.1
  ↓
系统自动/手动记录订单号
```

### 3. 解锁完整报告
```
用户访问 www.myjobtest.cn/report.html
  ↓
输入闲鱼订单号
  ↓
系统验证订单 → 显示个性化完整报告
  ↓
报告永久保存在本地（localStorage）
```

---

## 🔧 管理员操作

### 添加已付款订单

当用户在闲鱼付款后，需要手动添加订单号到系统：

```bash
cd /root/.openclaw/workspace/web-dashboard
python3 manage-orders.py add <订单号>
```

**示例：**
```bash
python3 manage-orders.py add 123456789012
```

### 查看订单列表

```bash
python3 manage-orders.py list
```

### 查看统计信息

```bash
python3 manage-orders.py stats
```

---

## 📁 数据存储

### 测试结果
位置：`/root/.openclaw/workspace/memory/career-test/`
文件：`result-XXXXXXXX.json`（每个结果一个文件）

### 订单数据
位置：`/root/.openclaw/workspace/memory/career-test/orders.json`

**订单格式：**
```json
{
  "123456789012": {
    "status": "paid",
    "paid_at": "2026-03-13T14:05:05.254802",
    "source": "manual",
    "result_id": "A1B2C3D4"
  }
}
```

---

## 🔐 安全机制

### Token 验证
- 每个测试结果生成唯一的访问 token
- token 基于 result_id + 密钥的 SHA256 哈希
- 只有通过 token 才能访问完整报告

### 订单验证
- 订单号必须存在于系统才能查看报告
- 一个订单号只能关联一个测试结果
- 订单状态必须为 "paid"

---

## ⚙️ 服务管理

### 启动 API 服务
```bash
cd /root/.openclaw/workspace/web-dashboard
python3 career-test-api.py > /tmp/career-test-api.log 2>&1 &
```

### 停止 API 服务
```bash
pkill -f "career-test-api.py"
```

### 查看日志
```bash
tail -f /tmp/career-test-api.log
```

### 配置开机自启（可选）

创建 systemd 服务：
```bash
sudo tee /etc/systemd/system/career-test.service > /dev/null <<'EOF'
[Unit]
Description=Career Test API Service
After=network.target

[Service]
Type=simple
User=root
WorkingDirectory=/root/.openclaw/workspace/web-dashboard
ExecStart=/usr/bin/python3 career-test-api.py
Restart=always

[Install]
WantedBy=multi-user.target
EOF

sudo systemctl daemon-reload
sudo systemctl enable career-test
sudo systemctl start career-test
```

---

## 🎯 闲鱼自动发货配置

### 方案 1：手动发货（推荐起步）
1. 用户付款后，你在闲鱼发送消息：
   ```
   感谢购买！🎉
   
   请访问：www.myjobtest.cn/report.html
   输入订单号即可查看你的专属完整报告
   
   报告永久有效，可收藏保存~
   ```
2. 同时在后台添加订单：
   ```bash
   python3 manage-orders.py add <订单号>
   ```

### 方案 2：自动发货（需要闲鱼工具）
配置闲鱼自动发货工具，付款后自动：
1. 发送报告链接
2. 调用 API 添加订单

---

## 📊 数据备份

定期备份数据目录：
```bash
tar -czf career-test-backup-$(date +%Y%m%d).tar.gz \
    /root/.openclaw/workspace/memory/career-test/
```

---

## 🐛 故障排查

### API 服务无法访问
```bash
# 检查服务是否运行
ps aux | grep career-test-api

# 查看日志
tail -f /tmp/career-test-api.log

# 测试 API
curl http://localhost:5001/api/status
```

### nginx 配置问题
```bash
# 测试配置
/usr/sbin/nginx -t

# 重载配置
/usr/sbin/nginx -s reload

# 查看错误日志
tail -f /var/log/nginx/error.log
```

### 订单验证失败
1. 检查订单号是否正确
2. 确认订单已添加到系统
3. 查看 API 日志

---

## 📈 扩展建议

### 短期优化
- [ ] 添加订单号批量导入功能
- [ ] 增加订单状态查询页面
- [ ] 添加简单的数据统计后台

### 长期规划
- [ ] 接入闲鱼 API 自动验证订单
- [ ] 增加微信支付/支付宝支付
- [ ] 支持更多测试主题
- [ ] 添加用户账户系统

---

## 📞 技术支持

遇到问题检查：
1. API 服务是否运行
2. nginx 配置是否正确
3. 数据目录权限是否正常
4. 日志文件中的错误信息
