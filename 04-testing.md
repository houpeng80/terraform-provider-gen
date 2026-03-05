---
description: 为华为云Terraform provider的资源和数据源生成测试用例
---

# Terraform Provider 测试用例生成 Skill

## 概述

此skill基于华为云Terraform provider资源和数据源的实现及文档，生成全面的测试用例。

## 前置条件

使用此skill前，请确保以下文件存在：

| 文件类型 | 路径 | 说明 |
|---------|------|------|
| 注册代码 | `./huaweicloud/provider.go` | 资源/数据源注册 |
| 实现文件 | `./huaweicloud/services/{service}/{resource_or_datasource}.go` | 实际代码 |
| 文档文件 | `./docs/{resources|data-sources}/{resource_name}.md` | 资源文档 |
| 测试文件 | `./huaweicloud/services/acceptance/{service}/{resource_or_datasource}_test.go` | 测试文件（待生成） |

## 核心原则：文档分析是测试用例生成的基础

**关键提醒**：测试场景不是固定的，必须从文档中仔细分析才能得出完整的测试覆盖。

### 文档分析三步法

#### 第一步：逐字段深度分析
对文档中的每个字段进行详细分析：

- **字段属性**：类型、是否必需、是否可更新
- **取值范围**：枚举值、数值范围、字符串格式
- **依赖关系**：字段间的依赖和约束
- **使用限制**：特殊的使用限制和规则
- **业务含义**：字段在业务中的含义和使用场景

#### 第二步：场景提取
从文档描述中提取具体的测试场景：

- **使用场景**：明确提到的使用场景
- **组合规则**：字段组合的使用规则
- **限制条件**：特殊的限制条件
- **边界场景**：边界值和错误场景
- **业务流程**：业务流程和状态变化

#### 第三步：规则验证识别
识别文档中明确提到的验证规则：

- **数值限制**：范围限制（如period_num: 1-9）
- **格式限制**：字符串格式限制（如不允许<>符号）
- **依赖规则**：字段依赖关系（如billing_mode与prepaid_options）
- **转换规则**：状态转换规则（如只允许某些值间的转换）
- **环境要求**：特定区域或条件要求

### 文档分析示例

以`huaweicloud_cc_bandwidth_package`为例，文档分析发现的测试场景：

#### **从billing_mode字段分析**：
- 文档说明："只能在值为3或4时编辑，只能编辑为1、2、5、6"
- **测试场景**：billing_mode编辑限制测试、模式转换测试

#### **从interflow_mode字段分析**：
- 文档说明："Area模式有15个spec_code，Region模式依赖具体区域"
- **测试场景**：不同interflow_mode的测试、spec_code组合测试

#### **从prepaid_options字段分析**：
- 文档说明："period_num范围：month(1-9), year(1-3)"
- **测试场景**：边界值测试、不同period_type测试

#### **从description字段分析**：
- 文档说明："不允许包含尖括号<>"
- **测试场景**：格式验证测试、特殊字符处理

### 测试用例生成策略

基于文档分析，按优先级生成测试用例：

#### **P0 - 基础功能测试**
- 创建、读取、更新、删除
- 导入功能
- 基本字段验证

#### **P1 - 字段规则测试**
- 文档中明确提到的所有规则
- 字段间的依赖关系
- 取值范围和枚举值

#### **P2 - 组合场景测试**
- 多个字段的组合使用
- 业务流程的完整测试
- 状态变化的验证

#### **P3 - 边界和错误测试**
- 数值边界值
- 格式限制验证
- 错误输入处理

### 文档分析检查清单

在生成测试用例前，确保完成了以下文档分析：

- [ ] **必填字段**：所有必填字段都有测试覆盖
- [ ] **可选字段**：主要可选字段有测试覆盖
- [ ] **字段依赖**：字段间的依赖关系有测试验证
- [ ] **取值限制**：枚举值、范围限制有边界测试
- [ ] **更新限制**：哪些字段可更新、何时可更新
- [ ] **特殊规则**：文档中提到的特殊使用规则
- [ ] **业务场景**：文档中描述的使用场景
- [ ] **错误场景**：可能的错误输入和限制
- [ ] **导入注意**：导入时的特殊处理要求
- [ ] **环境要求**：特殊的环境或前置条件

### 常见遗漏场景

基于实际项目经验，以下场景经常被遗漏：

1. **企业项目测试**：enterprise_project_id字段
2. **资源绑定测试**：resource_id、resource_type绑定
3. **计费模式转换**：不同billing_mode间的转换
4. **标签完整生命周期**：标签的增删改查
5. **格式验证**：特殊字符、长度限制
6. **边界值测试**：数值字段的最小值、最大值
7. **状态转换**：资源状态的完整变化流程
8. **导入后处理**：导入后的状态差异处理

## 测试用例编写注意事项

### 内部字段处理

**重要提醒**：内部字段完全不应该在测试用例中体现，包括配置、验证和ignore列表。

**内部字段特征**：
- `enable_force_new`：用于控制ForceNew行为的内部字段
- 其他带有`Internal: true`描述的字段

**正确做法**：
```go
// 正确：完全忽略内部字段，不在任何地方提及
ImportStateVerifyIgnore: []string{
    // 不包含enable_force_new等内部字段
}

// 错误：在测试配置中包含内部字段
resource "huaweicloud_xx_resource" "test" {
    name = "test"
    enable_force_new = "true"  // 不应该在测试配置中包含
}

// 错误：在Check中验证内部字段
resource.TestCheckResourceAttr(rName, "enable_force_new", "false")  // 不应该验证内部字段

// 错误：在ImportStateVerifyIgnore中包含内部字段
ImportStateVerifyIgnore: []string{
    "enable_force_new",  // 内部字段不应该出现在ignore列表中
}
```

**识别内部字段的方法**：
1. 查看资源Schema中的`Description`字段，如果包含`Internal: true`标记
2. 查看文档，内部字段通常不会在用户文档中出现

**ImportStateVerifyIgnore标准配置**：
```go
ImportStateVerifyIgnore: []string{
    // 只包含真正需要在导入时忽略的用户可见字段
    // 不包含任何内部字段如enable_force_new
}
```

### 字段更新测试设计

在测试用例设计时需要：

1. **识别可更新字段**：只有非ForceNew字段才能进行更新测试
2. **验证NonUpdatable特性**：对于ForceNew字段，更新后应该触发资源重建
3. **合理设计更新步骤**：只测试真正可更新的字段

```go
// 正确：只更新可更新的字段
{
    Config: testResource_update(name),
    Check: resource.ComposeTestCheckFunc(
        // 可更新字段验证
        resource.TestCheckResourceAttr(rName, "name", fmt.Sprintf("%s_update", name)),
        // NonUpdatable字段保持不变
        resource.TestCheckResourceAttr(rName, "domain_name", "example.com"),
    ),
}
```

### getResourceFunc编写注意事项

**重要提醒**：在编写getResourceFunc时，必须正确导入服务包并使用服务名前缀。

**正确做法**：
```go
// 正确：导入服务包并使用服务名前缀
import (
    "github.com/huaweicloud/terraform-provider-huaweicloud/huaweicloud/services/ccm"
)

func getResourceFunc(cfg *config.Config, state *terraform.ResourceState) (interface{}, error) {
    client, err := cfg.NewServiceClient("scm", acceptance.HW_REGION_NAME)
    if err != nil {
        return nil, fmt.Errorf("error creating CCM client: %s", err)
    }

    return ccm.GetCsr(client, state.Primary.ID)  // 使用服务名前缀
}
```

**错误做法**：
```go
// 错误：没有导入服务包，没有使用服务名前缀
func getResourceFunc(cfg *config.Config, state *terraform.ResourceState) (interface{}, error) {
    client, err := cfg.NewServiceClient("scm", acceptance.HW_REGION_NAME)
    if err != nil {
        return nil, fmt.Errorf("error creating CCM client: %s", err)
    }

    return GetCsr(client, state.Primary.ID)  // 错误：没有使用服务名前缀
}
```

**编写规范**：
1. **导入服务包**：必须导入对应的服务包，如`"github.com/huaweicloud/terraform-provider-huaweicloud/huaweicloud/services/ccm"`
2. **使用服务名前缀**：调用服务函数时必须使用服务名前缀，如`ccm.GetCsr`
3. **保持一致性**：即使测试文件和资源实现在同一包中，也要显式导入并使用前缀
4. **避免歧义**：使用完整路径调用函数，避免可能的命名冲突

**常见服务包导入示例**：
```go
import (
    "github.com/huaweicloud/terraform-provider-huaweicloud/huaweicloud/services/ccm"
    "github.com/huaweicloud/terraform-provider-huaweicloud/huaweicloud/services/cc"
    "github.com/huaweicloud/terraform-provider-huaweicloud/huaweicloud/services/ecs"
    "github.com/huaweicloud/terraform-provider-huaweicloud/huaweicloud/services/vpc"
)
```

## 使用方式

```
你来为资源xxx生成一个测试用例
你来为数据源xxx生成一个测试用例
```

## 资源/数据源类型及测试模式

### 1. 数据源类型

**参考示例**: `huaweicloud_ccm_deployed_resources`, `huaweicloud_ccm_csrs`, `huaweicloud_ccm_private_certificates_by_tags`, `huaweicloud_secmaster_subscription_products`, `huaweicloud_waf_web_basic_protection_rules`

**测试模式**:
- 使用 `TestAccDataSource{Service}{Resource}_basic` 函数名
- 使用 `data.huaweicloud_{service}_{resource}.test` 作为 dataSource 变量
- 使用 `acceptance.InitDataSourceCheck(dataSource)` 进行初始化
- 包含 `CheckResourceExists()` 和字段验证检查
- 对计算字段使用 `resource.TestCheckResourceAttrSet()`
- 对必需的环境变量包含预检查
- **重要**: 如果存在Optional过滤参数，必须验证过滤功能

**结构**:
```go
func TestAccDataSource{Service}{Resource}_basic(t *testing.T) {
    dataSource := "data.huaweicloud_{service}_{resource}.test"
    dc := acceptance.InitDataSourceCheck(dataSource)

    resource.ParallelTest(t, resource.TestCase{
        PreCheck: func() {
            acceptance.TestAccPreCheck(t)
            acceptance.TestAccPreCheck{Specific}(t) // 如需要
        },
        ProviderFactories: acceptance.TestAccProviderFactories,
        Steps: []resource.TestStep{
            {
                Config: testDataSource{Service}{Resource}_basic(),
                Check: resource.ComposeTestCheckFunc(
                    dc.CheckResourceExists(),
                    // 字段验证
                    // 过滤参数验证（如存在）
                ),
            },
        },
    })
}
```

#### 基础配置模板

**简单数据源模板**:
```go
func testDataSource{Service}{Resource}_basic() string {
    return fmt.Sprintf(`
data "huaweicloud_{service}_{resource}" "test" {
  // 必需参数（可自动生成字段）
  name = "test_name"
  action = "filter"
  
  // 必需参数（需要环境变量字段）
  instance_id = "%s"
}
`, acceptance.HW_INSTANCE_ID)
}
```

**多必需参数模板**:
```go
func testDataSource{Service}{Resource}_basic() string {
    return fmt.Sprintf(`
data "huaweicloud_{service}_{resource}" "test" {
  // 必需参数（可自动生成字段）
  name = "test_name"
  action = "filter"
  
  // 必需参数（需要环境变量字段）
  instance_id = "%s"
  certificate_id = "%s"
}
`, acceptance.HW_INSTANCE_ID, acceptance.HW_CCM_SSL_CERTIFICATE_ID)
}
```

**依赖资源模板**:
```go
func testDataSource{Service}{Resource}_basic(name string) string {
    return fmt.Sprintf(`
%s

data "huaweicloud_{service}_{resource}" "test" {
  depends_on = [huaweicloud_{service}_{dependency_resource}.test]
  
  // 必需参数（可自动生成字段）
  name = "test_name"
  
  // 必需参数（需要环境变量字段）
  instance_id = "%s"
}
`, test{Service}{DependencyResource}_basic(name), acceptance.HW_INSTANCE_ID)
}
```

#### 必需参数分类说明

**可自动生成的必需参数**（不需要环境变量）:
- `name` - 资源名称，可使用固定测试值
- `action` - 操作类型，通常在文档中明确指定（如"filter", "count"）
- `description` - 描述信息，可使用测试描述
- `enabled` - 布尔值，可使用true/false
- 其他可预设的枚举值或固定值

**需要环境变量的必需参数**（无法自动获取）:
- `instance_id` - 实例ID，需要运行环境中存在的实例
- `certificate_id` - 证书ID，需要有效的证书标识
- `policy_id` - 策略ID，需要存在的策略
- `domain_id` - 域名ID，需要有效的域名
- `resource_id` - 资源ID，需要存在的资源标识
- 其他依赖运行环境资源的ID类参数

#### 多层级字段验证模板

**适用于具有复杂嵌套结构的数据源**:
```go
Check: resource.ComposeTestCheckFunc(
    dc.CheckResourceExists(),
    // 顶层字段验证
    resource.TestCheckResourceAttrSet(dataSource, "basic.#"),
    resource.TestCheckResourceAttrSet(dataSource, "standard.#"),
    resource.TestCheckResourceAttrSet(dataSource, "professional.#"),
    
    // 嵌套字段验证
    resource.TestCheckResourceAttrSet(dataSource, "basic.0.cloud_service_type"),
    resource.TestCheckResourceAttrSet(dataSource, "basic.0.resource_type"),
    resource.TestCheckResourceAttrSet(dataSource, "basic.0.resource_spec_code"),
    resource.TestCheckResourceAttrSet(dataSource, "basic.0.usage_factor"),
    resource.TestCheckResourceAttrSet(dataSource, "basic.0.region_id"),
    
    resource.TestCheckResourceAttrSet(dataSource, "standard.0.cloud_service_type"),
    resource.TestCheckResourceAttrSet(dataSource, "standard.0.resource_type"),
    resource.TestCheckResourceAttrSet(dataSource, "standard.0.resource_spec_code"),
    resource.TestCheckResourceAttrSet(dataSource, "standard.0.usage_factor"),
    resource.TestCheckResourceAttrSet(dataSource, "standard.0.region_id"),
    
    // 深层嵌套字段验证
    resource.TestCheckResourceAttrSet(dataSource, "resources.0.resource_detail.0.cert_id"),
    resource.TestCheckResourceAttrSet(dataSource, "resources.0.resource_detail.0.cert_name"),
    resource.TestCheckResourceAttrSet(dataSource, "resources.0.resource_detail.0.domain"),
    resource.TestCheckResourceAttrSet(dataSource, "resources.0.resource_detail.0.cert_type"),
    resource.TestCheckResourceAttrSet(dataSource, "resources.0.resource_detail.0.domain_type"),
    resource.TestCheckResourceAttrSet(dataSource, "resources.0.resource_detail.0.sans"),
),
```

#### 过滤参数验证配置模板

**当datasource存在Optional过滤参数时，使用以下模式**:

```go
func testDataSource{Service}{Resource}_basic() string {
    return fmt.Sprintf(`
# 基础查询
data "huaweicloud_{service}_{resource}" "test" {
  // 必需参数
}

# 过滤参数1验证
locals {
  filter_value_1 = data.huaweicloud_{service}_{resource}.test.items[0].{field_1}
}

data "huaweicloud_{service}_{resource}" "filter_1" {
  {filter_param_1} = local.filter_value_1
}

locals {
  filter_1_result = [
    for v in data.huaweicloud_{service}_{resource}.filter_1.items[*].{field_1} : v == local.filter_value_1
  ]
}

output "filter_1_is_useful" {
  value = alltrue(local.filter_1_result) && length(local.filter_1_result) > 0
}

# 过滤参数2验证
locals {
  filter_value_2 = data.huaweicloud_{service}_{resource}.test.items[0].{field_2}
}

data "huaweicloud_{service}_{resource}" "filter_2" {
  {filter_param_2} = local.filter_value_2
}

locals {
  filter_2_result = [
    for v in data.huaweicloud_{service}_{resource}.filter_2.items[*].{field_2} : v == local.filter_value_2
  ]
}

output "filter_2_is_useful" {
  value = alltrue(local.filter_2_result) && length(local.filter_2_result) > 0
}

# 空结果验证（如适用）
data "huaweicloud_{service}_{resource}" "non_exist" {
  {filter_param} = "non-exist-value"
}

output "non_exist_is_zero" {
  value = length(data.huaweicloud_{service}_{resource}.non_exist.items) == 0
}
`, acceptance.{ENV_VAR})
}
```

**Check函数中添加过滤验证**:
```go
Check: resource.ComposeTestCheckFunc(
    dc.CheckResourceExists(),
    resource.TestCheckResourceAttrSet(dataSource, "items.0.{field}"),
    // ... 其他字段验证
    
    // 过滤参数验证
    resource.TestCheckOutput("filter_1_is_useful", "true"),
    resource.TestCheckOutput("filter_2_is_useful", "true"),
    resource.TestCheckOutput("non_exist_is_zero", "true"),
),
```

### 2. 一次性操作资源类型

**参考示例**: `huaweicloud_ccm_certificate_cancel_request`, `huaweicloud_waf_batch_update_whiteblackip_rules`, `huaweicloud_cbh_reset_login_mode`, `huaweicloud_aad_domain_certificate`

**测试模式**:
- 单步测试
- 无 `CheckDestroy`（设置为 `nil`）
- 无资源状态验证（action资源不维护状态）
- 专注于成功执行
- 可能需要依赖资源进行测试

**结构**:
```go
func TestAcc{Service}{Resource}_basic(t *testing.T) {
    name := acceptance.RandomAccResourceName()
    // 其他变量根据需要
    
    resource.ParallelTest(t, resource.TestCase{
        PreCheck: func() {
            acceptance.TestAccPreCheck(t)
            acceptance.TestAccPreCheck{Specific}(t) // 如需要
        },
        ProviderFactories: acceptance.TestAccProviderFactories,
        CheckDestroy:      nil, // action资源无销毁检查
        Steps: []resource.TestStep{
            {
                Config: testAcc{Service}{Resource}_basic(name, otherVars),
            },
        },
    })
}

func testAcc{Service}{Resource}_basic(name, otherVars string) string {
    return fmt.Sprintf(`
// 如需要，依赖资源

resource "huaweicloud_{service}_{resource}" "test" {
  // 必需参数
}
`, name, otherVars)
}
```

### 3. 不支持删除/查询的资源类型

**参考示例**: `huaweicloud_hss_policy_switch_status`

**测试模式**:
- 测试创建和更新场景
- 无 `CheckDestroy`（资源无法被销毁）
- 使用常量字符串进行配置
- 专注于状态转换

**结构**:
```go
func TestAcc{Service}{Resource}_basic(t *testing.T) {
    resource.ParallelTest(t, resource.TestCase{
        PreCheck: func() {
            acceptance.TestAccPreCheck(t)
        },
        ProviderFactories: acceptance.TestAccProviderFactories,
        Steps: []resource.TestStep{
            {
                Config: test{Service}{Resource}_basic,
            },
            {
                Config: test{Service}{Resource}_basic_update,
            },
        },
    })
}

const test{Service}{Resource}_basic = `
resource "huaweicloud_{service}_{resource}" "test" {
  // 初始配置
}
`

const test{Service}{Resource}_basic_update = `
resource "huaweicloud_{service}_{resource}" "test" {
  // 更新配置
}
`
```

### 4. 常规资源类型

**参考示例**: `huaweicloud_cc_bandwidth_package`, `huaweicloud_ccm_csr`, `huaweicloud_secmaster_pipe_consumption`, `huaweicloud_waf_dedicated_agency`, `huaweicloud_hss_policy_group`

**测试模式**:
- 完整生命周期测试（创建、多步骤更新、导入）
- 通过API调用进行资源存在性验证
- `CheckDestroy` 实现
- 多个测试场景：基础功能、计费模式转换、特殊配置等
- 对必需和计算属性进行字段验证
- 资源绑定和关联测试
- 复杂更新场景测试

**结构**:
```go
func TestAcc{Service}{Resource}_basic(t *testing.T) {
    var obj interface{}
    name := acceptance.RandomAccResourceName()
    rName := "huaweicloud_{service}_{resource}.test"

    rc := acceptance.InitResourceCheck(
        rName,
        &obj,
        getResourceFunc, // API调用函数
    )

    resource.ParallelTest(t, resource.TestCase{
        PreCheck: func() {
            acceptance.TestAccPreCheck(t)
            acceptance.TestAccPreCheck{Specific}(t) // 如需要
        },
        ProviderFactories: acceptance.TestAccProviderFactories,
        CheckDestroy:      rc.CheckResourceDestroy(),
        Steps: []resource.TestStep{
            {
                Config: testAcc{Service}{Resource}_basic(name),
                Check: resource.ComposeTestCheckFunc(
                    rc.CheckResourceExists(),
                    resource.TestCheckResourceAttr(rName, "name", name),
                    // 其他字段验证
                ),
            },
            {
                Config: testAcc{Service}{Resource}_update(name),
                Check: resource.ComposeTestCheckFunc(
                    rc.CheckResourceExists(),
                    resource.TestCheckResourceAttr(rName, "name", name+"-updated"),
                    // 更新字段验证
                ),
            },
            {
                ResourceName:      rName,
                ImportState:       true,
                ImportStateVerify: true,
            },
        },
    })
}
```

#### 资源检查函数

**模式1：简单服务函数调用**（推荐）
```go
func get{Service}{Resource}Func(cfg *config.Config, state *terraform.ResourceState) (interface{}, error) {
    client, err := cfg.NewServiceClient("{service_name}", acceptance.HW_REGION_NAME)
    if err != nil {
        return nil, fmt.Errorf("error creating {Service} client: %s", err)
    }

    return {service_package}.Get{Resource}(client, state.Primary.ID)
}
```

**模式2：无参数查询函数**
```go
func get{Service}{Resource}Func(cfg *config.Config, _ *terraform.ResourceState) (interface{}, error) {
    client, err := cfg.NewServiceClient("{service_name}", acceptance.HW_REGION_NAME)
    if err != nil {
        return nil, fmt.Errorf("error creating {Service} client: %s", err)
    }

    return {service_package}.Query{Something}(client)
}
```

**模式3：复杂API调用模式**

**使用前提**：
- 服务包没有提供对应的Get函数
- 需要复杂的API路径构建或特殊请求头
- 需要自定义响应处理逻辑

**实现指导**：
1. **参考资源代码中的Get方法**：查看`./huaweicloud/services/{service}/{resource}.go`中的Get实现
2. **复制API路径构建逻辑**：包括路径替换、参数处理
3. **保持一致的错误处理**：使用相同的错误格式和响应处理
4. **使用utils.FlattenResponse**：统一响应处理方式

**实现步骤**：
1. 分析资源文件中的Get方法实现
2. 提取API路径和请求逻辑
3. 复制到测试函数中，适配参数来源
4. 确保错误处理和响应处理一致

```go
func get{Service}{Resource}Func(cfg *config.Config, state *terraform.ResourceState) (interface{}, error) {
    client, err := cfg.NewServiceClient("{service_name}", acceptance.HW_REGION_NAME)
    if err != nil {
        return nil, fmt.Errorf("error creating {Service} client: %s", err)
    }

    // 构建API路径（参考资源文件中的Get方法）
    httpUrl := "v2/{project_id}/instances/{instance_id}/apps/{app_id}/resource"
    getPath := client.Endpoint + httpUrl
    getPath = strings.ReplaceAll(getPath, "{project_id}", client.ProjectID)
    getPath = strings.ReplaceAll(getPath, "{instance_id}", state.Primary.Attributes["instance_id"])
    getPath = strings.ReplaceAll(getPath, "{app_id}", state.Primary.Attributes["application_id"])
    
    // 发送请求（参考资源文件中的请求逻辑）
    opt := golangsdk.RequestOpts{
        KeepResponseBody: true,
    }
    requestResp, err := client.Request("GET", getPath, &opt)
    if err != nil {
        return nil, fmt.Errorf("error retrieving resource: %s", err)
    }
    return utils.FlattenResponse(requestResp)
}
```

**模式4：多参数查询函数**
```go
func get{Service}{Resource}Func(cfg *config.Config, state *terraform.ResourceState) (interface{}, error) {
    client, err := cfg.NewServiceClient("{service_name}", acceptance.HW_REGION_NAME)
    if err != nil {
        return nil, fmt.Errorf("error creating {Service} client: %s", err)
    }

    return {service_package}.Get{Something}(client, state.Primary.Attributes["workspace_id"], state.Primary.ID)
}
```

**函数命名规范**:
- 标准命名：`get{Service}{Resource}Func`
- 变体命名：`getResource{Something}Func`

**参数使用模式**:
1. **仅使用ID**：`state.Primary.ID`（最常见）
2. **使用属性+ID**：`state.Primary.Attributes["workspace_id"], state.Primary.ID`
3. **不使用state参数**：`_ *terraform.ResourceState`（无参数查询）
4. **使用多个属性**：`state.Primary.Attributes["instance_id"], state.Primary.Attributes["application_id"]`

**选择模式的指导原则**:
- **优先使用模式1**：如果服务包提供了对应的Get函数
- **使用模式2**：对于查询类资源，不需要特定ID
- **使用模式3**：当需要复杂的API路径构建或特殊请求头时
- **使用模式4**：当查询需要多个参数时（如workspace_id + resource_id）

#### 配置模板

**基础配置模板**:
```go
func testAcc{Service}{Resource}_basic(name string) string {
    return fmt.Sprintf(`
resource "huaweicloud_{service}_{resource}" "test" {
  // 基础配置
}
`, name)
}
```

**复杂配置模板示例**（基于实际测试用例）:
```go
func testBandwidthPackage_basic(name string) string {
    return fmt.Sprintf(`
resource "huaweicloud_cc_connection" "test" {
  name = "%[1]s"
}

resource "huaweicloud_cc_bandwidth_package" "test" {
  name                  = "%[1]s"
  local_area_id         = "Chinese-Mainland"
  remote_area_id        = "Chinese-Mainland"
  charge_mode           = "bandwidth"
  billing_mode          = 3
  bandwidth             = 5
  description           = "test description"
  resource_id           = huaweicloud_cc_connection.test.id
  resource_type         = "cloud_connection"
  interflow_mode        = "Area"
  spec_code             = "bandwidth.cmtocm"
  enterprise_project_id = "%[2]s"

  tags = {
    foo = "bar"
    key = "value"
  }
}
`, name, acceptance.HW_ENTERPRISE_PROJECT_ID)
}
```

**多步骤更新配置**:
```go
func testAcc{Service}{Resource}_update1(name string) string {
    return fmt.Sprintf(`
resource "huaweicloud_{service}_{resource}" "test" {
  // 第一次更新配置
}
`, name)
}

func testAcc{Service}{Resource}_update2(name string) string {
    return fmt.Sprintf(`
resource "huaweicloud_{service}_{resource}" "test" {
  // 第二次更新配置
}
`, name)
}
```

## 字段覆盖指南

1. **必填字段**: 始终使用精确值进行验证
2. **计算字段**: 使用 `resource.TestCheckResourceAttrSet()`
3. **可选字段**: 测试存在和不存在两种场景
4. **ForceNew字段**: 测试导入功能
5. **敏感字段**: 使用适当的验证方法
6. **嵌套块**: 使用适当的索引验证嵌套属性
7. **列表/映射属性**: 验证数量和单个元素
8. **资源关联**: 使用 `resource.TestCheckResourceAttrPair()` 验证资源间关联
9. **多步骤更新**: 测试多次更新场景，验证字段变化
10. **计费模式转换**: 如适用，测试不同计费模式间的转换
11. **ImportStateVerifyIgnore**: 对不稳定的字段使用导入忽略
12. **数据源过滤参数**: 对Optional过滤参数进行功能验证，使用`resource.TestCheckOutput`验证过滤有效性
13. **空结果验证**: 测试过滤条件不存在时返回空列表的场景
14. **多层级字段验证**: 对复杂嵌套结构进行逐层验证，包括顶层列表计数和深层嵌套属性
15. **多必需参数**: 验证所有必需参数的组合使用，确保数据源在完整参数下正常工作
16. **依赖资源验证**: 对需要依赖资源的数据源，使用`depends_on`确保依赖资源先创建

## 环境变量

常用预检查函数：
- `acceptance.TestAccPreCheck(t)` - 基础预检查
- `acceptance.TestAccPreCheckEpsID(t)` - 企业项目ID
- `acceptance.TestAccPreCheckWafInstance(t)` - WAF实例
- `acceptance.TestAccPreCheckWafPolicyId(t)` - WAF策略ID
- `acceptance.TestAccPreCheckCCMSSLCertificateId(t)` - CCM SSL证书

## 最佳实践

1. **使用 `resource.ParallelTest`** 进行并行执行
2. **适当包含 `// lintignore:AT001`** 注释
3. **使用 `acceptance.RandomAccResourceName()`** 生成唯一名称
4. **使用 `acceptance.RandomCidr()`** 生成随机CIDR块
5. **验证所有重要字段** 以实现全面覆盖
6. **遵循命名规范** 保持一致性
7. **包含导入测试** 针对可导入资源
8. **添加注释** 说明复杂测试场景

## 实施步骤

1. **分析资源/数据源实现** 以了解：
   - 必填与可选参数
   - 计算字段
   - ForceNew参数
   - API端点和方法
   - 资源生命周期支持

2. **查看文档** 以了解：
   - 使用示例
   - 参数描述
   - 重要说明和约束

3. **根据资源特性确定测试类型**

4. **按照相应模式生成测试文件**

5. **通过验证所有重要属性确保字段覆盖**

6. **为必需的环境变量添加必要的预检查**

7. **如适用，测试边界情况**（不同参数组合）

---

## 总结

此skill文档提供了全面的测试用例生成指南，强调**文档分析是测试用例生成的基础**。通过系统性的文档分析、规范化的测试模式和最佳实践，可以生成高质量、全面覆盖的测试用例。

**核心要点**：
- 深入文档分析，识别所有测试场景
- 按优先级组织测试用例（P0-P3）
- 正确处理内部字段和更新限制
- 规范编写getResourceFunc
- 根据资源类型选择合适的测试模式
- 遵循最佳实践确保测试质量
