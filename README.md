# 说明文档

## Rex.Utils.Core [![NuGet version (Rex.Utils.Core)](https://img.shields.io/nuget/v/Rex.Utils.Core.svg?style=flat-square)](https://www.nuget.org/packages/Rex.Utils.Core/)
- [功能说明](rex.utils.core.md "功能说明")
    - 1.读取 `appsettings.json`
    - 2.Convert 类型转换
    - 3.Linq 扩展
    - 4.加密
    - 5.基于系统方法的扩展
    - 6.验证与判断
    - 7.IO 扩展
    - 8.预留
    - 9.其他
- [更新日志](rex.utils.core.changelog.md "更新日志")


## Rex.Extensions.Models
- 统一返回客户端实体类:
    - `ResultData` 输出JSON结果: `{ "code": 1000, "msg": "string", "result": { } }`
    - `ResultDataModel` 输出JSON结果: `{ "code": 1000, "msg": "string", "data": { } }`
    - `ResultStatusModel` 输出JSON结果: `{ "status": 1000, "msg": "string", "data": { } }`
- 频率较高的实体验证特性
    - MinValue/MaxValue 数值类型最大值、最小值
    - ComplexPassword 强密码校验：必须包含数字、大小写字母、特殊符号
    - MobileNumber 国内手机号码校验：^1(3\d|4[5-9]|5[0-35-9]|6[567]|7[0-8]|8\d|9[0-35-9])\d{8}$<br>
      电信 133,149,153,173,174,177,180,181,189,191,193,199<br>
      移动 134,135,136,137,138,139,147,148,150,151,152,157,158,159,172,178,182,183,184,187,188,195,198<br>
      联通 130,131,132,145,146,155,156,166,175,176,185,186,196<br>
      广电 190,192,197<br>
      电信虚拟 162,1700,1701,1702<br>
      移动虚拟 165,1703,1705,1706<br>
      联通虚拟 167,1704,1707,1708,1709,171
- 持续补充中.

## Rex.Extensions.Microsoft.AspNetCore.Mvc
- 功能说明
    - 1.`AddCorsExt` 跨域扩展
    - 2.`AddJsonOptionsExt` `AddNewtonsoftJsonExt` JSON 序列化扩展
    - 3.`AddExceptionExt` 全局异常处理扩展
    - 4.`AddValidationModelExt` 实体验证扩展
- 更新日志


## Rex.Extensions.Helper.Core
- 功能说明
    - 1.AutoMapperHelper
    - 2.HttpHelper (简易版，仅支持 GET POST)
    - 3.QrCodeHelper
    - 4.HtmlAgilityPack.CssSelectors
- 更新日志


## Rex.Helper.Excel
- 功能说明
- 更新日志

