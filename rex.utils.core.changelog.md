## 6.6.0/6.5.0/6.4.0 (2022.04.24)
- 调整版本号
    - `6.6.x` 支持 `.NET 6`
    - `6.5.x` 支持 `.NET 5`，2022年05月08日，与微软同步停更，仅保持维护BUG
    - `6.4.x` 支持 `.NET Framework 4.7`，已停更，仅保持维护BUG
- 调整注释说明、优化代码
- 增加：`GetPinYinForAll`, `GetPinYinForName`, `GetFirstPinYinForName`, `GetFirstPinYinForAll`
- 增加：`Guid` 类型增加 `.IsNull()` 扩展方法，用于判断是否为 `null` 或 `Guid.Empty`
- 增加：`.IsZero()` 扩展方法，用于判断数值类型是否为 `null` 或 `0`
- 优化：`UtilsBasis.GetUnixTimestamp`，通过 `ConcurrentDictionary` 记录已生成时间戳，确保不重复



## 6.0.5 (2022.03.14)
- 调整：UtilsBasis.GetUnixTimestamp 去掉毫秒限制
- 增加：UtilsBasis.SnowflakeId 方便调用
- 调整注释说明、优化代码
- 创建 `Rex.Extensions.Models` 实体扩展类，`Rex.Utils.Core` 移除所有`ResultData`相关对象；
    - `ResultData` 输出JSON结果: `{ "code": 1000, "msg": "string", "result": { } }`
    - `ResultDataModel` 输出JSON结果: `{ "code": 1000, "msg": "string", "data": { } }`
    - `ResultStatusModel` 输出JSON结果: `{ "status": 1000, "msg": "string", "data": { } }`


## 6.0.4 (2022.01.11)
- 命名空间 `System.Text.Json.Extensions.DateTimeConverterExtensions` 改为 `System.Text.Json.ConverterExtensions.DateTimeConverterExtensions`
- 扩展 `System.Text.Json.Serialization.JsonConverter<T>`，可指定 number 类型，在 `JSON` 中以字符串方式输出。
  - `Int16ToStringJsonConverter` `Int16NullableToStringJsonConverter`
  - `Int32ToStringJsonConverter` `Int32NullableToStringJsonConverter`
  - `Int64ToStringJsonConverter` `Int64NullableToStringJsonConverter`
  - `DoubleToStringJsonConverter` `DoubleNullableToStringJsonConverter`
  - `DecimalToStringJsonConverter` `DecimalNullableToStringJsonConverter`
  - `SingleToStringJsonConverter` `SingleNullableToStringJsonConverter`
- `System.Text.Json` 在依赖注入时可提供自定义转换器，若提供时则完全采用，不提供则自动采用系统内置的 `DateTimeConverterExtensions`、`DateTimeNullableConverterExtensions ` 转换器。


**DEMO**
```csharp
public class Demo {
    public long BigNumber { get; set; }

    [System.Text.Json.Serialization.JsonConverter(typeof(System.Text.Json.ConverterExtensions.Int64ToStringJsonConverter))]
    public long BigNumberS { get; set; }

    [System.Text.Json.Serialization.JsonConverter(typeof(System.Text.Json.ConverterExtensions.Int64NullableToStringJsonConverter))]
    public long? BigNumberN { get; set; }
}


{
    "big_number": 268550809652445184,
    "big_number_s": "268550809652445185",
    "big_number_n": "268550809652445186"
}
```


## 6.0.3 (2022.01.06)
- 将 `string` 返回值为 `null` 的方法改为 `string.Empty`
- 增加 雪花（snowflake）算法：`Snowflake.Instance.NextId()`
- Linq 新增 `.SelectAsync()`

```csharp
var inputs = await events.SelectAsync(async ev => await ProcessEventAsync(ev));
```

```csharp
var query = await System.IO.Directory.GetFiles(@"C:\Windows", "*.xml")
    .SelectAsync(async path => {
        using (var fs = new System.IO.FileStream(path, System.IO.FileMode.Open, System.IO.FileAccess.Read)) {
            using (var reader = new System.IO.StreamReader(fs)) {
                return await reader.ReadToEndAsync();
            }
        }
    });
```


## 6.0 (2021.12.20)
- 6.0 开始仅支持 `.net6` 和 `.net5`
- 生成随机数调整：
  - `RandomCodeHelper.GetNumber` 代替 `UtilsBasis.RandomNumber`
  - `RandomCodeHelper.GetNumber` 代替 `UtilsBasis.RandomString()`
  - `RandomCodeHelper.GetNumber` 代替 `UtilsBasis.RandomStringByPattern`
  - 通过 `static readonly ConcurrentDictionary<string, string>` 记录已生成随机数，避免近期生成重复随机数。稍后解决清理字典机制。
- Linq 扩展
  - `Distinct` 改为 `DistinctBy` 保持与 `.net6` 新增方法名称一致；此方法仅在 `.net5` 中有效
  - `Replace` 参数调整
- `DataTable` 转 `List<T>` 保留 `ToListByEmit` 方法
- `Directory` `File` `Path` 增加扩展，如 `IOExt.DeleteDirectory` 等，整合一些操作时的判断、非空判断，避免写多余代码
- `MD5` `SHA1` `SHA256` `SHA512` 增加对 `byte[]` `Stream` 进行哈希值计算
- 针对 `.net6` 和 `.net5` 调整其他代码，涉及废弃一些方法


## 5.1.3 （2021.07.10）
- 调整部分代码
- JSON扩展命名空间调整，将优化默认配置修改为：
  - 使用 `System.NewtonsoftJson.Settings` 代替 `System.JsonNetExtensions.JsonNetSettings`
  - 使用 `System.MicrosoftJsonExtensions.Settings` 代替 `System.JsonMicrosoftExtensions.JsonOptions`


## 5.1.1 （2021.07.02）
- 不再支持 `.NET Standard 2.1`，仅支持 `.NET 5.x` 和 `.NET Framework 4.7+`，未来 `6.0` 版本将不支持 `.NET Framework`
- `.NET Framework 4.7+` 设置 C# 语言版本 8.0
- 调整部分代码


## 5.0.6 （2021.06.11）
- 移除 `ResultModel` 对象
- 增加 `ToJsonNS` `ToObjectNS` `ToJsonAsSystemNS` `ToObjectAsSystemNS` 方法，不设置任何序列化、反序列化配置项；
- 扩展 `System.IO`
  - `str.ReplaceInvalidFileName()` 将不允许在文件名中使用的字符替换成下划线；
  - `str.GetFileNameWithoutExtension()` 原始方法 `Path.GetFileNameWithoutExtension` 扩展，增加为空判断，避免异常；
  - `str.GetFileName()` 原始方法 `Path.GetFileName` 扩展，自动判断是否为空，避免异常；
  - `str.GetExtension()`  原始方法 `Path.GetExtension` 扩展，自动判断是否为空，避免异常；
  - `IOExt.DeleteFile()` 删除文件，自动判断源文件是否存在；
  - `IOExt.MoveFile()` 移动文件，自动判断源文件是否存在；
  - `IOExt.Reader()` 读取文本内容，源文件不存在则返回null
  - `IOExt.Writer()` 写入文本内容，自动创建目录结构；
  - `IOExt.CreateDirectory()` 创建目录，自动判断目录是否存在；
  - `IOExt.DeleteDirectory()` 删除目录，自动判断目录是否存在；
  - `IOExt.MoveDirectory` 将文件或目录及其内容移到新位置，自动判断目录是否存在；
  - `IOExt.GetFiles()` 原始方法 `Directory.GetFiles` 扩展，返回指定目录中文件的名称；
  - `IOExt.GetDirectories()` 原始方法 `Directory.GetDirectories` 扩展，返回指定目录中文件的名称；


## 5.0.5.3 （2021.03.24）
- `Rex.Utils.Core` 更新包版本
- `Rex.Microsoft.AspNetCore.Mvc.Extensions` 更新包版本
- `Rex.Helper.Core` 更新包版本


## 5.0.5 （2021.03.15）
- 调整 `ToJson()` 扩展方法，实体属性不设置 `JsonProperty` 时，默认采用 `SnakeCaseNamingStrategy` 规则，如：FooBar =输出=> foo_bar
- 调整 `ToJsonAsSystem()` 扩展方法，实体属性不设置 `JsonPropertyName` 时，默认采用 `SnakeCaseNamingStrategy` 规则，如：FooBar =输出=> foo_bar
- `ToJson()` 和 `ToJsonAsSystem()` 不再捕获异常，触发异常时直接抛出
- 优化部分代码


## 3.2.3 （2020.06.01）
- 增加 UtilsBasis.GetUnixTimestamp 返回毫秒时间戳（e.g. 1591115598575）
- 调整部分代码



## 3.1
- 增加 `AppSettings.GetValueToDirectory` 扩展方法



## 3.0
- 增加 枚举 ToList 方法，例如：`var list = new Color().ToList();`
- 扩展 `ResultModel`



## 2.2.1.5 （2020.05.01）
- 调整 `new ResultModel(int,string,object)` 返回 json `{ code:0,msg:"ok",data:{} }`
- 增加 `new ResultInfo(int,string,object)` 返回 json `{ code:0,msg:"ok",result:{} }`
- 增加 `new ResultInfo<T>(int,string,T)` 返回 json `{ code:0,msg:"ok",result:{} }`
- 增加 `ResultModel.Success()` `ResultModel.Error()`
- 增加 `ResultInfo.Success()` `ResultInfo.Error()`
- 增加 `ResultInfo<T>.Success()` `ResultInfo<T>.Error()`



## 2.2.1.4 （2020.04.22）
- 增加 `SplitsDefault` 扩展方法，当拆分字符串是 null 时返回一个默认集合
- 增加 `ResultModel<T>`，扩展 `ResultModel.Success(), ResultModel<T>.Success(), ResultModel.Error(), ResultModel<T>.Error()`



## 2.2.1 （2020.03.25）
- 增加 `IEnumerable<T>` 扩展 Add
- 优化部分代码



## 2.2.0 （2019.12.25）
- 调整部分代码的依赖关系
- 增加 针对 `IEnumerable` 类型的 `IndexOf` 扩展
- 增加 单例类助手 `Singleton` 例：`public class ClassA : Singleton<ClassA>`
- 增加 `Distinct` 扩展更符合 Linq 一贯的风格 `var p1 = products.Distinct(p => p.ID);`
- 修改 `ResultModel` 命名空间 `System.GeneralExtensions.ResultModel`【和自己的其他项目有冲突了。。。】



## 2.1.0.3 （2019.11.09）
- 支持 `.NET Standard 2.1` `.NET Framework 4.7`
- 重写部分代码
- 优化部分代码
- 增加 `ToDataTable<T>`
- 增加 `ToDateTimes` 支持类型 `DateTime?`，可将Excel日期转换（43799.9820949074 ==> 2019/11/30 23:34:13）
- 增加 `.Distinct()` 扩展 ==> `.Distinct(x=> x.Name)`



## 1.0.8 （2019.07.30）
- 增加：
    - 实体：System.Extend.Models.ResultModel

```csharp
var result = new ResultModel { Code = 1000, Msg = "成功", Data = new { id = 1, name = "xx" } };
var json = paramsErrorResult.ToJson();
```

输出 JSON

```javascript
{
    "code": 1000,
    "msg": "成功",
    "data": {
        "id": 1,
        "name": "xx"
    }
}
```


## 1.0.7 （2019.07.21）
- Bug修复和小的调整



## 1.0.6 （2019.07.20）
#### 添加：
##### 1. 时间相关：
- `ToChineseDate` 转换为农历年
- `ToChineseDay` 阿拉伯数字转换中文日期数字
- `ToChineseMonth` 阿拉伯数字转换成中文月份数字
- `ToStrings<T>`
- `EndOfDay`  一天末尾时间
- `EndOfMonth`  一个月末尾时间
- `EndOfWeek`  一周末尾时间
- `EndOfYear` 一年末尾时间
- `FirstDayOfWeek` 一周的第一天
- `GetAge` 根据出生日期计算年龄
- `GetDateDiff` 日期差计算
- `GetDays` 获取一个月有多少天
- `GetFriendlyString` 友好时间
- `GetWeekdays` 返回两个时间内工作日天数
- `GetWeekends` 返回两个时间内周末天数
- `GetWeekNumber` 获取日期是一年中第几个星期
- `IsToday` 日期是否是今天
- `IsWeekDay` 是否是工作日
- `IsWeekEnd` 是否是周末

##### 2. 字典相关：
- `AddOrReplace` 添加或替换内容
- `AddRange` 合并两个字典
- `GetValue` 字典取值

#### 调整：
- ` LogHelper.Fatal(MethodBase.GetCurrentMethod(), ex);` ==> `LogHelper.Debug(this, ex);`



## 1.0.5
- Bug修复和小的调整
- 更名：`IsInRange` ==> `IsRange`
