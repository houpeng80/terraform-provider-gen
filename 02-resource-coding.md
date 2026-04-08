# Resource 编码开发技能

## 1. 生成 resource函数

- 该函数的名称格式为 `Resource{XXX}`，其中 `{XXX}` 为该资源的功能信息，采用驼峰状格式，例如：`ResourcePublication`
- 该函数参数为空，返回值为 `*schema.Resource`
- 为 `CreateContext`、`UpdateContext`、`ReadContext`、`DeleteContext` 四个变量设置值，分别为对应的实现函数，格式为`resource{XXX}{YYY}`，其中 `{XXX}` 为该资源的功能信息，`{YYY}`为函数动作
- 如果该服务为全局函数，那么就不需要添加`region`，如果不是全局函数，那么就需要添加参数`region`，并且放到最前边，并且设置`Optional`和`Computed` 为`true`

```go
// 不支持修改的参数列表

// 使用到的API
func ResourcePublication() *schema.Resource {
    return &schema.Resource{
		CreateContext: resourcePublicationCreate, 
		UpdateContext: resourcePublicationUpdate, 
		ReadContext:   resourcePublicationRead, 
		DeleteContext: resourcePublicationDelete,

		// 导入信息
		
		// CustomizeDiff 信息
		
		// 超时时间
		
        Schema: map[string]*schema.Schema{
            "region": {
                Type:     schema.TypeString,
                Optional: true,
                Computed: true,
            },
            // URI参数
			
            // 请求参数
			
			// 包周期参数
			"charging_mode": {
				Type:     schema.TypeString,
				Optional: true,
				Computed: true,
				ValidateFunc: validation.StringInSlice([]string{
					"prePaid", "postPaid",
				}, false),
			},
			"period_unit": {
				Type:         schema.TypeString,
				Optional:     true,
				RequiredWith: []string{"period"},
				ValidateFunc: validation.StringInSlice([]string{
					"month", "year",
				}, false),
			},
			"period": {
				Type:         schema.TypeInt,
				Optional:     true,
				RequiredWith: []string{"period_unit"},
			},
			"auto_renew": common.SchemaAutoRenewUpdatable(nil),
			"auto_pay":   common.SchemaAutoPay(nil),
			
            // 更新参数

            // 响应消息
        },
    }
}

// 创建resource相关函数

// 更新resource相关函数

// 查询resource相关函数

// 删除resource相关函数

// 导入resource相关函数
```

## 2. 添加创建resource相关函数，生成创建资源所需方法

### 2.1 获取创建resource的数据

- 调用用户提供的创建资源的API，从结果中获取到当前服务信息、功能、URI信息（包括URI和参数信息）、请求参数和响应消息
- 禁止使用 `web reference` 的内容，使用 `WebFetch` 获取完整的页面内容

### 2.2 添加创建resource的参数

- 添加URI信息中的参数信息到resource函数中（示例中的`URI参数`位置），如果有参数`project_id`，就忽略掉，如果是必填，就设置`Required`为 `true`，如果是可选参数，就就设置`Optional`为 `true`
- 添加请求参数到resource函数中（示例中的`请求参数`位置），如果是必填，就设置`Required`为 `true`，如果是可选参数，就就设置`Optional`为 `true`，禁止设置`ForceNew`
- 如果请求参数类型为bool，那么就将类型设置为string，并且添加validation限制ValidateFunc: `validation.StringInSlice([]string{"true", "false"}, false)`
- 所有参数中，如果参数类型为对象，在这里对应的类型为list，设置`MaxItems`为 `1`，并且必须在子函数中展示该对象的参数，子函数名格式为`{XXX}{YYY}Schema`，其中`{XXX}`为该资源的功能信息， `{YYY}` 为该参数的驼峰形式，子函数放到resource函数后边
- 如果API请求参数中包含计费模式字段，如`charge_mode`或`charge_info`等字段，并且其说明中支持预付费模式，即包年/包月，或者取值包含`prePaid`，那么说明该API支持包周期
- 如果API支持包周期，那么就添加包周期参数： `charging_mode`、`period_unit`、`period`、`auto_renew`、`auto_pay`
- 如果API支持包周期，那么在资源函数前边加两个注释（示例中的`使用到的API`位置）：`@API BSS POST /v2/orders/subscriptions/resources/autorenew/{instance_id}`和`@API BSS DELETE /v2/orders/subscriptions/resources/autorenew/{instance_id}`
- 最后在resource函数前边加一个注释（示例中的`使用到的API`位置），格式为`// @API {service} {method} {path}`，其中`{service}`为当前服务名，`{method}`为URI中的请求方法，`{path}`为URI中的请求path

```go
// 使用到的API
func ResourcePublication() *schema.Resource {
    return &schema.Resource{
		CreateContext: resourcePublicationCreate, 
		UpdateContext: resourcePublicationUpdate, 
		ReadContext:   resourcePublicationRead, 
		DeleteContext: resourcePublicationDelete,

		// 导入信息
		
		// CustomizeDiff 信息
		
		// 超时时间

        Schema: map[string]*schema.Schema{
            "region": {
                Type:     schema.TypeString,
                Optional: true,
                Computed: true,
            },
            // URI参数
			"instance_id": {
                Type:     schema.TypeString,
				Required: true,
            },
            // 请求参数
            "bucket_name": {
                Type:     schema.TypeString,
                Optional: true,
            },
			"bucket_exist": {
                Type:     schema.TypeString,
                Optional: true,
                ValidateFunc: validation.StringInSlice([]string{
					"true", "false",
				}, false),
            },
            "subscription_options": {
				Type:     schema.TypeList,
				Optional: true,
				MaxItems: 1,
				Elem:     publicationSubscriptionOptionsSchema(),
			},
            "tables": {
				Type:     schema.TypeSet,
				Optional: true,
				Elem:     publicationTablesSchema(),
			},
			// 包周期参数
			"charging_mode": {
				Type:     schema.TypeString,
				Optional: true,
				Computed: true,
				ValidateFunc: validation.StringInSlice([]string{
					"prePaid", "postPaid",
				}, false),
			},
			"period_unit": {
				Type:         schema.TypeString,
				Optional:     true,
				RequiredWith: []string{"period"},
				ValidateFunc: validation.StringInSlice([]string{
					"month", "year",
				}, false),
			},
			"period": {
				Type:         schema.TypeInt,
				Optional:     true,
				RequiredWith: []string{"period_unit"},
			},
			"auto_renew": common.SchemaAutoRenewUpdatable(nil),
			"auto_pay":   common.SchemaAutoPay(nil),
			// ......
        },
		
		// ......
    }
}

func publicationSubscriptionOptionsSchema() *schema.Resource {
	return &schema.Resource{
		Schema: map[string]*schema.Schema{
			"independent_agent": {
				Type:         schema.TypeString,
				Optional:     true,
				ValidateFunc: validation.StringInSlice([]string{"true", "false"}, false),
			},
			"snapshot_always_available": {
				Type:         schema.TypeString,
				Optional:     true,
			},
		},
	}
}

func publicationTablesSchema() *schema.Resource {
	return &schema.Resource{
		Schema: map[string]*schema.Schema{
			"table_name": {
				Type:     schema.TypeString,
				Required: true,
			},
			"schema": {
				Type:     schema.TypeString,
				Optional: true,
			},
		},
	}
}
```

### 2.3 生成创建resource的函数

- 生成一个创建资源的函数，格式为`resource{XXX}Create`，其中 `{XXX}` 为该资源的功能信息，例如：`resourcePublicationCreate`
- 该函数总共包含：函数名称、参数定义、创建client、构造请求参数、构造请求体、发送请求、解析结果、设置资源id、等待任务或订单完成

```go
func resourcePublicationCreate(ctx context.Context, d *schema.ResourceData, meta interface{}) diag.Diagnostics {
    // 参数定义

	// 创建client

    // 构造请求参数和请求体
	
	// 发送请求
	
	// 解析结果
	
	// 设置资源id
	
	// 等待任务或订单完成

	return resourcePublicationRead(ctx, d, meta)
}
```

### 2.4 参数定义，创建client

- 创建client时，其中第一个参数为服务类型，错误信息中的服务类型需要大写

```go
	cfg := meta.(*config.Config)
	region := cfg.GetRegion(d)

    var (
		httpUrl = "v3/{project_id}/instances/{instance_id}/replication/publications"
		product = "rds"
	)

	client, err := cfg.NewServiceClient(product, region)
	if err != nil {
		return diag.Errorf("error creating RDS client: %s", err)
	}
```

### 2.5 构造请求参数和请求体

- 将path参数替换为具体的值，禁止使用`utils.ReplacePathVariables`，使用`strings.ReplaceAll`
- 构造请求体，函数名为 `buildCreate{XXX}BodyParams`， 其中 `{XXX}` 为该资源的功能信息，最后使用`utils.RemoveNil`去除掉值为nil的参数
- 需要引入包`ithub.com/chnsz/golangsdk`

```go
    createPath := client.Endpoint + httpUrl
    createPath = strings.ReplaceAll(createPath, "{project_id}", client.ProjectID)
    createPath = strings.ReplaceAll(createPath, "{instance_id}", d.Get("instance_id").(string))
	
	createOpt := golangsdk.RequestOpts{
		KeepResponseBody: true,
		MoreHeaders: map[string]string{"Content-Type": "application/json"},
	}
	createOpt.JSONBody = utils.RemoveNil(buildCreatePublicationBodyParams(d))
```

### 2.6 发送请求、解析结果

```go
    createResp, err := client.Request("POST", getPath, &getOpt)
	if err != nil {
		return diag.Errorf("error creating RDS publication: %s", err)
	}

    createRespBody, err := utils.FlattenResponse(createResp)
	if err != nil {
		return diag.FromErr(err)
	}
```

### 2.7 设置资源id

- 如果用户说明了获取资源`id`的位置，那么就从用户指定的位置获取`id`
- 如果用户没有说明获取`id`的位置，那么就直接从API响应消息中查找`id`

```go
    id := utils.PathSearch("id", createRespBody, "").(string)
	if id == "" {
		return diag.Errorf("error creating RDS publication: ID is not found in API response")
	}
	d.SetId(id)
```

### 2.8 等待任务或订单完成

#### 2.8.1 添加等待订单完成逻辑

- 从API响应消息中获取订单ID信息，如`order_id`或`order_info`，如果存在，那么就等待订单的完成
- 如果API支持包周期，那么在resource函数前边加一个注释（示例中的`使用到的API`位置）：`@API BSS GET /v2/orders/customer-orders/details/{order_id}`

```go
    orderId := utils.PathSearch("order_id", createRespBody, "").(string)
	if orderId != "" {
		bssClient, err := cfg.BssV2Client(region)
		if err != nil {
			return diag.Errorf("error creating BSS v2 client: %s", err)
		}
		err = common.WaitOrderComplete(ctx, bssClient, orderId, d.Timeout(schema.TimeoutCreate))
		if err != nil {
			return diag.FromErr(err)
		}
	}
```

#### 2.8.2 添加等待任务完成逻辑

- 从API响应消息中获取任务ID信息，如果存在，那么就等待任务的完成
- 函数名为 `check{XXX}JobFinish`， 其中 `{XXX}` 为该资源的功能信息

```go
    jobId := utils.PathSearch("job_id", createRespBody, "").(string)
	if jobId != "" {
		if err = checkInstanceJobFinish(client, jobId, d.Timeout(schema.TimeoutCreate)); err != nil {
			return diag.Errorf("error creating publication: %s", err)
		}
	}
```

### 2.9 生成函数请求体

- 首先需要构造请求体，函数名为 `buildCreate{XXX}BodyParams`， 其中 `{XXX}` 为该资源的功能信息
- 如果API支持包周期，那么参数中的自动支付参数，如`auto_pay`，`is_auto_pay`， 直接设置为`true`
- 将函数放到创建resource的函数的后边

```go
func buildCreatePublicationBodyParams(d *schema.ResourceData) map[string]interface{} {
	bodyParams := map[string]interface{}{
		"publication_name":               d.Get("publication_name"),
		"publication_database":           d.Get("publication_database"),
		"is_create_snapshot_immediately": isCreateSnapshotImmediately,
		"subscription_options":           buildPublicationSubscriptionOptionsBodyParams(d.Get("subscription_options")),
		"job_schedule":                   buildPublicationJobScheduleBodyParams(d.Get("job_schedule")),
		"extend_tables":                  utils.ValueIgnoreEmpty(d.Get("extend_tables").(*schema.Set).List()),
		"tables":                         tables,
	}
}
```

### 2.10 生成等待任务函数

#### 2.10.1 添加等待时间

- 在资源函数中添加创建资源等待时间（示例中的`超时时间`位置）

```go
    Timeouts: &schema.ResourceTimeout{
		Create:  schema.DefaultTimeout(30 * time.Minute),
		// ....
	},
```

#### 2.10.2 创建等待任务函数

- 首先需要构造请求体，函数名为 `check{XXX}JobFinish`， 其中 `{XXX}` 为该资源的功能信息
- 首先定义一个变量`stateConf`，其值为`resource.StateChangeConf`的引用，创建该变量时需要设置`Pending`、`Target`、`Refresh`、`Timeout`、`PollInterval`
- 其中`Pending`为一个包含等待状态的字符串数组，其值为 `Pending`， 其中`Target`为一个包含等待状态的字符串数组，其值为 `Completed`
- `Refresh` 为刷新状态函数，需要定义一个，函数名为`{XXX}JobStatusRefreshFunc`， 其中 `{XXX}` 为该资源的功能信息
- `Timeout` 为等待的超时时长，从参数中获取，`PollInterval` 固定为`10 * time.Second`
- 在resource函数前边加一个注释（示例中的`使用到的API`位置），格式为`// @API {service} {method} {path}`，其中`{service}`为当前服务名，`{method}`为URI中的请求方法，`{path}`为URI中的请求path
- 将函数放到最后边

```go
func checkPublicationJobFinish(ctx context.Context, client *golangsdk.ServiceClient, jobID string,
	timeout time.Duration) error {
	stateConf := &resource.StateChangeConf{
		Pending:      []string{"Pending"},
		Target:       []string{"Completed"},
		Refresh:      publicationJobStatusRefreshFunc(client, jobID),
		Timeout:      timeout,
		PollInterval: 10 * time.Second,
	}
	if _, err := stateConf.WaitForStateContext(ctx); err != nil {
		return fmt.Errorf("error waiting for RDS publication job (%s) to be completed: %s ", jobID, err)
	}
	return nil
}
```

#### 2.10.3 创建刷新状态函数

- 首先需要构造请求体，函数名为 `{XXX}JobStatusRefreshFunc`， 其中 `{XXX}` 为该资源的功能信息
- 禁止使用`utils.ReplacePathVariables`，使用`strings.ReplaceAll`
- 调用查询任务API，从结果中获取`status`，如果中间过程有报错，那么返回状态为`Failed`，对应的返回格式为`return nil, "Failed", err`
- 如果没有获取到`status`，则返回状态值为`Failed`，对应的返回格式为`return nil, "Failed", err`
- 如果`status`值为`success`、`completed`等类似成功的状态，则返回状态为`Completed`, 对应的返回格式为`return getRespBody, "Completed", nil`
- 如果`status`值为`fail`、`error`等类似失败的状态，则返回状态为`Failed`, 对应的返回格式为`return getRespBody, "Failed", fmt.Errorf("the job is fail")`
- 如果上述情况都不满足，那么就返回等待状态`Pending`，对应的返回格式为`return getRespBody, "Pending", nil`

```go
func publicationJobStatusRefreshFunc(client *golangsdk.ServiceClient, jobId string) resource.StateRefreshFunc {
	return func() (interface{}, string, error) {
		var (
			httpUrl = "v3/{project_id}/jobs?id={job_id}"
		)

		getPath := client.Endpoint + httpUrl
		getPath = strings.ReplaceAll(getPath, "{project_id}", client.ProjectID)
		getPath = strings.ReplaceAll(getPath, "{job_id}", jobId)

		getOpt := golangsdk.RequestOpts{
			KeepResponseBody: true,
			MoreHeaders: map[string]string{"Content-Type": "application/json"},
		}
		getResp, err := client.Request("GET", getPath, &getOpt)
		if err != nil {
			return nil, "Failed", err
		}

		getRespBody, err := utils.FlattenResponse(getResp)
		if err != nil {
			return nil, "Failed", err
		}

		status := utils.PathSearch("status", getRespBody, "").(string)
		if status == "ERROR" {
			return nil, status, fmt.Errorf("the job is fail")
		}
		if status == "SUCCESS" {
			return getRespBody, "Completed", nil
		}
		return getRespBody, "Pending", nil
	}
}
```

## 3. 生成查询resource相关函数，生成查询资源所需方法

- 禁止使用查询任务的API查询resource信息

### 3.1 如果没有提供查询资源API（不包含查询任务的API）

- 生成一个空函数，格式为`resource{XXX}Read`，其中 `{XXX}` 为该资源的功能信息，例如：`resourcePublicationRead`

```go
func resourcePublicationRead(_ context.Context, _ *schema.ResourceData, _ interface{}) diag.Diagnostics {
	return nil
}
```

### 3.2 如果提供了查询资源API（不包含查询任务的API）

#### 3.2.1 获取查询resource的数据

- 调用用户提供的查询资源的API，从结果中获取到当前服务信息、功能、URI信息（包括URI和参数信息）、请求参数和响应消息
- 禁止使用 `web reference` 的内容，使用 `WebFetch` 获取完整的页面内容

#### 3.2.2 添加查询resource的参数

##### 3.2.2.1 API不为分页接口

- 从响应消息中获取返回的resource信息
- 如果响应消息中的参数在前边创建的 `resource 函数` 的请求参数中不存在，那么就添加到`resource 函数`中的`响应消息`处，并设置`Computed`为 `true`

##### 3.2.2.2 API为分页接口

- 从响应消息中获取返回的resource信息，找到要查询的信息列表，并获取列表的元素详情
- 如果列表元素详情中的参数在前边创建的 `resource 函数` 的请求参数中不存在，那么就添加到`resource 函数`中的`响应消息`处，并设置`Computed`为 `true`

#### 3.2.3 添加查询resource的参数

- 生成一个查询资源的函数，格式为`resource{XXX}Read`，其中 `{XXX}` 为该资源的功能信息，例如：`resourcePublicationRead`
- 该函数总共包含：函数名称、参数定义、创建client、构造请求参数、构造请求体、发送请求、解析结果、设置返回参数
- 最后在resource函数前边加一个注释（示例中的`使用到的API`位置），格式为`// @API {service} {method} {path}`，其中`{service}`为当前服务名，`{method}`为URI中的请求方法，`{path}`为URI中的请求path

```go
func resourcePublicationRead(ctx context.Context, d *schema.ResourceData, meta interface{}) diag.Diagnostics {
    // 参数定义

	// 创建client

    // 构造请求参数
	
	// 构造请求体
	
	// 发送请求
	
	// 解析结果
	
	// 设置返回参数
	
	// 设置其他API查询参数
}
```

#### 3.2.4 参数定义，创建client

- 创建client时，其中第一个参数为服务类型，错误信息中的服务类型需要大写

```go
	cfg := meta.(*config.Config)
	region := cfg.GetRegion(d)

	var (
		httpUrl = "v3/{project_id}/instances/{instance_id}/replication/publications"
		product = "rds"
	)

	client, err := cfg.NewServiceClient(product, region)
	if err != nil {
		return diag.Errorf("error creating RDS client: %s", err)
	}
```

#### 3.2.5 构造请求url

- 将path参数替换为具体的值，禁止使用`utils.ReplacePathVariables`，使用`strings.ReplaceAll`
- 如果查询参数是path参数，那么就需要添加分页参数之外的所有查询参数，通过`buildGet{XXX}QueryParams`方法实现，其中 `{XXX}` 为该资源的功能信息

```go
    getPath := client.Endpoint + httpUrl
    getPath = strings.ReplaceAll(getPath, "{project_id}", client.ProjectID)
    getPath = strings.ReplaceAll(getPath, "{instance_id}", d.Get("instance_id").(string))
    getPath += buildGetPublicationQueryParams(d)
```

#### 3.2.6 构造请求体

- 如果API支持分页，并且请求URI中的请求方法不为GET，那么需要生成请求体，参数名为：`getOpt`
- 如果API不支持分页，那么就需要生成请求体，参数名为：`getOpt`
- 使用`golangsdk.RequestOpts`生成请求体，禁止使用`utils.BaseRequestOpts()`

```go
	getOpt := golangsdk.RequestOpts{
        KeepResponseBody: true,
		MoreHeaders: map[string]string{"Content-Type": "application/json"}
	}
```

#### 3.2.7 发送请求、解析结果

##### 3.2.7.1 如果API不支持分页

- 直接发送请求，参数依次为：URI中的请求方法、请求url、请求体，最后解析结果
- 最后使用`utils.PathSearch`从结果中获取结果中根据`id`获取数据，如果结果为nil，就直接返回`common.CheckDeletedDiag`错误，其中包含 `Method`，`URL`，`RequestId`，`Body`

```go
    getResp, err := client.Request("GET", getPath, &getOpt)
	if err != nil {
		return diag.Errorf("error retrieving RDS publication: %s", err)
	}

	getRespBody, err := utils.FlattenResponse(getResp)
	if err != nil {
		return diag.FromErr(err)
	}
	publication := utils.PathSearch(fmt.Sprintf("publications[?id=='%s']|[0]", d.Id()), getRespBody, nil)
	if publication == nil {
		return common.CheckDeletedDiag(d, golangsdk.ErrDefault404{
			ErrUnexpectedResponseCode: golangsdk.ErrUnexpectedResponseCode{
				Method:    "GET",
				URL:       "/v3/{project_id}/instances/{instance_id}/replication/publications",
				RequestId: "NONE",
				Body:      []byte(fmt.Sprintf("the RDS publication (%s) does not exist", d.Id())),
			},
		}, "error retrieving RDS publication")
	}
```

##### 3.2.7.2 如果API支持分页，并且URI中的请求方法为 GET，那么直接使用 pagination 包中的 ListAllItems 函数

- 分页参数为 limit + offset, ListAllItems中第二个参数qType为"offset"
- 分页参数为 pagesize + page, ListAllItems中第二个参数qType为"page"
- 分页参数为 limit + marker, ListAllItems中第二个参数qType为"marker"， 如果返回值中包含下一页的marker时，第四个参数中的MarkerField为下一页的marker
- 最后使用`utils.PathSearch`从结果中获取结果中根据`id`获取数据，如果结果为nil，就直接返回`common.CheckDeletedDiag`错误，其中包含 `Method`，`URL`，`RequestId`，`Body`

```go
    getResp, err := pagination.ListAllItems(
		client,
		"offset",
		listPath,
		&pagination.QueryOpts{MarkerField: ""})
    getRespJson, err := json.Marshal(getResp)
	if err != nil {
		return diag.FromErr(err)
	}
	var getRespBody interface{}
	err = json.Unmarshal(getRespJson, &getRespBody)
	if err != nil {
		return diag.FromErr(err)
	}
	
	publication := utils.PathSearch(fmt.Sprintf("publications[?id=='%s']|[0]", d.Id()), getRespBody, nil)
	if publication == nil {
		return common.CheckDeletedDiag(d, golangsdk.ErrDefault404{
			ErrUnexpectedResponseCode: golangsdk.ErrUnexpectedResponseCode{
				Method:    "GET",
				URL:       "/v3/{project_id}/instances/{instance_id}/replication/publications",
				RequestId: "NONE",
				Body:      []byte(fmt.Sprintf("the RDS publication (%s) does not exist", d.Id())),
			},
		}, "error retrieving RDS publication")
	}
```

##### 3.2.7.3 如果API支持分页，并且URI中的请求方法不为 GET

- 首先定义一个零时变量 `res` 保存查询结果
- 定义分页参数，如果API中包含`limit`，记录API中允许设置的最大值，记为`maxLimit`，定义变量`offset`，如果API是使用page+size分页，那么定义变量`page`，并且赋值为 `1`
- 如果请求参数在请求体中，首先需要构造请求体，函数名为 `buildGet{XXX}BodyParams`， 其中 `{XXX}` 为该资源的功能信息，其中要包含分页参数， 最后使用utils.RemoveNil去除掉值为nil的参数
- 在for循环查询所有页的结果，请求参数依次为：URI中的请求方法、请求url、请求体
- 使用`utils.PathSearch`从结果中获取结果中根据`id`获取数据，如果结果不为空，说明匹配到了数据，将结果赋值给`res`变量，然后使用`break`跳出循环
- 如果结果为空，那么使用`utils.PathSearch`从结果中获取返回数据列表，如果结果为空，那么说明已经没有数据，就使用`break`跳出循环
- 更新分页参数，如果API中包含`offset`，那么`offset`增加`maxLimit`, 如果API是使用page+size分页，那么定义变量`page`， 那么`page`增加`1`
- 最后判断`res`是否为空，如果`res`为空，那么就直接返回`common.CheckDeletedDiag`错误，其中包含 `Method`，`URL`，`RequestId`，`Body`

```go
    offset := 0
	res := make([]map[string]interface{}, 0)
	for {
		getOpt.JSONBody = utils.RemoveNil(buildGetBackupDatabasesBodyParams(d, limit))
		getResp, err := client.Request("POST", getPath, &getOpt)
		if err != nil {
			return diag.Errorf("error retrieving RDS publication: %s", err)
		}

		getRespBody, err := utils.FlattenResponse(getResp)
		if err != nil {
			return diag.FromErr(err)
		}
		res = utils.PathSearch(fmt.Sprintf("publications[?id=='%s']|[0]", d.Id()), getRespBody, nil)
		if res != nil {
		    break
		}
        publications = utils.PathSearch("publications", getRespBody, make([]interface{}, 0)).([]interface{})
        if len(publications) == 0 {
            break
        }
		
		offset += 100
	}
	if res == nil {
		return common.CheckDeletedDiag(d, golangsdk.ErrDefault404{
			ErrUnexpectedResponseCode: golangsdk.ErrUnexpectedResponseCode{
				Method:    "GET",
				URL:       "/v3/{project_id}/instances/{instance_id}/replication/publications",
				RequestId: "NONE",
				Body:      []byte(fmt.Sprintf("the RDS publication (%s) does not exist", d.Id())),
			},
		}, "error retrieving RDS publication")
	}
```

#### 3.2.8 设置返回参数

- 如果该服务为全局函数，那么就不需要返回`region`，如果不是全局函数，那么就需要返回`region`
- 如果参数类型不为对象或者列表，那么就直接从返回结果中获取后设置即可
- 如果参数类型为对象或者列表， 需要使用函数 `flattenGet{XXX}Body` 解析当前查询结果，其中 `{XXX}` 为该资源的功能信息，如果有嵌套，那么`{XXX}` 为该资源的功能信息加上变量名，为驼峰格式

```go
    mErr := multierror.Append(
		d.Set("region", region),
		d.Set("publication_name", utils.PathSearch("publication_name", publication, nil)),
		d.Set("publication_database", utils.PathSearch("publication_database", publication, nil)),
		d.Set("subscription_options", flattenPublicationSubscriptionOptions(publication)),
		d.Set("job_schedule", flattenPublicationJobSchedule(publication)),
		d.Set("is_select_all_table", strconv.FormatBool(
			utils.PathSearch("is_select_all_table", publication, false).(bool))),
		d.Set("extend_tables", utils.PathSearch("extend_tables", publication, nil)),
		d.Set("tables", flattenPublicationTables(publication)),
		d.Set("status", utils.PathSearch("status", publication, nil)),
		d.Set("subscription_count", utils.PathSearch("subscription_count", publication, nil)),
	)

	return diag.FromErr(mErr.ErrorOrNil())
```

#### 3.2.9 设置其他API查询参数

// todo

#### 3.2.10 生成参数函数、请求体函数、解析结果函数

##### 3.2.10.1 生成参数函数

- 函数名为 `buildGet{XXX}QueryParams`， 其中 `{XXX}` 为该资源的功能信息
- 如果参数为 `Required`，那么不需要判断，直接添加到结果即可，如果分页参数为必填，也需要添加上
- 如果参数为 `Optional`，那么需要先判断是否为空
- 如果参数类型为`bool`，但是通过`ValidateFunc`函数限制为`true`和`false`，那么需要先将其转换为`bool`，然后再添加到结果
- 如果参数类型为`list`，那么需要遍历该链表，然后逐个添加到结果中
- 将函数放到查询resource的函数的后边

```go
func buildGetInstanceQueryParams(d *schema.ResourceData) string {
    res := ""
	res = fmt.Sprintf("%s&version_name=%v", res, v)
    if v, ok := d.GetOk("bucket_name"); ok {
    	res = fmt.Sprintf("%s&bucket_name=%v", res, v)
    }
    if v, ok := d.GetOk("bucket_exist"); ok {
        bucketExist, _ := strconv.ParseBool(v["bucket_exist"].(string))
    	res = fmt.Sprintf("%s&is_flexus=%v", res, v)
    }
    if v, ok := d.GetOk("bucket_params"); ok {
		for _, param := range v.([]interface{}) {
			res = fmt.Sprintf("%s&bucket_params=%v", res, param)
		}
	}
    // ... 其他查询参数
     
    if res != "" {
    	res = "?" + res[1:]
    }
    return res
}
```

##### 3.2.10.2 生成请求体函数

- 函数名为 `buildGet{XXX}BodyParams`， 其中 `{XXX}` 为该API要获取的功能信息
- 如果API中包含`offset`，那么使用分页参数`offset`，`limit`为API中允许设置的最大值，记为`maxLimit`，如果API是使用`page+size`分页，那么使用分页参数`page`
- 该请求体要和上边构建请求参数的函数有区别，参数判空不要使用`d.GetOk`，使用`utils.ValueIgnoreEmpty`
- 将函数放到查询resource函数的后边

```go
func buildGetInstanceBodyParams(d *schema.ResourceData, offset int) map[string]interface{} {
	bodyParams := map[string]interface{}{
		"states":   utils.ValueIgnoreEmpty(d.Get("states")),
		"group_id": utils.ValueIgnoreEmpty(d.Get("group_id")),
		"limit":    100,
		"offset":    offset,
	}
	return bodyParams
}
```

##### 3.2.10.3 解析结果函数

###### 3.2.10.1.1 返回参数类型为 list

- 函数名为 `flattenGet{XXX}Body`， 其中 `{XXX}` 为该参数的驼峰形式，返回值类型为`[]interface{}`
- 首先使用`utils.PathSearch`从结果中获取结果，将结果转为数组后遍历，依次将数组对象中的每个元素添加到对应的map中，然后添加到返回对象中
- 如果数组元素中参数类型为对象或者列表，那么就调调用解析结果函数，在子函数中实现，函数名定义和当前函数名类似规则
- 如果数组中元素有递归结果，那么就只保留第一层，下边的直接将结果转换为json格式的字符串
- 最后返回结果
- 将函数放到查询resource的函数的后边

```go
func flattenGetBackupDatabasesBody(resp interface{}) []interface{} {
	curJson := utils.PathSearch("instance_publications", resp, make([]interface{}, 0))
	curArray := curJson.([]interface{})
	res := make([]interface{}, 0, len(curArray))
	for _, v := range curArray {
		res = append(res, map[string]interface{}{
			"database_id":   utils.PathSearch("database_id", v, nil),
			"database_name": utils.PathSearch("database_name", v, nil),
			"database_attr": flattenGetBackupDatabasesDatabaseAttrBody(v),
		})
	}
	return res
}
```

###### 3.2.10.1.2 返回参数类型为对象

- 函数名为 `flattenGet{XXX}Body`， 其中 `{XXX}` 为该参数的驼峰形式，返回值类型为`[]interface{}`
- 首先使用`utils.PathSearch`从结果中获取结果，奖结果对象中的参数依次添加到一个map中，然后将该map添加到一个数组中
- 如果对象中参数类型为对象或者列表，那么就调用递归调用解析结果函数，函数名定义和当前函数名类似规则
- 最后返回结果
- 将函数放到查询resource的函数的后边

```go
func flattenGetInstanceBody(resp interface{}) []interface{} {
	curJson := utils.PathSearch("instance_publications", resp, nil)
	if curJson == nil {
	    return nil
	}
	
	res := []interface{}{
		map[string]interface{}{
			"database_id": utils.PathSearch("database_id", resp, nil),
			"database_name": utils.PathSearch("database_name", v, nil),
			"database_attr": flattenGetBackupDatabasesDatabaseAttrBody(v),
		},
	}
	return res
}
```

#### 3.2.11 生成导入信息

##### 3.2.11.1 如果没有提供查询resource的API

- 禁止生成导入信息

##### 3.2.11.2 如果有提供查询resource的API

###### 3.2.11.2.1 如果没有提供导入id格式

- 直接使用默认导入方法，在资源函数中添加导入信息（示例中的`导入信息`位置）

```go
    Importer: &schema.ResourceImporter{
	    StateContext: schema.ImportStatePassthroughContext,
	},
```

###### 3.2.11.2.2 如果有提供导入id格式

- 需要使用自定义导入函数，函数名为： `resource{XXX}ImportState`，其中 `{XXX}` 为该资源的功能信息

```go
    Importer: &schema.ResourceImporter{
		StateContext: resourceInstanceImportState,
	},
```

- 生成自定义导入函数，函数名为： `resource{XXX}ImportState`，其中 `{XXX}` 为该资源的功能信息
- 函数首先使用`strings.Split`对导入id（`d.Id()`），分割符号为`/`
- 然后判断分割结果长度是否符合预期，如果不符合，那么就直接返回异常
- 然后重新设置资源id，一般为分割后结果的最后一个元素
- 最后将分割结果设置到对应的参数中，返回结果
- 将该函数放到最后边

```go
func resourceInstanceImportState(_ context.Context, d *schema.ResourceData, _ interface{}) ([]*schema.ResourceData,
	error) {
	parts := strings.Split(d.Id(), "/")

	if len(parts) != 2 {
		return nil, errors.New("invalid format specified for import ID, must be <instance_id>/<id>")
	}

	d.SetId(parts[1])
	mErr := multierror.Append(nil,
		d.Set("instance_id", parts[0]),
	)

	return []*schema.ResourceData{d}, mErr.ErrorOrNil()
}
```



## 4. 生成更新resource相关函数，生成更新资源所需方法

### 4.1 如果没有提供更新API，并且参数中没有 `enterprise_project_id`、 `tags`和 `auto_renew`

- 生成一个空函数，格式为`resource{XXX}Update`，其中 `{XXX}` 为该资源的功能信息，例如：`resourcePublicationUpdate`

```go
func resourcePublicationUpdate(_ context.Context, _ *schema.ResourceData, _ interface{}) diag.Diagnostics {
	return nil
}
```

### 4.2 如果提供了更新API，或者有 `enterprise_project_id`、 `tags`和 `auto_renew` 至少一个参数

- 生成一个创建资源的函数，格式为`resource{XXX}Update`，其中 `{XXX}` 为该资源的功能信息，例如：`resourcePublicationUpdate`
- 该函数总共包含：函数名称、参数定义、创建client、更新参数方法、更新enterprise_project_id、更新tags、更新auto_renew

```go
func resourcePublicationUpdate(ctx context.Context, d *schema.ResourceData, meta interface{}) diag.Diagnostics {
    // 参数定义

	// 创建client

    // 更新参数方法
	
	// 更新enterprise_project_id
	
	// 更新tags
	
	// 更新auto_renew

	return resourcePublicationRead(ctx, d, meta)
}
```

#### 4.2.1 参数定义，创建client

- 创建client时，其中第一个参数为服务类型，错误信息中的服务类型需要大写
- 如果参数中包含包周期参数，那么就需要创建更新包周期参数所需的client，变量名为`bssClient`

```go
	cfg := meta.(*config.Config)
	region := cfg.GetRegion(d)

	var (
		product = "rds"
	)

	client, err := cfg.NewServiceClient("rds", region)
	if err != nil {
		return diag.Errorf("error creating RDS client: %s", err)
	}
	bssClient, err := cfg.BssV2Client(region)
	if err != nil {
		return diag.Errorf("error creating bss V2 client: %s", err)
	}
```

#### 4.2.2 更新参数方法

- 根据用户提供的更新resource参数API，按照以下步骤，依次添加参数更新方法

##### 4.2.2.1 获取更新resource参数的数据

- 调用用户提供的查询资源的API，从结果中获取请求参数
- 禁止使用 `web reference` 的内容，使用 `WebFetch` 获取完整的页面内容

##### 4.2.2.2 生成更新参数逻辑

- 如果API请求参数只有一个，那么就使用`d.HasChange`方法判断参数是否修改
- 如果API请求参数有多个，那么就使用`d.HasChanges`方法判断参数是否修改
- 更新参数通过`update{XXX}{YYY}`方法实现，其中 `{XXX}` 为该资源的功能信息，`{YYY}` 为参数名称，函数名最终采用驼峰格式
- 函数参数首先需要包含`d`和`client`
- 如果API响应消息中包含`job_id`等任务信息，那么函数参数需要包含`ctx`
- 如果API请求消息中包含包周期相关参数，例如`自动支付（is_auto_pay）`，或者API响应消息中包含`order_id`，那么函数参数需要包含`bssClient`
- 最后判断返回错误信息是否为空，如果为空，就要return错误信息，使用`diag.FromErr`处理错误信息

```go
    if d.HasChange("name") {
		if err = updateInstanceName(d, client); err != nil {
			return diag.FromErr(err)
		}
	}
```

```go
	if d.HasChanges("kms_key_id", "description", "event_subscriptions") {
		if err := updateInstance(ctx, d, client); err != nil {
			return diag.FromErr(err)
		}
	}
```

```go
    if d.HasChange("flavor") {
		if err = updateInstanceFlavor(ctx, d, client, bssClient); err != nil {
			return diag.FromErr(err)
		}
	}
```

#### 4.2.3 更新enterprise_project_id

- 如果请求`创建资源API`参数中包含 `enterprise_project_id`，那么就添加`更新enterprise_project_id`
- 定义一个参数`migrateOpts`，创建一个对象`config.MigrateResourceOpts`，其中`ResourceId`为资源id，`ResourceType`为资源服务名的小写，`RegionId`为`region`，`ProjectId`为资源`client.ProjectID`
- 使用`cfg.MigrateEnterpriseProject`方法去更新`enterprise_project_id`，如果返回`err`不为空，则返回错误信息
- 最后在资源函数前边加一个注释（示例中的`使用到的API`位置）：`// @API EPS POST /v1.0/enterprise-projects/{enterprise_project_id}/resources-migrat`

```go
	if d.HasChange("enterprise_project_id") {
		migrateOpts := config.MigrateResourceOpts {
			ResourceId:   instanceID,
			ResourceType: "rds",
			RegionId:     region,
			ProjectId:    client.ProjectID,
		}
		if err = cfg.MigrateEnterpriseProject(ctx, d, migrateOpts); err != nil {
			return diag.FromErr(err)
		}
	}
```

#### 4.2.4 更新tags

- 如果`创建资源API`参数中包含 `tags`，那么就添加`更新tags`
- 使用`utils.UpdateResourceTags`方法去更新`tags`，如果返回`err`不为空，则返回错误信息
- 最后在资源函数前边加一个注释（示例中的`使用到的API`位置）：`// @API RDS POST /v3/{project_id}/instances/{id}/tags/action`

```go
	if d.HasChange("tags") {
		if err = utils.UpdateResourceTags(client, d, "instances", instanceID); err != nil {
			return diag.Errorf("error updating tags of RDS instance (%s): %s", instanceID, err)
		}
	}
```

#### 4.2.5 更新auto_renew

- 如果`创建资源API`参数中包含 `auto_renew`，那么就添加`更新auto_renew`
- 使用`common.UpdateAutoRenew`方法去更新`auto_renew`，如果返回`err`不为空，则返回错误信息
- 最后在资源函数前边加两个注释（示例中的`使用到的API`位置）：`// @API BSS POST /v2/orders/subscriptions/resources/autorenew/{instance_id}` 和 `// @API BSS DELETE /v2/orders/subscriptions/resources/autorenew/{instance_id}`

```go
	if d.HasChange("auto_renew") {
		if err = common.UpdateAutoRenew(bssClient, d.Get("auto_renew").(string), instanceID); err != nil {
			return diag.Errorf("error updating the auto-renew of the instance (%s): %s", instanceID, err)
		}
	}
```

#### 4.2.6 生成更新参数函数

##### 4.2.6.1 获取更新resource参数的数据

- 调用用户提供的更新资源的API，从结果中获取请求参数
- 禁止使用 `web reference` 的内容，使用 `WebFetch` 获取完整的页面内容
- 最后在资源函数前边加一个注释（示例中的`使用到的API`位置），格式为`// @API {service} {method} {path}`，其中`{service}`为当前服务名，`{method}`为URI中的请求方法，`{path}`为URI中的请求path

##### 4.2.6.2 生成更新参数函数体

- 函数名为`update{XXX}{YYY}`，其中 `{XXX}` 为该资源的功能信息，`{YYY}` 为参数名称，函数名最终采用驼峰格式
- 函数参数首先需要包含`d`和`client`
- 如果API响应消息中包含`job_id`等任务信息，那么函数参数需要包含`ctx`
- 如果API请求消息中包含包周期相关参数，例如`自动支付（is_auto_pay）`，或者API响应消息中包含`order_id`，那么函数参数需要包含`bssClient`
- 将函数放到更新resource的函数的后边

```go
func updateRdsInstanceFlavor(ctx context.Context, d *schema.ResourceData, client, bssClient *golangsdk.ServiceClient) diag.Diagnostics {
    // 构造请求参数和请求体

	// 发送请求
	
	// 解析结果
	
	// 等待任务或订单完成
}
```

##### 4.2.6.3 构造请求参数和请求体

- 将path参数替换为具体的值，禁止使用`utils.ReplacePathVariables`，使用`strings.ReplaceAll`
- 首先需要构造请求体，函数名为 `buildUpdate{XXX}{YYY}BodyParams`， 其中 `{XXX}` 为该资源的功能信息，`{YYY}` 为参数名称，函数名最终采用驼峰格式

```go
    httpUrl := "v3/{project_id}/instances/{instance_id}/name" 
    updatePath := client.Endpoint + httpUrl
	updatePath = strings.ReplaceAll(updatePath, "{project_id}", client.ProjectID)
	updatePath = strings.ReplaceAll(updatePath, "{instance_id}", fmt.Sprintf("%v", d.Id()))

	updateOpt := golangsdk.RequestOpts{
		KeepResponseBody: true,
	}
	updateOpt.JSONBody = buildUpdateInstanceNameBodyParams(d)
```
##### 4.2.6.4 发送请求、解析结果

###### 4.2.6.4.1 如果API响应消息为空

- 那么就不需要处理返回结果

```go
    _, err = client.Request("POST", getPath, &getOpt)
	if err != nil {
		return diag.Errorf("error updating RDS instance name: %s", err)
	}
```

##### 4.2.6.4.2 如果API响应消息不为空

- 获取更新结果，然后使用`utils.FlattenResponse`去解析

```go
    updateResp, err := client.Request("POST", getPath, &getOpt)
	if err != nil {
		return diag.Errorf("error updating RDS instance name: %s", err)
	}

    updateRespBody, err := utils.FlattenResponse(updateResp)
	if err != nil {
		return diag.FromErr(err)
	}
```

##### 4.2.6.5 等待任务或订单完成

###### 4.2.6.5.1 添加等待时间

- 在资源函数中添加更新资源等待时间（示例中的`超时时间`位置）
- 如果已经存在，则不需要再次添加

```go
    Timeouts: &schema.ResourceTimeout{
		Update:  schema.DefaultTimeout(30 * time.Minute),
		// ....
	},
```

###### 4.2.6.5.2 添加等待订单完成逻辑

- 从API响应消息中获取订单ID信息，如果存在，那么就等待订单的完成

```go
    orderId := utils.PathSearch("order_id", createRespBody, "").(string)
	if orderId != "" {
		err = common.WaitOrderComplete(ctx, bssClient, orderId, d.Timeout(schema.TimeoutUpdate))
		if err != nil {
			return diag.FromErr(err)
		}
	}
```

###### 4.2.6.5.3 添加等待任务完成逻辑

- 从API响应消息中获取任务ID信息，如果存在，那么就等待任务的完成
- 函数名为 `check{XXX}JobFinish`， 其中 `{XXX}` 为该资源的功能信息


```go
    jobId := utils.PathSearch("job_id", createRespBody, "").(string)
	if jobId != "" {
		if err = checkInstanceJobFinish(client, jobId, d.Timeout(schema.TimeoutUpdate)); err != nil {
			return diag.Errorf("error updating publication: %s", err)
		}
	}
```

##### 4.2.6.6 生成函数请求体

- 首先需要构造请求体，函数名为 `buildUpdate{XXX}{YYY}BodyParams`， 其中 `{XXX}` 为该资源的功能信息，`{YYY}` 为参数名称，函数名最终采用驼峰格式
- 将函数放到最后边

```go
func updateRdsInstanceNameBodyParams(d *schema.ResourceData) map[string]interface{} {
	bodyParams := map[string]interface{}{
		"name": d.Get("name"),
	}
}
```

### 4.3 添加不支持更新的参数列表

- 如果创建API的参数不被更新API支持更新，那么该参数不支持更新，那么就在resource函数中添加不支持修改的参数列表（示例中的`不支持修改的参数列表`位置），类型为字符串数组
- 如果参数类型为list，那么就需要使用星号（`*`）来分开层级

```go
var rdsInstanceNonUpdatableParams = []string{
	"name",
	"images",
	"building_config.*.cluster", "building_config.*.image_pull_secrets",
}
```

### 4.4 添加 CustomizeDiff 信息

- 如果4.3资源函数中添加了不支持修改的参数列表，那么就在资源函数中添加CustomizeDiff 信息（示例中的`CustomizeDiff 信息`位置）

```go
    CustomizeDiff: config.FlexibleForceNew(rdsInstanceNonUpdatableParams),
```

## 5. 生成删除resource的函数

### 5.1 如果没有提供删除API

#### 5.1.1 如果没有包周期参数

- 生成一个返回warn信息的函数，格式为`resource{XXX}Delete`，其中 `{XXX}` 为该资源的功能信息，例如：`resourcePublicationDelete`

```go
func resourcePublicationDelete(_ context.Context, _ *schema.ResourceData, _ interface{}) diag.Diagnostics {
	errorMsg := "Deleting RDS publication resource is not supported. The resource is only removed from the state."
	return diag.Diagnostics{
		diag.Diagnostic{
			Severity: diag.Warning,
			Summary:  errorMsg,
		},
	}
}
```

#### 5.1.2 如果有包周期参数

- 生成一个删除资源的函数，格式为`resource{XXX}Delete`，其中 `{XXX}` 为该资源的功能信息，例如：`resourceInstanceDelete`
- 该函数总共包含：函数名称、参数定义、创建client、退订资源


```go
func resourceInstanceDelete(_ context.Context, d *schema.ResourceData, meta interface{}) diag.Diagnostics {
	// 参数定义

	// 创建client
	
	// 退订资源
	
	return nil
}
```

##### 5.1.2.1 参数定义，创建client

- 创建退订资源的client，变量名为`bssClient`

```go
	cfg := meta.(*config.Config)
	region := cfg.GetRegion(d)

	bssClient, err := cfg.BssV2Client(region)
	if err != nil {
		return diag.Errorf("error creating bss V2 client: %s", err)
	}
```

##### 5.1.2.2 退订资源

- 判断 `charging_mode` 参数是否设置，如果设置了就判断是否为`prePaid`，如果是就使用`common.UnsubscribePrePaidResource`去退订资源

```go
    if v, ok := d.GetOk("charging_mode"); ok && v.(string) == "prePaid" {
		if err = common.UnsubscribePrePaidResource(d, cfg, []string{d.Id()}); err != nil {
			return diag.Errorf("error unsubscribe RDS instance: %s", err)
		}
	}
```

### 5.2 如果有提供删除API

#### 5.2.1 如果没有包周期参数

##### 5.2.1.1 获取删除资源的数据

- 调用用户提供的更新资源的API，从结果中获取请求参数
- 禁止使用 `web reference` 的内容，使用 `WebFetch` 获取完整的页面内容
- 最后在资源函数前边加一个注释（示例中的`使用到的API`位置），格式为`// @API {service} {method} {path}`，其中`{service}`为当前服务名，`{method}`为URI中的请求方法，`{path}`为URI中的请求path

##### 5.2.1.2 生成删除资源函数

- 生成一个删除资源的函数，格式为`resource{XXX}Delete`，其中 `{XXX}` 为该资源的功能信息，例如：`resourceInstanceDelete`

```go
func resourcePublicationDelete(_ context.Context, _ *schema.ResourceData, _ interface{}) diag.Diagnostics {
	// 参数定义

	// 创建client

    // 构造请求参数和请求体
	
	// 发送请求
	
	// 解析结果
	
	// 等待任务
}
```

##### 5.2.1.3 参数定义，创建client

- 创建client时，其中第一个参数为服务类型，错误信息中的服务类型需要大写

```go
	cfg := meta.(*config.Config)
	region := cfg.GetRegion(d)

    var (
		httpUrl = "v3/{project_id}/instances/{instance_id}/replication/publications"
		product = "rds"
	)

	client, err := cfg.NewServiceClient(product, region)
	if err != nil {
		return diag.Errorf("error creating RDS client: %s", err)
	}
```

##### 5.2.1.4 构造请求参数和请求体

- 将path参数替换为具体的值，禁止使用`utils.ReplacePathVariables`，使用`strings.ReplaceAll`
- 首先需要构造请求体，函数名为 `buildDelete{XXX}BodyParams`， 其中 `{XXX}` 为该资源的功能信息

```go
    deletePath := client.Endpoint + httpUrl
    deletePath = strings.ReplaceAll(deletePath, "{project_id}", client.ProjectID)
    deletePath = strings.ReplaceAll(deletePath, "{instance_id}", d.Id())
	
	deleteOpt := golangsdk.RequestOpts{
		KeepResponseBody: true,
		MoreHeaders: map[string]string{"Content-Type": "application/json"},
	}
	deleteOpt.JSONBody = utils.RemoveNil(buildDeleteInstanceBodyParams(d))
```

##### 5.2.1.5 发送请求、解析结果

###### 5.2.1.5.1 如果API响应消息为空

- 那么就不需要处理返回结果

```go
    _, err = client.Request("DELETE", deletePath, &deleteOpt)
	if err != nil {
		return diag.Errorf("error deleting RDS instance: %s", err)
	}
```

###### 5.2.1.5.2 如果API响应消息不为空

- 获取删除结果，然后使用`utils.FlattenResponse`去解析

```go
    deleteResp, err := client.Request("DELETE", deletePath, &deleteOpt)
	if err != nil {
		return diag.Errorf("error deleting RDS instance: %s", err)
	}

    deleteRespBody, err := utils.FlattenResponse(deleteResp)
	if err != nil {
		return diag.FromErr(err)
	}
```

##### 5.2.1.6 等待任务

- 从API响应消息中获取任务ID信息，如果存在，那么就等待任务的完成
- 函数名为 `check{XXX}JobFinish`， 其中 `{XXX}` 为该资源的功能信息


```go
    jobId := utils.PathSearch("job_id", deleteRespBody, "").(string)
	if jobId != "" {
		if err = checkInstanceJobFinish(client, jobId, d.Timeout(schema.TimeoutUpdate)); err != nil {
			return diag.Errorf("error deleting instance: %s", err)
		}
	}
```

#### 5.2.2 如果有包周期参数

##### 5.2.2.1 获取删除资源的数据

- 调用用户提供的更新资源的API，从结果中获取请求参数
- 禁止使用 `web reference` 的内容，使用 `WebFetch` 获取完整的页面内容
- 最后在资源函数前边加一个注释（示例中的`使用到的API`位置），格式为`// @API {service} {method} {path}`，其中`{service}`为当前服务名，`{method}`为URI中的请求方法，`{path}`为URI中的请求path

##### 5.2.2.2 生成删除资源函数

- 生成一个删除资源的函数，格式为`resource{XXX}Delete`，其中 `{XXX}` 为该资源的功能信息，例如：`resourceInstanceDelete`

```go
func resourcePublicationDelete(_ context.Context, _ *schema.ResourceData, _ interface{}) diag.Diagnostics {
	// 参数定义

	// 创建client

    // 退订资源/删除资源
	
	return nil
}
```

##### 5.2.2.3 参数定义，创建client

- 创建client时，其中第一个参数为服务类型，错误信息中的服务类型需要大写

```go
	cfg := meta.(*config.Config)
	region := cfg.GetRegion(d)

	var (
		product = "rds"
	)

	client, err := cfg.NewServiceClient("rds", region)
	if err != nil {
		return diag.Errorf("error creating RDS client: %s", err)
	}
	bssClient, err := cfg.BssV2Client(region)
	if err != nil {
		return diag.Errorf("error creating bss V2 client: %s", err)
	}
```

##### 5.2.2.4 退订资源/删除资源

- 判断 `charging_mode` 参数是否设置，如果设置了就判断是否为`prePaid`，如果是就使用`common.UnsubscribePrePaidResource`去退订资源
- 否则就执行删除资源函数，函数名为`delete{XXX}`，其中 `{XXX}` 为该资源的功能信息

```go
    if v, ok := d.GetOk("charging_mode"); ok && v.(string) == "prePaid" {
		if err = common.UnsubscribePrePaidResource(d, cfg, []string{d.Id()}); err != nil {
			return diag.Errorf("error unsubscribe RDS instance: %s", err)
		}
	} else {
		err = deleteRdsInstance(ctx, d, client)
		if err != nil {
			return diag.FromErr(err)
		}
	}
```

##### 5.2.2.5 生成删除资源函数

###### 5.2.2.5.1 生成删除资源函数

- 生成一个删除资源的函数，格式为`resource{XXX}Delete`，其中 `{XXX}` 为该资源的功能信息，例如：`resourceInstanceDelete`

```go
func resourcePublicationDelete(_ context.Context, _ *schema.ResourceData, _ interface{}) diag.Diagnostics {
    // 构造请求参数和请求体
	
	// 发送请求
	
	// 解析结果
	
	// 等待任务
}
```

###### 5.2.2.5.2 构造请求参数和请求体

- 将path参数替换为具体的值
- 首先需要构造请求体，函数名为 `buildDelete{XXX}BodyParams`， 其中 `{XXX}` 为该资源的功能信息

```go
    httpUrl := "v3/{project_id}/instances/{instance_id}"
    deletePath := client.Endpoint + httpUrl
    deletePath = strings.ReplaceAll(deletePath, "{project_id}", client.ProjectID)
    deletePath = strings.ReplaceAll(deletePath, "{instance_id}", d.Id())
	
	deleteOpt := golangsdk.RequestOpts{
		KeepResponseBody: true,
		MoreHeaders: map[string]string{"Content-Type": "application/json"},
	}
	deleteOpt.JSONBody = utils.RemoveNil(buildDeleteInstanceBodyParams(d))
```

###### 5.2.2.5.3  发送请求、解析结果

####### 5.2.2.5.3.1 如果API响应消息为空

- 那么就不需要处理返回结果

```go
    _, err = client.Request("POST", deletePath, &deleteOpt)
	if err != nil {
		return diag.Errorf("error deleting RDS instance: %s", err)
	}
```

####### 5.2.2.5.3.2 如果API响应消息不为空

- 获取删除结果，然后使用`utils.FlattenResponse`去解析

```go
    deleteResp, err := client.Request("DELETE", deletePath, &deleteOpt)
	if err != nil {
		return diag.Errorf("error deleting RDS instance: %s", err)
	}

    deleteRespBody, err := utils.FlattenResponse(deleteResp)
	if err != nil {
		return diag.FromErr(err)
	}
```

###### 5.2.2.5.4 等待任务

- 从API响应消息中获取任务ID信息，如果存在，那么就等待任务的完成
- 函数名为 `check{XXX}JobFinish`， 其中 `{XXX}` 为该资源的功能信息


```go
    jobId := utils.PathSearch("job_id", deleteRespBody, "").(string)
	if jobId != "" {
		if err = checkInstanceJobFinish(client, jobId, d.Timeout(schema.TimeoutUpdate)); err != nil {
			return diag.Errorf("error deleting instance: %s", err)
		}
	}
```

## 6. 导入信息

// todo

## 7. 调整包的顺序

- 按照go标准库包、github.com/hashicorp、github.com/chnsz、github.com/huaweicloud的顺序，并且中间用空行隔开

```go
import (
	"context"
	"encoding/json"
	"fmt"
	"strings"

	"github.com/hashicorp/go-multierror"
	"github.com/hashicorp/go-uuid"
	"github.com/hashicorp/terraform-plugin-sdk/v2/diag"
	"github.com/hashicorp/terraform-plugin-sdk/v2/helper/schema"

	"github.com/chnsz/golangsdk/pagination"

	"github.com/huaweicloud/terraform-provider-huaweicloud/huaweicloud/config"
	"github.com/huaweicloud/terraform-provider-huaweicloud/huaweicloud/utils"
)
```

## 8. 文件的包名为服务名的小写，不是`package huaweicloud`，例如`package rds`