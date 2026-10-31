# W01 Assignment Notes: Build .NET Applications with C#

Student: Yesid Augusto Romero Ruiz

Environment: .NET 8 SDK (8.0.427). Both projects target `net8.0`.

## 1. Pizzas evidence (Create a web API with ASP.NET Core controllers)

The module's seed data in `Services/PizzaService.cs` has two pizzas: `Classic Italian` (id 1) and `Veggie` (id 2). I added a third one, `Pepperoni` (id 3), and changed `nextId` from `3` to `4` so that the next pizza created through POST does not get a duplicated id.

```csharp
static int nextId = 4;
static PizzaService()
{
    Pizzas = new List<Pizza>
    {
        new Pizza { Id = 1, Name = "Classic Italian", IsGlutenFree = false },
        new Pizza { Id = 2, Name = "Veggie", IsGlutenFree = true },
        new Pizza { Id = 3, Name = "Pepperoni", IsGlutenFree = false }
    };
}
```

Request (API freshly started, before any other change):

```http
GET http://localhost:5147/pizza/
Accept: application/json
```

Response (200 OK), showing the two existing pizzas and the additional one:

```json
[
  {
    "id": 1,
    "name": "Classic Italian",
    "isGlutenFree": false
  },
  {
    "id": 2,
    "name": "Veggie",
    "isGlutenFree": true
  },
  {
    "id": 3,
    "name": "Pepperoni",
    "isGlutenFree": false
  }
]
```

## 2. API operations: request, response and status code

All requests were sent from `ContosoPizza.http` in Visual Studio Code with the API running at `http://localhost:5147`, started from scratch so the first created pizza gets id 4.

### POST: create a pizza (201 Created)

Request:

```http
POST http://localhost:5147/pizza/
Content-Type: application/json

{
    "name": "Hawaii",
    "isGlutenFree": false
}
```

Response:

```http
HTTP/1.1 201 Created
Connection: close
Content-Type: application/json; charset=utf-8
Date: Sat, 31 Oct 2026 20:41:45 GMT
Server: Kestrel
Location: http://localhost:5147/Pizza/4
Transfer-Encoding: chunked

{
  "id": 4,
  "name": "Hawaii",
  "isGlutenFree": false
}
```

Status code: **201 Created**

### PUT: update a pizza (204 No Content)

Request:

```http
PUT http://localhost:5147/pizza/4
Content-Type: application/json

{
    "id": 4,
    "name": "Hawaiian",
    "isGlutenFree": false
}
```

Response:

```http
HTTP/1.1 204 No Content
Connection: close
Date: Sat, 31 Oct 2026 20:42:02 GMT
Server: Kestrel
```

Status code: **204 No Content**

### GET: read the updated pizza (200 OK)

Request:

```http
GET http://localhost:5147/pizza/4
Accept: application/json
```

Response:

```http
HTTP/1.1 200 OK
Connection: close
Content-Type: application/json; charset=utf-8
Date: Sat, 31 Oct 2026 20:42:17 GMT
Server: Kestrel
Transfer-Encoding: chunked

{
  "id": 4,
  "name": "Hawaiian",
  "isGlutenFree": false
}
```

Status code: **200 OK**

### DELETE: remove a pizza (204 No Content)

Request:

```http
DELETE http://localhost:5147/pizza/4
```

Response:

```http
HTTP/1.1 204 No Content
Connection: close
Date: Sat, 31 Oct 2026 20:42:32 GMT
Server: Kestrel
```

Status code: **204 No Content**

### Verification after DELETE (200 OK)

Request:

```http
GET http://localhost:5147/pizza/
Accept: application/json
```

Response: `HTTP/1.1 200 OK` (Date: Sat, 31 Oct 2026 20:42:46 GMT). The list contains only the three original pizzas, so pizza 4 was removed:

```json
[
  { "id": 1, "name": "Classic Italian", "isGlutenFree": false },
  { "id": 2, "name": "Veggie", "isGlutenFree": true },
  { "id": 3, "name": "Pepperoni", "isGlutenFree": false }
]
```

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
