# W01 Assignment Notes: Build .NET Applications with C#

Student: Yesid Augusto Romero Ruiz

## 1. Pizzas evidence (Create a web API with ASP.NET Core controllers)

_Pending: complete after module 6._

## 2. API operations: request, response and status code

_Pending: complete after module 6._

| Operation | Request | Response | Status code |
|-----------|---------|----------|-------------|
| GET       |         |          |             |
| POST      |         |          |             |
| PUT       |         |          |             |
| DELETE    |         |          |             |

## 3. Sales summary function (Work with files and directories in a .NET app)

Project: `mslearn-dotnet-files/Program.cs` (.NET 8, `net8.0`).

The function reads every sales file, adds up each file's `Total`, and builds the report with a `StringBuilder`. Amounts are formatted as currency with the `en-US` culture so they always look like `$1,234.56`, regardless of the computer's regional settings. File names are shown as paths relative to the `stores` folder, because several files share the same name (`sales.json`).

Required `using` statements:

```csharp
using System.Globalization;
using System.Text;
using Newtonsoft.Json;
```

The function:

```csharp
string GenerateSalesSummary(IEnumerable<string> salesFiles, string baseDirectory)
{
    var culture = CultureInfo.GetCultureInfo("en-US");
    var details = new StringBuilder();
    double grandTotal = 0;

    foreach (var file in salesFiles)
    {
        string salesJson = File.ReadAllText(file);
        SalesData? data = JsonConvert.DeserializeObject<SalesData?>(salesJson);
        double fileTotal = data?.Total ?? 0;

        grandTotal += fileTotal;

        string relativeName = Path.GetRelativePath(baseDirectory, file);
        details.AppendLine($"  {relativeName}: {fileTotal.ToString("C", culture)}");
    }

    var report = new StringBuilder();
    report.AppendLine("Sales Summary");
    report.AppendLine("----------------------------");
    report.AppendLine($" Total Sales: {grandTotal.ToString("C", culture)}");
    report.AppendLine();
    report.AppendLine(" Details:");
    report.Append(details);

    return report.ToString();
}
```

How it is called (after `CalculateSalesTotal`):

```csharp
var summary = GenerateSalesSummary(salesFiles, storesDirectory);
File.WriteAllText(Path.Combine(salesTotalDir, "salesSummary.txt"), summary);
```

Supporting record used for deserialization:

```csharp
record SalesData(double Total);
```

Output generated in `salesTotalDir/salesSummary.txt`:

```text
Sales Summary
----------------------------
 Total Sales: $2,012.20

 Details:
  sales.json: $88.88
  201\sales.json: $501.22
  201\salestotals.json: $0.00
  202\sales.json: $1,234.22
  202\salestotals.json: $0.00
  203\sales.json: $99.00
  203\salestotals.json: $0.00
  204\sales.json: $88.88
  204\salestotals.json: $0.00
```

Check: 88.88 + 501.22 + 1,234.22 + 99.00 + 88.88 = 2,012.20, which matches the total. The `salestotals.json` files are included because the module selects every `.json` file; they contribute `$0.00`.
