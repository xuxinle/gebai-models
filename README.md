# models — GEBAI 资源子仓库

模型等大体积资源文件的集中存放地（独立 git 仓库，主仓库通过 `.gitignore` 的 `/models/` 与本仓库隔离）。
位置约定：`{GEBAI_HOME}/models/`（源码/dev 形态 = 项目根目录；单二进制形态 = `~/.gebai/models/`）。

## 目录约定

```
detect/               检测模型（GEBAI_CV_DETECT_MODEL 指向此处的 ONNX）
  screenparser-best.pt     训练权重源文件（ScreenParser，docling-project，Apache-2.0）
  screenparser-best.onnx   运行时 ONNX（yolo export format=onnx imgsz=1280 导出；55 类 UI 组件）
vendor/               运行时原生依赖（体积大、网络受限环境难获取，随本仓库分发）
  onnxruntime-node/   GPU sidecar 的原生推理包（npm onnxruntime-node，含平台二进制；
                      sidecar 按 {GEBAI_HOME}/models/vendor/onnxruntime-node 自动解析，免 .env 配置）
```

## 使用

- 检测模型：主仓库 `.env` 设 `GEBAI_CV_DETECT_MODEL=<绝对路径>/models/detect/screenparser-best.onnx`
- GPU sidecar：无需配置——放入 `vendor/onnxruntime-node/` 即自动生效（解析顺序见主仓库 `core/cv/sidecar.ts`）
- OCR（PP-OCR）不在此处：随主仓库 `packages/server/assets/cv-models/` 构建内嵌（既有管线）

## 更换 / 补充模型

```bash
# 例：新检测模型放入 detect/ 后，.env 的 GEBAI_CV_DETECT_MODEL 改指新 ONNX 即可
# .pt → ONNX 导出（ultralytics）：
yolo export model=<name>.pt format=onnx imgsz=1280
```

## 版本管理说明

- 本仓库为嵌套独立仓库：`git add . && git commit` 在本目录内提交，不进主仓库历史
- 需要远端备份时在本目录 `git remote add origin <url>` 推送；若托管平台对大文件有限制可转 Git LFS
- `vendor/onnxruntime-node` 的 npm 包内含全平台原生绑定（darwin/linux/win32 的 x64/arm64），可直接跨平台使用；升级版本时在可联网机器 `npm pack onnxruntime-node` 解包替换
