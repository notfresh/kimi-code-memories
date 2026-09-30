# Kimi Code Memories — 给 AI 协作者的操作约定

> **必读**:本仓库是 AX Graph(任务 0012)的**外部 base 实例**,**必须依赖 AX Graph 才能使用**。没有 AX Graph 工具,本仓库的 TOML 文件只是纯文本,无法被解析、查询、校验。
>
> AX Graph 仓库:`/root/projects/axgraph/`(详细文档见 `axgraph/docs/AGENTS.md`)
>
> 进入本目录后:先读本文件,再按本文件指引调用 `bin/ax`。

## 1. 这是什么

- **定位**:一个**独立仓库**,数据是关于 **MoonshotAI/kimi-code 开源仓库**的事实记忆(架构 / 原则 / 源码事实 / 读码笔记 / 坑点 / 教训等)。
- **依赖**:**必须依赖 [AX Graph](https://github.com/notfresh/axgraph)(本地路径 `/root/projects/axgraph/`)才能用**。本仓库不自带工具,所有解析/查询/校验都通过 AX Graph 的 `bin/ax` 命令完成。
- **维护者**:郑旭(Hermes 用户)。AI 是协作者,**人在回路**:新增/修改节点前必须由人发起。

## 2. 数据布局

```
kimi-code-memories/                                     ← 仓根(就是 AX Graph 的一个外部 base)
├── AGENTS.md                                           ← 本文件(给 AI 说明怎么用)
├── numbering-rule.txt                                  ← 编号规则(number 字段定义 + 使用规范)
└── .axgraph/
    └── base-dir-kimi-code-memories/                    ← AX Graph 实际数据目录
        ├── base.toml                                   ← AX Graph base 元信息(local_kinds 扩展在此)
        └── Layer-longlife.toml                         ← 长期语义层(principle / constraint / decision / invariant ...)
```

说明:
- 数据**实际位于 `.axgraph/base-dir-kimi-code-memories/`**(AX Graph 自动生成),**不在仓根平铺**。
- Layer 文件以仓的实际内容为准:本仓目前只有 `Layer-longlife.toml` 一个长期层文件;需要新层时再 `ax init` 增加。
- `Layer-*.toml.detail.toml` 是节点长笔记的扩展存储,**与主 Layer 文件独立处理**:number / id / kind 等结构化字段只在主文件维护,detail.toml 走另一套流程(详见 `numbering-rule.txt`)。

## 3. 怎么用(AX Graph 工具)

```bash
# 1. 直接 cd 到本仓根
cd /root/projects/kimi-code-memories

# 2. 用 AX Graph 的 ax 命令(工具会自动识别 cwd 下的 base.toml)
ax query --bases               # 确认本仓作为 base 已识别
ax <节点id>                    # 查节点
ax <关键词>                    # 模糊匹配
ax <id> -e                     # 人话解读
ax <id> -d                     # 末尾显示详细介绍
ax --validate                  # 全量校验
ax --validate <节点id>         # 单点校验
```

完整命令清单见 `axgraph/docs/AGENTS.md` §4。

## 4. schema 规范

**完全沿用 AX Graph**(`/root/projects/axgraph/docs/SCHEMA.md`)。本仓库**不另立 schema,不发明字段**。

- 节点必填:`id` / `kind` / `path` / `layer`
- kind 取值用 AX Graph 已支持的集合,优先长期层四类:`principle` / `constraint` / `decision` / `invariant`
- 本仓特有 kind 通过 `base.toml` 的 `[base] local_kinds = ["..."]` **局部扩展**(2026-05-25 引入,使用范例 `hypothesis`)。校验会合并全局 + 本地,无需改 axgraph 全局
- 边关系也直接用 AX Graph 的(`JUSTIFIES` / `DERIVED_FROM` / `CONFLICTS_WITH` 等)
- id 命名、校验规则、行号漂移处理:**全部参考 AX Graph 文档**

### 4.1 本仓扩展字段:`number`(口语引用号)

与 AX Graph 全局 schema 平行的、本仓特有的字段。**仅在主 Layer TOML 文件维护**,`Layer-*.toml.detail.toml` 不要求。

- **含义**:口语引用「第 N 条记忆」的短锚点;用户对话/笔记/9616 页面会用到
- **位置**:与 `id` 同级,字段顺序一般在 `id` 之后、`kind` 之前
- **类型**:正整数(int)
- **规则**:全仓唯一、严格递增、严禁跳号/重复/回填改号
- **与 id 的关系**:`id` 是机器字符串(唯一、不可改);`number` 是口语编号(本仓内、谨慎新增)。两者同级,`number` **不替代** `id`
- **使用规范**:完整规则(注释规范、格式约定、与 id 的关系)见 `numbering-rule.txt`
- **新增时的取号规则**:取当前最大 number + 1,新节点追加到 Layer 文件末尾

AI 进入本仓处理「第 N 条记忆」「number 是 X 的节点」之类的口语指令时:
1. **先查主 Layer TOML 的 `number` 字段**(不是 id,不是 status,不是其它)
2. Layer TOML 缺字段 = 视为「该节点无 number」,**不要用 id 序号或数组下标凑**
3. `Layer-*.detail.toml` 里**不要**查找或补 number 字段(独立处理,见 §2 数据布局)
4. 完整定义以 `numbering-rule.txt` 为准,本节是给 AI 的速查

## 5. 工作流程(新增一条记忆)

1. **读 schema**:AX Graph 的 `docs/SCHEMA.md`(`/root/projects/axgraph/docs/SCHEMA.md`)
2. **跟用户确认 kind 一次**(就一次)。默认是长期层四选一
3. **决定放哪个 Layer 文件**(新建或追加到现有的 `Layer-*.toml`)
4. **写节点 + 边**(严格按 AX Graph 字段)
5. **`ax --validate <节点id>`** → 0 Error
6. **`ax <节点id>` 验证输出**

## 6. 铁律

1. **不另写工具**——查询/校验全用 `bin/ax`
2. **schema 不发明**——字段、命名、校验全部沿用 AX Graph
3. **写入后必跑 `ax --validate`**
4. **冲突建 `CONFLICTS_WITH` 边,不静默覆盖**
5. **过时用 `SUPERSEDES` + `status="deprecated"`**
6. **人在回路**:AI 提议,人确认

## 7. 背景

- 项目创建:2026-09-30(周三)
- 是 AX Graph 框架的「外部 base 实例」,需要 AX Graph 工具配合使用
- 数据主题:**MoonshotAI/kimi-code 开源仓库**(架构 / 源码事实 / 读码笔记 / 坑点)
- 模板来源:仿 `/root/projects/hermes-agent-memories` 的结构与约定
- 任务编号:**待分配**(用户决定后填入)