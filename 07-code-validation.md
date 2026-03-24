---
name: code-validation
description: 用于检查terraform-provider-huaweicloud项目代码规范的SKILL。当用户需要检查代码是否符合规范、运行代码检查脚本、验证schema和文档一致性、检查UT覆盖率时使用此SKILL。适用于PR代码检查、开发过程中的代码质量验证、提交前的代码审查等场景。
---

# 终端环境检测与命令执行规范

## 步骤0：终端类型检测（必须首先执行）

**重要**：在执行任何检查命令之前，必须先检测用户当前使用的终端类型。

### 检测方法

通过执行`uname -a`命令，根据输出结果判断终端类型：

```bash
uname -a
```

### 终端类型判断规则

根据`uname -a`命令的输出结果，判断终端类型：

| 终端类型 | `uname -a`输出特征 | 命令执行格式 |
|---------|------------------|-------------|
| **PowerShell** | `uname : 无法将"uname"项识别为 cmdlet、函数、脚本文件或可运行程序的名称。` | `wsl bash -lc "命令"` |
| **JavaScript DEBUG Terminal** | `uname : 无法将"uname"项识别为 cmdlet、函数、脚本文件或可运行程序的名称。` | `wsl bash -lc "命令"` |
| **Git Bash** | `MINGW64 xxxxx` | `wsl bash -lc "命令"` |
| **CMD** | `'uname' 不是内部或外部命令，也不是可运行的程序或批处理文件` | `wsl bash -lc "命令"` |
| **Ubuntu (WSL)** | `Linux xxxxx` | 直接执行命令（不需要wsl前缀） |

### 命令执行格式

**情况1：Windows终端（PowerShell、JavaScript DEBUG Terminal、Git Bash、CMD）**

所有命令必须使用以下格式：
```bash
wsl bash -lc "命令"
```

**情况2：Ubuntu终端（WSL）**

直接执行命令，不需要wsl前缀：
```bash
命令
```

#### 示例命令

**查看Go版本（Windows终端）：**
```bash
wsl bash -lc "go version"
```

**查看Go版本（Ubuntu终端）：**
```bash
go version
```

**获取当前项目目录（Windows终端）：**
```bash
wsl bash -lc "pwd"
# 输出示例：/mnt/d/code/terraform-provider-huaweicloud
```

**获取当前项目目录（Ubuntu终端）：**
```bash
pwd
# 输出示例：/mnt/d/code/terraform-provider-huaweicloud
```

**执行检查脚本（Windows终端）：**
```bash
wsl bash -lc "cd /mnt/d/code/terraform-provider-huaweicloud && ./scripts/codecheck.sh ./huaweicloud/services/dws"
```

**执行检查脚本（Ubuntu终端）：**
```bash
cd /mnt/d/code/terraform-provider-huaweicloud && ./scripts/codecheck.sh ./huaweicloud/services/dws
```

### 记录要求

- 记录检测到的终端类型（PowerShell/JavaScript DEBUG Terminal/Git Bash/CMD/Ubuntu）
- 记录后续命令的执行格式（是否需要wsl前缀）
- 在后续所有步骤中，根据检测到的终端类型执行对应的命令格式

### 路径转换规则

**Windows终端路径转换：**
- **Windows路径**：`d:\code\terraform-provider-huaweicloud`
- **WSL路径**：`/mnt/d/code/terraform-provider-huaweicloud`
- **转换规则**：将盘符`d:`转换为`/mnt/d/`，反斜杠`\`转换为正斜杠`/`

**Ubuntu终端路径：**
- 直接使用Linux路径，无需转换

# 代码规范检查SKILL

## 概述

此SKILL用于检查terraform-provider-huaweicloud项目的代码是否符合规范要求。它会执行一系列检查，包括基本代码检查、schema验证、单元测试覆盖率、文档一致性等。

## 检查范围

### 重要：必须执行的4个检查脚本

**每次代码检查都必须完整运行以下4个shell脚本，缺一不可：**

1. **`gofmtcheck.sh`** - 代码格式检查
2. **`errcheck.sh`** - 未检查错误检查
3. **`codecheck.sh`** - 代码复杂度和lint检查
4. **`coverage.sh`** - 单元测试覆盖率检查

**强制执行规则：**
- 所有4个脚本必须全部执行
- 任何一个脚本检查失败都需要修复后重新检查
- 只有所有脚本都通过才能认为代码检查通过
- 不允许跳过任何一个脚本的检查

### 1. 基本检查
使用项目根目录下`/scripts`中的脚本进行基本检查：
- `codecheck.sh` - 代码复杂度、golangci-lint、misspell等
- `coverage.sh` - 单元测试覆盖率
- `errcheck.sh` - 未检查错误
- `gofmtcheck.sh` - 代码格式

### 2. 重点检查文件类型
- **schema文件**：resource和data source的schema定义
- **单元测试文件**：确保UT覆盖率达到80%及以上
- **md文档**：检查文档与schema的一致性
- **provider.go文件**：检查provider的注册和配置

## 检查步骤

### 步骤1：确定检查范围
首先确定要检查的文件范围。如果用户指定了特定的PR或文件，则只检查这些文件；否则检查所有新增和修改的文件。

使用git命令获取变更文件：

**Windows终端：**
```bash
wsl bash -lc "git diff --name-only HEAD~1 HEAD"
```

**Ubuntu终端：**
```bash
git diff --name-only HEAD~1 HEAD
```

或者如果指定了PR：

**Windows终端：**
```bash
wsl bash -lc "git diff origin/main...HEAD --name-only"
```

**Ubuntu终端：**
```bash
git diff origin/main...HEAD --name-only
```

**检查用户是否指定了HTML文件**：

如果用户在请求中指定了HTML文件路径（例如：`huaweicloud/services/acceptance/iam/iam_coverage.html`），则需要记录该HTML文件路径，后续步骤5将直接使用该HTML文件提取覆盖率，而不再运行单元测试。

**记录要求**：
- 记录用户指定的HTML文件路径（如果指定）
- 如果没有指定HTML文件，记录为"未指定"

### 步骤2：分类检查文件
将变更文件分类到以下类别：
1. schema文件（`resource_huaweicloud_*.go`和`data_source_huaweicloud_*.go`）
2. 单元测试文件（`*_test.go`）
3. md文档文件（`docs/resources/*.md`和`docs/data-sources/*.md`）
4. provider.go文件（`huaweicloud/provider.go`）
5. 其他Go文件

### 步骤3：执行基本检查

#### 3.0 环境变量配置检查（一次性）

**重要**：在执行任何检查脚本之前，必须先检查环境变量文件是否存在并记录下来。

**检查环境变量文件**：

需要先查询当前项目的根目录，根据根目录查询环境变量文件。

**Windows终端：**
```bash
wsl bash -lc "pwd"
```

**Ubuntu终端：**
```bash
pwd
```

按以下顺序检查环境变量文件，找到第一个存在的文件即停止检查：

**Windows终端：**
```bash
wsl bash -lc "if [ -f /mnt/d/code/terraform-provider-huaweicloud/.vscode/.env ]; then cat /mnt/d/code/terraform-provider-huaweicloud/.vscode/.env; elif [ -f /mnt/d/code/terraform-provider-huaweicloud/.trae/.env ]; then cat /mnt/d/code/terraform-provider-huaweicloud/.trae/.env; elif [ -f /mnt/d/code/terraform-provider-huaweicloud/.cursor/.env ]; then cat /mnt/d/code/terraform-provider-huaweicloud/.cursor/.env; elif [ -f /mnt/d/code/terraform-provider-huaweicloud/.codebuddy/.env ]; then cat /mnt/d/code/terraform-provider-huaweicloud/.codebuddy/.env; elif [ -f /mnt/d/code/terraform-provider-huaweicloud/.continue/.env ]; then cat /mnt/d/code/terraform-provider-huaweicloud/.continue/.env; else echo '未找到环境变量文件'; fi"
```

**Ubuntu终端：**
```bash
if [ -f /mnt/d/code/terraform-provider-huaweicloud/.vscode/.env ]; then cat /mnt/d/code/terraform-provider-huaweicloud/.vscode/.env; elif [ -f /mnt/d/code/terraform-provider-huaweicloud/.trae/.env ]; then cat /mnt/d/code/terraform-provider-huaweicloud/.trae/.env; elif [ -f /mnt/d/code/terraform-provider-huaweicloud/.cursor/.env ]; then cat /mnt/d/code/terraform-provider-huaweicloud/.cursor/.env; elif [ -f /mnt/d/code/terraform-provider-huaweicloud/.codebuddy/.env ]; then cat /mnt/d/code/terraform-provider-huaweicloud/.codebuddy/.env; elif [ -f /mnt/d/code/terraform-provider-huaweicloud/.continue/.env ]; then cat /mnt/d/code/terraform-provider-huaweicloud/.continue/.env; else echo '未找到环境变量文件'; fi
```

**检查目录顺序**：
1. `.vscode`（默认目录）
2. `.trae`
3. `.cursor`
4. `.codebuddy`
5. `.continue`

**要求**：
- 只读取环境变量文件中未被注释掉的变量（不以`#`开头的行）
- 如果环境变量中配置了目录相关的变量，需要将Windows路径转换为Linux路径
- 如果没有配置环境变量，立即停止并告知用户需要进行配置

**记录要求**：
- 记录环境变量检查结果：配置完成/未配置
- 记录找到的环境变量文件路径（后续所有命令都使用此路径）

#### 3.1 代码格式检查

**Windows终端：**
```bash
wsl bash -lc "cd /mnt/d/code/terraform-provider-huaweicloud && ./scripts/gofmtcheck.sh"
```

**Ubuntu终端：**
```bash
cd /mnt/d/code/terraform-provider-huaweicloud && ./scripts/gofmtcheck.sh
```

**注意**：此脚本会检查所有文件，需要从输出结果中获取与本次resource/data source相关的内容。

**记录要求**：
- 记录执行状态：成功/失败
- 如果失败，记录失败原因

#### 3.2 错误检查

**只检查指定文件**：

**Windows终端：**
```bash
wsl bash -lc "cd /mnt/d/code/terraform-provider-huaweicloud && ./scripts/errcheck.sh -f huaweicloud/services/dws/data_source_huaweicloud_dws_host_nets.go"
```

**Ubuntu终端：**
```bash
cd /mnt/d/code/terraform-provider-huaweicloud && ./scripts/errcheck.sh -f huaweicloud/services/dws/data_source_huaweicloud_dws_host_nets.go
```

**判断标准**：
- **通过**：errcheck没有检查出任何错误（输出为空或只包含正常信息）
- **不通过**：errcheck检查出错误（输出包含具体的错误信息）

**注意**：
- 只检查指定的文件，可以指定schema文件或测试文件
- 使用`-f`参数指定要检查的文件路径
- 需要根据errcheck的输出结果判断是否通过
- 如果输出包含错误信息，则检查不通过；如果输出为空或只有正常信息，则检查通过

**记录要求**：
- 记录执行状态：成功/失败
- 记录检查结果：通过/不通过
- 如果不通过，记录检查出的错误信息

#### 3.3 代码复杂度和lint检查

**只检查指定文件**：

**Windows终端：**
```bash
wsl bash -lc "cd /mnt/d/code/terraform-provider-huaweicloud && ./scripts/codecheck.sh ./huaweicloud/services/dws -f data_source_huaweicloud_dws_host_nets.go"
```

**Ubuntu终端：**
```bash
cd /mnt/d/code/terraform-provider-huaweicloud && ./scripts/codecheck.sh ./huaweicloud/services/dws -f data_source_huaweicloud_dws_host_nets.go
```

**注意**：
- 只检查指定的文件，可以指定schema文件或测试文件
- 使用`-f`参数指定要检查的文件路径
- 第一个参数是服务包路径（如`./huaweicloud/services/dws`）

**记录要求**：
- 记录执行状态：成功/失败
- 如果失败，记录失败原因

### 步骤4：检查schema文件

#### 4.1 API注释检查
使用`api_comment_check`工具检查schema文件是否有正确的API注释：

**Windows终端：**
```bash
wsl bash -lc "cd /mnt/d/code/terraform-provider-huaweicloud/scripts/api_comment_check && go run main.go -basePath ../../huaweicloud/"
```

**Ubuntu终端：**
```bash
cd /mnt/d/code/terraform-provider-huaweicloud/scripts/api_comment_check && go run main.go -basePath ../../huaweicloud/
```

检查每个schema文件是否包含：
- `// @API Product httpMethod requestPath`格式的注释
- 所有使用的API端点都有对应的注释

**记录要求**：
- 记录检查结果：通过/不通过
- 如果不通过，记录缺少的API注释

#### 4.2 Schema定义检查
检查schema文件是否符合规范：
- 所有resource必须包含`Importer`字段
- ReadContext中必须使用`common.CheckDeletedDiag`
- 必须使用`go-multierror`包处理多个错误
- 错误消息中不应包含"HuaweiCloud"关键字

**记录要求**：
- 记录检查结果：通过/不通过
- 如果不通过，记录不符合规范的项目

### 步骤5：检查单元测试覆盖率

#### 5.0 判断是否需要运行单元测试

**重要**：根据步骤1中记录的信息，判断是否需要运行单元测试。

**情况1：用户指定了HTML文件**
- 如果用户在步骤1中指定了HTML文件路径，则**不运行单元测试**
- 直接跳到步骤5.4，使用用户指定的HTML文件提取覆盖率
- 记录："使用用户指定的HTML文件，跳过单元测试"

**情况2：用户未指定HTML文件**
- 必须运行单元测试
- 继续执行步骤5.1到5.3
- 记录："用户未指定HTML文件，运行单元测试"

#### 5.1 确定单元测试函数（仅在未指定HTML文件时执行）

**重要**：需要识别单元测试文件中的所有测试函数，并运行所有测试函数。

**示例**：以`huaweicloud_identity_regions`为例

1. **查找对应的单元测试文件**：
   - Schema文件：`huaweicloud/services/identity/data_source_huaweicloud_identity_regions.go`
   - 测试文件：`huaweicloud/services/acceptance/identity/data_source_huaweicloud_identity_regions_test.go`

2. **根据包名和文件名寻找schema文件的完整路径**：
   - 包名：`identity`（从测试文件路径`huaweicloud/services/acceptance/identity/`中提取）
   - 测试文件名：`data_source_huaweicloud_identity_regions_test.go`
   - Schema文件名：将测试文件名的`_test.go`后缀去掉，得到`data_source_huaweicloud_identity_regions.go`
   - Schema文件完整路径：`huaweicloud/services/identity/data_source_huaweicloud_identity_regions.go`

   **通用规则**：
   - 从测试文件路径`huaweicloud/services/acceptance/<package_name>/`中提取包名
   - 将测试文件名的`_test.go`后缀去掉，得到schema文件名
   - Schema文件路径：`huaweicloud/services/<package_name>/<schema_file_name>.go`

3. **提取测试函数名称**：

**Windows终端：**
   ```bash
   wsl bash -lc "grep -E '^func TestAcc' huaweicloud/services/acceptance/identity/data_source_huaweicloud_identity_regions_test.go | sed 's/func //'"
   ```

**Ubuntu终端：**
   ```bash
   grep -E '^func TestAcc' huaweicloud/services/acceptance/identity/data_source_huaweicloud_identity_regions_test.go | sed 's/func //'
   ```

   输出示例：`TestAccDataRegions_basic`

4. **运行所有测试函数**：
   - 如果只有1个测试函数，运行1次
   - 如果有多个测试函数，需要为每个函数运行一次

**记录要求**：
- 记录找到的测试函数列表
- 记录每个测试函数的执行状态

#### 5.2 计算覆盖率（仅在未指定HTML文件时执行）

**读取环境变量并运行测试：**

使用步骤3.0中记录的环境变量 **.env** 文件路径，拼接命令运行测试。

**示例命令格式（以huaweicloud_identity_regions为例）：**

假设在步骤3.0中找到的环境变量文件路径为`.vscode/.env`：

**Windows终端：**
```bash
wsl bash -lc 'cd /mnt/d/code/terraform-provider-huaweicloud && export $(cat .vscode/.env | grep -v "^#" | sed "s/\r$//" | xargs) && ./scripts/coverage.sh -o identity -f TestAccDataRegions_basic'
```

**Ubuntu终端：**
```bash
cd /mnt/d/code/terraform-provider-huaweicloud && export $(cat .vscode/.env | grep -v "^#" | sed "s/\r$//" | xargs) && ./scripts/coverage.sh -o identity -f TestAccDataRegions_basic
```

**对于多个测试函数的情况：**

**Windows终端：**
```bash
wsl bash -lc 'cd /mnt/d/code/terraform-provider-huaweicloud && export $(cat .vscode/.env | grep -v "^#" | sed "s/\r$//" | xargs) && ./scripts/coverage.sh -o identity -f TestAccDataRegions_basic'
wsl bash -lc 'cd /mnt/d/code/terraform-provider-huaweicloud && export $(cat .vscode/.env | grep -v "^#" | sed "s/\r$//" | xargs) && ./scripts/coverage.sh -o identity -f TestAccDataRegions_filter'
```

**Ubuntu终端：**
```bash
cd /mnt/d/code/terraform-provider-huaweicloud && export $(cat .vscode/.env | grep -v "^#" | sed "s/\r$//" | xargs) && ./scripts/coverage.sh -o identity -f TestAccDataRegions_basic
cd /mnt/d/code/terraform-provider-huaweicloud && export $(cat .vscode/.env | grep -v "^#" | sed "s/\r$//" | xargs) && ./scripts/coverage.sh -o identity -f TestAccDataRegions_filter
```

**注意**：
- 使用`sed "s/\r$//"`去除Windows换行符
- 使用`grep -v "^#"`过滤掉注释行
- 路径转换：将Windows路径（如`D:\DOCS\tmp\tf_win.log`）转换为Linux路径（如`/mnt/d/DOCS/tmp/tf_win.log`）
- 必须为每个测试函数指定`-f`参数
- 环境变量文件路径使用步骤3.0中记录的路径

**记录要求**：
- 记录每个测试函数的执行状态：成功/失败/跳过
- 记录测试跳过的原因（如缺少环境变量）

#### 5.4 从HTML和COV文件计算覆盖率

**重要**：必须从HTML文件中提取覆盖率数据。

**HTML文件来源**：
- **如果用户指定了HTML文件**：使用步骤1中记录的HTML文件路径
- **如果用户未指定HTML文件**：使用coverage.sh生成的HTML文件

**生成文件位置（未指定HTML文件时）**：
```
huaweicloud/services/acceptance/<service_name>/<service_name>_coverage.html
huaweicloud/services/acceptance/<service_name>/<service_name>_coverage.cov
```

**示例**：
- HTML文件：`huaweicloud/services/acceptance/identity/identity_coverage.html`
- COV文件：`huaweicloud/services/acceptance/identity/identity_coverage.cov`

**提取覆盖率数据**：

1. **查看HTML文件**：

**Windows终端：**
   ```bash
   wsl bash -lc "cat huaweicloud/services/acceptance/identity/identity_coverage.html"
   ```

**Ubuntu终端：**
   ```bash
   cat huaweicloud/services/acceptance/identity/identity_coverage.html
   ```

2. **查看COV文件**（可选）：

**Windows终端：**
   ```bash
   wsl bash -lc "cat huaweicloud/services/acceptance/identity/identity_coverage.cov"
   ```

**Ubuntu终端：**
   ```bash
   cat huaweicloud/services/acceptance/identity/identity_coverage.cov
   ```

3. **提取特定文件覆盖率**：
   **从HTML源码中搜索对应的resource/data source名称**
   
   在HTML文件中搜索对应的文件名，括号中的数字就是覆盖率。
   
   示例命令（以data_source_huaweicloud_identity_regions.go为例）：

**Windows终端：**
   ```bash
   wsl bash -lc "grep -o 'data_source_huaweicloud_identity_regions.go ([0-9.]*%)' huaweicloud/services/acceptance/identity/identity_coverage.html"
   ```

**Ubuntu终端：**
   ```bash
   grep -o 'data_source_huaweicloud_identity_regions.go ([0-9.]*%)' huaweicloud/services/acceptance/identity/identity_coverage.html
   ```
   
   输出示例：
   ```html
   <option value="file23">github.com/huaweicloud/terraform-provider-huaweicloud/huaweicloud/services/iam/data_source_huaweicloud_identity_regions.go (80.0%)</option>
   ```
   提取结果：80.0%

**记录要求**：
- 用户指定的schema文件的覆盖率
- 记录使用的HTML文件路径（用户指定或自动生成）
- 记录覆盖率计算结果：达标（>=80%）/不达标（<80%）

#### 5.5 验证覆盖率
检查生成的覆盖率报告，确保：
- 用户指定的schema文件的覆盖率 >= 80%

如果覆盖率不足，需要提出建议：
1. 识别未覆盖的代码路径
2. 添加或修改测试用例
3. 重新运行覆盖率测试

**记录要求**：
- 记录覆盖率验证结果：通过/不通过
- 如果不通过，记录覆盖率不足的函数

### 步骤6：检查md文档

#### 6.1 文档与schema一致性检查
使用`markdown_check`工具检查：

**Windows终端：**
```bash
wsl bash -lc "cd /mnt/d/code/terraform-provider-huaweicloud/scripts/markdown_check && go run main.go"
```

**Ubuntu终端：**
```bash
cd /mnt/d/code/terraform-provider-huaweicloud/scripts/markdown_check && go run main.go
```

检查项包括：
- schema中的所有参数是否在文档中都有说明
- 文档中的参数是否在schema中存在
- 参数类型、必填性、默认值是否一致
- Deprecated标记是否一致

**记录要求**：
- 记录检查结果：通过/不通过
- 如果不通过，记录不一致的项目

#### 6.2 文档格式检查
检查文档是否符合格式要求：
- 是否包含Example Usage
- 是否包含Argument Reference
- 是否包含Attribute Reference
- 是否包含Import说明（如果支持导入）
- 链接是否正确

**记录要求**：
- 记录检查结果：通过/不通过
- 如果不通过，记录格式问题

### 步骤7：检查provider.go

如果provider.go被修改，检查：
1. 新增的resource/data source是否已注册
2. 导入路径是否正确
3. 是否有重复注册
4. 版本号是否需要更新

**记录要求**：
- 记录检查结果：通过/不通过
- 如果不通过，记录问题

### 步骤8：生成检查报告

汇总所有检查结果，生成包含以下内容的报告：

```md
## 代码规范检查报告

### 检查范围
- 检查的文件列表
- 检查的时间

### 环境变量配置检查
✅️ 环境变量文件已找到
- 环境变量文件路径：.vscode/.env
- 执行状态：成功

### 基本检查结果（4个脚本必须全部通过）

#### 1. 代码格式检查 (gofmtcheck.sh)
✅️ 检查通过
- 执行状态：成功

#### 2. 错误检查 (errcheck.sh)
✅️ 检查通过
- 执行状态：成功
- 检查结果：通过（未检查出错误）

#### 3. 代码复杂度和lint检查 (codecheck.sh)
✅️ 检查通过
- 执行状态：成功

#### 4. 单元测试覆盖率检查 (coverage.sh)
❌️️ 检查失败
- 执行状态：失败
- 覆盖率：xx%
- HTML文件来源：用户指定/自动生成
- HTML文件路径：xxx
- 失败原因：xxx

### Schema检查
- ✅️ API注释完整性
- ✅️ Schema定义规范性

### 文档检查
- ✅️ 文档与schema一致性
- ✅️ 文档格式

### Provider检查
- ✅️ 资源注册

### 问题列表
1. 问题描述
   - 文件：xxx
   - 位置：xxx
   - 修复建议：xxx

### 覆盖率报告
- 整体覆盖率：xx%
- 各函数覆盖率：
  - 函数1: xx%
  - 函数2: xx%

### 总结
- ❌️️ **不通过**
- 需要修复的问题数量
- 必须修复的检查脚本：列出未通过的脚本名称
```

**报告格式要求**：
- ✅️：表示检查通过（成功）
- ❌️️：表示检查失败（错误）
- ⚠️：表示警示或需要注意的问题
- 检查通过时，不显示"失败原因"字段
- 检查失败时，显示"失败原因"字段
- 清晰标注每个检查项的状态
- 提供具体的修复建议

## 检查标准

### 通过标准（必须满足所有条件）

**强制要求：所有以下条件必须同时满足：**

1. **所有4个基本检查脚本必须全部通过**
   - `gofmtcheck.sh` 必须通过
   - `errcheck.sh` 必须通过（errcheck没有检查出错误）
   - `codecheck.sh` 必须通过
   - `coverage.sh` 必须通过（覆盖率 >= 80%）

2. **Schema文件符合规范**
   - 包含正确的API注释
   - Schema定义符合项目规范

3. **文档与schema一致**
   - 所有参数都有对应说明
   - 参数类型、必填性一致

4. **没有严重错误**
   - 无未处理的错误
   - 无代码格式问题

### 不通过的情况

**出现以下任一情况即为不通过：**

1. **任何基本检查脚本失败**
   - `gofmtcheck.sh` 检查失败
   - `errcheck.sh` 检查失败
   - `codecheck.sh` 检查失败
   - `coverage.sh` 覆盖率不足（< 80%）

2. **代码格式错误**
   - 不符合Go标准格式

3. **有未处理的错误**
   - 函数调用的错误返回值未检查

4. **UT覆盖率不足**
   - 整体覆盖率 < 80%
   - 关键函数覆盖率 < 80%

5. **文档与schema不一致**
   - schema修改后未更新文档
   - 参数说明缺失或不匹配

6. **缺少必要的API注释**
   - 新增的API调用未添加注释

7. **Schema定义不符合规范**
   - Resource缺少Importer字段
   - 未使用go-multierror包处理错误
   - 错误消息包含"HuaweiCloud"关键字

## 常见问题及修复建议

### 问题1：gofmt检查失败
**原因**：代码格式不符合Go标准格式
**修复**：运行格式化命令

**Windows终端：**
```bash
wsl bash -lc "cd /mnt/d/code/terraform-provider-huaweicloud && gofmt -w ."
```

**Ubuntu终端：**
```bash
cd /mnt/d/code/terraform-provider-huaweicloud && gofmt -w .
```

### 问题2：errcheck检查失败
**原因**：有未检查的错误返回值
**修复**：检查所有函数调用的错误返回值并处理

### 问题3：UT覆盖率不足
**原因**：测试用例不完整
**修复**：
1. 运行覆盖率测试查看详情

**Windows终端：**
```bash
wsl bash -lc "cd /mnt/d/code/terraform-provider-huaweicloud && export $(cat .vscode/.env | grep -v '^#' | sed 's/\r$//' | xargs) && go test -coverprofile=coverage.out"
```

**Ubuntu终端：**
```bash
cd /mnt/d/code/terraform-provider-huaweicloud && export $(cat .vscode/.env | grep -v '^#' | sed 's/\r$//' | xargs) && go test -coverprofile=coverage.out
```

2. 识别未覆盖的代码路径
3. 添加测试用例覆盖这些路径

### 问题4：文档与schema不一致
**原因**：schema修改后未更新文档
**修复**：
1. 运行`markdown_check`工具查看具体不一致项

**Windows终端：**
```bash
wsl bash -lc "cd /mnt/d/code/terraform-provider-huaweicloud/scripts/markdown_check && go run main.go"
```

**Ubuntu终端：**
```bash
cd /mnt/d/code/terraform-provider-huaweicloud/scripts/markdown_check && go run main.go
```

2. 更新文档以匹配schema
3. 确保所有参数都有说明

### 问题5：缺少API注释
**原因**：新增的API调用未添加注释
**修复**：
在resource函数前添加注释：
```go
// @API VPC POST /v1/{project_id}/vpcs
// @API VPC GET /v1/{project_id}/vpcs/{id}
func ResourceVirtualPrivateCloudV1() *schema.Resource {
    ...
}
```

## 使用示例

### 示例1：检查PR的代码
用户请求："检查这个PR的代码是否符合规范"
1. 获取PR的变更文件
2. 分类文件
3. 执行所有检查
4. 生成报告

### 示例2：检查特定服务
使用技能检查 huaweicloud_vpcs 相关文件是否符合代码规范

**Windows终端：**
1. 执行`wsl bash -lc "cd /mnt/d/code/terraform-provider-huaweicloud && ./scripts/codecheck.sh ./huaweicloud/services/vpc -f data_source_huaweicloud_vpcs.go"`
2. 执行`wsl bash -lc "cd /mnt/d/code/terraform-provider-huaweicloud && ./scripts/errcheck.sh -f huaweicloud/services/vpc/data_source_huaweicloud_vpcs.go"`
3. 执行`wsl bash -lc 'cd /mnt/d/code/terraform-provider-huaweicloud && export $(cat .vscode/.env | grep -v "^#" | sed "s/\r$//" | xargs) && ./scripts/coverage.sh -o vpc -f TestAccDataVpcs_basic'`
4. 检查相关文档
5. 生成报告

**Ubuntu终端：**
1. 执行`cd /mnt/d/code/terraform-provider-huaweicloud && ./scripts/codecheck.sh ./huaweicloud/services/vpc -f data_source_huaweicloud_vpcs.go`
2. 执行`cd /mnt/d/code/terraform-provider-huaweicloud && ./scripts/errcheck.sh -f huaweicloud/services/vpc/data_source_huaweicloud_vpcs.go`
3. 执行`cd /mnt/d/code/terraform-provider-huaweicloud && export $(cat .vscode/.env | grep -v "^#" | sed "s/\r$//" | xargs) && ./scripts/coverage.sh -o vpc -f TestAccDataVpcs_basic`
4. 检查相关文档
5. 生成报告

### 示例3：检查单个文件
用户请求："检查resource_huaweicloud_vpc.go是否符合规范"
1. 检查API注释
2. 检查schema定义
3. 检查对应的测试文件覆盖率
4. 检查对应的文档
5. 生成报告

### 示例4：检查huaweicloud_identity_regions
用户请求："检查huaweicloud_identity_regions的代码"

1. 确定文件：
   - Schema文件：`huaweicloud/services/identity/data_source_huaweicloud_identity_regions.go`
   - 测试文件：`huaweicloud/services/acceptance/identity/data_source_huaweicloud_identity_regions_test.go`

2. 提取测试函数：

**Windows终端：**
   ```bash
   wsl bash -lc "grep -E '^func TestAcc' huaweicloud/services/acceptance/identity/data_source_huaweicloud_identity_regions_test.go | sed 's/func //'"
   ```

**Ubuntu终端：**
   ```bash
   grep -E '^func TestAcc' huaweicloud/services/acceptance/identity/data_source_huaweicloud_identity_regions_test.go | sed 's/func //'
   ```

   输出：`TestAccDataRegions_basic`

3. 执行检查：

**Windows终端：**
   ```bash
   wsl bash -lc "cd /mnt/d/code/terraform-provider-huaweicloud && ./scripts/gofmtcheck.sh"
   wsl bash -lc "cd /mnt/d/code/terraform-provider-huaweicloud && ./scripts/errcheck.sh"
   wsl bash -lc "cd /mnt/d/code/terraform-provider-huaweicloud && ./scripts/codecheck.sh ./huaweicloud/services/identity"
   wsl bash -lc 'cd /mnt/d/code/terraform-provider-huaweicloud && export $(cat .vscode/.env | grep -v "^#" | sed "s/\r$//" | xargs) && ./scripts/coverage.sh -o identity -f TestAccDataRegions_basic'
   ```

**Ubuntu终端：**
   ```bash
   cd /mnt/d/code/terraform-provider-huaweicloud && ./scripts/gofmtcheck.sh
   cd /mnt/d/code/terraform-provider-huaweicloud && ./scripts/errcheck.sh
   cd /mnt/d/code/terraform-provider-huaweicloud && ./scripts/codecheck.sh ./huaweicloud/services/identity
   cd /mnt/d/code/terraform-provider-huaweicloud && export $(cat .vscode/.env | grep -v "^#" | sed "s/\r$//" | xargs) && ./scripts/coverage.sh -o identity -f TestAccDataRegions_basic
   ```

4. 从HTML文件提取覆盖率：

**Windows终端：**
   ```bash
   wsl bash -lc "cat huaweicloud/services/acceptance/identity/identity_coverage.html"
   ```

**Ubuntu终端：**
   ```bash
   cat huaweicloud/services/acceptance/identity/identity_coverage.html
   ```

5. 生成报告

### 示例5：检查指定文件（使用-f参数）
用户请求："检查huaweicloud_dws_host_nets的指定文件"

1. 确定文件：
   - Schema文件：`huaweicloud/services/dws/data_source_huaweicloud_dws_host_nets.go`
   - 测试文件：`huaweicloud/services/acceptance/dws/data_source_huaweicloud_dws_host_nets_test.go`

2. 执行检查（只检查指定文件）：

**Windows终端：**
   ```bash
   wsl bash -lc "cd /mnt/d/code/terraform-provider-huaweicloud && ./scripts/gofmtcheck.sh"
   wsl bash -lc "cd /mnt/d/code/terraform-provider-huaweicloud && ./scripts/errcheck.sh -f huaweicloud/services/dws/data_source_huaweicloud_dws_host_nets.go"
   wsl bash -lc "cd /mnt/d/code/terraform-provider-huaweicloud && ./scripts/codecheck.sh ./huaweicloud/services/dws -f data_source_huaweicloud_dws_host_nets.go"
   ```

**Ubuntu终端：**
   ```bash
   cd /mnt/d/code/terraform-provider-huaweicloud && ./scripts/gofmtcheck.sh
   cd /mnt/d/code/terraform-provider-huaweicloud && ./scripts/errcheck.sh -f huaweicloud/services/dws/data_source_huaweicloud_dws_host_nets.go
   cd /mnt/d/code/terraform-provider-huaweicloud && ./scripts/codecheck.sh ./huaweicloud/services/dws -f data_source_huaweicloud_dws_host_nets.go
   ```

3. 检查schema文件规范和文档
4. 生成报告

## 注意事项

1. **步骤0：终端类型检测（必须首先执行）**
   - 在执行任何检查命令之前，必须先执行`uname -a`命令检测终端类型
   - 根据输出结果判断终端类型（PowerShell/JavaScript DEBUG Terminal/Git Bash/CMD/Ubuntu）
   - 记录检测到的终端类型和对应的命令执行格式
   - 后续所有步骤中，根据检测到的终端类型执行对应的命令格式

2. **命令执行格式**
   - Windows终端（PowerShell、JavaScript DEBUG Terminal、Git Bash、CMD）：使用`wsl bash -lc "命令"`格式
   - Ubuntu终端（WSL）：直接执行命令，不需要wsl前缀
   - 在所有命令示例中都提供了Windows和Ubuntu两种格式

3. **强制执行4个检查脚本**
   - 每次代码检查必须完整执行所有4个shell脚本
   - 不允许跳过任何一个脚本的检查
   - 任何一个脚本失败都需要修复后重新运行所有检查
   - 只有所有脚本都通过才能认为代码检查通过

4. **环境变量配置**
   - 在步骤3.0中一次性检查环境变量文件（`.vscode`、`.trae`、`.cursor`、`.codebuddy`、`.continue`）
   - 只读取未被注释的变量
   - 需要进行Windows路径到Linux路径的转换（仅Windows终端）
   - 如果没有配置环境变量，立即停止并告知用户
   - 后续所有命令都使用步骤3.0中记录的环境变量文件路径

5. **单元测试函数识别**
   - 必须从测试文件中提取所有测试函数
   - 如果只有1个测试函数，运行1次
   - 如果有多个测试函数，需要为每个函数运行一次
   - 必须使用`-f`参数指定测试函数名称

6. **覆盖率计算**
   - 必须从coverage.sh生成的HTML文件中提取覆盖率数据
   - HTML文件位置：`huaweicloud/services/acceptance/<service_name>/<service_name>_coverage.html`
   - 需要提取整体覆盖率和各函数覆盖率

7. **检查记录**
   - 必须记录每个脚本的执行状态（成功/失败/跳过）
   - 必须记录失败或跳过的原因
   - 检查报告中必须包含所有检查项的状态

8. **检查报告**
   - 在检查报告中使用表情符号：✓（成功）、✗（失败）、⚠️（警示）
   - 清晰标注每个检查项的状态
   - 提供具体的修复建议

9. **环境要求**：
   - 需要Go环境
   - 需要安装必要的工具：golangci-lint, scc, misspell, gocyclo, errcheck
   - 需要在git仓库中运行

10. **性能考虑**：
    - coverage测试可能耗时较长，可以并行运行
    - 对于大型PR，可以分批检查

11. **特殊情况**：
    - 对于deprecated的资源，可以跳过某些检查
    - 对于内部使用的资源，文档检查可以放宽

## 输出格式

检查完成后，输出结构化的检查报告，包括：
- 检查概览
- 详细结果（包含表情符号）
- 问题列表
- 修复建议
- 覆盖率统计
- 最终结论（通过/不通过）

**表情符号使用说明**：
- ✓：表示检查通过或成功
- ✗：表示检查失败或错误
- ⚠️：表示警示或需要注意的问题
