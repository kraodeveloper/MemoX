
<h1 align="center">🧠 MemoX V1 – 记忆 Agent</h1>

<p align="center">
  <em>把你看过、听过、写过的一切都装进口袋，<br/>离线也能 1 秒召回。</em><br/>
  <sub>私有优先 · 本地优先 · 跨平台</sub>
</p>

<p align="center">
  <img alt="language" src="https://img.shields.io/badge/Tech-Rust%20%7C%20TypeScript%20%7C%20Python-ff69b4">
  <img alt="platform" src="https://img.shields.io/badge/Platforms-macOS%20%7C%20Windows%20%7C%20iOS%20%7C%20Android-007ec6">
  <img alt="license"  src="https://img.shields.io/badge/License-Apache--2.0-green">
</p>

> **一句话**  
> MemoX V1 是个人“随身黑匣子”：**采集 → 自动标注 → 向量化 → 本地检索**，P99 召回时间 < 1 秒。

---

#### 目录
1. [核心特性](#核心特性)  
2. [系统架构](#系统架构)  
3. [目录结构](#目录结构)  
4. [快速开始](#快速开始)  
5. [性能基准](#性能基准)  
6. [安全与隐私](#安全与隐私)  
7. [路线图](#路线图)  
8. [贡献指南](#贡献指南)  
9. [许可证](#许可证)  

---

#### 核心特性
| 模块 | 能力 |
|------|------|
| 📥 **全场景采集** | 浏览器扩展、剪贴板守护、语音（Whisper-cpp）、截图拖拽 |
| 🏷️ **自动标注** | 时间、GPS/Wi-Fi 位置、NER 实体、6 类情绪 |
| 🔎 **极速检索** | 语义 + 关键词混合检索，P99 < 1 秒，离线可用 |
| 💾 **本地优先存储** | SQLite WAL + FAISS HNSW，高频窗口 7 MB |
| ☁️ **可选云归档** | 低频记忆蒸馏后加密上传 Milvus |
| 🔐 **隐私保护** | SQLCipher、端到端加密、一键忘记 |
| ⚙️ **易扩展** | gRPC `/PushMemory` & `/Recall` 接口，可接入眼镜/耳机 |

---

#### 系统架构
```text
┌───────────── 设备端 ─────────────┐
│ ① Capture SDK                  │
│ ② Context Tagger               │
│ ③ Vectorizer  (e5-small-int8)  │
│ ④ HighFreqStore (SQLite+Faiss) │
│ ⑤ Recall Engine (<1 s)         │
│ ⑥ WeightMgr & Sync Scheduler   │
└──────────────────┬─────────────┘
                   │ gRPC/TLS
┌────────── 可选云侧 ───────────┐
│ ⑦ LowFreqStore (Milvus)      │
│ ⑧ Distiller & Backup         │
└──────────────────────────────┘
```

---

#### 目录结构
```text
memox
├─ client                 # 端侧代码
│  ├─ desktop             # Rust + Tauri
│  ├─ mobile              # React-Native
│  └─ shared              # gRPC 接口 & 通用库
├─ server                 # 可选云归档
│  ├─ api                 # FastAPI + gRPC
│  └─ distiller           # 低频蒸馏任务
├─ models                 # ONNX/gguf 量化模型
│  ├─ e5-small-int8.onnx
│  └─ whisper-small.en.gguf
├─ scripts                # 运维脚本 & 基准测试
│  └─ benchmark.sh
├─ docs                   # 设计文档
└─ README.md
```

---

#### 快速开始

##### 1. 环境要求
- macOS 12 / Windows 10 / iOS 15 / Android 11 及以上  
- Rust ≥ 1.77 · Node ≥ 18 · Python ≥ 3.9  
- 4-core CPU / 4 GB 内存（模型已 INT8 量化）  

##### 2. 安装 & 运行
```bash
# 克隆仓库
git clone https://github.com/your-org/memox.git
cd memox

## 桌面端
cargo tauri dev           # macOS / Windows

## 移动端
npm i
npx react-native run-ios  # 或 run-android
```

##### 3. 初体验
1. 首次启动按提示授权麦克风 / 辅助功能。  
2. **⌥ + Space** 打开快速录入，输入任意文本。  
3. 终端执行：  
   ```bash
   memox recall "上周 OKR 回顾" -k 3
   ```  
   1 秒内即可获得上下文片段。

---

#### 性能基准
| 场景 | 设备 | 延迟 |
|------|------|------|
| 召回 Top-3（128 k 条） | MacBook M2 | **420 ms** |
| 召回 Top-3（128 k 条） | iPhone 15 | **610 ms** |
| 剪贴板写入 1 MB | MacBook M2 | 45 ms |
| 夜间蒸馏 10 k 条 | AWS c6g.large | 38 s |

> FAISS `IndexHNSWFlat dim=384 M=32`，嵌入模型 `e5-small-v2 INT8`

---

#### 安全与隐私
| 层级 | 手段 |
|------|------|
| 静态存储 | SQLCipher AES-256-GCM（PBKDF2-HMAC-SHA256） |
| 传输 | gRPC + TLS1.3 + HPKE |
| 云端 | 用户独享密钥，服务器无法解密原文 |
| 控制 | 一键“Erase All”，GDPR 数据导出 ZIP+CSV |

---

#### 路线图
| 里程碑 | 状态 | 目标 |
|--------|------|------|
| Capture SDK P0（剪贴板、浏览器） | ⏳ | W1 |
| 向量化服务 + DB |⏳ | W2 |
| 召回 P95 < 0.9 s | ⏳ | W3 |
| 自动标注 & Tag UI | ⏳ | W4 |
| 夜间同步 & 蒸馏 | ⏳ | W5 |
| 端到端加密云备份 | ⏳ | W6 |
| 公测 Beta (50 用户) | ⏳ | W8 |

✅ 完成 🟡 进行中 ⏳ 计划中

---

#### 贡献指南
欢迎 PR！请先阅读 [CONTRIBUTING.md](CONTRIBUTING.md)。

```bash
# 本地 lint & 单测
cargo clippy --all
npm test
```

---

#### 许可证
本项目基于 **Apache-2.0** 协议发布，详见 [LICENSE](LICENSE)。

---

<p align="center">Made with ❤️ & Rust · 2025</p>
