# models — GEBAI 资源子仓库

模型等大体积资源文件的集中存放地（独立 git 仓库，主仓库通过 `.gitignore` 的 `/models/` 与本仓库隔离）。
位置约定：`{GEBAI_HOME}/models/`（源码/dev 形态 = 项目根目录；单二进制形态 = `~/.gebai/models/`）。

## 目录约定

```
detect/               检测模型（GEBAI_CV_DETECT_MODEL 指向此处的 ONNX）
  screenparser-best.pt     训练权重源文件（ScreenParser，docling-project，Apache-2.0）
  screenparser-best.onnx   运行时 ONNX（yolo export format=onnx imgsz=1280 导出；55 类 UI 组件）
vendor/               运行时原生依赖（体积大、网络受限环境难获取，随本仓库分发）
  node_modules/       GPU sidecar 的原生推理包完整依赖闭包（onnxruntime-node + onnxruntime-common
                      等 16 包，npm 布局——依赖经 node 标准向上查找天然可用；
                      sidecar 按 {GEBAI_HOME}/models/vendor/node_modules 自动解析，免 .env 配置）
```

## 使用

**整个仓库放到 `{GEBAI_HOME}/models/` 即零配置可用**（dev=项目根目录，二进制=`~/.gebai`）：

- 检测模型：`detect/` 下唯一 `.onnx` 自动发现（`GEBAI_CV_DETECT_MODEL` 可覆盖；放多个时列出候选要求显式指定）
- GPU sidecar：`vendor/node_modules/` 自动解析（免任何环境变量；检测与 OCR 推理共用）
- OCR（PP-OCR）不在此处：随主仓库 `packages/server/assets/cv-models/` 构建内嵌（既有管线）

## 更换 / 补充模型

```bash
# 例：新检测模型放入 detect/ 后，.env 的 GEBAI_CV_DETECT_MODEL 改指新 ONNX 即可
# .pt → ONNX 导出（ultralytics）：
yolo export model=<name>.pt format=onnx imgsz=1280
```

## 版本管理说明

- 本仓库为嵌套独立仓库（主仓库 `.gitignore` 的 `/models/` 隔离）：`git add . && git commit` 在本目录内提交，不进主仓库历史
- 远端备份：`https://github.com/xuxinle/gebai-models`（私有）——`*.pt`/`*.onnx` 已走 **Git LFS**（超过 GitHub 100MB 单文件硬限制）；克隆后 `git lfs pull` 拉取权重本体（若检出文件为 ~134B 指针文本，说明 LFS 未安装或未 smudge——装好后 `git lfs checkout` 还原）
- 推送走代理时：`git -c http.proxy=http://127.0.0.1:29290 push origin master`
- `vendor/node_modules/onnxruntime-node` 的 npm 包内含全平台原生绑定（darwin/linux/win32 的 x64/arm64），可直接跨平台使用；升级版本时在可联网机器 `npm install --prefix <临时目录> onnxruntime-node` 后整体替换 `vendor/node_modules/`
