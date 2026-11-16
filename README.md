# 故事图谱 StoryAtlas

> 自己定义人物、情节与关系；应用自动梳理人物关联、指出剧情矛盾，并在需要时帮你生成草稿。

[![License: MIT](https://img.shields.io/badge/License-MIT-green.svg)](LICENSE)
[![CI](https://github.com/storyatlas-app/storyatlas/actions/workflows/ci.yml/badge.svg)](https://github.com/storyatlas-app/storyatlas/actions/workflows/ci.yml)

StoryAtlas 面向小说作者、跑团主持（GM）、编剧与设定爱好者。它**不自带任何剧情或示例人物**——所有内容都由你创建；应用负责把你写下的东西连起来，并告诉你哪里可能不自洽。

三条主线：

| 你在做 | 应用在做 |
| --- | --- |
| 定义人物（性格、外貌、背景、目标、生卒年） | 识别人名与别名，建立人物档案 |
| 写情节（时间、地点、正文） | 从左到右读上下文，判断谁和谁是什么关系 |
| 补关系、改设定 | **体检**：指出时间线矛盾、关系冲突、性格与行为不符 |

## 功能

### 1. 完全自定义的世界

- **人物**：姓名、别名、一句话简介、性格标签（内置预设 + 任意自定义）、外貌、背景、目标/动机、出生年、去世年、代表色
- **情节**：标题、发生时间、地点、正文、关联人物、标签
- **关系**：9 类（爱情 / 亲情 / 友情 / 师徒 / 同事 / 上下级 / 敌对 / 同盟 / 其他），强度 0–100，有向类型支持方向

### 2. 基于上下文的人物关联

本地规则引擎（纯 Dart、确定性、可测试）分三层判断：

1. **同篇章共现**：两人出现在同一情节即建立联系；提及距离越近、跨篇重复越多，权重越高；与主角共现额外加权
2. **关系关键词**：中文词汇表覆盖 9 类关系。关键词必须满足两个条件才计入——
   - 不属于任何人名的内部（「苏婉娘」里的「娘」不算亲情）
   - 归属于离它最近的两位在场人物，且落在两人的提及区间内
3. **方向推断**：有向关系默认「先提及者为主动方」，遇到反向句式（「拜…为师」「被…害死」「X 的下属 Y」）会定位标记词后最近的人物来翻转方向

最终按 **手动 > AI > 自动** 的优先级合并，同一对人物只保留一条；自动分析产生的噪声边可以一键忽略。

可选接入 OpenAI 兼容接口（DeepSeek、通义千问等）做语义分析，本地规则始终兜底。

### 3. 剧情体检：指出哪里可能不妥

本地体检引擎（`ConsistencyChecker`）会给出分级的检查结果，每条都带**具体说明 + 修改建议 + 可跳转对象**：

| 类别 | 检查项 |
| --- | --- |
| 时间线 | 情节早于人物出生、人物去世后仍出场、出生年晚于去世年、同年多篇情节缺少月日 |
| 人物关系 | 师徒/上下级互相指向、关系指向已删除的人物、手动关系与正文证据不符、同一对人物关系前后矛盾（先爱情后敌对） |
| 人物设定 | 姓名/别名被多个人物共用、性格标签自相矛盾、标「温和」却反复主动施暴、人物完全没有设定 |
| 情节 | 正文过短、没有识别到任何人物、勾选了人物但正文没提到、情节没有时间 |
| 完整度 | 人物从未进入任何情节（孤岛角色） |

体检结果是**建议而非判决**：刻意写的反转、多面性格都很正常，应用只提示你复核。

### 4. AI 自动生成

配置好接口后可以：

- **生成人物**：给出题材与要求，一次生成 1–8 个有区分度的角色
- **生成情节**：选定出场人物与时间，生成一篇严格只使用这些角色的情节草稿
- **润色正文**：保持原意与人物设定，修正语病、增强画面感

生成的内容会直接进入你的项目，可以继续手动修改。未配置 AI 时，分析与体检功能完全可用。

### 5. 三个视图

- **时间交际线**：按年份分组的情节节点，逐篇展示「谁 ⇄ 谁 · 类型 · 原文证据」
- **思维视图**：以任意人物为根的径向思维导图，两层关系 + 该人物的情节；点节点可切换根
- **人物关系网**：力导向布局，边按类型着色、粗细按强度、有向关系带箭头；可缩放平移、点选查看、以某人为中心重排

## 架构

「引擎 + 应用壳」单一仓库结构。核心引擎**零运行时依赖、纯 Dart**，可独立发布复用：

```
storyatlas/
├── packages/storyatlas_core/      # 核心引擎（pub: storyatlas_core）
│   ├── lib/src/models/            #   人物 / 情节 / 关系 / 项目数据
│   ├── lib/src/analysis/          #   提及识别、关键词、归属过滤、关系合并、AI 分析与生成
│   ├── lib/src/check/             #   体检规则与问题模型
│   ├── lib/src/layout/            #   力导向布局、思维导图径向布局
│   └── test/                      #   77 项单元测试
├── lib/                           # Flutter 应用壳
│   ├── src/state/                 #   全局状态、自动保存、体检联动
│   ├── src/services/              #   JSON 文件存储、OpenAI 兼容客户端
│   ├── src/ui/                    #   人物 / 情节 / 时间线 / 思维图 / 关系网 / 体检 / 创作 / 设置
│   └── src/painters/              #   关系网与思维图的自绘画笔
└── .github/workflows/ci.yml       # 分析 + 测试 + 三平台构建
```

## 快速开始

需要 Flutter ≥ 3.35（Dart ≥ 3.3）。

```bash
git clone https://github.com/storyatlas-app/storyatlas.git
cd storyatlas
flutter pub get

# 仓库不提交平台目录，首次运行前生成所需平台工程
flutter create --org app.storyatlas --project-name storyatlas --platforms android .
flutter run
```

只跑核心引擎（不需要 Flutter SDK，仅需 Dart ≥ 3.3）：

```bash
cd packages/storyatlas_core
dart pub get && dart analyze && dart test
```

### 打包 APK / 桌面版

```bash
flutter create --org app.storyatlas --project-name storyatlas --platforms android .
flutter build apk --release
# 产物：build/app/outputs/flutter-apk/app-release.apk
```

推送到 GitHub 后，[CI 工作流](.github/workflows/ci.yml) 会自动执行静态分析、全部测试，并构建 **Android APK / Linux / Windows** 三平台 Release 产物（Actions → Artifacts）。

## 使用流程

1. **建人物**：姓名是正文识别的关键，别名用来兼容不同称呼。性格标签会参与体检。
2. **写情节**：正文里直接写人物名字。保存后立即分析关系并重跑体检。
3. **看视图**：时间线看先后，思维图看某个人的世界，关系网看全局。
4. **体检**：顶栏 `✓` 图标，红点是问题数。点条目直接跳到要改的地方。
5. **AI**：顶栏 `✨` 图标。先生成人物，再生成情节，最后可润色。
6. **修正关系**：自动结果只是参考，可在关系列表或人物详情「手动修正 / 删除 / 忽略」，手动关系优先级最高。

## 数据存储

数据保存在应用支持目录的 `storyatlas/project.json`，修改后 600 ms 内自动保存。设置页可导出 / 导入 JSON 备份（`schemaVersion: 2`，旧版数据可向前兼容导入）。

## 测试

```bash
cd packages/storyatlas_core && dart test   # 引擎、体检、布局、AI 解析
flutter test                                # 组件测试
```

## 路线图

- [ ] 节点拖拽手动微调关系网布局
- [ ] 体检规则可配置（开关、阈值）
- [ ] 关系与设定的修改历史 / 撤销
- [ ] 更多语言的关键词表与界面 i18n
- [ ] 导出 PNG / 长图分享
- [ ] 桌面端原生文件选择器

欢迎提 Issue 与 PR，详见 [CONTRIBUTING.md](CONTRIBUTING.md)。

## 许可证

[MIT](LICENSE) © 2026 StoryAtlas Contributors
