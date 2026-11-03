# AdvancedSearch

An ASP.NET Core + React sample that implements the **advanced search scenario**: a full-page query
builder where users pick entities and columns, define conditions, run the query and work with
the result. The page is a React component (`ClientApp/`), and the ASP.NET Core app serves the
EasyQuery API to it.

What you will see on the **EasyQuery** page (`/easy-query`, the last item of the navigation bar):

* **Entities** panel: the tree of all entities and attributes of the data model; drag or click to add them to the query.
* **Columns** panel: the result columns (with sorting and aggregate functions).
* **Conditions** panel: the query conditions, grouped with AND/OR.
* **Query Menu**: Clear, Load, Storage (New, Save, Save as, Remove) and Fetch data. Three predefined queries are included (`App_Data/dm-nwind/queries`).
* **Result** panel: the result grid (paging by 30 rows) and a chart, with export to PDF, Excel and CSV.

The browser also remembers the current query (`localStorage`), so the page opens with the last
query you worked on.

All EasyQuery server settings are in `Startup.cs`: `AddEasyQuery()` in `ConfigureServices` and
`MapEasyQuery(...)` in `Configure`. The client settings are in `ClientApp/src/components/EasyQuery.js`.

## Tech info

| | |
|---|---|
| .NET | .NET 6 (`net6.0`) |
| Platform | ASP.NET Core |
| UI framework | **React 16** (Create React App, `react-scripts` 5), served through `Microsoft.AspNetCore.SpaServices.Extensions` |
| EasyQuery for .NET | **7.5.0**: `Korzh.EasyQuery.AspNetCore`, `Korzh.EasyQuery.EntityFrameworkCore.Relational`, `Korzh.EasyQuery.DataExport`, `Korzh.EasyQuery.SqLiteGate`, `Korzh.EasyQuery.SqlServerGate` |
| EasyData exporters | 1.6.0: `EasyData.Exporters.PdfSharp` (PDF), `EasyData.Exporters.ClosedXML` (Excel) |
| EasyQuery.JS | **7.5.1** from npm (`@easyquery/core`, `@easyquery/ui`, `@easyquery/enterprise` 7.5.1; `@easydata/core`, `@easydata/ui` 1.6.0) |
| Query engine | SQL (`UseSqlManager()`) |
| Data model source | EF Core `DbContext` (`UseDbContext<AppDbContext>`) |
| EasyQuery API | Middleware: `endpoints.MapEasyQuery(...)`, endpoint `/api/easyquery` |
| Database | SQLite through EF Core 6 (SQL Server is prepared in commented code) |
| Query storage | `FileQueryStore`: JSON files in `App_Data/dm-nwind/queries` |

## Project structure

| File / folder | What's inside |
|---|---|
| `Startup.cs` | **The EasyQuery settings**: services (DbContext, CORS, SPA, EasyQuery + exporters), `MapEasyQuery`, the React dev server, demo DB initialization |
| `Program.cs` | The host; it only starts `Startup` |
| `ClientApp/src/components/EasyQuery.js` | **The page component**: `AdvancedSearchView` options and initialization, the query kept in `localStorage` |
| `ClientApp/src/components/EasyQueryHtml.js` | The page markup: the panels and the buttons the view finds by ID |
| `ClientApp/src/index.js`, `ClientApp/src/css/index.css` | The EasyQuery styles (`@easyquery/ui/dist/assets/css/easyquery.ui.css`) and one rule of the sample's own |
| `ClientApp/src/App.js`, `ClientApp/src/components/NavMenu.js` | The `/easy-query` route and its menu item |
| `ClientApp/package.json`, `ClientApp/.npmrc` | The npm packages; the commented settings for the Korzh MyGet feed (pre-release packages) |
| `Models/NWind/`, `Data/AppDbContext.cs` | Entity classes and the DbContext of the demo (Northwind) database |
| `Seed/DbInitializeExtensions.cs` | Creates and seeds the database on the first run |
| `App_Data/dm-nwind/queries/` | Predefined queries for the `nwind` model |
| `App_Data/EqDemoData.zip` | Demo data used to seed the database |
| `Controllers/SampleDataController.cs`, `Counter.js`, `FetchData.js`, `Home.js` | Left over from the project template; not used by EasyQuery |

## How to start

1. Install the [.NET 6 SDK](https://dotnet.microsoft.com/download/dotnet/6.0) (or later, with the
   .NET 6 runtime) and [Node.js](https://nodejs.org/) (LTS; `npm` must be in `PATH`).
2. You don't need a license key to start. On the first run the page shows a **No license**
   dialog that gets you a free 30-day trial key. A purchased key goes into `appsettings.json`
   (`EasyQuery:LicenseKey`); see [License key](../README.md#license-key).
3. Run the project:

   ```bash
   dotnet run
   ```

   or open `../Samples.React.sln` and start `EqDemo.AspNetCoreReact.AdvancedSearch`. The first
   build runs `npm install` in `ClientApp`, which takes a few minutes. In Development, the app
   also starts the React dev server (`npm start`).
4. Open https://localhost:5001 (or http://localhost:5000) and click **EasyQuery** in the
   navigation bar.

On the first run the application creates the SQLite database `eqdemo-sqlite.db` in the project
folder and fills it with the demo data from `App_Data/EqDemoData.zip`. To reset the data, stop
the app and delete the `eqdemo-sqlite.db*` files.

Try it: open **Load** and pick one of the predefined queries, or build your own: add a few
columns from the Entities panel, add a condition, press **Fetch data**. Then export the result
to Excel.

## Next steps

### Connect the sample to your database

The sample builds its data model from `AppDbContext`, so every `DbSet` becomes an entity in the
Entities panel. It runs the queries as SQL through the same EF Core connection. The React client
doesn't change. To show your database instead of Northwind:

1. **Connection string.** In `appsettings.json`, uncomment the `EqDemoDb` entry under
   `ConnectionStrings` and put your connection string there.
2. **EF Core provider.** In `Startup.cs` (`ConfigureServices`), replace
   `options.UseSqlite(Configuration.GetConnectionString("EqDemoSqLite"))` with the commented
   `options.UseSqlServer(Configuration.GetConnectionString("EqDemoDb"))` line. The SQL Server
   provider is already referenced. For another database, add its EF Core 6 provider package
   (e.g. `Npgsql.EntityFrameworkCore.PostgreSQL`) and use its `Use...` method.

   No DB gate or SQL dialect setting needs to change: while the model comes from a `DbContext`,
   the SQL follows its provider. The commented `.RegisterDbGate<...>()` lines are only for
   [loading the model from the database schema](#load-the-model-directly-from-the-database-schema-uncomment--change).
3. **Your entities.** Replace the Northwind classes in `Models/NWind/` and the `DbSet`s in
   `Data/AppDbContext.cs` with yours. If you don't have the classes yet, generate them from the
   database (the example is for SQL Server):

   ```bash
   dotnet new tool-manifest
   dotnet tool install dotnet-ef --version 6.0.36
   dotnet add package Microsoft.EntityFrameworkCore.Design --version 6.0.1
   dotnet ef dbcontext scaffold "<your connection string>" Microsoft.EntityFrameworkCore.SqlServer --output-dir Models/MyDb --namespace EqDemo.Models --context AppDbContext --context-dir Data --context-namespace EqDemo --no-onconfiguring --force
   ```

   * The project targets .NET 6, so it needs `dotnet-ef` 6. A newer, global `dotnet-ef` stops with
     *Could not load file or assembly 'System.Runtime, Version=8.0.0.0'*. The first two commands
     install version 6 for this folder only.
   * `dotnet add package` puts the version into `../Directory.Packages.props`.
   * The scaffolding builds the project first, and the first build runs `npm install`.

   The last command overwrites `Data/AppDbContext.cs` with a context for your tables. Then delete
   `Models/NWind/`.
4. **Demo seeding.** Remove `app.EnsureDbInitialized(Configuration, env);` at the end of
   `Configure` in `Startup.cs`. After that, `Seed/DbInitializeExtensions.cs`,
   `App_Data/EqDemoData.zip` and the `Korzh.DbUtils.*` packages are no longer used and can go too.
5. **Northwind leftovers.** Delete the predefined queries in `App_Data/dm-nwind/queries`. The
   browser still remembers the last Northwind query, so clear the
   `easyquerycomponent-query` key in its local storage, or open the page in a private window.

Start the app: the Entities panel now lists your tables.

### Add EasyQuery to your own project

These steps bring the advanced search page into an existing ASP.NET Core application with a React
client, for example one made from Visual Studio's *React and ASP.NET Core* template: an ASP.NET
Core server project plus a Vite React app. The server works with its database through EF Core.

1. **Install the server packages** (in the server project):

   ```bash
   dotnet add package Korzh.EasyQuery.AspNetCore
   dotnet add package Korzh.EasyQuery.EntityFrameworkCore.Relational
   dotnet add package Korzh.EasyQuery.DataExport
   dotnet add package EasyData.Exporters.PdfSharp
   dotnet add package EasyData.Exporters.ClosedXML
   ```

   The two `EasyData.Exporters.*` packages add PDF and Excel export; leave them out if you don't
   need it. `Korzh.EasyQuery.DataExport` gives CSV and HTML for Excel. The sample's versions are in
   [`Directory.Packages.props`](../Directory.Packages.props).
2. **Add the server side** in `Program.cs`. Replace `MyDbContext` with your context:

   ```csharp
   using EasyData.Export;
   using Korzh.EasyQuery.Services;
   ...
   builder.Services.AddEasyQuery()
       .UseSqlManager()
       .AddDefaultExporters()
       .AddDataExporter<PdfDataExporter>("pdf")
       .AddDataExporter<ExcelDataExporter>("excel");

   // the PDF exporter needs the non-Unicode code pages
   System.Text.Encoding.RegisterProvider(System.Text.CodePagesEncodingProvider.Instance);
   ...
   var app = builder.Build();
   ...
   app.MapEasyQuery(options => {          // the EasyQuery API at /api/easyquery
       options.DefaultModelId = "mydb";
       options.SaveNewQuery = false;
       options.UseDbContext<MyDbContext>();
       options.UseQueryStore(_ => new FileQueryStore("App_Data"));
   });
   ```

   The sample also registers an "allow all" CORS policy. You need one only if the browser calls
   the API on another origin. With the dev server's proxy (step 4) or with the client served by
   the same app, it isn't needed.
3. **Install the client packages** (in the React app's folder), the same versions as in the
   sample's [`ClientApp/package.json`](ClientApp/package.json):

   ```bash
   npm install @easyquery/core@7.5.1 @easyquery/ui@7.5.1 @easyquery/enterprise@7.5.1 @easydata/core@1.6.0 @easydata/ui@1.6.0
   ```

   Keep `@easyquery/*` in line with the `Korzh.EasyQuery.*` NuGet packages, and `@easydata/*`
   with the `EasyData.*` ones.
4. **Add the UI.**
   * Copy [`EasyQuery.js`](ClientApp/src/components/EasyQuery.js) and
     [`EasyQueryHtml.js`](ClientApp/src/components/EasyQueryHtml.js) into your components. In a
     Vite app, name them `EasyQuery.jsx` and `EasyQueryHtml.jsx`, since Vite compiles JSX only
     in `.jsx` files.
   * In `EasyQuery.jsx`, remove the `.setAppName(...)` and `.useBackend(...)` lines: they tell
     the library that it runs in this sample. Keep `.useEndpoint('/api/easyquery')` in line with
     the server.
   * Import the styles once, in your entry file:
     `import '@easyquery/ui/dist/assets/css/easyquery.ui.css';`. Copy the sample's
     `#EntitiesPanel { user-select: none; }` rule into your CSS too.
   * Show the component on a route, e.g. with React Router 6 or later,
     `<Route path="/easy-query" element={<EasyQuery />} />`.
   * In development, send the API calls to the server through the dev server's proxy. In Vite,
     that's `server: { proxy: { '/api': 'http://localhost:<server port>' } }` in `vite.config.js`.
   * The component has no cleanup for unmounting, and React's `<StrictMode>` mounts every
     component twice in development, so EasyQuery starts twice. Render the page outside
     `<StrictMode>`.
5. **Connect your data.** `options.UseDbContext<...>()` from step 2 is all it takes: every
   `DbSet` of your context becomes an entity, and the queries run through the context's
   connection, in the SQL of its provider. Hide what users shouldn't see with
   `[EqEntity(false)]` on a class or `[EqEntityAttr(false)]` on a property.
6. **Choose where queries are stored.** `FileQueryStore` keeps them as JSON files in
   `App_Data/dm-<model ID>/queries`; the folder is created on the first save. For anything else,
   implement `IQueryStore`.
7. **Set the license key**, as described in [License key](../README.md#license-key). Until there
   is one, the page shows the **No license** dialog instead of the query builder.

More in the docs: [From a sample to your own project](https://korzh.com/easyquery/docs/getting-started/from-sample-to-your-own-project)
and [Getting started for ASP.NET Core + React](https://korzh.com/easyquery/docs/getting-started/asp-net-core/aspnet-core-react).

### Change the sample

#### Move the demo database to SQL Server (uncomment)

To keep the Northwind data but have it in SQL Server, do steps 1 and 2 of
[Connect the sample to your database](#connect-the-sample-to-your-database). The commented
`EqDemoDb` entry has an example LocalDB connection string. Leave the seeding in place: in
`Seed/DbInitializeExtensions.cs`, replace `options.UseSqlite(...)` with the commented
`options.UseSqlServer(...)` line. The demo data is then created in the new database on the
first start.

#### Load the model directly from the database schema (uncomment + change)

In `Startup.cs`, uncomment the `.RegisterDbGate<SqLiteGate>()` line (or `SqlServerGate`, to match
your database), and in `MapEasyQuery(...)` replace `options.UseDbContext<AppDbContext>();` with:

```csharp
options.UseDbConnection<SqliteConnection>(Configuration.GetConnectionString("EqDemoSqLite"));
options.UseDbConnectionModelLoader();
```

Add `using Microsoft.Data.Sqlite;` at the top of the file. Saved queries refer to entities and
attributes by their IDs, and those of a schema-based model may differ, so the predefined queries
may no longer load.

#### Let users save new queries

`options.SaveNewQuery = false` in `MapEasyQuery(...)` means a new query is saved only when the user
presses Save. Set it to `true` to save each new query as soon as it's created.

#### Don't restore the last query

`EasyQuery.js` saves the current query to `localStorage` on every change and loads it on start
(`loadQueryFromLocalStorage()`). Remove the `addEventListener('ready', ...)` block to always start
with an empty query. Or set `loadQueryOnStart: true` and `defaultQueryId: '<query id>'` in
`viewOptions` to open one of the saved queries instead.

#### Change the page behavior

In `viewOptions` of `EasyQuery.js`:

* `widgets.easyGrid.paging.pageSize`: the page size of the result grid;
* `result.showChart`: the chart under the result;
* `serverExporters` and the export links in `EasyQueryHtml.js`: the export formats. `pdf` and
  `excel` must be registered with `.AddDataExporter<...>()` in `Startup.cs`.

#### Update EasyQuery.JS or use pre-release packages

* Change the `@easyquery/*` and `@easydata/*` versions in `ClientApp/package.json` and run
  `npm install`. Keep them in line with the NuGet versions in `../Directory.Packages.props`.
* To get pre-release builds from the Korzh MyGet feed, uncomment the two registry lines in
  `ClientApp/.npmrc`.
