---
title: Prediction Service
weight: 1
description: Learn how to prepare data, call the prediction API, and interpret model outputs.
---

## 概述

预测模块提供统一的推理服务，将训练完成的模型部署为 HTTP API，支撑实时与批量业务场景。页面将介绍必要的准备工作、接口规范以及常见运维操作，帮助你快速完成集成。

## 核心能力

- **一体化部署**：模型包上传后自动完成容器构建与弹性扩缩。
- **多种输入格式**：支持 JSON、CSV 文件以及对象存储 URL。
- **版本管理**：单模型多版本并存，可按流量比例灰度。
- **监控告警**：内置请求延迟、成功率、资源利用率指标，并支持自定义阈值。

## 快速开始

1. **准备模型文件**：确保导出的模型符合平台支持的框架（TensorFlow SavedModel、PyTorch TorchScript 或 ONNX）。  
2. **上传模型版本**：通过 `prediction-cli upload` 或控制台上传模型包，并填写版本标签、资源规格。  
3. **发布服务**：上传成功后点击发布，系统会自动生成在线推理实例和测试 URL。  
4. **验证请求**：使用下方示例脚本发送推理请求，确认返回结果无误后即可在业务系统中调用。

## HTTP 接口

### 认证

所有请求需携带 `Authorization: Bearer <API_KEY>` 头部，可在控制台的「Access Tokens」页面生成。

### 推理请求

`POST /api/v1/prediction/{model_id}/infer`

请求示例：

```bash
curl -X POST "https://api.example.com/api/v1/prediction/credit-risk/infer" \
  -H "Authorization: Bearer $API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "inputs": [
      {"age": 35, "income": 120000, "loan_amount": 30000, "tenure": 24},
      {"age": 52, "income": 78000, "loan_amount": 18000, "tenure": 12}
    ],
    "options": {
      "explain": true,
      "confidence_threshold": 0.6
    }
  }'
```

返回结果：

```json
{
  "model_version": "credit-risk@20241104",
  "predictions": [
    {"score": 0.83, "label": "approve", "explanations": {"income": 0.42, "tenure": 0.18}},
    {"score": 0.27, "label": "reject", "explanations": {"loan_amount": -0.36, "age": -0.22}}
  ],
  "metrics": {"latency_ms": 48}
}
```

### 批量推理

`POST /api/v1/prediction/{model_id}/batch`

- 上传 CSV 文件或对象存储地址 (`s3://`、`oss://`)。  
- 返回任务 ID，可在「批量任务」页面查看处理进度与下载结果。

## 监控与告警

- **实时监控**：默认收集 QPS、P99 延迟、错误率，可在控制台图表查看或导出 Prometheus 指标。  
- **自定义阈值**：支持为成功率、延迟设置阈值，触发后通过邮件与钉钉机器人告警。  
- **日志检索**：集成集中日志系统，支持按请求 ID、模型版本过滤。

## 常见问题

- **返回 401**：检查 API Key 是否过期或未绑定对应模型。  
- **推理延迟高**：可以升级计算规格，或开启异步批量模式。  
- **模型加载失败**：确认依赖库版本与 `requirements.txt` 一致，必要时重新导出模型。

如需更多帮助，请联系平台运营团队或提交工单。
