# resources — GEBAI 资源子仓库

模型与运行时原生依赖等大体积资源的集中存放地（独立 git 仓库，主仓库通过 `.gitignore` 的
`/resources/` 与本仓库隔离）。

位置约定：`{GEBAI_HOME}/resources/`
（源码/dev 形态 = 项目根目录下的 `resources/`；单二进制形态 = `~/.gebai/resources/`）。

## 目录结构

两种形态共用同一套结构：源码/dev 形态直接读本仓库；单二进制形态把构建期内嵌的资源释放到
`{GEBAI_HOME}/resources/` 下的相同相对路径（见「二进制形态」）——解析路径与 drop-in 规则因此
在两种形态下完全一致。

```
models/                      模型资产（按领域分组，放入即自动发现）
  cv/
    ocr/                     OCR（PP-OCRv4 mobile）三件套（GEBAI_CV_MODELS_DIR 缺省目录）
      det.onnx               文本检测 ONNX（~4.7MB，RapidOCR 托管，Apache-2.0）
      rec.onnx               文本识别 ONNX（~10.9MB；字典内嵌于 character 元数据）
      dict.txt               CTC 字符表（从 rec 元数据提取，可手工替换）
    detect/                  检测模型目录（GEBAI_CV_DETECT_MODEL 指向此处的 ONNX）
      screenparser-best.pt       训练权重源文件（ScreenParser，docling-project，Apache-2.0）
      screenparser-best.onnx     运行时 ONNX（yolo export format=onnx imgsz=1280 导出；55 类 UI 组件）
vendor/                      运行时依赖（非模型资产）
  cv/                        单二进制形态释放的 CV 运行时：ort wasm 入口与本体、cv-driver.mjs
  node_modules/              GPU sidecar 的原生推理包依赖闭包（onnxruntime-node + onnxruntime-common
                             等 16 包，npm 布局——依赖经 node 标准向上查找天然可用）
```

## 获取资源

三种等价方式（任选其一，产物目录结构相同）：

1. **主仓库脚本（推荐）**：`bun run resources:download` —— 按主仓库清单 `scripts/resources.manifest.json`
   逐文件下载（modelscope / hf-mirror / huggingface 多源轮换、断点续传、size/sha256 校验），
   并按本仓库相同结构铺开到 `{GEBAI_HOME}/resources/`；`--check` 只校验现状、`--only` 限定条目、
   `--source` 指定来源优先级、`--skip-vendor` 跳过 vendor 依赖
2. **克隆本仓库**：整仓放到 `{GEBAI_HOME}/resources/`（权重走 Git LFS，克隆后 `git lfs pull`）
3. **手工放置**：按下方目录结构自行放入（文件名与目录固定）

下载清单与脚本属主仓库（`scripts/`），本仓库只存资源本体。

## 使用

**整个仓库放到 `{GEBAI_HOME}/resources/` 即零配置可用**（dev=项目根目录，二进制=`~/.gebai`）：

- 检测模型：`models/cv/detect/` 下唯一 `.onnx` 自动发现（`GEBAI_CV_DETECT_MODEL` 可覆盖；放多个时列出候选要求显式指定）
- OCR 模型：`models/cv/ocr/` 三件套自动生效（解析顺序 `GEBAI_CV_MODELS_DIR` → 二进制内嵌释放 → 本目录），构建时缺失则自动下载到此处
- GPU sidecar 原生依赖：`vendor/node_modules/` 自动解析（免任何环境变量；检测与 OCR 推理共用）

## 二进制形态（bun --compile 单二进制）

构建期内嵌的 CV 资产在运行时释放到 `{GEBAI_HOME}/resources/`，相对结构与本仓库一致：

- PP-OCR 三件套 → `models/cv/ocr/`：**文件已存在则保留不覆盖**，用户替换或自备的模型优先
- ort wasm 运行时（入口 + 本体）与 `cv-driver.mjs` → `vendor/cv/`：版本 marker 为
  `vendor/cv.version`，版本变化时重建该目录
- 检测模型不内嵌：把 ONNX 放入 `models/cv/detect/` 即生效（与源码形态同一路径）

## 更换 / 补充模型

```bash
# 例：新检测模型放入 models/cv/detect/ 后，.env 的 GEBAI_CV_DETECT_MODEL 改指新 ONNX 即可
# .pt → ONNX 导出（ultralytics）：
yolo export model=<name>.pt format=onnx imgsz=1280
```

## 版本管理说明

- 本仓库为嵌套独立仓库（主仓库 `.gitignore` 的 `/resources/` 隔离）：`git add . && git commit`
  在本目录内提交，不进主仓库历史
- 远端：`https://github.com/xuxinle/gebai-resources`（公开）——`*.pt`/`*.onnx` 已走 **Git LFS**
  （超过 GitHub 100MB 单文件硬限制）；克隆后 `git lfs pull` 拉取权重本体（若检出文件为 ~134B
  指针文本，说明 LFS 未安装或未 smudge——装好后 `git lfs checkout` 还原）
- 推送走代理时：`git -c http.proxy=http://127.0.0.1:29290 push origin master`
- `vendor/node_modules/onnxruntime-node` 的 npm 包内含全平台原生绑定（darwin/linux/win32 的
  x64/arm64），可直接跨平台使用；升级版本时在可联网机器
  `npm install --prefix <临时目录> onnxruntime-node` 后整体替换 `vendor/node_modules/`
