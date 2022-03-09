# Rex.Helper.Excel

## 简介
框架 `.NET 6.0` `.NET 5.0` `.NET Standard 2.1` `.NET Framework 4.7`，

- > **using [EPPlus](https://www.nuget.org/packages/EPPlus "EPPlus") , [EPPlus.Core.Extensions](https://www.nuget.org/packages/EPPlus.Core.Extensions "EPPlus.Core.Extensions")**
- > EPPlus 5+
  - > .net: `appSettings` 增加 `<add key="EPPlus:ExcelPackage.LicenseContext" value="NonCommercial" />`
  - > core: `appsettings.json` 根节点增加 `"EPPlus": { "ExcelPackage": { "LicenseContext": "NonCommercial" } }`


## 1.DataTable 集合生成 Excel 文件

```csharp
DataTable dataTable = new DataTable();

dataTable.Columns.Add("学号");
dataTable.Columns.Add("姓名");
dataTable.Columns.Add("性别");
dataTable.Columns.Add("语文");
dataTable.Columns.Add("数学");
dataTable.Columns.Add("英语");
dataTable.Columns.Add("物理");
dataTable.Columns.Add("化学");
dataTable.Columns.Add("生物");
dataTable.Columns.Add("历史");
dataTable.Columns.Add("地理");
dataTable.Columns.Add("政治");

Random random = new Random(DateTime.Now.Millisecond);
for (int i = 1; i <= 30; i++) {
    dataTable.Rows.Add("stu" + i, "name" + i, "M",
        random.Next(60, 100), random.Next(60, 100), random.Next(60, 100),
        random.Next(60, 100), random.Next(60, 100), random.Next(60, 100),
        random.Next(60, 100), random.Next(60, 100), random.Next(60, 100));
}

var t1 = dataTable.Copy();
t1.TableName = "sheet_name_1";

var t2 = dataTable.Copy();
t2.TableName = "sheet_name_2";

var t3 = dataTable.Copy();
t3.TableName = "sheet_name_3";

var tables = new List<DataTable> { t1, t2, t3 };
var bytes = tables.ToXlsx();
File.WriteAllBytes(string.Concat(ExcelPath, DateTime.Now.ToString("yyyMMddHHmmssfff"), ".xlsx"), bytes);

var t4 = dataTable.Copy();
tables = new List<DataTable> { t4, t4 };
bytes = tables.ToXlsx();
File.WriteAllBytes(string.Concat(ExcelPath, DateTime.Now.ToString("yyyMMddHHmmssfff"), ".xlsx"), bytes);

```


## 2.泛型集合生成 Excel 文件
- `ToWorksheet` 创建新Sheet
- `NextWorksheet` 创建下一个Sheet
- `WithTitle` 为表格创建标题行
- `WithColumn` 为表格创建表头
- `WithoutHeader` 不显示表格标题行
- `WithConfiguration` 为表格设置样式
  - `WithWorksheetConfiguration` 设置Sheet全局样式，优先级0
    - `SetBackgroundColor` 设置背景色
    - `SetDefaultRowHeight` 设置行高
    - `SetFont` 设置字体
    - `SetFontColor` 设置字体颜色
    - `SetFreezePanes` 设置冻结首行
    - `SetHorizontalAlignment` 设置水平对齐方式
    - `SetShowGridLines` 设置显示网格线
    - `SetVerticalAlignment` 设置垂直对齐方式
  - `WithTitleConfiguration` 设置表格标题样式，不设置表格标题（WithTitle）此属性无效，优先级1
    - `SetBackgroundColor` 设置背景色
    - `SetBorderAround` 设置单元格边框线
    - `SetFont` 设置字体
    - `SetFontAsBold` 设置字体加粗
    - `SetFontColor` 设置字体颜色
    - `SetFontName` 设置字体名称
    - `SetHorizontalAlignment` 设置水平对齐方式
    - `SetVerticalAlignment` 设置垂直对齐方式
  - `WithHeaderConfiguration` 设置表格列头样式，优先级2
    - `SetBackgroundColor` 设置背景色
    - `SetBorderAround` 设置单元格边框线
    - `SetFont` 设置字体
    - `SetFontAsBold` 设置字体加粗
    - `SetFontColor` 设置字体颜色
    - `SetFontName` 设置字体名称
    - `SetHorizontalAlignment` 设置水平对齐方式
    - `SetVerticalAlignment` 设置垂直对齐方式
  - `WithHeaderRowConfiguration` 设置表格列头行样式，优先级3
    - `SetBackgroundColor` 设置背景色
    - `SetBorderAround` 设置单元格边框线
    - `SetFont` 设置字体
    - `SetFontAsBold` 设置字体加粗
    - `SetFontColor` 设置字体颜色
    - `SetFontName` 设置字体名称
    - `SetHorizontalAlignment` 设置水平对齐方式
    - `SetVerticalAlignment` 设置垂直对齐方式
  - `WithCellConfiguration` 设置单元格样式（或对数据进行其他操作），优先级4
    - `SetBackgroundColor` 设置背景色
    - `SetBorderAround` 设置单元格边框线
    - `SetFont` 设置字体
    - `SetFontAsBold` 设置字体加粗
    - `SetFontColor` 设置字体颜色
    - `SetFontName` 设置字体名称
    - `SetHorizontalAlignment` 设置水平对齐方式
    - `SetVerticalAlignment` 设置垂直对齐方式
  - `WithColumnConfiguration` 设置表格列样式，优先级5
    - `SetBackgroundColor` 设置背景色
    - `SetBorderAround` 设置有效列边框线
    - `SetFont` 设置字体
    - `SetFontAsBold` 设置字体加粗
    - `SetFontColor` 设置背景色
    - `SetFontName` 设置字体名称
    - `SetHorizontalAlignment` 设置水平对齐方式
    - `SetVerticalAlignment` 设置垂直对齐方式
  - *优先级高覆盖低*

*[C#] 泛型集合  ==> Excel*

```csharp
var data = new List<Person>
{
    new Person {FirstName = "Daniel", LastName = "Day-Lewis", YearBorn = 1957},
    new Person {FirstName = "Sally", LastName = "Field", YearBorn = 1946},
    new Person {FirstName = "David", LastName = "Strathairn", YearBorn = 1949},
    new Person {FirstName = "Joseph", LastName = "Gordon-Levitt", YearBorn = 1981},
    new Person {FirstName = "James", LastName = "Spader", YearBorn = 1960},
    new Person {FirstName = "堵", LastName = "华荣", YearBorn = 1961},
    new Person {FirstName = "楚", LastName = "笑萍", YearBorn = 1962},
    new Person {FirstName = "犹", LastName = "格", YearBorn = 1963},
    new Person {FirstName = "阎", LastName = "奕", YearBorn = 1964},
    new Person {FirstName = "茆", LastName = "夏蓉", YearBorn = 1965},
    new Person {FirstName = "路", LastName = "琨瑶", YearBorn = 1966},
    new Person {FirstName = "孟", LastName = "琼芳", YearBorn = 1967},
    new Person {FirstName = "南", LastName = "晓灵", YearBorn = 1968},
    new Person {FirstName = "廉", LastName = "心诺", YearBorn = 1969},
    new Person {FirstName = "苟", LastName = "琦", YearBorn = 1970},
    new Person {FirstName = "谯", LastName = "亦云", YearBorn = 1971},
    new Person {FirstName = "朴", LastName = "春蕾", YearBorn = 1972},
    new Person {FirstName = "户", LastName = "念真", YearBorn = 1973},
};

var worksheet = _data.Where(x => x.YearBorn < 1950)
    /*** 创建 sheet1 ***/
    .ToWorksheet("< 1950") //设置 sheet 名称
    .WithTitle(">= 1950") //设置第一行为标题行，并将有效列合并
    .WithColumn(x => x.FirstName, "First Name") //设置当前列标题（优先级高于属性的 ExcelTableColumnAttribute）
    .WithColumn(x => x.YearBorn, "Year Born")
    /*** 创建 sheet2 ***/
    .NextWorksheet(_data.Where(x => x.YearBorn >= 1950), " >= 1950 ") //创建新 sheet
    .WithoutHeader() //不显示列标题（但 WithTitle 不受影响）
    /*** 创建 sheet3 ***/
    .NextWorksheet(_data.Where(x => x.YearBorn >= 1950 && x.YearBorn <= 1980)) //创建新 sheet，自动创建工作簿名称
    .WithTitle("1950 between 1980") //设置第一行为标题行，并将有效列合并
    /*** 配置工作簿样式 ***/
    .WithConfiguration(config => config
        /*** 设置Sheet全局样式，优先级0 ***/
        .WithWorksheetConfiguration(sheet => sheet
            .SetFont(new Font("宋体", 11))
            .SetDefaultRowHeight(17)
            .SetShowGridLines(false)
            .SetFreezePanes(3, 1)
            .SetVerticalAlignment(ExcelVerticalAlignment.Center)
            .SetHorizontalAlignment(ExcelHorizontalAlignment.Left)
        )
        /*** 设置表格标题样式（不设置 WithTitle 此属性无效），优先级1 ***/
        .WithTitleConfiguration(title => title
            .SetBackgroundColor(Color.Yellow)
            .SetBorderAround(Color.Pink)
            .SetBorderAround(ExcelBorderStyle.MediumDashDotDot)
        )
        /*** 设置表格列头样式，优先级2 ***/
        .WithHeaderConfiguration(header => header
            .SetFont(new Font("Arial", 13, FontStyle.Bold))
            .SetFontColor(Color.FromArgb(231, 230, 230))
            .SetBackgroundColor(Color.FromArgb(68, 84, 106))
        )
        /*** 设置表格列头行样式，优先级3 ***/
        .WithHeaderRowConfiguration(headerRow => headerRow
            .SetBorderAround(Color.Red, ExcelBorderStyle.Double)
            .SetFontName("Verdana")
        )
        /*** 设置单元格样式（或对数据进行其他操作），优先级4 ***/
        .WithCellConfiguration((range, entity) => {
            range
                .SetFont(new Font("Times New Roman", 13))
                .SetFontAsBold();
            entity.YearBorn = entity.YearBorn % 2 == 0 ? entity.YearBorn : 1990; //对属性单独处理
        })
        /*** 设置表格有效列样式，优先级5 ***/
        .WithColumnConfiguration(column => column    
            .SetFont(new Font("微软雅黑", 11))
            .SetHorizontalAlignment(ExcelHorizontalAlignment.Left)
            .SetVerticalAlignment(ExcelVerticalAlignment.Top)
        )
    );

var bytes = worksheet.ToXlsx();
File.WriteAllBytes(string.Concat(ExcelPath, DateTime.Now.ToString("yyyMMddHHmmssfff"), ".xlsx"), bytes);

```

## 3.读取 Excel 到 DataTable
- 目前只支持 **`.xlsx`** 格式文档
- 索引从0开始

```csharp
var path = Path.Combine(AppDomain.CurrentDomain.BaseDirectory, "demo.xlsx");
//将第1个 sheet，返回 DataTable
var table = ExcelHelper.ToTable(path);
//将第3个 sheet，返回 DataTable
table = ExcelHelper.ToTable(path, 2);

//读取所有sheet，返回 List<DataTable>
var tables = ExcelHelper.ToTables(path);
```


## 3. Excel => `DataTable`
- 目前只支持 **`.xlsx`** 格式文档
- .NET Framework 版本索引从 1 开始，.NET Core 版本索引从 0 开始

```csharp
var path = Path.Combine(AppDomain.CurrentDomain.BaseDirectory, "demo.xlsx");
//将第1个 sheet，返回 DataTable
var table = ExcelHelper.ToTable(path);
//将第3个 sheet，返回 DataTable
table = ExcelHelper.ToTable(path, 2);

//读取所有sheet，返回 List<DataTable>
var tables = ExcelHelper.ToTables(path);
```


## 3. Excel => `List<T>`
- 目前只支持 **`.xlsx`** 格式文档
- .NET Framework 版本索引从 1 开始，.NET Core 版本索引从 0 开始
- `ExcelTableColumn` 属性映射到Excel列，别名和索引不可同时使用（不允许`[ExcelTableColumn(2, ColumnName = "Year of Birth")]`）
- 推荐：`属性顺序` 保持与 `Excel列顺序` 一致


```csharp
class Person {
    public string FirstName { get; set; }

    [ExcelTableColumn]
    public string LastName { get; set; }

    [ExcelTableColumn("Year of Birth")]
    public int? YearBorn { get; set; }
}

var _data = new List<Person>
{
    new Person {FirstName = "Daniel", LastName = "Day-Lewis", YearBorn = 1957},
    new Person {FirstName = "Sally", LastName = "Field", YearBorn = 1946},
    new Person {FirstName = "David", LastName = "Strathairn", YearBorn = 1949},
    new Person {FirstName = "Joseph", LastName = "Gordon-Levitt", YearBorn = 1981},
};

var sheet = _data.Where(x => x.YearBorn < 2000).ToWorksheet("sheet1");
var bytes = sheet.ToXlsx();
var path = Path.Combine(ExcelPath, $"{DateTime.Now:yyyMMddHHmmssfff}.xlsx");
File.WriteAllBytes(path, bytes);

var list = ExcelHelper.ToList<Person>(path, 0); // list.Count == 4

```

