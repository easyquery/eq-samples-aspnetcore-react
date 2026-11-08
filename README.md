# EasyQuery samples for ASP.NET Core + React

[![NuGet](https://img.shields.io/nuget/v/Korzh.EasyQuery.AspNetCore)](https://www.nuget.org/packages/Korzh.EasyQuery.AspNetCore) [![npm](https://img.shields.io/npm/v/@easyquery/ui)](https://www.npmjs.com/package/@easyquery/ui) [![Build](https://github.com/easyquery/eq-samples-aspnetcore-react/actions/workflows/build.yml/badge.svg)](https://github.com/easyquery/eq-samples-aspnetcore-react/actions/workflows/build.yml) [![License: MIT](https://img.shields.io/badge/license-MIT-blue)](LICENSE)

[EasyQuery](https://korzh.com/easyquery) adds a query builder, ad-hoc reporting and data filtering
to .NET applications. The samples in this repository show it on ASP.NET Core + React.

## Samples

| Sample | Scenario | Stack |
|---|---|---|
| [AdvancedSearch](AdvancedSearch) | Advanced search | ASP.NET Core 10 · React 16 · C#, JavaScript |

* **Advanced search.** A full-page query builder: users pick the columns, combine any conditions they need, run the query and work with the result — sorting, paging and export.

## Quick start

1. Get the code: [download the ZIP](https://github.com/easyquery/eq-samples-aspnetcore-react/archive/refs/heads/main.zip),
   or `git clone https://github.com/easyquery/eq-samples-aspnetcore-react.git`.
2. Open `Samples.React.sln` in Visual Studio 2022 (17.2 or later).
3. Set the sample you want as the startup project and run it. Or, from the command line, `cd` into a sample folder and run `dotnet run`; each sample's README has the exact steps.

Every sample creates and fills its own demo database on the first start. That can take a minute
or two; later starts take a few seconds.

## Package versions

All NuGet package versions of this repository, EasyQuery and EasyData included, are set in one
file: [`Directory.Packages.props`](Directory.Packages.props) (NuGet central package
management, which needs Visual Studio 2022 17.2+ or the .NET 6.0.300+ SDK). To try another
EasyQuery version, change it there. The npm packages (`@easyquery/*`, `@easydata/*`) are in the
`package.json` of the client app; keep them in line with the NuGet ones.

## License key

You don't need one to start. On the first run the sample shows a **No license** dialog: enter your
email, then the code we send to it, and you get a free 30-day trial key on the spot. The key is saved to `EasyQuery.lck`; restart the sample to apply it.

Already have a key? Put it into `appsettings.json` of the app that serves the EasyQuery API and the dialog won't appear:

```json
"EasyQuery": {
  "LicenseKey": "your key goes here"
}
```

Ready to buy? See the [licensing options on korzh.com](https://korzh.com/easyquery?utm_source=github&utm_medium=sample&utm_campaign=eq-samples-aspnetcore-react#asp-core-edition).

## Next steps

### Need a complete reporting application, not a page?

The samples show one EasyQuery page at a time. If what you're after is a whole self-service
reporting portal, start from [EasyReport Starter Kit (ERSK)](https://korzh.com/easy-report-starter-kit?utm_source=github&utm_medium=sample&utm_campaign=eq-samples-aspnetcore-react)
instead. It's a ready-made ASP.NET Core solution built on EasyQuery, with its full source code
and a Clean Architecture layout, that you rebrand and extend:

* users create, edit and share reports themselves, in the visual query builder;
* results as tables, charts and pivot tables, with grouping and totals;
* user management with roles and guest access;
* several database connections (SQL Server, MySQL, PostgreSQL, Oracle), multi-tenancy, and
  scheduled reports sent by email as Excel, PDF or CSV (Pro edition).

See it in action at [easy.report](https://easy.report), the live demo of ERSK.

### Connect a sample to your database

The sample runs on its own demo database. Here is how you can connect it to your own data
instead: keep the sample as it is and swap the database under it. The ASP.NET Core part reaches
the data through EF Core — an `AppDbContext` with the Northwind entities in `Models/NWind/` — and
builds the EasyQuery data model from it, so the change is about that `DbContext` and its
connection. The React client stays as it is:

1. **Point the connection string at your database** in `appsettings.json`.
2. **Switch the EF Core provider** (`options.UseSqlite(...)` in `Startup.cs`) to yours. The
   SQL Server line is already there, commented out; for another database add its EF Core
   provider package.
3. **Replace the demo `DbContext`** with yours: your entity classes instead of `Models/NWind/`,
   your `DbSet`s in `Data/AppDbContext.cs`. No classes yet? `dotnet ef dbcontext scaffold`
   generates them from the database.
4. **Turn off the demo seeding**: the `EnsureDbInitialized(...)` call at the end of `Startup.cs`.
   It creates the Northwind tables and fills them on the first start.
5. **Remove what refers to Northwind**: the predefined queries in `App_Data/dm-nwind`.

The model is built from the whole `DbContext`, so all its entities show up in the query builder
at once. The sample's own steps:

| Sample | Data model | Steps |
|---|---|---|
| Advanced search | Whole `DbContext`, SQL | [AdvancedSearch](AdvancedSearch/README.md#connect-the-sample-to-your-database) |

### Add EasyQuery to your own project

To get the same page into an application you already have, repeat what the sample does, step by
step:

1. **Install the packages.** On the server: `Korzh.EasyQuery.AspNetCore` and
   `Korzh.EasyQuery.EntityFrameworkCore.Relational` (the SQL engine), and, if users should
   export the result, the exporters: `Korzh.EasyQuery.DataExport` (CSV and HTML for Excel),
   `EasyData.Exporters.ClosedXML` (Excel files), `EasyData.Exporters.PdfSharp` (PDF). In the
   React app: `@easyquery/core`, `@easyquery/ui`, `@easyquery/enterprise`, `@easydata/core` and
   `@easydata/ui`.
2. **Add the server side**: `AddEasyQuery()` in the services and `MapEasyQuery(...)` in the
   request pipeline.
3. **Add the UI**: copy the React component that holds the EasyQuery widgets, with its markup
   and styles, and give it a route.
4. **Connect your data**: tell EasyQuery your `DbContext`. The queries then run through its
   connection, in the SQL of its provider.
5. **Choose where queries are stored**: files in `App_Data`, as in the sample, or your own
   `IQueryStore`.
6. **Set the license key**, as described in [License key](#license-key). Until there is one,
   the page shows the **No license** dialog instead of the EasyQuery widgets.

The exact packages, code and files:

| Sample | What you copy | Steps |
|---|---|---|
| Advanced search | The `MapEasyQuery` settings, the `EasyQuery` component and its markup | [AdvancedSearch](AdvancedSearch/README.md#add-easyquery-to-your-own-project) |

For the details behind each step, see [From a sample to your own project](https://korzh.com/easyquery/docs/getting-started/from-sample-to-your-own-project)
and [Getting started for ASP.NET Core + React](https://korzh.com/easyquery/docs/getting-started/asp-net-core/aspnet-core-react)
in the documentation.

## Other platforms

* [ASP.NET Core Razor Pages and MVC](https://github.com/easyquery/eq-samples-aspnetcore-razor-mvc) — 6 samples
* [ASP.NET Core + Angular](https://github.com/easyquery/eq-samples-aspnetcore-angular) — 2 samples
* [ASP.NET Core + Blazor](https://github.com/easyquery/eq-samples-aspnetcore-blazor) — 2 samples
* [ASP.NET Core + Vue](https://github.com/easyquery/eq-samples-aspnetcore-vue) — 2 samples
* [ASP.NET Core + Stencil](https://github.com/easyquery/eq-samples-aspnetcore-stencil) — 1 sample
* [ASP.NET MVC 5 (.NET Framework)](https://github.com/easyquery/eq-samples-aspnet4-mvc) — 5 samples
* [ASP.NET Web Forms (.NET Framework)](https://github.com/easyquery/eq-samples-aspnet4-webforms) — 2 samples
* [Windows Forms](https://github.com/easyquery/eq-samples-winforms) — 3 samples
* [WPF](https://github.com/easyquery/eq-samples-wpf) — 2 samples

## Links

* [EasyQuery home page](https://korzh.com/easyquery)
* [Documentation](https://korzh.com/easyquery/docs)
* [Live demo](https://korzh.com/demo)
* [All EasyQuery samples](https://github.com/easyquery/DotNetSamples)
