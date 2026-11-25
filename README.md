# Scene Regions · 整图精细分区

把照片中的可见表面合并成适合施加特效的大区块，导出带编号的彩色预览、原图尺寸的黑白蒙版和机器可读的清单。道路、草地、人行道默认归为地面。同类对象可以共用一张蒙版，无明确主体的风景可以仅有天空、水面和山林。推理在本机 CPU 上运行。

## 安装与使用

下载 [完整技能包 scene-regions.zip](https://github.com/wyb0508/scene-regions/releases/latest/download/scene-regions.zip)。解压后进入 `scene-regions`，使用 Python 3.10–3.13 安装依赖。以下命令适用于 macOS 或 Linux。

```sh
python3 -m venv .venv
.venv/bin/python -m pip install -r requirements.txt
./run.sh "/path/to/photo.jpg" --out "/path/to/new-output"
```

在 Codex 里使用时，可将解压出的目录放到 `~/.codex/skills/scene-regions`，然后说「用 $scene-regions 给这张照片分区」。分组明确时直接交付。若某块暗区可能属于夜空或背景，先展示整图编号预览，再询问归属。

## 输出与衔接

- `preview.jpg` 展示整图分组和图例。
- `masks/*.png` 每组一张。白色属于该组，各张蒙版互斥，合起来覆盖整张原图。
- `manifest.json` 提供稳定的分组键、相对路径、尺寸、像素数和原图 SHA-256，用于下游读取。
- `regions.png` 保存本次输出的区块编号。`classes.png` 保存 ADE20K 类别 ID，可供天光天气壁纸 Agent 导入。

默认精细模式用 SegFormer 生成语义分组，再由 CascadePSP 和 SlimSAM 细化相邻区块的交界。快速预览可加 `--quality fast`。主体选择、同类拆分、图文栏和局部修正的参数见 [技能说明](SKILL.md)。

## 从源码运行

克隆后，将同版本完整技能包中的 `scene-regions/models/` 内容复制到仓库的 `models/`，再按上文安装依赖。运行回归测试可使用以下命令。

```sh
.venv/bin/python -m unittest discover -s tests
```

## 许可证与模型来源

本项目原创代码和文档采用 [MIT License](LICENSE)。发行包内的第三方权重及配套文件按各自上游许可分发。**其中 SegFormer 仅限非商业研究或评估**，使用随包模型时须遵守该条款。

| 模型 | 用途 | 来源与许可 |
| --- | --- | --- |
| SegFormer B0 / B2 | ADE20K 语义分区 | [B0 ONNX](https://huggingface.co/Xenova/segformer-b0-finetuned-ade-512-512/tree/d3e5499fa8701ff0453ca940a8dfeae39b2f1504) · [B2 ONNX](https://huggingface.co/Xenova/segformer-b2-finetuned-ade-512-512/tree/df795789e70f4089c8658907679c6fd2367c89a5) · [NVIDIA 条款](models/LICENSE-SegFormer.txt) |
| U²-NetP | 显著性与初始主体轮廓 | [U-2-Net](https://github.com/xuebinqin/U-2-Net) · [rembg 权重](https://github.com/danielgatis/rembg/releases/tag/v0.0.0) · [Apache 2.0](models/LICENSE-U2Net.txt) |
| SlimSAM | 提示点驱动的边界候选 | [ONNX 权重](https://huggingface.co/Xenova/slimsam-77-uniform/tree/5850ab45f587c112167512ffef949107115e26a0) · [Apache 2.0](models/refiners/LICENSE-SlimSAM.txt) |
| CascadePSP | 表面分组边界细化 | [ONNX 权重](https://huggingface.co/hkchengrex/CascadePSP-onnx/tree/a6b21ce1d485c1208650473234775114873d881c) · [MIT](models/refiners/LICENSE-CascadePSP.txt) |
