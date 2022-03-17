# Rex.Utils.Core

## 简介
- 扩展库包括：类型转换、类型检测、加密/解密、日志、常用扩展方法、文字处理、配置文件读取等
- 支持 `.NET 5` `.NET 6`

## 目录
* [更新日志（2022.03.03）](rex.utils.core.changelog.md "更新日志")
* [1.读取 `appsettings.json`](#1读取-appsettingsjson)
* [2.Convert 类型转换](#2convert-类型转换)
* [3.Linq 扩展](#3linq-扩展)
* [4.加密](#4加密)
* [5.基于系统方法的扩展](#5基于系统方法的扩展)
* [6.验证与判断](#6验证与判断)
* [7.IO 扩展](#7io-扩展)
* [8.预留](#8预留)
* [9.其他](#9其他)


## 1.读取 appsettings.json
- `AppSettings.GetValue( )`
- `.NET Core` 读取 `appsettings.json`
- `.NET Framework` 读取 `web.config`
- key 不分区大小写
----

*AppSettings.json*
```JSON
{
  "AppSettings": {
    "Origins": [ "*" ],
    "Headers": [ "*" ],
    "Methods": [ "*" ],
    "CacheTime": 7200,
    "DESKey": "123456!@#$%^",
    "Log4NetPath": "/App_Data/log4net.config"
  }
}
```


*web.config*
- `.NET Framework` 仅支持数值类型、字符串类型

```xml
<appSettings>
   <add key="Origins" value="*"/>
   <add key="Headers" value="*"/>
   <add key="Methods" value="*"/>
   <add key="CacheTime" value="7200" />
   <add key="DESKey" value="123456!@#$%^"/>
   <add key="Log4NetPath" value="/App_Data/log4net.config" />
</appSettings>
```


*[C#]*

```csharp
string path = AppSettings.GetValue("Log4NetPath");   // "/App_Data/log4net.config"

// 以下内容 Framework 不支持

string path = AppSettings.GetValueToPath("Log4NetPath");  // "C:\\Users\\......\\bin\\Debug\\net5.0\\App_Data\\log4net.config"

int t = AppSettings.GetValue<int>("CacheTime");  // 7200

int t = AppSettings.GetValue<int>("Time", 3600);  // key不存在，返回默认值 3600

string[] origins = AppSettings.GetValue<string[]>("arr"); // string[] { "aa", "bb", "cc" }

```

#### .Net Core 支持泛型序列化
- `AppSettings.GetAppSettings<T>( key, path )`
- 可继承基类进行扩展

```csharp
public class AppSettingsBaseEntity {
    public string[] Origins { get; set; }
    public string[] Headers { get; set; }
    public string[] Methods { get; set; }
    public string DESKey { get; set; }
    public string Log4NetPath { get; set; }
    public int CacheTime { get; set; }
}

var app = AppSettings.GetAppSettings<AppSettingsBaseEntity>("AppSettings", "appsettings.json");

```
----


## 2.Convert 类型转换
- [1.Boolean](#201-boolean)
- [2.DataTable 与 List 互转](#202-datatable-与-list-互转)
- [3.Dictionary](#203-dictionary)
- [4.Enum](#204-enum)
- [5.Guid](#205-guid)
- [6.Number](#206-number)
- [7.JSON](#207-json)
- [8.Stream 与 byte[] 互转](#208-stream-与-byte[]-互转)
- [9.String](#209-string)
- [10.Time](#210-time)
- [11.Words](#211-words)
----


### 2.01 Boolean
- 默认支持以下内容的转换
  - `true`
  - `yes`
  - `1`
  - `on`
  - `是`

```csharp
"true".ToBool(); // true
"yes".ToBool();  // true
"1".ToBool();    // true
"on".ToBool();   // true
"是".ToBool();   // true

bool? v = null;
v.ToBool() // false
```

----


### 2.02 DataTable 与 List 互转
- `DataTable` 通过反射转 `List` ==> `ToListByReflect`
- `DataTable` 通过动态生成代码转 `List` ==> `ToListByDynamic`
- `IEnumerable<T>` 转 `DataTable` ==> `ToDataTable`

```csharp
var dic = new Dictionary<int, int?>();
dic.Add(1, 123);
dic.Add(2, 234);
dic.Add(3, null);
var table = dic3.ToDataTable();
// table.Columns.Count ==> 2
// table.Rows[0][0] ==> 1
// table.Rows[0][1] ==> 123
// table.Rows[1][0] ==> 2
// table.Rows[1][1] ==> 234
// table.Rows[2][0] ==> 3
// table.Rows[2][1] ==> DBNull.Value


var arr = new[] { 1, 2, 3, 4 };
var table = arr.ToDataTable();
// table.Columns.Count ==> 1
// table.Rows[0][0] ==> 1
// table.Rows[1][0] ==> 2
// table.Rows[2][0] ==> 3
// table.Rows[3][0] ==> 4

// 暂不支持 Nullable 类型
// 暂不支持 var arr = new int?[] { 1, 2, 3, 4, null };


var list = new List<Demo>
{
    new Demo {PackageId = "1", ReleaseDate = DateTime.Parse("2019-12-01"), Version = "1.1"},
    new Demo {PackageId = "2", ReleaseDate = DateTime.Parse("2019-12-02"), Version = "1.2", Age = 20},
    new Demo {PackageId = "3", ReleaseDate = DateTime.Parse("2019-12-03"), Version = "1.3"},
};
table = list.ToDataTable();
// table.Columns.Count ==> 3
// table.Rows[0]["PackageId"] ==> "1"
// table.Rows[0]["ReleaseDate"] ==> 2019-12-01 00:00:00
// table.Rows[0]["Version"] ==> "1.1"
// table.Rows[0]["Age"] ==> DBNull.Value

// table.Rows[1]["PackageId"] ==> "2"
// table.Rows[1]["ReleaseDate"] ==> 2019-12-02 00:00:00
// table.Rows[1]["Version"] ==> "1.2"
// table.Rows[1]["Age"] ==> 20

// table.Rows[2]["PackageId"] ==> "3"
// table.Rows[2]["ReleaseDate"] ==> 2019-12-03 00:00:00
// table.Rows[2]["Version"] ==> "1.3"
// table.Rows[2]["Age"] ==> DBNull.Value


var rng = new Random();
var query = Enumerable.Range(1, 10000)
    .Select(index => new WeatherForecast {
        Date = DateTime.Now.AddDays(index),
        TemperatureC = rng.Next(-20, 55),
        Summary = Summaries[rng.Next(Summaries.Length)]
    })
    .ToArray();
var o = table.ToListByEmit<WeatherForecast>();
```

----


### 2.03 Dictionary
- `ToDictionary` 目前只支持[枚举]转[字典]

```csharp
enum MediaType { MP3 = 1,MP4,AVI,MOV,WMV,WAV }

var media = typeof(MediaType).ToDictionary();
```

----


### 2.04 Enum
```csharp
public enum DateTimePart {
    [Description("年")]
    Year = 1,
    [Description("月")]
    Month,
    [Description("日")]
    Day,
    [Description("小时")]
    Hour,
    [Description("分钟")]
    Minute,
    [Description("秒")]
    Second,
    
    millisecond,
}

var ls = new DateTimePart().ToList(); // 返回元组类型集合
var c = ls.Count();        // 6

var last = ls.Last();      // (6, "Second", "秒")
// last.name ==> 6;
// last.value  ==> "Second";
// last.description ==> "秒";

var (value, name, description) = ls.Last();
// name ==> 6;
// value  ==> "Second";
// description ==> "秒";

typeof(EnumExtensions.DateTimePart).ToList().Select(x => x.name).ToArray();  // new[] { "Year", "Month", "Day", "Hour", "Minute", "Second" }

typeof(EnumExtensions.DateTimePart).ToList().Select(x => x.value).ToArray();  // new[] { 1, 2, 3, 4, 5, 6 }

var s = DateTimePart.Hour.GetDescription(); // "小时"
// 当枚举未提供 Description 属性时，返回自身，例：
enum DatePart { Year, Month, Day }
DatePart.Month.GetDescription() // Month


var day = "Day".TryParse<EnumExtensions.DateTimePart>();      // DateTimePart.Day
var month = "Month".TryParse<EnumExtensions.DateTimePart>();  // EnumExtensions.DateTimePart.Month

"aaa".TryParse<EnumExtensions.DateTimePart>()   // 0 or (EnumExtensions.DateTimePart)0
"aaa".TryParse(DateTimePart.Hour)               // 返回设定的默认值：DateTimePart.Hour

```

----


### 2.05 Guid

```csharp
string str = "61835e0b-8d89-4d39-aa13-900e192ff824";
Guid gid = str.ToGuid();


Guid? v = null;
Guid gid = v.ToGuid(); // gid == Guid.Empty 即 gid == "00000000-0000-0000-0000-000000000000"

```

----


### 2.06 Number
  * `ToInt16`
  * `ToInt32`
  * `ToInt64`
  * `ToFloat`
  * `ToByte`
  * `ToDecimal`
  * `ToDouble`

```csharp
object obj = 1;
string str = "2";

var a1 = obj.ToInt16();
var a2 = str.ToInt16();

short? v = null;
v.ToInt16() // 0
v.ToInt16(-1) // -1


var b1 = obj.ToInt32();
var b2 =str.ToInt32();

int? v = null;
v.ToInt32() // 0
v.ToInt32(-1) // -1


var c1 = obj.ToInt64();
var c2 = str.ToInt64();

long? v = null;
v.ToInt64() // 0
v.ToInt64(-1) // -1


var d1 = obj.ToFloat();
var d2 = str.ToFloat();

float? v = null;
v.ToFloat() // 0
v.ToFloat(-1) // -1


var e1 = obj.ToByte();
var e2 = str.ToByte();

byte? v = null;
v.ToByte() // 0
v.ToByte(255) // 255


var f1 = obj.ToDecimal();
var f2 = str.ToDecimal();

decimal? v = null;
v.ToDecimal() // 0
v.ToDecimal(-1) // -1


var g1 = obj.ToDouble();
var g2 = str.ToDouble();

double? v = null;
v.ToDouble() // 0
v.ToDouble(-1) // -1

```

----


### 2.07 JSON
- `ToJson` `ToObject`
  - 采用 `Newtonsoft.Json` 进行序列化与反序列化
  - 支持自定义 `Newtonsoft.Json.JsonSerializerSettings`
  - `JsonNetExtensions.JsonNetSettings` 默认配置：
    - 输出忽略所有 null 值属性
    - 输出日期格式：`yyyy/MM/dd HH:mm:ss`
    - 使用蛇形命名策略，如："FooBar" ==> "foo_bar"，注：仅在未设置 JsonProperty 时有效
    - 压缩输出
    - 支持日期属性自定义输出格式


- `ToJsonNS` `ToObjectNS`
  - 序列化与反序列化时不采用任何 `JsonSerializerSettings` 设置参数


- `ToJsonAsSystem` `ToObjectAsSystem`
  - 采用 `System.Text.Json`
  - 支持自定义 `System.Text.Json.JsonSerializerOptions`
  - `JsonMicrosoftExtensions.JsonOptions` 默认配置：
    - 输出忽略所有 null 值属性
    - 输出日期格式：`yyyy/MM/dd HH:mm:ss`
    - 使用蛇形命名策略，如："FooBar" ==> "foo_bar"，注：尽在不设置 JsonPropertyName 时有效
    - 压缩输出
    - 支持中文字符编码


- `ToJsonAsSystemNS` `ToObjectAsSystemNS`
  - 序列化与反序列化时不采用任何 `JsonSerializerOptions` 设置参数


```csharp
[JsonObject(MemberSerialization.OptIn)]
[Serializable]
class Movie {
    [System.Text.Json.Serialization.JsonPropertyName("nn")] // System.Text.Json 自定义 JSON 属性名
    [Newtonsoft.Json.JsonProperty("nn")]                    // Newtonsoft.Json 自定义 JSON 属性名
    public string Name { get; set; }

    [System.Text.Json.Serialization.JsonPropertyName("release_date")]
    [Newtonsoft.Json.JsonProperty("release_date"), Newtonsoft.Json.JsonConverter(typeof(Newtonsoft.Json.Converters.DateTimeConverterExtensions))]
    public DateTime ReleaseDate { get; set; }

    [System.Text.Json.Serialization.JsonPropertyName("rd")]
    [Newtonsoft.Json.JsonProperty("rd"), Newtonsoft.Json.JsonConverter(typeof(Newtonsoft.Json.Converters.DateTimeConverterExtensions), "yyyy.MM.dd")]
    public DateTime? ReleaseDate2 { get; set; }

    [System.Text.Json.Serialization.JsonPropertyName("g")]
    [Newtonsoft.Json.JsonProperty("g")]
    public string[] Genres { get; set; }

    [System.Text.Json.Serialization.JsonIgnore]     // System.Text.Json  忽略单个属性
    [Newtonsoft.Json.JsonIgnore]                    // Newtonsoft.Json  忽略单个属性
    public int Year { get; set; }

    [System.Text.Json.Serialization.JsonPropertyName("age")]
    [Newtonsoft.Json.JsonProperty("age")]
    public int? Age { get; set; }
}

var movie = new Movie();
movie.Title = "F9: The Fast Saga";
movie.ReleaseDate = DateTime.Parse("2021-05-21");
movie.ReleaseDate2 = movie.ReleaseDate;
movie.Genres = new[] { "动作", "冒险", "犯罪", "惊悚片" };
movie.Years = 2021;
movie.Runtime = 143;


var s1 = movie.ToJson()
//{
//	"title": "F9: The Fast Saga",
//	"release_date": "2021/05/21 00:00:00",
//	"rd": "2021.05.21",
//	"genres": ["动作", "冒险", "犯罪", "惊悚片"],
//	"run_time": 143
//}
var m1 = s1.ToObject<Movie>();


var s2 = m.ToJsonNS();
//{
//	"Title": "F9: The Fast Saga",
//	"ReleaseDate": "2021-05-21T00:00:00",
//	"ReleaseDate2": "2021-05-21T00:00:00",
//	"Genres": ["动作", "冒险", "犯罪", "惊悚片"],
//	"Years": 2021,
//	"Runtime": 143
//}
var m2 = s2.ToObjectNS<Movie>();


var s3 =  movie.ToJsonAsSystem();
//{
//	"title": "F9: The Fast Saga",
//	"release_date": "2021/05/21 00:00:00",
//	"rd": "2021/05/21 00:00:00",
//	"genres": ["动作", "冒险", "犯罪", "惊悚片"],
//	"run_time": 143
//}
var m3 = s3.ToObjectAsSystem<Movie>();


var s4 = movie.ToJsonAsSystemNS();
//{
//	"Title": "F9: The Fast Saga",
//	"ReleaseDate": "2021-05-21T00:00:00",
//	"ReleaseDate2": "2021-05-21T00:00:00",
//	"Genres": ["\u52A8\u4F5C", "\u5192\u9669", "\u72AF\u7F6A", "\u60CA\u609A\u7247"],
//	"Years": 2021,
//	"Runtime": 143
//}
var m4 = s4.ToObjectAsSystemNS<Movie>();

```


> 备注：在依赖注入时可提供自定义转换器，若提供时则完全采用，不提供则自动采用系统内置的 `DateTimeConverterExtensions`、`DateTimeNullableConverterExtensions ` 转换器。

**`System.Text.Json` 均支持 .NET5/6**
```csharp
public void ConfigureServices(IServiceCollection services) {
    // 采用系统内置时间格式：yyyy/MM/dd HH:mm:ss
    services.AddJsonOptionsExt();
    
    // 自定义其他时间格式，如：yyyy-MM-dd HH:mm:ss
    // services.AddJsonOptionsExt(
    //     new System.Text.Json.ConverterExtensions.DateTimeConverterExtensions("yyyy-MM-dd HH:mm:ss"),
    //     new System.Text.Json.ConverterExtensions.DateTimeNullableConverterExtensions("yyyy-MM-dd HH:mm:ss")
    // );
}
```

**`Newtonsoft.Json` 均支持 .NET5/6**
```csharp
public void ConfigureServices(IServiceCollection services) {
    // 采用系统内置时间格式：yyyy/MM/dd HH:mm:ss
    services.AddNewtonsoftJsonExt();
}
```

自定义时间格式：

```csharp
public class Demo {
    [Newtonsoft.Json.JsonConverter(typeof(Newtonsoft.Json.ConverterExtensions.DateTimeConverterExtensions), "yyyy-MM-dd HH:mm:ss")]
    public DateTime Date { get; set; }
}
```

----


### 2.08 Stream 与 byte[] 互转
支持以下类型转换
- `byte[]` to `MemoryStream`
  - 当发生异常时，会释放由 `MemoryStream` 使用的所有资源。
- `byte[]` to `Stream`
  - 当发生异常时，会释放由 `Stream` 使用的所有资源。
- `MemoryStream` to `byte[]`
- `Stream` to `byte[]`


```csharp
string host = @"C:\Windows\System32\drivers\etc\hosts";
byte[] buffer;

using (var stream = new FileStream(host, FileMode.OpenOrCreate, FileAccess.ReadWrite)) {
    buffer = stream.ToBytes();
}

MemoryStream m1 = buffer.ToMemoryStream();
byte[] bs1 = ms1.ToBytes();

Stream m2 = buffer.ToStream();
byte[] bs2 = ms2.ToBytes();

```

----


### 2.09 String
支持以下类型转换成
- `DateTime` `DateTime?` ==> 默认格式 `yyyy/MM/dd HH:mm:ss`
- `bool` `bool?`
- `DataTable` `DataRow[]` `DataRow`
- `IEnumerable<T>`

```csharp
DateTime? dt1 = null;
dt1.ToStrings()                         // null
dt1.ToStrings("yyyy/MM/dd")             // null
dt1.ToStrings("yyyy-MM-dd HH:mm:ss")    // null

DateTime dt1 = DateTime.Parse("2020-11-11");
dt1.ToStrings()                         // "2019/05/01 00:00:00"
dt1.ToStrings("yyyy/MM/dd")             // "2019/05/01"
dt1.ToStrings("yyyy-MM-dd HH:mm:ss")    // "2019-05-01 00:00:00"


bool? b1 = true;
b1.ToStrings()  // true

b1 = false;
b1.ToStrings()  // false

b1 = null;
b1.ToStrings()  // false


var dt = new DataTable();
dt.Columns.Add("PackageId", typeof(string));
dt.Columns.Add("Version", typeof(string));
dt.Columns.Add("ReleaseDate", typeof(DateTime));
dt.Rows.Add("Newtonsoft.Json", "11.0.1", new DateTime(2018, 2, 17));
dt.Rows.Add("Newtonsoft.Json", "10.0.3", new DateTime(2017, 6, 18));

dt.ToStrings(0, 1)                  // "11.0.1"
dt.ToStrings(0, "Version")          // "11.0.1"

foreach (DataRow item in dt.Rows)
{
    item.ToStrings(0)               // "Newtonsoft.Json"
    item.ToStrings("PackageId")     // "Newtonsoft.Json"
}

DataRow[] rows = dt.Select();
rows.ToStrings(1, 2)                // "2017/6/18 0:00:00"
rows.ToStrings(1, "ReleaseDate")    // "2017/6/18 0:00:00"



var arr = new[] { 1, 2, 3, 4, 5 };
arr.ToStrings(",")      // "1,2,3,4,5"

var ls = arr.Where(x => x % 2 == 0);
ls.ToStrings("|");      // 2|4


object o1 = DBNull.Value;
o1.ToStrings()   // null

object o2 = null;
o2.ToStrings()   // null
```

----


### 2.10 Time
  * `ToDateTime` 字符串转时间
  * `ToUnixTimestamp` 时间格式转换为 Unix 时间戳格式
  * `EndOfDay`  一天末尾时间
  * `EndOfMonth`  一个月末尾时间
  * `EndOfWeek`  一周末尾时间
  * `EndOfYear` 一年末尾时间
  * `FirstDayOfWeek` 一周的第一天
  * `GetAge` 根据出生日期计算年龄
  * `GetDateDiff` 日期差计算
  * `GetDays` 获取一个月有多少天
  * `GetFriendlyString` 友好时间
  * `GetWeekdays` 返回两个时间内工作日天数
  * `GetWeekends` 返回两个时间内周末天数
  * `GetWeekNumber` 获取日期是一年中第几个星期
  * `IsToday` 日期是否是今天
  * `IsWeekDay` 是否是工作日
  * `IsWeekEnd` 是否是周末

```csharp
object o = DateTime.Parse("2019/5/1");
o.ToDateTime();                         // 2019/5/1 00:00:00
o.ToDateTime("yyyy-MM-dd HH:mm:ss");    // 2019-5-1 00:00:00
"2019/5/1".ToDateTime();                // 2019/5/1 00:00:00


// Unix 时间戳 ==> DateTime
long? val = 1556687655000;
val.ToDateTime()                                        // 2019/5/1 13:14:15
DateTime.Parse("2019/5/1 13:14:15").ToUnixTimestamp();  // 1556687655000


// Excel 日期转换，如 Excel 日期 43799.9820949074 ==> 2019/11/30 23:34:13，失败时返回 null
double? dou = 43799.9820949074;
DateTime? dts = dou.ToDateTime();   // 2019/11/30 23:34:13

dou = null;
dts = dou.ToDateTime();             // null


//一天末尾时间
DateTime.Now.EndOfDay(); // 2019/7/19 23:59:59
//一个月末尾时间
DateTime.Now.EndOfMonth(); // 2019/7/31 23:59:59
//一周末尾时间
DateTime.Now.EndOfWeek(); // 2019/7/21 23:59:59
//一年末尾时间
DateTime.Now.EndOfYear(); // 2019/12/31 23:59:59
//工作周的第一天（星期一）
DateTime.Now.FirstDayOfWeek(); // 2021/7/26 0:00:00
//根据出生日期计算年龄
"2000/7/19".ToDateTime().GetAge(); // 19
//日期差计算
var a = DateTime.Parse("2000-07-31");
var b = DateTime.Parse("2021-07-31");
a.GetDateDiff(b, DateTimePart.Year); // 21

//获取一个月有多少天
DateTime.Parse("2021-01-10").GetDays(); // 31

//友好时间
DateTime.Now.AddDays(-3).GetFriendlyString();
// 32秒前
// 1分钟之前
// 3分钟
// 1小时前
// 3小时前
// 昨天
// 3天之前
// 1个月之前
// 3月之前
// 3年前

//返回两个时间内工作日天数
var t1 = new DateTime(2019, 7, 1);
var t2 = new DateTime(2019, 7, 10);
t1.GetWeekdays(t2); // 7

//返回两个时间内周末天数
var t1 = new DateTime(2019, 7, 1);
var t2 = new DateTime(2019, 7, 19);
t1.GetWeekends(t2); // 4

//获取该日期是一年中的第几周（星期一为本周第一天）
DateTime.Parse("2021-12-18").GetWeekNumber() // 51

//日期是否是今天
DateTime.Now.IsToday();                     // true
DateTime.Parse("1987-09-27").IsToday()      // false
//是否是工作日
DateTime.Parse("2021-12-17").IsWeekDay()    // true
DateTime.Parse("2021-12-18").IsWeekDay()    // false
//是否是周末（周六日）
DateTime.Parse("2021-12-17").IsWeekEnd()    // false
DateTime.Parse("2021-12-18").IsWeekEnd()    // true

```

----


### 2.11 Words
以下内容请移步 `ToolGood.Words` 官网查看详情 **<https://github.com/toolgood/ToolGood.Words>**
- `ToSBC` 半角转全角
- `ToDBC` 全角转半角
- `ToChineseRMB` 数字转成中文大写
- `ToNumber` 大写中文数字转数字
- `ToSimplifiedChinese` 繁体转简体
- `ToTraditionalChinese` 简体转繁体
- `HasChinese` 是否含有中文
- `IsAllChinese` 是否全为中文（不能有任何符号与空格）
- `HasEnglish` 是否含有英语
- `IsAllEnglish` 是否全为英语（不能有任何符号与空格）
- `GetFirstPinYin` 获取首字母
- `GetPinYin` 获取拼音全拼

```csharp
DateTime.Now.ToChineseDate() // "庚子年十月廿六"

(1).ToChineseDay();     // "一"
(11).ToChineseDay();    // "十一"
(0).ToChineseDay();     // null
(33).ToChineseDay();    // null

(1).ToChineseMonth();   // "一月"
(11).ToChineseMonth();  // "十一月"
(0).ToChineseMonth();   // null
(33).ToChineseMonth();  // null


"abcABC123".ToSBC()         // "ａｂｃＡＢＣ１２３"
"ａｂｃＡＢＣ１２３".ToDBC()    // "abcABC123"

(123.45).ToChineseRMB()         // "壹佰贰拾叁元肆角伍分"
"壹佰贰拾叁元肆角伍分".ToNumber()   // 123.45


"简体转换繁体".ToTraditionalChinese() // "簡體轉換繁體"
"簡體轉換繁體".ToSimplifiedChinese()  // "简体转换繁体"

var chs = "GO语言算法优化：在GO版本移植成功后，测试后性能不理想，性能 GO < JAVA < C#，因为我没有完全理解GO语言。JAVA与C#采用了新的算法，GO还是采用老的算法。";
chs.HasChinese();       // true
chs.IsAllChinese();     // false
chs.HasEnglish();       // true
chs.IsAllEnglish();     // false

chs = "细分敏感词分类涉政文本涉爆文本色情文本辱骂文本非法交易文本广告导流文本";
chs.HasChinese();       // true
chs.IsAllChinese();     // true

chs = "JsonNETisapopularhighperformanceJSONframeworkforNET";
chs.HasEnglish();       // true
chs.IsAllEnglish();     // true

chs = "我骑着自行车去的中国人民银行";
chs.GetFirstPinYin();   // "WQZXCQZGRMYH"
chs.GetPinYin();        // "WoQiZiXingCheQuZhongGuoRenMinYinHang"
chs.GetPinYinAsSpace(); // "Wo Qi Zi Hang Che Qu Zhong Guo Ren Min Yin Hang"
chs[3].GetAllPinYin();  // new[] { "Hang", "Xing", "Heng" }

```

----


## 3.Linq 扩展
- [1.Add](#301-add)
- [2.DistinctBy](#302-distinctby)
- [3.ForEach](#303-foreach)
- [4.IndexOf](#304-indexof)
- [5.Insert](#305-insert)
- [6.RemoveAt](#306-removeat)
- [7.Replace](#307-replace)
- [8.SelectAsync](#308-selectasync)
----


### 3.01 Add
- 扩展方法内部避免了 `null` 引发异常

```csharp
IEnumerable<string> q1 = null;
IEnumerable<string> v1 = q1.Add("10"); // new[] { "10" }


IEnumerable<string> q2 = null;
string s2 = null;
IEnumerable<string> v2 = q2.Add(s2); // new string[0]，空集合并非 null


List<int> list = null;
IEnumerable<int> v3 = list.Add(11, 22, 33); // new[] { 11, 22, 33 }

IEnumerable<int> list2 = new List<int> { 1, 2, 3 };
IEnumerable<int> v4 = list2
                        .Add(10)
                        .Add(11, 12)
                        .Add(new[] { 13, 14 })
                        .Add(v3);
// new[] { 1, 2, 3, 10, 11, 12, 13, 14, 11, 22, 33 }
```

---


### 3.02 DistinctBy
- 为 `.net5` 增加了 `keySelector` 用作源类型的比较鉴别器

> 备注：`.net6` 系统新增 `DistinctBy`，特此将 `Distinct` 扩展方法改名为 `DistinctBy`，保持与 `.net6` 一致

```csharp
var lis = new List<Movie>
{
    new Movie {Name = "钢铁侠", Year = 2008},
    new Movie {Name = "无敌浩克", Year = 2008},
    new Movie {Name = "钢铁侠2", Year = 2010},
    new Movie {Name = "雷神", Year = 2011},
    new Movie {Name = "美国队长：复仇者先锋", Year = 2011},
    new Movie {Name = "复仇者联盟", Year = 2012},

    new Movie {Name = "钢铁侠", Year = 2008},
    new Movie {Name = "钢铁侠2", Year = 2010},
};

var year = lis
    .Distinct(x => new { x.Year })
    .Select(x => x.Year);
// new[] { 2008, 2010, 2011, 2012 }


var name = lis
    .Distinct(x => new { x.Name })
    .Select(x => x.Name);
// new[] { "钢铁侠", "无敌浩克", "钢铁侠2", "雷神", "美国队长：复仇者先锋", "复仇者联盟" }


var mo = lis
    .Distinct(x => new { x.Name, x.Year })
    .Select(x => x);
/*
 new Movie[]
 {
     new Movie {Name = "钢铁侠", Year = 2008},
     new Movie {Name = "无敌浩克", Year = 2008},
     new Movie {Name = "钢铁侠2", Year = 2010},
     new Movie {Name = "雷神", Year = 2011},
     new Movie {Name = "美国队长：复仇者先锋", Year = 2011},
     new Movie {Name = "复仇者联盟", Year = 2012}
 }
*/


```

---


### 3.03 ForEach
- 增加了对 `IEnumerable<T>` 的支持

```csharp
IEnumerable<int> lis1 = new[] { 1, 2, 3 };

lis1.ForEach(value => {
    Console.WriteLine(value);
});

// 1 2 3


lis1.ForEach((value, index) => {
    Console.WriteLine($"value: {value}, index: {index}");
});

// value: 1, index: 0
// value: 2, index: 1
// value: 3, index: 2
```

---


### 3.04 IndexOf
- 增加了对 `IEnumerable<T>` 的支持

```csharp
IEnumerable<int> list = new[] { 1, 2, 3, 4, 5, 6, 7, 8, 9, 10 };

list.IndexOf(1);    // 0
list.IndexOf(3);    // 2
list.IndexOf(7);    // 6
list.IndexOf(9);    // 8
```

---


### 3.05 Insert
- 将新序列插入到原序列的指定索引处
- `原序列`与`新序列`不能为 `null`
- `index` 索引不能小于 `0`
- `index` 索引大于原序列长度时，采用原序列长度代替 `index`

```csharp
var seq0 = new[] { 97, 98, 99 };

seq0.Insert(-1, 0)      // new[] { -1, 97, 98, 99 }
seq0.Insert(-1, 1)      // new[] { 97, -1, 98, 99 }
seq0.Insert(-1, 2)      // new[] { 97, 98, -1, 99 }

seq0.Insert(-1, -10)    // new[] { 97, 98, 99 }
seq0.Insert(-1, 3)      // new[] { 97, 98, 99 }


var seq1 = new[] { 0, 1, 2 };
seq1.Insert(seq0, 0)    // new[] { 97, 98, 99, 0, 1, 2, }
seq1.Insert(seq0, 10)   // new[] { 0, 1, 2, 97, 98, 99, }
seq1.Insert(seq0, 1)    // new[] { 0, 97, 98, 99, 1, 2, }

```
---


### 3.06 RemoveAt
- 从 `IEnumerable` 中移除特定对象的第一个匹配项

```csharp
var seq0 = new[] { 97, 98, 99 };

seq0.RemoveAt(0)   // new[] { 98, 99 }
seq0.RemoveAt(1)   // new[] { 11, 98, 99 }
seq0.RemoveAt(2)   // new[] { 97, 98 }

seq0.RemoveAt(4)   // new[] { 97, 98, 99 }
seq0.RemoveAt(-4)  // new[] { 97, 98, 99 }
```

---


### 3.07 Replace
- 根据索引替换相应的值，返回全新 `IEnumerable` 序列

```csharp
var seq0 = new[] { 97, 98, 99 };

// 索引 0 的值替换为 11
seq0.Replace(11, 0)   // new[] { 11, 98, 99 }
seq0.Replace(22, 1)   // new[] { 97, 22, 99 }
seq0.Replace(33, 2)   // new[] { 97, 98, 33 }

seq0.Replace(44, 4)   // new[] { 97, 98, 99 }
seq0.Replace(00, -4)  // new[] { 97, 98, 99 }
```

---


### 3.08 SelectAsync
- 异步将序列中的每个元素投影到新表单。
- `.SelectAsync()`

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

----


## 4.加密
- [1.Base64](#401-base64)
- [2.DES](#402-des)
- [3.MD5](#403-md5)
- [4.SHA1](#404-sha1)
- [5.SHA256](#405-sha256)
- [6.SHA512](#406-sha512)
----


### 4.01 Base64

```csharp
// 默认编码 utf-8
var b64 = val.Base64Encode();
var str = b64.Base64Decode();

// 自定义编码
var b64 = val.Base64Encode(Encoding.Unicode);
var str = b64.Base64Decode(Encoding.Unicode);
```

----


### 4.02 DES

*[appsettings.json]*
```JSON
{
    "AppSettings": {
        "DESKey": "DES-keys"
    }
}
```

*[web.config]*
```xml
<appSettings>
  <add key="DESKey" value="DES-keys"/>
</appSettings>
```

*[C#]*

```csharp
// 默认自动读取 AppSettings 的 DESKey
var pwd = val.DESEncrypt();
var str = pwd.DESDecrypt();

// 自定义加密 key
var key = "DES-keys"; // AppSettings.GetValue("DESKey")
var pwd = val.DESEncrypt(key);
var str = pwd.DESDecrypt(key);
```

----


### 4.03 MD5

```csharp
str.MD5();      // 168F8A2D6C745561C1588DC94369DFE4
str.MD5(false); // 168f8a2d6c745561c1588dc94369dfe4

string val = null;
val.MD5();      // null


var fs = new FileStream(@"C:\Windows\System32\drivers\etc\hosts", FileMode.Open));
fs.ToBytes().MD5();     // D73548E94AEF269BED300599BC23E7E3

fs.MD5Async().Result;   // D73548E94AEF269BED300599BC23E7E3
```

----


### 4.04 SHA1

```csharp
str.SHA1();         // EFE6BA133D57D1952E206B8CA170308A81C7D053
str.SHA1(false);    // efe6ba133d57d1952e206b8ca170308a81c7d053

string val = null;
val.SHA1();         // null


var fs = new FileStream(@"C:\Windows\System32\drivers\etc\hosts", FileMode.Open));
fs.ToBytes().SHA1();    // 99CCBC9E355F799B671E6CC418AAD654E8AED438

fs.SHA1Async().Result;  // 99CCBC9E355F799B671E6CC418AAD654E8AED438
```

----


### 4.05 SHA256

```csharp
str.SHA256();         // 0EC6907A844FEC978B5A1532073CAD487945EE2152DC4167A81B91513C11B6B3
str.SHA256(false);    // 0ec6907a844fec978b5a1532073cad487945ee2152dc4167a81b91513c11b6b3

string val = null;
val.SHA1();           // null


var fs = new FileStream(@"C:\Windows\System32\drivers\etc\hosts", FileMode.Open));
fs.ToBytes().SHA256();    // CF2A0381EDEB234AC80E31F438092DB4A27CF486C952260F0E7074B4B8716953

fs.SHA256Async().Result;  // CF2A0381EDEB234AC80E31F438092DB4A27CF486C952260F0E7074B4B8716953
```

----


### 4.06 SHA512

```csharp
str.SHA512();         // 554882C2AA7CCEDFB17E241C69769624D3346ABEC81EF9D053FE2D8D4948E0025046D2623ECFD4AB99ECD9450676EA09F9435FA3F0E8541EB953FEB89DB431BB
str.SHA512(false);    // 554882c2aa7ccedfb17e241c69769624d3346abec81ef9d053fe2d8d4948e0025046d2623ecfd4ab99ecd9450676ea09f9435fa3f0e8541eb953feb89db431bb

string val = null;
val.SHA512();         // null


var fs = new FileStream(@"C:\Windows\System32\drivers\etc\hosts", FileMode.Open));
fs.ToBytes().SHA512();    // 29B779F07495D939E3C6D240ABE668AFA3C49E1A3373B9241B32491538D191FB87D7342117A382FA6D7F787B73E024B562360CA02C320B7268CA38AE8D3BF30C

fs.SHA512Async().Result;  // 29B779F07495D939E3C6D240ABE668AFA3C49E1A3373B9241B32491538D191FB87D7342117A382FA6D7F787B73E024B562360CA02C320B7268CA38AE8D3BF30C
```

----


## 5.基于系统方法的扩展
- [1.Append](#501-append)
- [2.Joins](#502-joins)
- [3.Lengths](#503-lengths)
- [4.MapPaths](#504-mappaths)
- [5.Replaces](#505-replaces)
- [6.Splits](#506-splits)
- [7.Substrings](#507-substrings)
- [8.ToTitleCase](#508-totitlecase)
- [9.Trims](#509-trims)
- [10.StringCut](#510-stringcut)
----


### 5.01 Append

```csharp
string s1 = "string";
object o1 = "object";
int i1 = 123;
char c1 = '-';
string c2 = "-";

var val = s1
    .Append(c1)
    .Append(o1)
    .Append(c2)
    .Append(i1);                    // "string-object-123"

var v2 = val.AppendBefore("@@");    // "@@string-object-123"
```

----


### 5.02 Joins

```csharp
var list = new List<string>();
list.Joins("_");    // string.Empty

list = null;
list.Joins("_");    // null


list = new List<string>();
list.Add("A");
list.Add("B");
list.Add("C");
list.Joins("_");    // "A_B_C"
```

----


### 5.03 Lengths

```csharp
var str = "获取当前System.String";

var i1 = str.Lengths(); // 21
var i2 = str.Length;    // 17
```

----


### 5.04 MapPaths
- 文件存在返回绝对路径，文件不存在返回 `null`
- 网址路径，则直接返回网址本身，支持 `http://` `https://`

```csharp
var v0 = "https://github.com/xxx/.../master/appsettings.json".MapPaths();   // 网址本身

var v2 = "\\App_Data\\log4net.config".MapPaths();       // C:\IIS\App_Data\log4net.config
var v3 = "App_Data/log4net.config".MapPaths();          // C:\IIS\App_Data\log4net.config
```

----


### 5.05 Replaces

```csharp
var str = "获取当前System.String";

str.Replaces("S");                          // "获取当前ystem.tring"
str.Replaces("S", "A");                     // "获取当前Aystem.Atring"
str.Replaces(new[] { "S", "获取" }, "@");    // "@当前@ystem.@tring"
```

----


### 5.06 Splits
- 返回值不包括包含空字符串的数组元素
- 数组每项会删除开头和结尾空白字符
- 返回序列的每个元素均已去除收尾空格

```csharp
var str = "获取当前System.String对象中的字符数";
str.Splits(".", "的", "当");     // new[] { "获取", "前System", "String对象中", "字符数" }
str.Splits(null);               // new[] { "获取当前System.String对象中的字符数" }
str.Splits(new string[] { });   // new[] { "获取当前System.String对象中的字符数" }


var s1 = "/Sys //tem/ / .String ";
s1.Split('/');                  // 系统方法  new[] { "", "Sys ", "", "tem", " ", " .String " }
s1.Splits("/");                 //         new[] { "Sys", "tem", ".String" }
```

----


### 5.07 Substrings
- 截取字符串，区分中英文字符长度，中文按1:2，英文按1:1 计算长度

```csharp
var str = "获取当前System.String对象中的字符数";

str.Substrings(7);          // "获取当"
str.Substrings(11);         // "获取当前Sys"
str.Substrings(11, "___");  // "获取当前Sys___"
str.Substring(0, 11);       // "获取当前System."
```

----


### 5.08 ToTitleCase
- 将指定字符串转换为词首字母大写

```csharp
"codec_long_name".ToTitleCase();    // "Codec_Long_Name"
"codeclongname".ToTitleCase();      // "Codeclongname"
"codec long name".ToTitleCase();    // "Codec Long Name"
"一个.net core平台".ToTitleCase();   // "一个.Net Core平台"

string[] values = {
    "a tale of two cities",
    "gROWL to the rescue",
    "inside the US government",
    "sports and MLB baseball",
    "The Return OF Sherlock Holmes",
    "UNICEF and children"
};
foreach (var item in values) {
    Console.WriteLine("{0} --> {1}", item, item.ToTitleCase());
}

/*
* a tale of two cities --> A Tale Of Two Cities
* gROWL to the rescue --> Growl To The Rescue
* inside the US government --> Inside The US Government
* sports and MLB baseball --> Sports And MLB Baseball
* The Return of Sherlock Holmes --> The Return Of Sherlock Holmes
* UNICEF and children --> UNICEF And Children
*/
```

----


### 5.09 Trims

```csharp
var val = str.Trims();         // 等同于 str?.Trim() ?? string.Empty;

var str = "获取当前System.String对象中的字符数";
str.Trims('获', '数');          // 取当前System.String对象中的字符
str.Trims(null);               // 获取当前System.String对象中的字符数
```

----


### 5.10 StringCut
- 字符串裁剪

```csharp
var str = "Newtonsoft.Json";
str.StringCut("New", ".");      // tonsoft
```

----


## 6.验证与判断
- [1.IsNull](#61-isnull)
- [2.IsRange](#62-isrange)
----


### 6.1 IsNull
- 判断是否为 null or empty，简化 string.IsNullOrWhiteSpace 方便调用
- 支持已下类型：
  - `string`
  - `object`
  - `DBNull`
  - `DataTable`
  - `DataRowCollection`
  - `DataRow`
  - `DataRow[]`
  - `DataSet`
  - `IEnumerable`


```csharp
object o = null;
o.IsNull()				// true

string str = null;
str.IsNull()				// true
"".IsNull()				// true
" ".IsNull()				// true

var o = DBNull.Value;
o.IsNull()				// true



DataTable table = null;
table.IsNull()				// true

table = new DataTable();
table.IsNull()				// true
table.Rows.IsNull()			// true

table.Rows.Add()
table.Rows.Add()
table.IsNull()				// false
table.Rows.IsNull()			// false


DataSet ds = null;
ds.IsNull()				// true

ds = new DataSet();
ds.IsNull()				// true

ds.Tables.Add(new DataTable());
ds.Tables.Add(new DataTable());
ds.IsNull();				// false


var list = new List<string>();
list.IsNull();				// true
list = null;
list.IsNull();				// true
```
----


### 1.2 IsRange
- 支持检测 数值/时间/集合 是否在的范围内


```csharp
var a = 1;
var b = 10;
var c = 5;
var d = 14;
c.IsRange(a, b);		// true
d.IsRange(a, b);		// false


var a = DateTime.Parse("2019/5/1");
var b = DateTime.Parse("2019/5/5");
var c = DateTime.Parse("2019/5/2");
var d = DateTime.Parse("2019/5/6");
c.IsRange(a, b);		// true
d.IsRange(a, b);		// false


var list = new List<int> { 1, 2, 3, 4, 5, 6, 7, 8, 9, 10 };
var val1 = 3;
var val2 = 55;
val1.IsRange(list);		// true
val2.IsRange(list);		// false
```
----


## 7.IO 扩展
> 仅对 `Directory` `File` `Path` 整合一些操作时的判断、非空判断，避免写多余代码；
> 
> 习惯使用原生方法的可以忽略此部分；
----


- **Directory**
  - `IOExt.CreateDirectory` 创建目录，目录已存在的则忽略；
  - `IOExt.DeleteDirectory` 删除目录，目录不存在的则忽略；
  - `IOExt.MoveDirectory` 移动目录，移动的目录存在 且 新目录不存在 才移动；
  - `IOExt.GetFiles` 返回目录中文件名称，习惯 `Directory.GetFiles` 方法的忽略此处；
  - `IOExt.GetDirectories` 返回目录中子目录的名称，习惯 `Directory.GetDirectories` 方法的忽略此处；



- **File**
  - `IOExt.ConvertFileSize` 文件单位转换 Bytes,KB,MB,GB,TB；
  - `IOExt.DeleteFile` 删除指定的文件，不存在的文件则忽略；
  - `IOExt.MoveFile` 移动文件，文件不存在的则忽略；可设置是否要覆盖目标文件；习惯 `File.Move` 方法的忽略此处；
  - `IOExt.Reader` 读取文档内容，默认 UTF8 格式，文件不存在返回 null；
  - `IOExt.ReaderAsync` 异步读取文档，默认 UTF8 格式，文件不存在返回 null；
  - `IOExt.Writer` 写入文档内容，默认 UTF8 格式，保存文件路径不存在则自动创建目录；
  - `IOExt.WriterAsync` 异步写入文档内容，默认 UTF8 格式，保存文件路径不存在则自动创建目录；



- **Path**
  - `IOExt.ReplaceInvalidFileName` 将不允许在文件名中使用的字符替换成下划线
  - `IOExt.GetFileNameWithoutExtension` 返回不具有扩展名的指定路径字符串的文件名，习惯 `Path.GetFileNameWithoutExtension` 方法的忽略此处；
  - `IOExt.GetFileName` 返回指定路径字符串的文件名和扩展名，习惯 `Path.GetFileName` 方法的忽略此处；
  - `IOExt.GetExtension` 返回指定路径字符串的扩展名（包括句点“.”），习惯 `Path.GetExtension` 方法的忽略此处；

----


## 8.预留
> 预留


----


## 9.其他
- [1.GetIPv4](#91-getipv4)
- [2.GetUnixTimestamp](#92-获取时间戳) 获取时间戳（毫秒），13位
- [3.ConvertBytes](#93-convertbytes) 单位转换 Bytes,KB,MB,GB,TB
- [4.Random](#94-random)
----


### 9.1 GetIPv4


```csharp
string[] ip = UtilsBasis.GetIPv4;	// new[] { "192.168.132.1", "192.168.126.1", "192.168.3.100" }
```
----


### 9.2 获取时间戳
- 获取时间戳（毫秒），13位
- 获取雪花Id，18位


```csharp
UtilsBasis.GetUnixTimestamp;	// 1607875432593

UtilsBasis.SnowflakeId;
```

### 9.3 ConvertBytes
- 单位转换 Bytes, KB, MB, GB, TB
- isStandardValue参数：true 按1024计算，false 按1000计算

```csharp
IOExt.ConvertFileSize(245134695)		// "245.13 MB"
IOExt.ConvertFileSize(245134695, true)		// "233.78 MB"

IOExt.ConvertFileSize(1592494871)		// "1.59 GB"
IOExt.ConvertFileSize(1592494871, true)		// "1.48 GB"

IOExt.ConvertFileSize(73181050853)		// "73.18 GB"
IOExt.ConvertFileSize(73181050853, true)	// "68.16 GB"

IOExt.ConvertFileSize(316969)			// "316.97 KB"
IOExt.ConvertFileSize(316969, true)		// "309.54 KB"
```
----


### 9.4 Random
- `RandomCodeHelper.GetNumber`: 随机纯数字
- `RandomCodeHelper.GetString`: 随机数字+字母
- `RandomCodeHelper.GetStringByPattern`: 随机支持自定义生成样式，默认：##??**
  - "?"代表一个字符
  - "#"代表一个一位数字
  - "*"代表一个字符串或一个一位数字


> 通过 `static readonly ConcurrentDictionary<string, string>` 记录已经产生的随机数，避免重复产生。


```csharp
RandomCodeHelper.GetNumber()		// "707785"
RandomCodeHelper.GetNumber(10)		// "6102154083"


RandomCodeHelper.GetString()		// "20NB66"
RandomCodeHelper.GetString(10)		// "DX6ZVVNRPF"


RandomCodeHelper.GetStringByPattern()		// "36hBLo"
RandomCodeHelper.GetStringByPattern("####-???")	// "5065-CEc"
```

----
