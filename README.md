# 豆包 AI 聊天助手

基于 Vue 3 的智能对话界面，支持 DeepSeek 大模型、语音输入、自动朗读。

## 功能

- 智能对话（DeepSeek-V3.2）
- 语音输入（浏览器语音识别）
- 自动朗读（可开关，状态保存）
- 多轮上下文记忆

## 安装与运行

```bash
yarn install
yarn dev

## API 配置

默认接口：https://api.siliconflow.cn/v1/chat/completions

模型：deepseek-ai/DeepSeek-V3.2


## 使用说明

输入文字，按 Enter 发送

点击麦克风图标语音输入

点击「朗读」按钮开关自动播报

## 注意事项

语音识别需要 HTTPS 或 localhost 环境

API Key 为明文存储，生产环境建议后端代理
