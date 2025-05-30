# 网站维护页面

这是一个使用Vite + Vue 3 + TypeScript构建的网站维护页面，适用于Vercel部署。

## 功能特性
- 从TextDB在线存储获取维护状态数据
- 实时显示维护状态（即将开始/进行中/已完成）
- 显示维护计划起止时间
- 动态显示当前时间
- 剩余时间倒计时功能
- 响应式现代化设计
- 加载和错误状态处理

## 配置说明
1. 复制.env.example为.env文件
2. 在TextDB创建维护数据，格式如下：
```json
{
  "example_com_20250530_1": {
    "status": "进行中",
    "startTime": "2025-05-30T10:00:00", 
    "endTime": "2025-05-30T18:00:00"
  },
  "another_com_20250530_1": {
    "status": "即将",
    "startTime": "2025-05-30T12:00:00",
    "endTime": "2025-05-30T20:00:00"
  }
}
```
3. 在.env中设置：
   - TEXTDB_ID: TextDB共享码
   - MAINTENANCE_ID: 要显示的维护信息ID (格式: domain_time_rev)

## 开发运行
```bash
npm install
npm run dev
```

## 构建部署
```bash
npm run build
```
项目已配置vercel.json，可直接部署到Vercel平台。

## 自定义样式
修改`src/App.vue`中的`<style>`部分来调整页面样式。

## 数据更新
只需更新TextDB中的JSON数据，页面将自动获取最新状态。
