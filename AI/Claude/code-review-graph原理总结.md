# code-review-graph 核心原理总结

## 目录

1. [快速入门](#一快速入门)
2. [快速更新代码调用链的原理](#二快速更新代码调用链的原理)
3. [Tree-sitter 技术原理](#三tree-sitter-技术原理)
4. [BFS 广度优先搜索（最多 2 跳）](#四bfs-广度优先搜索最多-2-跳)
5. [全量构建 vs 增量更新](#五全量构建-vs-增量更新)
6. [调用链查询（超过 2 跳的处理）](#六调用链查询超过-2-跳的处理)
7. [安装与初始化流程](#七安装与初始化流程)
8. [性能优势与关键配置](#八性能优势与关键配置)
9. [参考文献](#九参考文献)

---

## 一、快速入门

### 第一步：安装 code-review-graph

```bash
# 使用 pip 安装
pip install code-review-graph

# 或使用 pipx 安装（推荐，全局安装并隔离环境）
pipx install code-review-graph
```

### 第二步：在项目中初始化

```bash
cd /path/to/your/project

# 自动检测并配置所有支持的平台（推荐）
code-review-graph install
```

### 第三步：构建知识图谱

```bash
# 完整解析代码库并构建知识图谱
code-review-graph build
```

### 第四步：重启 AI 编码工具

重启 Claude Code、Cursor 或其他工具以加载新配置。

---

## 二、快速更新代码调用链的原理

### 1. 增量更新机制

通过 **Git/SVN 差异检测** 识别变更文件，而非重新解析整个代码库：

```python
# 检测变更文件
changed_files = get_changed_files(repo_root, base)  # 默认对比 HEAD~1

# 查找依赖文件（最多 2 跳）
for rel_path in changed_files:
    deps = find_dependents(store, full_path)  # BFS 遍历依赖关系
```

**关键实现**：`code-review-graph/code_review_graph/incremental.py#L919`

### 2. 依赖追踪算法

通过 **BFS 广度优先搜索** 追踪文件间的依赖关系：

- **单跳依赖**：查找直接导入或调用目标文件的文件
- **多跳扩展**：默认 2 跳深度，防止过度扩展
- **智能截断**：超过 500 个依赖文件时自动截断

**依赖类型**：`IMPORTS_FROM`、`CALLS`、`INHERITS`、`IMPLEMENTS`

### 3. 哈希校验优化

每个文件存储 **SHA256 哈希值**，更新时跳过未真正变化的文件：

```python
raw = abs_path.read_bytes()
fhash = hashlib.sha256(raw).hexdigest()
existing_nodes = store.get_nodes_by_file(str(abs_path))
if existing_nodes and existing_nodes[0].file_hash == fhash:
    continue  # 跳过未变文件
```

### 4. 高效的图存储引擎

使用 **SQLite + WAL 模式**：
- WAL 模式提升并发写入性能
- 索引优化：`idx_nodes_file`、`idx_edges_target` 等
- 批量操作：`store_file_batch()` 支持事务批量写入

#### WAL 模式原理

WAL（Write-Ahead Logging，预写日志）是 SQLite 的高性能事务日志模式，核心原理是**分离读写操作**和**顺序写入**：

**传统模式 vs WAL 模式**：

| 特性 | 传统模式 | WAL 模式 |
|------|---------|----------|
| **写入方式** | 直接修改数据库文件 | 追加写入 WAL 文件 |
| **读写并发** | 写阻塞读 | 读写并行 |
| **锁粒度** | 数据库级 | 行级 |
| **磁盘 I/O** | 随机写 | 顺序写（WAL） |

### 传统模式 vs WAL 模式写入流程对比

#### 传统模式写入流程：
```
┌─────────────────────────────────────────────────────────────┐
│ 1. 开始事务                                                  │
│ 2. 将原始数据写入 rollback journal（备份）                    │
│ 3. 直接修改主数据库文件（随机写，性能低）                      │
│ 4. 提交事务                                                  │
│ 5. 删除 rollback journal                                     │
└─────────────────────────────────────────────────────────────┘
```

**问题**：
- 写操作需要锁定整个数据库
- 读操作必须等待写操作完成
- 随机写性能低
- 每次写入都备份原数据

#### WAL 模式写入流程：
```
┌─────────────────────────────────────────────────────────────┐
│ 1. 开始事务                                                  │
│ 2. 将修改内容追加写入 WAL 文件（顺序写入，性能高）              │
│ 3. 提交事务（写入 commit 记录）                               │
│ 4. 后台 checkpointer 异步将 WAL 内容合并到主数据库             │
└─────────────────────────────────────────────────────────────┘
```

**优势**：
- 写操作不阻塞读操作
- 顺序写性能远高于随机写
- 只有追加操作，减少磁盘 I/O
- Checkpoint 可以异步后台执行

**关键机制**：

1. **双文件架构**：
   - `database.db`：主数据库文件
   - `database.db-wal`：WAL 文件（存储未合并的增量修改）

2. **读写并发实现**：
   - 写操作：仅写入 WAL 文件，获得写锁后立即释放
   - 读操作：从主数据库 + WAL 文件合并读取，不受写锁影响

3. **Checkpoint 机制**：
   - 自动触发：WAL 文件达到主数据库的 1/3 或事务数达到 1000
   - 后台异步将 WAL 内容合并到主数据库

**项目中的配置**：`code-review-graph/code_review_graph/graph.py`
```python
self._conn.execute("PRAGMA journal_mode=WAL")    # 启用 WAL
self._conn.execute("PRAGMA synchronous=NORMAL")  # 平衡性能与安全性
self._conn.execute("PRAGMA busy_timeout=5000")   # 忙等待时间
```

**优势**：
- 并行解析场景：多进程可以并发写入
- 实时查询场景：写入数据时可同时查询
- 性能需求：增量更新需要快速写入大量节点和边

### 5. 并行解析加速

项目使用 **多进程/多线程混合策略** 实现并行解析，充分利用多核 CPU 提升解析速度：

#### 核心实现机制

**执行器选择**（`code-review-graph/code_review_graph/incremental.py`）：

```python
_MAX_PARSE_WORKERS = min(os.cpu_count() or 4, 8)  # 最多 8 个工作进程

def _select_executor_kind():
    explicit = os.environ.get("CRG_PARSE_EXECUTOR", "").strip().lower()
    if explicit in ("process", "thread"):
        return explicit
    # Windows MCP 环境自动切换为线程模式，避免管道句柄继承死锁
    if sys.platform == "win32" and not sys.stdin.isatty():
        return "thread"
    return "process"  # Linux/macOS 默认使用多进程
```

**执行器类型选择策略**：

| 平台/环境 | 默认执行器 | 原因 |
|----------|-----------|------|
| Linux/macOS | `ProcessPoolExecutor` | 充分利用多核，GIL 释放 |
| Windows MCP | `ThreadPoolExecutor` | 避免管道句柄继承死锁（issues #46, #136） |
| 显式配置 | 按配置 | 通过 `CRG_PARSE_EXECUTOR` 环境变量 |

#### 并行解析流程

```
┌─────────────────────────────────────────────────────────────┐
│ 1. 收集所有可解析文件                                        │
│    - 过滤二进制文件、符号链接、不支持的语言                    │
└─────────────────────────┬───────────────────────────────────┘
                          │
                          ▼
┌─────────────────────────────────────────────────────────────┐
│ 2. 判断是否并行解析                                          │
│    - 文件数 < 8：串行解析（避免进程开销）                     │
│    - 文件数 >= 8：并行解析                                   │
└─────────────────────────┬───────────────────────────────────┘
                          │
                          ▼
┌─────────────────────────────────────────────────────────────┐
│ 3. 构建任务参数列表                                          │
│    args_list = [(rel_path, repo_root), ...]                  │
└─────────────────────────┬───────────────────────────────────┘
                          │
                          ▼
┌─────────────────────────────────────────────────────────────┐
│ 4. 并行执行解析（chunksize=20）                              │
│    executor.map(_parse_single_file, args_list)              │
└─────────────────────────┬───────────────────────────────────┘
                          │
                          ▼
┌─────────────────────────────────────────────────────────────┐
│ 5. 串行写入数据库（SQLite 单写限制）                          │
│    store.store_file_nodes_edges(...)                        │
└─────────────────────────────────────────────────────────────┘
```

#### 单文件解析函数

```python
def _parse_single_file(args: tuple[str, str]) -> tuple[str, list, list, str | None, str]:
    """在工作进程中解析单个文件"""
    rel_path, repo_root = args
    full_path = Path(repo_root) / rel_path

    try:
        source = full_path.read_bytes()
        fhash = hashlib.sha256(source).hexdigest()

        # 检测语言并解析
        parser = CodeParser()
        nodes, edges = parser.parse_bytes(rel_path, source)

        return (rel_path, nodes, edges, None, fhash)
    except Exception as e:
        return (rel_path, [], [], str(e), "")
```

#### 关键优化点

1. **自适应工作进程数**：
   - 默认：CPU 核心数（最多 8 个）
   - 可通过环境变量 `CRG_PARSE_WORKERS` 自定义

2. **Chunksize 优化**：
   - `executor.map(..., chunksize=20)`：批量分发任务，减少调度开销

3. **GIL 释放**：
   - Tree-sitter 解析在 C 层执行，释放 Python GIL
   - 多线程模式下也能获得较好的并行效果（<30% 性能损失）

4. **串行写入**：
   - SQLite 虽然支持 WAL 并发读，但写操作仍是串行
   - 解析结果收集后串行写入，避免锁竞争

5. **进度日志**：
   - 每解析 200 个文件输出一次进度
   - 便于用户了解解析进度

#### 性能对比

| 场景 | 串行解析 | 并行解析（8核） |
|------|---------|----------------|
| 1000 个文件 | ~60s | ~10s（6x 加速） |
| 100 个文件 | ~8s | ~3s（2.7x 加速） |
| 8 个文件 | ~1s | ~1.5s（进程开销） |

#### 配置参数

| 参数 | 默认值 | 说明 |
|------|--------|------|
| `CRG_PARSE_WORKERS` | CPU核心数（最多8） | 工作进程/线程数 |
| `CRG_PARSE_EXECUTOR` | auto | 执行器类型：`process` 或 `thread` |

#### 自适应串行策略

当文件数量较少（< 8 个）时，使用串行解析：

```python
if file_count < 8:
    # 串行解析，避免进程创建开销
    for rel_path in files:
        ...
else:
    # 并行解析
    with _make_executor(_MAX_PARSE_WORKERS) as executor:
        ...
```

### 6. Watch 模式实时更新

通过 **文件系统监听** + **防抖机制**（300ms）实现实时更新，支持在开发过程中自动同步代码变更到知识图谱。

#### 核心实现机制

**代码位置**：`code-review-graph/code_review_graph/incremental.py`

```python
_DEBOUNCE_SECONDS = 0.3  # 300ms 防抖窗口

def watch(repo_root: Path, store: GraphStore, on_files_updated: Optional[Callable] = None):
    """监听文件变更并自动更新图谱"""
    from watchdog.events import FileSystemEventHandler
    from watchdog.observers import Observer

    class GraphUpdateHandler(FileSystemEventHandler):
        def __init__(self):
            self._pending: set[str] = set()
            self._lock = threading.Lock()
            self._timer: threading.Timer | None = None
```

#### 完整工作流程

```
┌─────────────────────────────────────────────────────────────┐
│ 1. 启动文件监听                                               │
│    - 使用 watchdog 库监听整个代码库                          │
│    - 递归监听所有子目录                                      │
└─────────────────────────┬───────────────────────────────────┘
                          │
                          ▼
┌─────────────────────────────────────────────────────────────┐
│ 2. 文件系统事件触发                                          │
│    ┌──────────────┬──────────────┬──────────────┐          │
│    │ on_modified  │ on_created   │ on_deleted   │          │
│    │ 文件修改     │ 文件新建     │ 文件删除     │          │
│    └──────────────┴──────────────┴──────────────┘          │
└─────────────────────────┬───────────────────────────────────┘
                          │
                          ▼
┌─────────────────────────────────────────────────────────────┐
│ 3. 过滤事件（_should_handle）                                │
│    - 跳过符号链接                                            │
│    - 检查 .gitignore 等忽略规则                             │
│    - 过滤不支持的语言文件                                    │
│    - 过滤二进制文件                                          │
└─────────────────────────┬───────────────────────────────────┘
                          │
                          ▼
┌─────────────────────────────────────────────────────────────┐
│ 4. 防抖调度（_schedule）                                     │
│    - 添加到待处理文件集合                                    │
│    - 取消前一个定时器                                        │
│    - 设置新的 300ms 定时器                                   │
└─────────────────────────┬───────────────────────────────────┘
                          │
                          ▼
┌─────────────────────────────────────────────────────────────┐
│ 5. 批量处理（_flush）                                        │
│    - 获取所有待处理文件                                      │
│    - 清空待处理集合                                          │
│    - 逐个更新文件                                            │
└─────────────────────────┬───────────────────────────────────┘
                          │
                          ▼
┌─────────────────────────────────────────────────────────────┐
│ 6. 更新单个文件（_update_file）                              │
│    - 读取文件内容                                            │
│    - 计算 SHA256 哈希值                                      │
│    - Tree-sitter 解析提取节点和边                            │
│    - 存储到 SQLite 数据库                                    │
│    - 更新 last_updated 元数据                               │
└─────────────────────────┬───────────────────────────────────┘
                          │
                          ▼
┌─────────────────────────────────────────────────────────────┐
│ 7. 后处理回调（on_files_updated）                            │
│    - 重建 FTS 全文搜索索引                                  │
│    - 检测执行流程                                            │
│    - 更新社区结构                                            │
└─────────────────────────────────────────────────────────────┘
```

#### 防抖机制详解

**问题场景**：开发者连续保存多个文件时，如果每个保存都触发一次完整更新，会造成大量重复工作。

**解决方案**：300ms 防抖窗口

```python
def _schedule(self, abs_path: str):
    """添加文件到待处理集合，重置防抖定时器"""
    with self._lock:
        self._pending.add(abs_path)

        # 取消前一个定时器
        if self._timer is not None:
            self._timer.cancel()

        # 设置新的定时器，300ms 后执行
        self._timer = threading.Timer(_DEBOUNCE_SECONDS, self._flush)
        self._timer.start()

def _flush(self):
    """防抖窗口结束后批量处理所有待更新文件"""
    with self._lock:
        paths = list(self._pending)
        self._pending.clear()
        self._timer = None

    for abs_path in paths:
        self._update_file(abs_path)
```

#### 线程安全设计

```python
class GraphUpdateHandler(FileSystemEventHandler):
    def __init__(self):
        self._pending: set[str] = set()
        self._lock = threading.Lock()  # 互斥锁保护待处理文件集合
        self._timer: threading.Timer | None = None
```

- 使用 `threading.Lock` 保护 `self._pending` 集合
- 避免文件系统事件和定时器回调的竞态条件
- 保证待处理文件的原子性操作

#### 删除文件处理

```python
def on_deleted(self, event):
    # 仅处理我们跟踪的文件
    if _should_ignore(rel, ignore_patterns):
        return

    # 从数据库中删除文件数据
    store.remove_file_data(event.src_path)
    store.commit()
    logger.info("Removed: %s", rel)
```

#### 后台线程启动

```python
def start_watch_thread(repo_root: Path, store: GraphStore, daemon: bool = True):
    """在后台线程中启动 watch 模式"""
    thread = threading.Thread(
        target=watch,
        args=(repo_root, store),
        daemon=daemon,
        name="crg-watch",
    )
    thread.start()
```

#### 关键配置

| 配置 | 默认值 | 说明 |
|------|--------|------|
| `_DEBOUNCE_SECONDS` | 0.3s | 防抖窗口时间 |
| 线程名 | `crg-watch` | 后台线程标识 |
| 监听模式 | 递归 | 监听所有子目录 |

#### 使用方式

```bash
# 命令行启动 watch 模式
uv run code-review-graph watch

# 程序方式启动
start_watch_thread(Path("/path/to/repo"), store, daemon=True)
```

#### 性能优势

| 特性 | 传统方案 | Watch 模式 |
|------|---------|-----------|
| 响应延迟 | 手动更新，分钟级 | 实时更新，秒级 |
| 资源占用 | 定期全量扫描 | 仅更新变更文件 |
| 开发体验 | 需要手动触发 | 自动同步，无感更新 |
| 错误处理 | 出错后停止 | 单个文件失败不影响其他 |

---

## 三、Tree-sitter 技术原理

### 什么是 Tree-sitter？

Tree-sitter 是一个**解析器生成工具和增量解析库**，核心特性：
- **通用性**：支持 50+ 编程语言
- **快速**：实时解析，每次按键时更新
- **健壮**：语法错误时仍能提供有用结果
- **无依赖**：纯 C11 实现

### 增量解析原理

| 传统解析器 | Tree-sitter |
|-----------|-------------|
| 每次修改重新解析整个文件 | 只重新解析修改的区域 |
| 语法错误导致解析失败 | 错误恢复，继续解析 |
| 构建 AST 丢弃后重建 | 维护并增量更新语法树 |

### 在 code-review-graph 中的应用

**代码位置**：`code-review-graph/code_review_graph/parser.py#L777`

```python
def parse_bytes(self, path: Path, source: bytes):
    language = self.detect_language(path)
    parser = self._get_parser(language)
    tree = parser.parse(source)  # Tree-sitter 生成语法树

    # 提取节点和边
    self._extract_from_tree(tree.root_node, source, language, ...)
```

**支持的语言**：Python、JavaScript/TypeScript、Go、Rust、Java、C/C++、Solidity、Vue 等。

---

## 四、BFS 广度优先搜索（最多 2 跳）

### 概念解释

BFS 是一种**逐层遍历**图的算法：

```
目标文件 A
  ↑ 1 跳
文件 B（直接调用者）
  ↑ 2 跳
文件 C（间接调用者）
```

### 实现代码

**代码位置**：`code-review-graph/code_review_graph/incremental.py#L757`

```python
def find_dependents(store, file_path, max_hops=2):
    all_dependents = set()
    visited = {file_path}
    frontier = {file_path}

    for _hop in range(max_hops):
        next_frontier = set()
        for fp in frontier:
            deps = _single_hop_dependents(store, fp)
            new_deps = deps - visited
            all_dependents.update(new_deps)
            next_frontier.update(new_deps)
        visited.update(next_frontier)
        frontier = next_frontier
```

### 为什么限制为 2 跳？

| 跳数 | 特点 |
|------|------|
| 1 跳 | 速度最快，但可能遗漏重要依赖 |
| 2 跳 | **默认值**，平衡深度和性能 |
| 3+ 跳 | 覆盖更全面，但返回过多文件 |

可通过环境变量 `CRG_DEPENDENT_HOPS` 自定义跳数。

---

## 五、全量构建 vs 增量更新

### 核心区别

| 特性 | `build`（全量） | `update`（增量） |
|------|---------------|-----------------|
| **解析范围** | 所有文件 | 仅变更文件 + 2 跳依赖 |
| **深度限制** | 无 | 2 跳（默认） |
| **哈希检查** | 否（强制重解析） | 是（跳过未变文件） |
| **适用场景** | 首次初始化、分支切换 | 日常开发、小改动 |
| **速度** | 较慢 | 极快（毫秒级） |

### 全量构建流程

```
┌─────────────────────────────────────────────────────────────┐
│ 1. 收集所有文件                                              │
│ 2. 并行解析所有文件（Tree-sitter）                           │
│ 3. 存储节点和边到 SQLite                                     │
│ 4. 后期处理：重建索引、检测流程、社区聚类                    │
└─────────────────────────────────────────────────────────────┘
```

### 增量更新流程

```
┌─────────────────────────────────────────────────────────────┐
│ 1. 检测变更文件（Git diff）                                  │
│ 2. BFS 查找 2 跳依赖文件                                     │
│ 3. 哈希校验跳过未变文件                                      │
│ 4. 并行解析变更文件                                          │
│ 5. 事务性更新图数据库                                        │
└─────────────────────────────────────────────────────────────┘
```

---

## 六、调用链查询（超过 2 跳的处理）

### 增量更新 vs 查询的区别

**2 跳限制仅用于增量更新**，查询场景支持更深遍历：

| 场景 | 使用函数 | 默认深度 |
|------|---------|----------|
| 增量更新 | `find_dependents` | 2 跳 |
| 影响分析 | `get_impact_radius` | 2 跳（可自定义） |
| 关系查询 | `query_graph` | 无限制 |
| 图遍历 | `traverse_graph_func` | 3 跳（支持 1-6） |

### 查询超过 2 跳的方法

```python
# 方法 1：使用 traverse_graph_func
result = traverse_graph_func(
    query="format_date",
    mode="bfs",
    depth=4,  # 4 跳
    token_budget=3000
)

# 方法 2：自定义 max_depth
result = get_impact_radius(
    changed_files=["utils/helpers.py"],
    max_depth=3
)
```

---

## 七、安装与初始化流程

### 命令概述

`code-review-graph install` 是项目初始化命令，用于：
- 配置 AI 编码平台的 MCP 服务器
- 生成 Claude Code Skills
- 安装钩子（Hooks）自动同步代码变更
- 注入项目说明文档

**代码位置**：`code-review-graph/code_review_graph/cli.py#L177`

```python
def _handle_init(args: argparse.Namespace) -> None:
    """Set up MCP config for detected AI coding platforms."""
    repo_root = Path(args.repo) if args.repo else find_repo_root()
    # 配置平台 MCP 服务器
    # 生成 Skills
    # 安装 Hooks
    # 注入说明文档
```

### 完整安装流程

```
┌─────────────────────────────────────────────────────────────────┐
│ 1. 检测项目根目录                                               │
│    - 使用 find_repo_root() 查找 git 仓库根目录                  │
│    - 默认使用当前工作目录                                       │
└────────────────────┬────────────────────────────────────────────┘
                     │
                     ▼
┌─────────────────────────────────────────────────────────────────┐
│ 2. 安装 MCP 服务器配置                                          │
│    - 检测并配置多个平台：Claude、Cursor、Windsurf、Zed 等       │
│    - 写入平台特定的配置文件（.mcp.json、.cursor/mcp.json 等）   │
└────────────────────┬────────────────────────────────────────────┘
                     │
                     ▼
┌─────────────────────────────────────────────────────────────────┐
│ 3. 更新 .gitignore 文件                                         │
│    - 添加 .code-review-graph/ 到 .gitignore                     │
│    - 创建或更新 .gitignore                                      │
└────────────────────┬────────────────────────────────────────────┘
                     │
                     ▼
┌─────────────────────────────────────────────────────────────────┐
│ 4. 生成 Claude Code Skills                                     │
│    - 在 .claude/skills/ 目录下创建 4 个 Skill 目录              │
│    - 每个 Skill 包含 skill.md 定义文件                          │
└────────────────────┬────────────────────────────────────────────┘
                     │
                     ▼
┌─────────────────────────────────────────────────────────────────┐
│ 5. 注入项目说明文档                                             │
│    - 向 CLAUDE.md 添加 code-review-graph 使用指引              │
│    - 向其他平台配置文件注入说明                                 │
└────────────────────┬────────────────────────────────────────────┘
                     │
                     ▼
┌─────────────────────────────────────────────────────────────────┐
│ 6. 安装 Hooks                                                  │
│    - 生成 Claude Code settings.json hooks 配置                 │
│    - 安装 Git pre-commit hook                                   │
│    - 为其他平台安装相应的 hooks                                 │
└────────────────────┬────────────────────────────────────────────┘
                     │
                     ▼
┌─────────────────────────────────────────────────────────────────┐
│ 7. 完成安装指引                                                 │
│    - 提示运行 code-review-graph build 构建知识图谱             │
│    - 提示重启 AI 编码工具加载配置                                │
└─────────────────────────────────────────────────────────────────┘
```

### 生成的产物结构

```
project-root/
├── .mcp.json                    # Claude Code MCP 服务器配置
├── .claude/
│   ├── skills/                  # Claude Code Skills 目录
│   │   ├── explore-codebase/    # 探索代码库 Skill
│   │   │   └── skill.md
│   │   ├── review-changes/      # 代码审查 Skill
│   │   │   └── skill.md
│   │   ├── debug-issue/         # 调试问题 Skill
│   │   │   └── skill.md
│   │   └── refactor-safely/     # 安全重构 Skill
│   │       └── skill.md
│   └── settings.json            # Hooks 配置
├── .git/
│   └── hooks/
│       └── pre-commit          # Git pre-commit hook
├── CLAUDE.md                   # Claude 项目说明（已注入）
└── .code-review-graph/         # 知识图谱数据库（build 后生成）
    └── graph.db
```

### 支持的 AI 编码平台

| 平台 | 配置文件 | Skill 支持 | Hook 支持 |
|------|----------|------------|-----------|
| Claude Code | `.mcp.json` | ✅ | ✅ |
| Cursor | `.cursor/mcp.json` | | ✅ |
| Windsurf | `~/.codeium/windsurf/mcp_config.json` | | |
| Zed | `~/.config/zed/settings.json` | | |
| Continue | `~/.continue/config.json` | | |
| OpenCode | `.opencode.json` | | ✅ |
| Gemini CLI | `.gemini/settings.json` | ✅ | ✅ |
| Qwen Code | `~/.qwen/settings.json` | | |
| Codex | `~/.codex/config.toml` | | ✅ |

### 四个核心 Skills

#### 1. Explore Codebase（探索代码库）

**使用 `get_minimal_context` 快速获取概况：**

**MCP 工具调用（输入样例）：**
```json
{
  "method": "tools/get_minimal_context",
  "params": {
    "task": "explore codebase structure",
    "base": "HEAD~1",
    "repo_root": "/path/to/project"
  }
}
```

**或在 AI 对话中使用：**
```
使用 code-review-graph 工具获取代码库概况，task="explore codebase structure"
```

**输出样例：**
```json
{
  "status": "ok",
  "task": "explore codebase structure",
  "stats": {
    "total_files": 142,
    "total_functions": 897,
    "total_classes": 156,
    "total_edges": 3241
  },
  "top_communities": [
    "auth (23 files)",
    "api (41 files)",
    "database (18 files)"
  ],
  "risk": "low",
  "next_tool_suggestions": [
    "get_architecture_overview",
    "list_communities",
    "semantic_search_nodes"
  ]
}
```

**使用 `get_architecture_overview` 理解架构：**

**MCP 工具调用（输入样例）：**
```json
{
  "method": "tools/get_architecture_overview",
  "params": {
    "detail_level": "minimal",
    "repo_root": "/path/to/project"
  }
}
```

**输出样例：**
```json
{
  "communities": [
    {
      "name": "auth",
      "size": 23,
      "key_entities": [
        "UserService",
        "JWTManager",
        "AuthMiddleware"
      ]
    },
    {
      "name": "api",
      "size": 41,
      "key_entities": [
        "UserController",
        "OrderController",
        "PaymentController"
      ]
    }
  ],
  "flows": [
    "login_flow",
    "checkout_flow"
  ],
  "next_tool_suggestions": [
    "get_community",
    "list_flows"
  ]
}
```

**使用 `semantic_search_nodes` 搜索代码：**

**MCP 工具调用（输入样例）：**
```json
{
  "method": "tools/semantic_search_nodes",
  "params": {
    "query": "date format",
    "limit": 5,
    "detail_level": "minimal"
  }
}
```

**输出样例：**
```json
{
  "results": [
    {
      "name": "format_date",
      "qualified_name": "utils.py::format_date",
      "kind": "Function",
      "file_path": "/src/utils.py"
    },
    {
      "name": "DateFormatter",
      "qualified_name": "utils.py::DateFormatter",
      "kind": "Class",
      "file_path": "/src/utils.py"
    }
  ],
  "total": 2,
  "next_tool_suggestions": [
    "query_graph callers_of"
  ]
}
```

**使用 `query_graph` 追踪调用关系：**

**MCP 工具调用（查询调用者）：**
```json
{
  "method": "tools/query_graph",
  "params": {
    "pattern": "callers_of",
    "target": "format_date",
    "detail_level": "minimal"
  }
}
```

**MCP 工具调用（查询被调用者）：**
```json
{
  "method": "tools/query_graph",
  "params": {
    "pattern": "callees_of",
    "target": "generate_invoice",
    "detail_level": "standard"
  }
}
```

**MCP 工具调用（查询测试用例）：**
```json
{
  "method": "tools/query_graph",
  "params": {
    "pattern": "tests_for",
    "target": "validateUser"
  }
}
```

**输出样例：**
```json
{
  "status": "ok",
  "pattern": "callers_of",
  "target": "format_date",
  "results": [
    {
      "name": "generate_invoice",
      "qualified_name": "billing.py::generate_invoice",
      "kind": "Function"
    },
    {
      "name": "UserController.getProfile",
      "qualified_name": "api/users.py::UserController.getProfile",
      "kind": "Function"
    }
  ],
  "edges": [
    {
      "source_qualified": "billing.py::generate_invoice",
      "target_qualified": "utils.py::format_date",
      "kind": "CALLS"
    }
  ]
}
```

#### 2. Review Changes（代码审查）

**使用 `detect_changes` 进行风险评估：**

**MCP 工具调用（自动检测变更）：**
```json
{
  "method": "tools/detect_changes",
  "params": {
    "base": "HEAD~1",
    "detail_level": "standard",
    "repo_root": "/path/to/project"
  }
}
```

**MCP 工具调用（指定变更文件）：**
```json
{
  "method": "tools/detect_changes",
  "params": {
    "changed_files": ["auth/utils.py", "auth/jwt.py"],
    "base": "HEAD~5",
    "detail_level": "minimal"
  }
}
```

**输出样例：**
```json
{
  "risk_score": 0.68,
  "risk_level": "medium",
  "changed_functions": [
    {
      "name": "validateUser",
      "file": "auth/utils.py",
      "is_test": false,
      "impact_count": 12
    },
    {
      "name": "JWTManager.verify",
      "file": "auth/jwt.py",
      "is_test": false,
      "impact_count": 24
    }
  ],
  "test_gaps": [
    "validateUser has no tests",
    "JWTManager.verify has no tests"
  ],
  "impacted_communities": ["auth", "api"],
  "next_tool_suggestions": [
    "get_affected_flows",
    "get_impact_radius"
  ]
}
```

**使用 `get_affected_flows` 查找受影响路径：**

**MCP 工具调用（输入样例）：**
```json
{
  "method": "tools/get_affected_flows",
  "params": {
    "changed_files": ["auth/utils.py"],
    "detail_level": "standard"
  }
}
```

**输出样例：**
```json
{
  "affected_flows": [
    {
      "name": "login_flow",
      "risk": "high",
      "changed_count": 3,
      "impacted_count": 15
    },
    {
      "name": "register_flow",
      "risk": "medium",
      "changed_count": 2,
      "impacted_count": 8
    }
  ],
  "summary": "2 flows affected: login_flow (high risk)"
}
```

**使用 `get_impact_radius` 分析影响范围：**

**MCP 工具调用（输入样例）：**
```json
{
  "method": "tools/get_impact_radius",
  "params": {
    "changed_files": ["auth/utils.py"],
    "max_depth": 3,
    "detail_level": "standard"
  }
}
```

**输出样例：**
```json
{
  "changed_files": ["auth/utils.py"],
  "changed_nodes": [
    {
      "name": "validateUser",
      "qualified_name": "auth/utils.py::validateUser"
    }
  ],
  "impacted_nodes": [
    {
      "name": "AuthMiddleware",
      "qualified_name": "middleware/auth.py::AuthMiddleware"
    },
    {
      "name": "UserService.login",
      "qualified_name": "services/user.py::UserService.login"
    }
  ],
  "impacted_files": [
    "middleware/auth.py",
    "services/user.py",
    "api/users.py"
  ],
  "max_depth": 2,
  "total_impacted": 8,
  "risk": "medium"
}
```

**检查测试覆盖率：**

**MCP 工具调用（输入样例）：**
```json
{
  "method": "tools/query_graph",
  "params": {
    "pattern": "tests_for",
    "target": "validateUser"
  }
}
```

**输出样例：**
```json
{
  "tests_for": [
    {
      "name": "test_validateUser_valid",
      "qualified_name": "tests/auth/test_utils.py::test_validateUser_valid",
      "is_test": true,
      "file_path": "/tests/auth/test_utils.py"
    }
  ],
  "test_coverage": "partial",
  "missing_tests": [
    "edge cases for invalid emails",
    "empty input validation"
  ]
}
```

#### 3. Debug Issue（问题调试）

**使用 `semantic_search_nodes` 查找相关代码：**

**MCP 工具调用（输入样例）：**
```json
{
  "method": "tools/semantic_search_nodes",
  "params": {
    "query": "JWT token login authentication",
    "limit": 10
  }
}
```

**输出样例：**
```json
{
  "results": [
    {
      "name": "login",
      "qualified_name": "services/user.py::UserService.login",
      "kind": "Function"
    },
    {
      "name": "JWTManager.verify",
      "qualified_name": "auth/jwt.py::JWTManager.verify",
      "kind": "Method"
    }
  ],
  "total": 5,
  "next_tool_suggestions": [
    "query_graph callers_of",
    "get_flow"
  ]
}
```

**使用 `query_graph` 追踪调用链：**

**MCP 工具调用（查询调用者）：**
```json
{
  "method": "tools/query_graph",
  "params": {
    "pattern": "callers_of",
    "target": "JWTManager.verify"
  }
}
```

**MCP 工具调用（查询被调用者）：**
```json
{
  "method": "tools/query_graph",
  "params": {
    "pattern": "callees_of",
    "target": "UserService.login"
  }
}
```

**输出样例：**
```json
{
  "callers": [
    {
      "name": "AuthController.login",
      "qualified_name": "api/auth.py::AuthController.login"
    }
  ],
  "callees": [
    {
      "name": "validateUser",
      "qualified_name": "auth/utils.py::validateUser"
    },
    {
      "name": "JWTManager.generate",
      "qualified_name": "auth/jwt.py::JWTManager.generate"
    }
  ]
}
```

**使用 `get_flow` 查看执行路径：**

**MCP 工具调用（输入样例）：**
```json
{
  "method": "tools/get_flow",
  "params": {
    "flow_name": "login_flow",
    "detail_level": "standard"
  }
}
```

**输出样例：**
```json
{
  "name": "login_flow",
  "criticality": 0.92,
  "nodes": [
    "api/auth.py::AuthController.login",
    "services/user.py::UserService.login",
    "auth/utils.py::validateUser",
    "database/users.py::UserRepository.find"
  ],
  "edges": [
    { "source": "AuthController.login", "target": "UserService.login" },
    { "source": "UserService.login", "target": "validateUser" }
  ],
  "changed_in_flow": ["validateUser"],
  "risk": "high"
}
```

**使用 `detect_changes` 检查最近变更：**

**MCP 工具调用（输入样例）：**
```json
{
  "method": "tools/detect_changes",
  "params": {
    "base": "HEAD~5",
    "detail_level": "minimal"
  }
}
```

**输出样例：**
```json
{
  "base": "HEAD~5",
  "changed_files": [
    "auth/utils.py",
    "auth/jwt.py"
  ],
  "risk_score": 0.75,
  "risk_level": "high",
  "summary": "High-risk changes in authentication flow"
}
```

#### 4. Refactor Safely（安全重构）

**使用 `refactor_tool` 获取重构建议：**

**MCP 工具调用（获取建议）：**
```json
{
  "method": "tools/refactor_tool",
  "params": {
    "mode": "suggest",
    "target_community": "auth"
  }
}
```

**MCP 工具调用（查找死代码）：**
```json
{
  "method": "tools/refactor_tool",
  "params": {
    "mode": "dead_code"
  }
}
```

**输出样例：**
```json
{
  "mode": "suggest",
  "suggestions": [
    {
      "type": "rename",
      "from": "checkCreds",
      "to": "validateCredentials",
      "reason": "Improved readability, consistent with codebase style"
    },
    {
      "type": "extract",
      "from": "UserService.login",
      "extract_range": "lines 45-80",
      "suggest_name": "performAuthentication",
      "reason": "Long function, can be decomposed"
    }
  ],
  "dead_code": [
    "utils.py::deprecatedMethod",
    "auth/old.py"
  ],
  "next_tool_suggestions": [
    "refactor_tool rename_preview"
  ]
}
```

**重命名预览：**

**MCP 工具调用（输入样例）：**
```json
{
  "method": "tools/refactor_tool",
  "params": {
    "mode": "rename_preview",
    "rename_from": "checkCreds",
    "rename_to": "validateCredentials"
  }
}
```

**输出样例：**
```json
{
  "mode": "rename_preview",
  "rename_from": "checkCreds",
  "rename_to": "validateCredentials",
  "impacted": [
    {
      "file": "auth/utils.py",
      "line": 15,
      "type": "definition"
    },
    {
      "file": "services/user.py",
      "line": 67,
      "type": "call_site"
    },
    {
      "file": "api/auth.py",
      "line": 23,
      "type": "call_site"
    }
  ],
  "file_count": 3,
  "call_site_count": 5,
  "test_sites_affected": 2,
  "risk": "low",
  "safety_score": 0.95
}
```

**使用 `get_impact_radius` 确认安全边界：**

**MCP 工具调用（输入样例）：**
```json
{
  "method": "tools/get_impact_radius",
  "params": {
    "changed_nodes": ["auth/utils.py::checkCreds"],
    "max_depth": 3
  }
}
```

**输出样例：**
```json
{
  "changed_nodes": ["auth/utils.py::checkCreds"],
  "impacted_nodes": [
    "services/user.py::UserService.login",
    "api/auth.py::AuthController.login"
  ],
  "impacted_files": [
    "services/user.py",
    "api/auth.py"
  ],
  "max_depth": 2,
  "total_impacted": 4,
  "risk": "low",
  "safe_to_change": true
}
```

**使用 `detect_changes` 验证重构结果：**

**MCP 工具调用（输入样例）：**
```json
{
  "method": "tools/detect_changes",
  "params": {
    "base": "HEAD~1",
    "detail_level": "minimal"
  }
}
```

**输出样例：**
```json
{
  "risk_score": 0.08,
  "risk_level": "low",
  "changed_functions": [
    {
      "name": "validateCredentials",
      "file": "auth/utils.py",
      "is_test": false,
      "impact_count": 5,
      "test_covered": true
    }
  ],
  "test_gaps": [],
  "summary": "Refactoring successful - no new risks introduced"
}
```

### Hooks 工作机制

#### Claude Code Hooks

**代码位置**：`code-review-graph/code_review_graph/skills.py#L546`

```python
{
    "hooks": {
        "PostToolUse": [   # 工具使用后触发
            {
                "matcher": "Edit|Write|Bash",
                "hooks": [
                    {
                        "type": "command",
                        "command": "code-review-graph update --skip-flows",
                        "timeout": 30
                    }
                ]
            }
        ],
        "SessionStart": [   # 会话开始时触发
            {
                "matcher": "",
                "hooks": [
                    {
                        "type": "command",
                        "command": "code-review-graph status",
                        "timeout": 10
                    }
                ]
            }
        ]
    }
}
```

#### Git Pre-Commit Hook

```bash
#!/bin/sh
# code-review-graph 安装的 git 钩子
if command -v code-review-graph >/dev/null 2>&1; then
    code-review-graph update || true
    code-review-graph detect-changes --brief || true
fi
```

### 完整交互流程

```
┌─────────────┐     ┌─────────────┐     ┌─────────────┐
│   开发者    │────▶│  Claude AI  │────▶│   Project   │
│  修改代码   │     │   工具      │     │   代码库    │
└─────────────┘     └─────────────┘     └─────────────┘
                          │
                          ▼
                    ┌──────────────┐
                    │  PostToolUse │
                    │    Hook      │
                    └──────┬───────┘
                          │
                          ▼
                    ┌──────────────┐
                    │  update 更新 │
                    │  知识图谱    │
                    └──────┬───────┘
                          │
                          ▼
                    ┌──────────────┐
                    │  MCP 服务器  │
                    │  (stdio)     │
                    └──────┬───────┘
                          │
                          ▼
                    ┌──────────────┐
                    │  SQLite DB   │
                    │  (.code-     │
                    │   review-   │
                    │   graph/)   │
                    └──────────────┘

当开发者使用 Claude Code 时：
─────────────────────────────────────
1. 启动会话 → SessionStart Hook 运行 → 检查图谱状态
2. 编辑文件 → PostToolUse Hook 运行 → 自动更新图谱
3. 使用 Skills → 调用 MCP 工具 → 查询知识图谱
4. 提交代码 → Git Pre-Commit Hook 运行 → 显示变更风险
```

### MCP 服务器配置示例

**Claude Code `.mcp.json` 配置：**

```json
{
  "mcpServers": {
    "code-review-graph": {
      "command": "code-review-graph",
      "args": ["serve"],
      "cwd": "/path/to/project"
    }
  }
}
```

### Token 效率原则

所有 Skills 都遵循以下规则以节省 Token：
1. **总是先调用** `get_minimal_context(task="...")`
2. **使用** `detail_level="minimal"` 模式
3. **目标**：≤5 个工具调用，≤800 输出 Token
4. **需要时再升级**到 `detail_level="standard"`

### 安装命令选项

| 选项 | 说明 |
|------|------|
| `--repo <path>` | 指定项目根目录 |
| `--platform <target>` | 指定安装目标平台（默认 all） |
| `--no-skills` | 跳过 Skills 安装 |
| `--no-hooks` | 跳过 Hooks 安装 |
| `--no-instructions` | 跳过说明文档注入 |
| `--yes` | 自动确认所有操作 |
| `--dry-run` | 预览操作，不实际修改文件 |

### 指定特定平台

如果只想配置特定平台：

```bash
# 仅配置 Claude Code
code-review-graph install --platform claude

# 仅配置 Codex
code-review-graph install --platform codex

# 仅配置 Cursor
code-review-graph install --platform cursor

# 仅配置 Kiro
code-review-graph install --platform kiro

# 仅配置 OpenCode
code-review-graph install --platform opencode

# 仅配置 Gemini CLI
code-review-graph install --platform gemini-cli
```

---

## 八、性能优势与关键配置

### 流程总结

#### 首次构建

```
用户运行 → build → 收集所有文件 → 并行解析 → 存储节点/边 → 建立索引 → 完成
```

#### 日常更新

```
代码修改 → 触发 update → 检测变更 → BFS 2跳依赖 → 哈希过滤 → 解析变更文件 → 更新图 → 完成
```

#### 查询调用链

```
用户查询 → query_graph/traverse_graph → SQLite 查询 → 返回完整调用链 → 完成
```

### 性能优势

| 场景 | 传统方式 | code-review-graph |
|------|----------|-------------------|
| 首次构建 | 慢（全量解析） | 快（并行解析） |
| 增量更新 | 慢（重新解析） | 极快（只解析变更文件） |
| 错误容忍 | 失败 | 继续解析 |
| 内存占用 | 高 | 低（增量更新） |

### 关键配置

| 配置项 | 默认值 | 说明 |
|------|--------|------|
| `CRG_DEPENDENT_HOPS` | 2 | 增量更新的依赖跳数 |
| `CRG_PARSE_WORKERS` | CPU核心数（最多8） | 并行解析工作进程数 |
| `_MAX_DEPENDENT_FILES` | 500 | 依赖文件数量上限 |

---

## 九、参考文献

1. [Tree-sitter 官方文档](https://tree-sitter.github.io/tree-sitter/)
2. `code-review-graph/` - 源码目录
3. `code-review-graph/code_review_graph/incremental.py` - 增量更新核心实现
4. `code-review-graph/code_review_graph/parser.py` - Tree-sitter 解析实现
5. `code-review-graph/code_review_graph/graph.py` - 图存储实现
6. `code-review-graph/code_review_graph/cli.py` - CLI 命令实现
7. `code-review-graph/code_review_graph/skills.py` - Skills 和 Hooks 生成实现
