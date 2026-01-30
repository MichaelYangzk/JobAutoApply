# 新业务流程说明

## 流程概述

新流程采用**本地副本管理机制**，不再直接修改OneDrive中的Jobs.xlsx文件：

1. **智能复制**：从OneDrive复制Jobs.xlsx到当前目录
2. **本地处理**：在本地副本上运行LLM和Notion同步
3. **保护机制**：本地已处理的NEW状态数据不会被OneDrive覆盖

## 核心特性

### 1. 智能合并逻辑

当从OneDrive复制文件时，系统会：

- 检查本地是否已存在`Jobs.xlsx`
- 如果存在，比对`message_id`
- **保护本地NEW状态**：如果本地某条记录的`llm_status=NEW`，即使OneDrive有相同`message_id`的记录，也以本地为准
- 防止覆盖已开始处理但尚未完成的数据

### 2. 文件路径

- **OneDrive源文件**: `/Users/cm/Library/CloudStorage/OneDrive-UCIrvine/Jobs.xlsx`
- **本地工作副本**: `/Users/cm/Downloads/email_ingestion/Jobs.xlsx`

## 使用方法

### 基础运行

```python
python main.py
```

这会执行完整流程：
1. 从OneDrive智能复制到本地
2. 处理LLM分类
3. 同步到Notion

### Python代码调用

```python
from main import run_full

# 标准运行（智能合并）
run_full()

# 强制刷新（忽略本地NEW状态，完全使用OneDrive数据）
run_full(force_refresh=True)

# 使用自定义OneDrive路径
run_full(onedrive_path="/path/to/your/Jobs.xlsx")
```

### 仅复制文件（测试）

```python
from local_copy_manager import copy_and_merge_to_local

# 智能合并复制
local_path = copy_and_merge_to_local(
    "/Users/cm/Library/CloudStorage/OneDrive-UCIrvine/Jobs.xlsx"
)
print(f"Local copy: {local_path}")

# 强制覆盖
local_path = copy_and_merge_to_local(
    "/Users/cm/Library/CloudStorage/OneDrive-UCIrvine/Jobs.xlsx",
    force_refresh=True
)
```

## 工作流示例

### 场景1：首次运行

```bash
$ python main.py
[STEP 1] Copying from OneDrive to local with smart merge...
[COPY] Source: /Users/cm/Library/CloudStorage/OneDrive-UCIrvine/Jobs.xlsx
[COPY] Target: /Users/cm/Downloads/email_ingestion/Jobs.xlsx
[COPY] No local copy found, creating new one
[COPY] Local copy created

[STEP 2] LLM stage start
Processing 10 NEW records...
[STEP 2] LLM stage done

[STEP 3] Notion sync start
Syncing to Notion...
[STEP 3] Notion sync done

[SUCCESS] All processing complete. Local file: /Users/cm/Downloads/email_ingestion/Jobs.xlsx
```

### 场景2：后续运行（有本地NEW数据）

```bash
$ python main.py
[STEP 1] Copying from OneDrive to local with smart merge...
[COPY] Local copy exists, performing smart merge...
[INFO] Found 3 local NEW records to preserve
[INFO] Merged result: 25 rows (local NEW: 3, OneDrive: 22)
[COPY] Local copy created

[STEP 2] LLM stage start
Processing 3 NEW records (preserved from local)...
```

## 数据保护逻辑

### 合并规则

```
本地记录A: message_id=123, llm_status=NEW  (正在处理中)
OneDrive记录A: message_id=123, llm_status=DONE

合并结果: 保留本地记录A (NEW状态)，忽略OneDrive的DONE状态
原因: 防止覆盖本地正在处理的数据
```

### message_id匹配

系统通过`message_id`唯一标识每条记录：

- 相同`message_id` + 本地`llm_status=NEW` → 本地优先
- 相同`message_id` + 本地`llm_status=DONE/ERROR` → OneDrive优先
- 不同`message_id` → 两条记录都保留

## 手动同步回OneDrive

处理完成后，本地`Jobs.xlsx`包含最新数据。可以手动复制回OneDrive：

```bash
# macOS/Linux
cp Jobs.xlsx "/Users/cm/Library/CloudStorage/OneDrive-UCIrvine/Jobs.xlsx"

# 或使用Python
import shutil
shutil.copy(
    "Jobs.xlsx",
    "/Users/cm/Library/CloudStorage/OneDrive-UCIrvine/Jobs.xlsx"
)
```

## 注意事项

1. **本地副本位置**: 始终在项目当前目录（`/Users/cm/Downloads/email_ingestion/`）
2. **状态保护**: 只有`llm_status=NEW`的记录受保护，`DONE`和`ERROR`会被OneDrive覆盖
3. **去重逻辑**: 按`message_id`去重，优先保留本地NEW记录
4. **force_refresh**: 使用此参数会忽略本地NEW状态，完全使用OneDrive数据

## 故障排查

### 问题：本地NEW数据被覆盖了

检查：
- 确认没有使用`force_refresh=True`
- 确认`message_id`字段存在且正确
- 检查控制台输出中的合并统计信息

### 问题：合并后记录数不对

检查：
- 查看控制台输出：`[INFO] Merged result: X rows (local NEW: Y, OneDrive: Z)`
- 确认是否有重复的`message_id`
- 检查是否有空的`message_id`值

## 模块说明

### local_copy_manager.py

核心功能模块：

- `copy_and_merge_to_local()`: 智能复制主函数
- `merge_dataframes_smart()`: 智能合并算法
- `get_local_copy_path()`: 获取本地副本路径

### main.py

更新内容：

- 新增`ONEDRIVE_XLSX`和`LOCAL_XLSX`常量
- `run_full()`函数现在先复制到本地再处理
- 所有操作在本地副本上执行
