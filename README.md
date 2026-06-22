> Status: Conceptual Design / Pre-implementation
# ⭐ GrepSVideo — 让 LLM 驱动像素，让 Rust 驾驭管道

> **一句话定位**：一个基于 Rust + CEF + FFmpeg 的 AI 视频生成引擎 —— 用自然语言描述视频，LLM 生成动画脚本，离屏渲染 + 零拷贝管道直出 4K 视频。

---

## ✨ 核心亮点

- **🤖 LLM 原生驱动**  
  你只需告诉它：“一个深蓝渐变背景，金色大字‘2026启航’，下方流动粒子线条”，它就能生成对应的 `.gsv` 脚本，无需任何人工编码。
- **⚡ 零拷贝像素管道**  
  基于 Rust 的异步管道（`tokio::process`）将 CEF 离屏渲染的 GPU 像素流 **直接喂给 FFmpeg**，显存常驻帧数 ≤ 2，彻底杜绝显存爆炸。
- **🎬 物理 + 纹理混合渲染**  
  支持 Three.js 物理引擎（重力/碰撞）与 AI 生成 PBR 纹理的混合模式，用 **“物理算运动，AI算皮囊”** 的极致分工，实现低成本真实感。
- **🦀 全栈 Rust 主控**  
  从 CEF 生命周期管理到 FFmpeg 子进程通信，全部由 Rust 的强类型和所有权系统守护，杜绝内存泄漏和并发死锁。
- **📦 自定义 .gsv 脚本格式**  
  头部 JSON 定义元数据（分辨率/帧率/时长/音轨），主体为 JavaScript 动画逻辑（支持 Three.js / Canvas），完全解耦内容与引擎。

---

## 🧠 技术架构（核心流水线）

```
用户输入 → LLM (DeepSeek/Claude) → 生成 .gsv 脚本
                                   ↓
                          GrepSVideo 主控 (Rust)
                                   ↓
                  ┌────────────────┴────────────────┐
                  │                                  │
           CEF 离屏渲染 (4K)               FFmpeg 编码管道
           (GPU 显存中保持 ≤3帧)     (NVENC/VAAPI 硬件加速)
                  │                                  │
                  └────────────────┬────────────────┘
                                   ↓
                           音视频混流 → 输出 .mp4/.mkv
```

- **CEF 离屏渲染**：视口强制 3840×2160，通过 `OnPaint` 回调吐出原生 4K RGBA 缓冲。
- **Pipeline 零拷贝**：`OnPaint` 缓冲指针通过无锁通道传递给 FFmpeg 的 `stdin`，数据不落 CPU 内存。
- **预览与录制解耦**：预览窗口仅显示缩放副本，录制 Pipeline 独立稳定运行。

---

## 🛠️ 技术栈

| 组件 | 技术选型 |
|------|----------|
| 主控语言 | Rust (async tokio 运行时) |
| 浏览器渲染 | Chromium Embedded Framework (CEF) |
| 视频编码 | FFmpeg (libavcodec with NVENC/VAAPI) |
| 3D/动画引擎 | Three.js / Canvas (由 LLM 生成) |
| LLM 接口 | DeepSeek / OpenAI / Claude API |
| 脚本格式 | `.gsv` (JSON 元数据 + JavaScript 逻辑) |
| TUI 界面 | ratatui 或 cursive (可选) |

---

## 🚀 快速启动（示例）

```bash
# 1. 克隆项目
git clone https://github.com/yourname/GrepSVideo.git
cd GrepSVideo

# 2. 编译 Rust 主控 (自动下载 CEF 和 FFmpeg 依赖)
cargo build --release

# 3. 运行：指定 LLM 生成的 .gsv 脚本
./target/release/grepsvideo -i demo.gsv -o output.mp4

# 4. 或直接让 LLM 在线生成（需配置 API Key）
./grepsvideo --prompt "展示产品三大特点，科技感蓝色背景，文字快闪" --output product.mp4
```

---

## 📂 .gsv 脚本示例

```json
{
  "width": 3840,
  "height": 2160,
  "fps": 30,
  "duration": 15,
  "audio": "background.mp3",
  "script": "// JavaScript/Three.js 动画代码\nconst scene = new THREE.Scene();\n..."
}
```

---

## 🧪 为什么是 Rust？

- **内存安全**：CEF 的 C++ 回调极易发生 UAF，Rust 通过生命周期和所有权将其隔绝在 `unsafe` 边界内。
- **零成本抽象**：`async/await` 让管道背压控制极其优雅，无需手动信号量。
- **跨平台**：Windows/Linux/macOS 一套代码，二进制分发方便。

---

## 🌟 未来规划

- [ ] 支持 `.gsv` 热重载，实时预览脚本修改效果
- [ ] 集成 AI 纹理生成（Stable Diffusion）作为视频内贴图
- [ ] 提供 Web 版（WASM 编译主控核心）
- [ ] 插件系统：支持自定义 FFmpeg 滤镜和 CEF 扩展

---

## 🤝 贡献

本项目处于 **概念验证（PoC）** 阶段，欢迎对 Rust / CEF / FFmpeg 感兴趣的极客一起造轮子！  
Issue / PR 请遵循 [Rust Code of Conduct](https://www.rust-lang.org/policies/code-of-conduct)。

---

## 📄 许可证

MIT / Apache-2.0 双许可

---

**“我们用 Rust 驯服了 GPU 的野性，用 LLM 解放了创意的枷锁。”**
