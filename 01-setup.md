# 项目结构介绍

## 概述

terraform-provider-huaweicloud 是华为云的 Terraform Provider 实现，用于通过 Terraform 管理华为云资源。
该项目基于 Terraform Plugin SDK v2 开发，遵循 Terraform Provider 的标准开发模式。

## 目录结构

```
terraform-provider-huaweicloud/
├── huaweicloud/                    # 主代码目录
│   ├── config/                     # 配置和认证相关
│   │   ├── config.go               # Provider 配置结构和方法
│   │   ├── auth.go                 # 认证逻辑
│   │   └── endpoints.go            # 服务端点管理
│   │
│   ├── common/                     # 公共函数和工具
│   │   ├── common.go               # 公共方法（CheckDeleted, WaitOrderComplete等）
│   │   ├── errors.go               # 错误处理
│   │   └── resource_schema.go      # Schema 辅助函数
│   │
│   ├── helper/                     # 辅助工具包
│   │   ├── filters/                # 过滤器工具
│   │   ├── hashcode/               # 哈希计算
│   │   ├── httphelper/             # HTTP 辅助工具
│   │   ├── mutexkv/                # 互斥锁工具
│   │   └── schemas/                # Schema 包装器
│   │
│   ├── services/                   # 各服务资源实现
│   │   ├── vpc/                    # VPC 服务
│   │   │   ├── resource_huaweicloud_vpc.go
│   │   │   ├── data_source_huaweicloud_vpcs.go
│   │   │   └── ...
│   │   ├── ecs/                    # ECS 服务
│   │   ├── evs/                    # EVS 服务
│   │   ├── rds/                    # RDS 服务
│   │   ├── cce/                    # CCE 服务
│   │   └── ...                     # 其他服务
│   │
│   ├── acceptance/                 # 单元测试验收
│   │   ├── acceptance.go           # 单元测试通用函数和公共方法
│   │   ├── resource_check.go       # 资源检查工具
│   │   └── vpc/                    # VPC 服务测试
│   │   │   ├── data_source_huaweicloud_vpcs_test.go
│   │   │   ├── resource_huaweicloud_vpc_test.go
│   │   │   └── ...
│   │   └── ...                     # 其他服务
│   │
│   ├── provider.go                 # Provider 定义和注册
│   └── utils/                      # 工具函数
│       └── utils.go                # 通用工具函数
│
├── docs/                           # 文档目录
│   ├── resources/                  # Resource 文档
│   │   └── vpc.md
│   └── data-sources/               # Data Source 文档
│       └── vpcs.md
│
├── examples/                       # 示例代码
├── scripts/                        # 构建脚本
├── GNUmakefile                     # Makefile
├── go.mod                          # Go 模块定义
└── go.sum                          # 依赖校验
```

## 核心组件说明

### 1. Provider 定义 (provider.go)

Provider 是 Terraform 与云服务交互的入口，定义了：
- 配置参数（region, access_key, secret_key 等）
- 资源映射表 (ResourcesMap)
- 数据源映射表 (DataSourcesMap)

```go
func Provider() *schema.Provider {
    return &schema.Provider{
        Schema:         providerSchema,
        ResourcesMap:   resourcesMap,
        DataSourcesMap: dataSourcesMap,
        ConfigureFunc:  configureProvider,
    }
}
```

### 2. 服务目录结构 (services/)

每个服务目录包含：
- `resource_huaweicloud_*.go` - 资源实现文件
- `data_source_huaweicloud_*.go` - 数据源实现文件

### 3. 配置管理 (config/)

`config.Config` 结构体管理：
- 认证信息（AK/SK, Token 等）
- 区域和项目信息
- 服务客户端缓存
- 企业项目配置

### 4. 公共工具 (common/)

提供通用功能：
- `CheckDeleted` / `CheckDeletedDiag` - 处理资源不存在的情况，在 ReadContext 和 DeleteContext 中使用。
- `WaitOrderComplete` - 等待订单完成
- `UnsubscribePrePaidResource` - 退订包周期资源
- `GetRegion` - 获取 region，默认使用环境变量 HC_REGION_NAME

### 5. 工具函数 (utils/)

提供类型转换、数据处理等工具：
- `ExpandToStringList` - 接口数组转字符串数组
- `PathSearch` - JMESPath 表达式查询，常用在读取API返回数据时使用。
- `RemoveNil` - 移除 map 中的 nil 值，调用API时使用避免空参数报错。
- `TagsToMap` / `ExpandResourceTags` - 标签处理

## 资源命名规范

### 包名

一般是服务名称简写，如vpc。

### 文件命名

| 类型 | 命名格式 | 示例 |
|------|----------|------|
| Resource | `resource_huaweicloud_<resource_name>.go` | `resource_huaweicloud_vpc.go` |
| Data Source | `data_source_huaweicloud_<data_source_name>.go` | `data_source_huaweicloud_vpcs.go` |
| Test | `<resource>_test.go` | `resource_huaweicloud_vpc_test.go` |
| Document | `<resource_name>.md` | `vpc.md` |

### 函数命名

| 函数类型 | 命名格式 | 示例 |
|----------|----------|------|
| Resource 定义 | `Resource<ResourceName>()` | `ResourceVirtualPrivateCloud()` |
| Create | `resource<ResourceName>Create()` | `resourceVirtualPrivateCloudCreate()` |
| Read | `resource<ResourceName>Read()` | `resourceVirtualPrivateCloudRead()` |
| Update | `resource<ResourceName>Update()` | `resourceVirtualPrivateCloudUpdate()` |
| Delete | `resource<ResourceName>Delete()` | `resourceVirtualPrivateCloudDelete()` |
| Data Source 定义 | `DataSource<DataSourcesName>()` | `DataSourceVpcs()` |
| Data Source Read | `dataSource<DataSourcesName>Read()` | `dataSourceVpcsRead()` |

## 依赖说明

### 主要依赖

import 引入的顺序：标准库 -> 三方库 -> 二方库 -> 一方库，每类库用空行分割，
其中：`github.com/chnsz/*` 和 `github.com/huaweicloud/<非本代码仓之外的其他仓库>` 视为二方库。

```go
import (
	// 标准库
    "context"
    "fmt"
    "log"
    "regexp"
    "strconv"
    "strings"
    "time"

	// 三方库
    "github.com/hashicorp/go-multierror"
    "github.com/hashicorp/terraform-plugin-sdk/v2/diag"
    "github.com/hashicorp/terraform-plugin-sdk/v2/helper/resource"
    "github.com/hashicorp/terraform-plugin-sdk/v2/helper/schema"
    
	// 二方库
    "github.com/chnsz/golangsdk"
    "github.com/chnsz/golangsdk/openstack/common/tags"
    v1rules "github.com/chnsz/golangsdk/openstack/networking/v1/security/rules"
    v1groups "github.com/chnsz/golangsdk/openstack/networking/v1/security/securitygroups"
    v2groups "github.com/chnsz/golangsdk/openstack/networking/v2/extensions/security/groups"
    v3groups "github.com/chnsz/golangsdk/openstack/networking/v3/security/groups"
    v3rules "github.com/chnsz/golangsdk/openstack/networking/v3/security/rules"
    
	// 一方库
    "github.com/huaweicloud/terraform-provider-huaweicloud/huaweicloud/common"
    "github.com/huaweicloud/terraform-provider-huaweicloud/huaweicloud/config"
    "github.com/huaweicloud/terraform-provider-huaweicloud/huaweicloud/utils"
    )
```

### SDK 选择

推荐使用 httpClient 的方式调用API，不推荐使用 SDK。

不推荐使用的 SDK：
1. **golangsdk** - OpenStack 风格 SDK，用于大部分 VPC、ECS 等基础服务
2. **huaweicloud-sdk-go-v3** - 华为云官方 SDK v3，用于新服务和高级功能

## 开发环境配置

### 必需的环境变量

必要的环境变量已配置完毕，可以直接启动调试。

### 运行测试

单元测试是测试 Terraform Provider 的一种主要方式，用于验证资源定义是否正确，单元测试覆盖率必须是 80% 以上。

```bash
# 运行单个测试，生成覆盖率报告
TF_ACC=1 go test -covermode=atomic -v -coverprofile=coverage.out  -coverpkg=./huaweicloud ./huaweicloud/services/acceptance/vpc -run TestAccVpcV1_basic -timeout 360m -parallel 4

# 运行所有 VPC 测试
TF_ACC=1 go test -covermode=atomic -v -coverprofile=coverage.out  -coverpkg=./huaweicloud ./huaweicloud/services/acceptance/vpc -run TestAcc* -timeout 360m -parallel 4
```

### 构建和安装

```bash
# 构建Provider
go build -o terraform-provider-huaweicloud

# 安装到本地（用于测试）
make install
```

## API 注释规范

每个资源文件顶部应包含 API 注释，用于文档生成。
注释的格式： `// @API <Service_Name> <API_METHOD> <API_PATH>`

```go
// @API VPC POST /v1/{project_id}/vpcs
// @API VPC GET /v1/{project_id}/vpcs/{id}
// @API VPC PUT /v1/{project_id}/vpcs/{id}
// @API VPC DELETE /v1/{project_id}/vpcs/{id}
```
