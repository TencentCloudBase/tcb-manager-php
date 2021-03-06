
FunctionManager 实例可以对云函数进行管理，包括创建、删除、更新、调用等云函数管理功能。

获得当前环境下的 FunctionManager 实例，示例代码如下：

```php
$funcManager = $tcbManager->getFunctionManager();
```

#### 目录

* [获取云函数列表](#获取云函数列表)
* [创建函数](#创建函数)
* [更新云函数代码](#更新云函数代码)
* [更新云函数配置](#更新云函数配置)
* [删除云函数](#删除云函数)
* [获取云函数详情](#获取云函数详情)
* [调用云函数](#调用云函数)
* [获取云函数调用日志](#获取云函数调用日志)

### 获取云函数列表

#### 接口定义

```php
listFunctions()
```

#### 参数说明

参数名                     |  类型  | 描述
------------------------- | ------ | -----------
$functionName             | String | 函数名称

**调用示例**

```php
$funcManager->listFunctions();
```

**返回示例**

```json
{
    "Functions": [
        {
            "FunctionId": "lam-xxxxxxx",
            "Namespace": "default",
            "FunctionName": "test",
            "ModTime": "2018-04-08 19:02:20",
            "AddTime": "2018-04-08 15:18:49",
            "Runtime": "Python2.7"            
        }
    ],
    "TotalCount": 1,
    "RequestID": "3c140219-cfe9-470e-b241-907877d6fb03"
}
```

**返回字段描述**

参数名                    |  类型  | 描述
------------------------ | ------ | -----------
RequestID                | String | 请求唯一标识
TotalCount               | Number | 总数
Functions                | Array  | 函数列表
Functions[].FunctionId   | String | 函数 ID
Functions[].FunctionName | String | 函数名称
Functions[].Namespace    | String | 命名空间
Functions[].Runtime      | String | 运行时间
Functions[].AddTime      | String | 创建时间
Functions[].ModTime      | String | 修改时间

### 创建函数

#### 接口定义

```php
createFunction(string $functionName, array $code, string $handler, string $runtime, array $options = [])
```

#### 参数说明

参数名                     |  类型  | 描述
------------------------- | ------ | -----------
$functionName             | String | 函数名称
$code                     | Array  | 源码资源，压缩包限制50M，以下参数必选一种方式上传源码文件
⁃ $ZipFile                | String | 包含函数代码文件及其依赖项的 zip 格式文件 经过 base64 编码后的字符串
⁃ $ZipFilePath            | String | 包含函数代码文件及其依赖项的 zip 格式文件路径
⁃ $SourceFilePath         | String | 源码文件路径
$handler                  | String | 函数调用入口，指明调用云函数时需要从哪个文件中的哪个函数开始执行。
$runtime                  | String | 函数运行时，目前支持 `Php7` 和 `Nodejs8.9`，请注意运行时与函数源文件对应，否则无法执行。
$options                  | Array  | 可选参数
⁃ Description             | String | 函数描述
⁃ Timeout                 | Number | 函数超时时间
⁃ MemorySize              | Number | 函数运行时内存大小，单位 MB，默认为 256，可选值 256 | 512
⁃ Environment             | Array  | 函数运行环境，见调用示例
⁃⁃ Variables              | String | 环境变量，在函数运行时可在环境变量里获取到相应的值，PHP 中获取环境变量函数为 getenv
⁃⁃⁃ Key                   | String | 环境变量名，注意：避免使用系统常用的环境变量名，建议统一前缀且大写，例如：ENV_PROJECTNAME_[KeyName]
⁃⁃⁃ Value                 | String | 环境变量值

> !handler 指定的入口文件需要和$code文件中真正的函数入口相对应，否则云函数无法成功部署。因入口文件只能在根目录中，所以自行压缩的 Zip包 需要注意入口文件要在压缩包的根路径

**概念**

- Runtime - 运行时
    PHP 运行时目前可填写 `Php7`，注意大小写
- Handler - 云函数入口
    执行方法表明了调用云函数时需要从哪个文件中的哪个函数开始执行。
    注意：入口文件只能在根目录中。通常写为 `index.main_handler`，指向的是 `index.[ext]` 文件内的 `main_handler` 函数方法。

    格式：不同语言的云函数格式是不同的，所以有不同的写法，但是目标都是要能够指明云函数的入口。
     - 一段式格式为 "[文件名]"，`Golang` 环境时使用，例如 "main";
     - 两段式格式为 "[文件名].[函数名]"，`Python，Node.js，PHP` 环境时使用，例如 "index.main_handler";
     - 三段式格式为 "[package].[class]::[method]"，`Java` 环境时使用，例如 "example.Hello::mainHandler";

    TCB 中目前支持 PHP 和 Node.js 的运行时，所以 handle 采用 两段式的写法，如果后续支持更多运行时，则可根据具体情况采用正确的写法。
    两段式的执行方法，前一段指向代码包中不包含后缀的文件名，后一段指向文件中的入口函数名。
    需要确保代码包中的文件名后缀与语言环境匹配，如 Python 环境为 .py 文件，Node.js 环境为 .js 文件。

>!请在测试时在 TCB 控制台确认函数创建并部署成功，有可能创建成功，`createFunction` 成功返回，但是部署失败，部署失败的原因通常为 `$handler` 参数与源码包不对应。


**Zip 压缩包文件示例**

入口文件为：`index.js`，必须在压缩包根目录。

**代码文件路径示例**

```sh
.
├── README.md
├── index.js
└── src
    └── index.js

1 directory, 3 files
```

>!该步骤是在源码根目录（不是上级目录）执行压缩。

**压缩 zip 文件**

```sh
zip -r code.zip .
  adding: README.md (stored 0%)
  adding: index.js (deflated 14%)
  adding: src/ (stored 0%)
  adding: src/index.js (stored 0%)
```

**查看 zip 包**

```sh
➜ unzip -l code.zip
Archive:  code.zip
  Length      Date    Time    Name
---------  ---------- -----   ----
        8  05-20-2019 16:19   README.md
      122  06-10-2019 21:06   index.js
        0  05-20-2019 16:19   src/
        0  05-20-2019 16:19   src/index.js
---------                     -------
      130                     4 files
```

**调用示例**

```php
$funcManager->createFunction(
    "functionName",
    [
       // 根据实际需要选择以下某种方式
       "ZipFile" => "base64 zip file content"
       // "ZipFilePath" => "path/to/zipFile"
       // "SourceFilePath" => "path/to/source-code"
    ],
    "index.main",
    "Php7",
    [
       "Description" => "this is function description",
       "Environment" => [
           "Variables" => [
               ["Key" => "ENV_PROJNAME_VERSION", "Value" => "v1.3.5"],
               ["Key" => "ENV_PROJNAME_ENDPOINT", "Value" => "api.your-domain.com"]
               ["Key" => "ENV_PROJNAME_ES_HOST", "Value" => "es-cluster.your-domain.com"]
           ]
       ]
    ]
);
```

**返回示例**

```json
{
    "RequestId": "eac6b301-a322-493a-8e36-83b295459397"
}
```

**返回字段描述**

参数名                    |  类型  | 描述
------------------------ | ------ | -----------
RequestID                | String | 请求唯一标识

以 JSON 对象描述，在 PHP 中为对应的数组结构，其他函数返回格式相同。


### 更新云函数代码

#### 接口定义

 ```php
 updateFunctionCode(string $functionName, string $code, string $handler, array $options = [])
 ```

#### 参数说明

参数名                     |  类型  | 描述
------------------------- | ------ | -----------
$functionName             | String | 函数名称
$code                     | Array  | 源码资源，压缩包限制50M，以下参数必选一种方式上传源码文件
⁃ $ZipFile                | String | 包含函数代码文件及其依赖项的 zip 格式文件 经过 base64 编码后的字符串
⁃ $ZipFilePath            | String | 包含函数代码文件及其依赖项的 zip 格式文件路径
⁃ $SourceFilePath         | String | 源码文件路径
$handler                  | String | 函数调用入口，同 createFunction
$options                  | Array  | 可选参数，同 createFunction

**调用示例**

```php
$funcManager->updateFunctionCode(
    "functionName",
    [
       // 根据实际需要选择以下某种方式
       "ZipFile" => "base64 zip file content"
       // "ZipFilePath" => "path/to/zipFile"
       // "SourceFilePath" => "path/to/source-code"
    ],
    "index.main",
    "Nodejs8.9"
);
```

**返回示例**

```json
{
    "RequestId": "eac6b301-a322-493a-8e36-83b295459397"
}
```

**返回字段描述**

参数名                    |  类型  | 描述
------------------------ | ------ | -----------
RequestID                | String | 请求唯一标识

### 更新云函数配置

#### 接口定义

```php
updateFunctionConfiguration(string $functionName, array $options = [])
```

#### 参数说明

参数名                     |  类型  | 描述
------------------------- | ------ | -----------
$functionName             | String | 函数名称
$options                  | Array  | 可选参数，同 createFunction

**调用示例**

```php
$funcManager->updateFunctionConfiguration(
    "functionName",
    [
        "Description" => "this is new description.",
        "Timeout" => 10,
        "Environment" => [
            "Variables" => [
               ["Key" => "ENV_PROJNAME_VERSION", "Value" => "v1.3.5"],
               ["Key" => "ENV_PROJNAME_ENDPOINT", "Value" => "api.your-domain.com"]
               ["Key" => "ENV_PROJNAME_ES_HOST", "Value" => "es-cluster.your-domain.com"]
            ]
        ]
    ]
);
```

**返回示例**

```json
{
    "RequestId": "eac6b301-a322-493a-8e36-83b295459397"
}
```

**返回字段描述**

参数名                    |  类型  | 描述
------------------------ | ------ | -----------
RequestID                | String | 请求唯一标识

### 删除云函数

#### 接口定义

```php
deleteFunction(string $functionName)
```

#### 参数说明

参数名                     |  类型  | 描述
------------------------- | ------ | -----------
$functionName             | String | 函数名称

**调用示例**

```php
$funcManager->deleteFunction("functionName");
```

**返回示例**

```json
{
    "RequestId": "eac6b301-a322-493a-8e36-83b295459397"
}
```
**返回字段描述**

参数名                    |  类型  | 描述
------------------------ | ------ | -----------
RequestID                | String | 请求唯一标识

### 获取云函数详情

#### 接口定义

```php
getFunction(string $functionName)
```

#### 参数说明

参数名                     |  类型  | 描述
------------------------- | ------ | -----------
$functionName             | String | 函数名称

**调用示例**

```php
$funcManager->getFunction("functionName");
```

**返回示例**

```json
{
    "RequestId": "a1ffbba5-5489-45bc-89c5-453e50d5386e",
    "FunctionName": "ledDummyAPITest",
    "FunctionVersion": "$LATEST",
    "Namespace": "default",
    "Runtime": "Python2.7",
    "Handler": "scfredis.main_handler",
    "Description": "",
    "ModTime": "2018-06-07 09:52:23",
    "Environment": {
        "Variables": []
    },
    "VpcConfig": {
        "SubnetId": "",
        "VpcId": ""
    },
    "Triggers": [],
    "ErrNo": 0,
    "UseGpu": "FALSE",
    "MemorySize": 128,
    "Timeout": 3,
    "CodeSize": 0,
    "CodeResult": "failed",
    "CodeInfo": "",
    "CodeError": "",
    "Role": ""
}
```

**返回字段描述**

参数名                         |  类型  | 描述
----------------------------- | ------ | --------------
RequestId                     | String | 请求唯一标识
FunctionName                  | String | 函数名称
Namespace                     | String | 命名空间
Runtime                       | String | 运行时
Handler                       | String | 函数入口
Description                   | String | 函数的描述信息
ModTime                       | String | 函数修改时间
Environment                   | Object | 函数的环境变量
Environment.Variables         | Array  | 环境变量数组
Environment.Variables[].Key   | String | 变量的 Key
Environment.Variables[].Value | String | 变量的 Value
MemorySize                    | Number | 函数的最大可用内存
Timeout                       | Number | 函数的超时时间

### 调用云函数

#### 接口定义

```php
invoke(string $functionName, array $options = [])
```

#### 参数说明

参数名                     |  类型  | 描述
------------------------- | ------ | -----------
$functionName             | String | 函数名称
$options                  | Array  | 可选参数
⁃ InvocationType          | String | `RequestResponse` (同步) 和 `Event` (异步)，默认为同步
⁃ ClientContext           | String | 运行函数时的参数，以 `JSONString` 格式传入，最大支持的参数长度是 `1M`
⁃ LogType                 | String | 同步调用时指定该字段，返回值会包含4K的日志，可选值为 None 和 Tail，默认值为 None。 当该值为 Tail 时，返回参数中的 logMsg 字段会包含对应的函数执行日志

**调用示例**

```php
$jsonString = "{\"userInfo\":{\"appId\":\"\",\"openId\":\"oaoLb4qz0R8STBj6ipGlHkfNCO2Q\"}}";
$funcManager->invoke("functionName", [
        "InvocationType" => "RequestResponse",
        "ClientContext" => json_encode($jsonString),
        "LogType" => "Tail"
    ]);
```

**返回示例**

```json
{
    "Result": {
        "MemUsage": 3207168,
        "Log": "",
        "RetMsg": "hello from scf",
        "BillDuration": 100,
        "FunctionRequestId": "6add56fa-58f1-11e8-89a9-5254005d5fdb",
        "Duration": 0.826,
        "ErrMsg": "",
        "InvokeResult": 0
    },
    "RequestId": "c2af8a64-c922-4d55-aee0-bd86a5c2cd12"
}
```

**返回字段描述**

参数名                    |  类型  | 描述
------------------------ | ------ | --------------------------------------------------
RequestId                | String | 请求唯一标识
Result                   | Object | 运行函数的返回
Result.FunctionRequestId | String | 此次函数执行的 ID
Result.Duration          | Number | 表示执行函数的耗时，单位是毫秒，异步调用返回为空
Result.BillDuration      | Number | 表示函数的计费耗时，单位是毫秒，异步调用返回为空
Result.MemUsage          | Number | 执行函数时的内存大小，单位为 Byte，异步调用返回为空
Result.InvokeResult      | Number | 0为正确，异步调用返回为空
Result.RetMsg            | String | 表示执行函数的返回，异步调用返回为空
Result.ErrMsg            | String | 表示执行函数的错误返回信息，异步调用返回为空
Result.Log               | String | 表示执行过程中的日志输出，异步调用返回为空

### 获取云函数调用日志

#### 接口定义

```php
getFunctionLogs(string $functionName, array $options = [])
```

#### 参数说明

参数名                     |  类型  | 描述
------------------------- | ------ | -----------
$functionName             | String | 函数名称
$options                  | Array  | 可选参数
⁃ FunctionRequestId       | String | 执行该函数对应的 requestId
⁃ Offset                  | Number | 数据的偏移量，Offset+Limit 不能大于10000
⁃ Limit                   | Number | 返回数据的长度，Offset+Limit 不能大于10000
⁃ Order                   | String | 以升序还是降序的方式对日志进行排序，可选值 desc 和 asc
⁃ OrderBy                 | String | 根据某个字段排序日志,支持以下字段：function_name, duration, mem_usage, start_time
⁃ StartTime               | String | 查询的具体日期，例如：2017 - 05 - 16 20:00:00，只能与 EndTime 相差一天之内
⁃ EndTime                 | String | 查询的具体日期，例如：2017 - 05 - 16 20:59:59，只能与 StartTime 相差一天之内

**调用示例**

```php
$funcManager->getFunctionLogs("functionName", [
    "Offset" => 0,
    "Limit" => 3
]);
```

**返回示例**

```json
{
    "TotalCount": 1,
    "Data": [
        {
            "MemUsage": 3174400,
            "RetCode": 1,
            "RetMsg": "Success",
            "Log": "",
            "BillDuration": 100,
            "InvokeFinished": 1,
            "RequestId": "bc309eaa-6d64-11e8-a7fe-5254000b4175",
            "StartTime": "2018-06-11 18:46:45",
            "Duration": 0.532,
            "FunctionName": "APITest"
        }
    ],
    "RequestId": "e2571ff3-da04-4c53-8438-f58bf057ce4a"
}
```

**返回字段描述**

参数名                    |  类型  | 描述
------------------------ | ------ | --------------------------------------------------
RequestId                | String | 请求唯一标识
TotalCount               | String | 函数日志的总数
Data[]                   | Array  | 运行函数的返回
Data[].RequestId         | String | 执行该函数对应的 requestId
Data[].FunctionName      | String | 函数的名称
Data[].RetCode           | Number | 函数执行结果，如果是0表示执行成功，其他值表示失败
Data[].InvokeFinished    | Number | 函数调用是否结束，如果是1表示执行结束，其他值表示调用异常
Data[].StartTime         | String | 函数开始执行时的时间点
Data[].Duration          | Number | 表示执行函数的耗时，单位是毫秒，异步调用返回为空
Data[].BillDuration      | Number | 表示函数的计费耗时，单位是毫秒，异步调用返回为空
Data[].MemUsage          | Number | 执行函数时的内存大小，单位为 Byte，异步调用返回为空
Data[].RetMsg            | String | 表示执行函数的返回，异步调用返回为空
Data[].Log               | String | 表示执行过程中的日志输出，异步调用返回为空
