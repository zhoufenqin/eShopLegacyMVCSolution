# Phase 0: Deep Analysis — eShopLegacyMVC

**Source framework:** ASP.NET MVC 5 + Web API 2 on .NET Framework 4.7.2  
**Target framework:** ASP.NET Core 2.2 on net461 (matching the reference project `eShopPorted`)  
**Reference project:** `eShopPorted/` — a partially-migrated sister project already on ASP.NET Core 2.2

---

## 1. Solution Structure

```
eShopLegacyMVCSolution.sln
├── eShopLegacy.Utilities/          ← shared library, .NET 4.8, legacy .csproj format
│   └── Serializing.cs
├── src/eShopLegacyMVC/             ← main web app, .NET 4.7.2, ASP.NET MVC 5
│   ├── App_Start/
│   ├── Controllers/
│   ├── Models/
│   ├── Modules/
│   ├── Services/
│   ├── ViewModel/
│   ├── Views/
│   ├── Content/   (CSS)
│   ├── Scripts/   (JS)
│   ├── Images/    (brand images)
│   ├── fonts/
│   ├── Pics/      (catalog item images)
│   └── Setup/     (CSV seed files)
└── eShopPorted/                    ← reference: ASP.NET Core 2.2 on net461
```

**Dependency graph:**
- `eShopLegacyMVC` → `eShopLegacy.Utilities` (project reference)
- `eShopPorted` → `eShopLegacy.Utilities` (project reference)

Migration order: `eShopLegacy.Utilities` first, then `eShopLegacyMVC`.

---

## 2. File Inventory

### eShopLegacy.Utilities (shared library)
| File | Change needed |
|---|---|
| `eShopLegacy.Utilities.csproj` | Convert to SDK-style `.csproj`; change target to `net461` |
| `Serializing.cs` | `BinaryFormatter` is deprecated (security risk) but available on net461; no change needed to compile |
| `Properties/AssemblyInfo.cs` | Delete — SDK-style projects generate this automatically |

### src/eShopLegacyMVC (main web project)
| File | Change needed |
|---|---|
| `eShopLegacyMVC.csproj` | Replace with SDK-style `Microsoft.NET.Sdk.Web` project; inline package references |
| `packages.config` | Delete — replaced by `<PackageReference>` items in new `.csproj` |
| `Global.asax` | Delete — replaced by `Program.cs` + `Startup.cs` |
| `Global.asax.cs` | Delete — replaced by `Program.cs` + `Startup.cs` |
| `Web.config` | Delete — replaced by `appsettings.json` |
| `Web.Debug.config` | Delete — replaced by `appsettings.Development.json` |
| `Web.Release.config` | Delete — no longer needed |
| `ApplicationInsights.config` | Delete — Application Insights configured in code/appsettings |
| `log4Net.xml` | Keep — log4net still used |
| `favicon.ico` | Move to `wwwroot/` |
| `App_Start/BundleConfig.cs` | Delete — bundling removed; use direct `<script>`/`<link>` tags |
| `App_Start/FilterConfig.cs` | Delete — error handling via `app.UseExceptionHandler()` in Startup |
| `App_Start/RouteConfig.cs` | Delete — routing via `app.UseMvc()` in Startup |
| `App_Start/WebApiConfig.cs` | Delete — Web API routes merged into MVC routing in Startup |
| `Modules/ApplicationModule.cs` | Update: remove `RegisterControllers`/`RegisterApiControllers` calls; keep service registrations |
| `Models/CatalogBrand.cs` | No change needed — no framework-coupled imports |
| `Models/CatalogType.cs` | No change needed — no framework-coupled imports |
| `Models/CatalogItem.cs` | No change needed — no framework-coupled imports |
| `Models/CatalogDBContext.cs` | Replace EF6 with EF Core: base class, constructor, model builder API |
| `Models/CatalogItemHiLoGenerator.cs` | Update: `db.Database.SqlQuery<T>()` → `db.Set<T>().FromSqlRaw()`; remove `System.Web` using |
| `Models/Infrastructure/CatalogDBInitializer.cs` | Replace `CreateDatabaseIfNotExists` base class; replace EF6 raw SQL APIs; replace `HostingEnvironment` with injected `IWebHostEnvironment`; replace `AppDomain.CurrentDomain.BaseDirectory` |
| `Models/Infrastructure/PreconfiguredData.cs` | No change needed |
| `Services/ICatalogService.cs` | No change needed |
| `Services/CatalogService.cs` | Update using directives: `System.Data.Entity` → `Microsoft.EntityFrameworkCore` |
| `Services/CatalogServiceMock.cs` | No change needed |
| `ViewModel/PaginatedItemsViewModel.cs` | No change needed |
| `Controllers/CatalogController.cs` | Update base class import, `HttpStatusCodeResult` → `BadRequest()`, `HttpNotFound()` → `NotFound()`, `[Bind(Include=)]` → `[Bind()]`, `Request.Url.Scheme` → `Request.Scheme` |
| `Controllers/PicController.cs` | Update base class import; replace `Server.MapPath` with injected `IWebHostEnvironment.WebRootPath` |
| `Controllers/Api/CatalogController.cs` | Update `System.Web.Mvc` → `Microsoft.AspNetCore.Mvc` |
| `Controllers/WebApi/BrandsController.cs` | Replace `ApiController` with `ControllerBase`; replace `IHttpActionResult` → `IActionResult`; replace `HttpResponseMessage` helpers; remove `System.Runtime.Remoting.Messaging` using |
| `Controllers/WebApi/FilesController.cs` | Same as BrandsController |
| `Views/_ViewStart.cshtml` | No change needed |
| `Views/Web.config` | Delete — not needed in ASP.NET Core |
| `Views/Shared/_Layout.cshtml` | Replace `@Styles.Render()`/`@Scripts.Render()` with direct `<link>`/`<script>` tags; replace `HttpContext.Current.Session` with `Context.Session`; add `@using Microsoft.AspNetCore.Http` |
| `Views/Shared/Error.cshtml` | No change needed |
| `Views/Catalog/Index.cshtml` | No change needed |
| `Views/Catalog/CatalogTable.cshtml` | No change needed |
| `Views/Catalog/Create.cshtml` | No change needed |
| `Views/Catalog/Edit.cshtml` | No change needed |
| `Views/Catalog/Delete.cshtml` | No change needed |
| `Views/Catalog/Details.cshtml` | No change needed |
| `Properties/AssemblyInfo.cs` | Delete — SDK-style projects auto-generate |
| `Content/*` | Move to `wwwroot/Content/` |
| `Scripts/*` | Move to `wwwroot/Scripts/` |
| `Images/*` | Move to `wwwroot/Images/` |
| `fonts/*` | Move to `wwwroot/fonts/` |
| `Pics/*` | Move to `wwwroot/Pics/` |
| `Setup/*` | Keep in place (content root relative paths updated) |

**New files to create:**
| File | Purpose |
|---|---|
| `Program.cs` | Modern entry point (`WebHost.CreateDefaultBuilder`) |
| `Startup.cs` | DI registration + middleware pipeline |
| `appsettings.json` | Configuration (connection string, UseMockData, UseCustomizationData) |
| `appsettings.Development.json` | Development overrides |
| `Views/_ViewImports.cshtml` | Shared `@using` and `@addTagHelper` directives |
| `Models/Config/CatalogTypeConfig.cs` | EF Core `IEntityTypeConfiguration<CatalogType>` |
| `Models/Config/CatalogBrandConfig.cs` | EF Core `IEntityTypeConfiguration<CatalogBrand>` |
| `Models/Config/CatalogItemConfig.cs` | EF Core `IEntityTypeConfiguration<CatalogItem>` |

---

## 3. Framework-Coupled API Catalog

### 3.1 Entry Point & Hosting

| Legacy API | Modern Equivalent | File |
|---|---|---|
| `System.Web.HttpApplication` base class | `Program.cs` + `Startup.cs` | `Global.asax.cs` |
| `AreaRegistration.RegisterAllAreas()` | No areas in this app — drop | `Global.asax.cs` |
| `GlobalConfiguration.Configure(WebApiConfig.Register)` | Merged into `app.UseMvc()` | `Global.asax.cs` |
| `FilterConfig.RegisterGlobalFilters(GlobalFilters.Filters)` | `app.UseExceptionHandler("/Home/Error")` | `Global.asax.cs` |
| `RouteConfig.RegisterRoutes(RouteTable.Routes)` | `app.UseMvc(routes => ...)` | `Global.asax.cs` |
| `BundleConfig.RegisterBundles(BundleTable.Bundles)` | Direct `<script>`/`<link>` tags | `Global.asax.cs` |
| `Database.SetInitializer<T>()` | EF Core `HasData()` / `context.Database.Migrate()` | `Global.asax.cs` |
| `HttpContext.Current.Session["key"]` | `HttpContext.Session.GetString("key")` | `Global.asax.cs`, `_Layout.cshtml` |
| `System.Configuration.ConfigurationManager.AppSettings["key"]` | `IConfiguration["key"]` or `.GetValue<bool>("key")` | `Global.asax.cs`, `CatalogDBInitializer.cs` |
| `Trace.CorrelationManager.ActivityId` | No direct equivalent — use middleware | `Global.asax.cs` |

### 3.2 Dependency Injection

| Legacy API | Modern Equivalent | File |
|---|---|---|
| `Autofac.Integration.Mvc.AutofacDependencyResolver` | `Autofac.Extensions.DependencyInjection.AutofacServiceProvider` | `Global.asax.cs` |
| `Autofac.Integration.WebApi.AutofacWebApiDependencyResolver` | Not needed — merged with MVC DI | `Global.asax.cs` |
| `builder.RegisterControllers(assembly)` | `services.AddMvc()` auto-registers controllers | `Modules/ApplicationModule.cs` |
| `builder.RegisterApiControllers(assembly)` | Not needed | `Modules/ApplicationModule.cs` |
| `Autofac.Mvc5` package | `Autofac.Extensions.DependencyInjection` package | `packages.config` |
| `CatalogDBContext` registered per-lifetime in Autofac | `services.AddDbContext<CatalogDBContext>()` in `ConfigureServices` | `Modules/ApplicationModule.cs` |

### 3.3 Data Access (EF6 → EF Core)

| Legacy API | Modern Equivalent | File |
|---|---|---|
| `System.Data.Entity.DbContext` | `Microsoft.EntityFrameworkCore.DbContext` | `CatalogDBContext.cs` |
| `DbContext("name=CatalogDBContext")` constructor | `DbContext(DbContextOptions options)` | `CatalogDBContext.cs` |
| `DbModelBuilder` parameter in `OnModelCreating` | `ModelBuilder` | `CatalogDBContext.cs` |
| `builder.Entity<T>()` inline fluent config | Separate `IEntityTypeConfiguration<T>` classes + `ApplyConfigurationsFromAssembly()` | `CatalogDBContext.cs` |
| `EntityTypeConfiguration<T>` (EF6 class) | `IEntityTypeConfiguration<T>` (EF Core interface) | `CatalogDBContext.cs` |
| `builder.Property(x).HasDatabaseGeneratedOption(DatabaseGeneratedOption.None)` | `builder.Property(x).ValueGeneratedNever()` | `CatalogDBContext.cs` |
| `builder.HasRequired<T>().WithMany().HasForeignKey()` | `builder.HasOne<T>().WithMany().HasForeignKey()` | `CatalogDBContext.cs` |
| `System.Data.Entity.ModelConfiguration` namespace | `Microsoft.EntityFrameworkCore.Metadata.Builders` | `CatalogDBContext.cs` |
| `CreateDatabaseIfNotExists<T>` base class | EF Core `HasData()` seeding in entity configs | `CatalogDBInitializer.cs` |
| `context.Database.SqlQuery<T>("SELECT ...")` | `context.Database.SqlQuery<T>(FormattableString)` or use raw SQL set extension | `CatalogDBInitializer.cs`, `CatalogItemHiLoGenerator.cs` |
| `context.Database.ExecuteSqlCommand(sql)` | `context.Database.ExecuteSqlRaw(sql)` | `CatalogDBInitializer.cs` |
| `System.Data.Entity.EntityState.Modified` | `Microsoft.EntityFrameworkCore.EntityState.Modified` | `CatalogService.cs` |
| `System.Data.Entity` usings (for `Include`, `DbSet`) | `Microsoft.EntityFrameworkCore` | `CatalogService.cs` |

### 3.4 Web Framework (MVC 5 → ASP.NET Core MVC)

| Legacy API | Modern Equivalent | File |
|---|---|---|
| `System.Web.Mvc.Controller` | `Microsoft.AspNetCore.Mvc.Controller` | All MVC controllers |
| `System.Web.Http.ApiController` | `Microsoft.AspNetCore.Mvc.ControllerBase` | `BrandsController.cs`, `FilesController.cs` |
| `System.Web.Mvc.ActionResult` | `Microsoft.AspNetCore.Mvc.ActionResult` | All controllers |
| `IHttpActionResult` | `IActionResult` | `BrandsController.cs` |
| `HttpStatusCodeResult(HttpStatusCode.BadRequest)` | `BadRequest()` | `CatalogController.cs`, `PicController.cs` |
| `HttpNotFound()` | `NotFound()` | `CatalogController.cs`, `PicController.cs` |
| `ResponseMessage(new HttpResponseMessage(HttpStatusCode.NotFound))` | `NotFound()` | `BrandsController.cs` |
| `ResponseMessage(new HttpResponseMessage(HttpStatusCode.OK))` | `Ok()` | `BrandsController.cs` |
| `[Bind(Include = "A,B,C")]` | `[Bind("A,B,C")]` | `CatalogController.cs` |
| `this.Request.Url.Scheme` | `Request.Scheme` | `CatalogController.cs` |
| `Server.MapPath("~/Pics")` | `_env.WebRootPath + "/Pics"` (inject `IWebHostEnvironment`) | `PicController.cs` |
| `System.Web.Mvc.SelectList` | `Microsoft.AspNetCore.Mvc.Rendering.SelectList` | `CatalogController.cs` |
| `System.Web.Mvc.HandleErrorAttribute` | `app.UseExceptionHandler()` middleware | `FilterConfig.cs` |
| `System.Web.Http.HttpConfiguration` | Not needed in ASP.NET Core | `WebApiConfig.cs` |
| `System.Runtime.Remoting.Messaging` | Not available in .NET Core — remove the unused `using` | `BrandsController.cs` |

### 3.5 Bundling & Static Files

| Legacy API | Modern Equivalent | File |
|---|---|---|
| `System.Web.Optimization.BundleCollection` | None — use direct file references | `BundleConfig.cs` |
| `@Styles.Render("~/Content/css")` | `<link rel="stylesheet" href="~/Content/bootstrap.css">` etc. | `_Layout.cshtml` |
| `@Scripts.Render("~/bundles/modernizr")` | `<script src="~/Scripts/modernizr-*.js"></script>` | `_Layout.cshtml` |
| `@Scripts.Render("~/bundles/jquery")` | `<script src="~/Scripts/jquery-*.js"></script>` | `_Layout.cshtml` |
| `@Scripts.Render("~/bundles/bootstrap")` | `<script src="~/Scripts/bootstrap.js"></script>` | `_Layout.cshtml` |
| Static assets in `Content/`, `Scripts/`, `Images/`, `fonts/` | Move to `wwwroot/` subdirectories; enable `app.UseStaticFiles()` | Multiple |

### 3.6 Configuration

| Legacy API | Modern Equivalent | File |
|---|---|---|
| `Web.config` `<connectionStrings>` | `appsettings.json` `"ConnectionStrings"` section | `Web.config` |
| `Web.config` `<appSettings>` (`UseMockData`, `UseCustomizationData`) | `appsettings.json` top-level keys | `Web.config` |
| `ConfigurationManager.AppSettings["UseMockData"]` | `IConfiguration.GetValue<bool>("UseMockData")` | `Global.asax.cs`, `CatalogDBInitializer.cs` |
| `Web.Debug.config` transform | `appsettings.Development.json` | `Web.Debug.config` |

---

## 4. NuGet Package Disposition

### Keep
| Package | Reason |
|---|---|
| `Autofac` | Same Autofac core, works on .NET Core |
| `log4net` | Still used for logging; no change required |
| `Newtonsoft.Json` | Still referenced; keep at 13.0.x (update from 12.0.1) |

### Replace
| Legacy Package | Modern Replacement |
|---|---|
| `Autofac.Mvc5` | `Autofac.Extensions.DependencyInjection 4.4.0` |
| `Autofac.Integration.WebApi` | Not needed separately — merged into MVC DI |
| `EntityFramework 6.2.0` | `Microsoft.EntityFrameworkCore 2.2.6` + `Microsoft.EntityFrameworkCore.SqlServer 2.2.6` |
| `Microsoft.AspNet.Mvc 5.2.7` | `Microsoft.AspNetCore.Mvc 2.2.0` (in SDK) |
| `Microsoft.AspNet.Web.Optimization` | None — direct file references |
| `Microsoft.AspNet.TelemetryCorrelation` | Built into ASP.NET Core |
| `Microsoft.ApplicationInsights.*` | `Microsoft.ApplicationInsights.AspNetCore` (if still needed) |
| `Microsoft.AspNet.SessionState.SessionStateModule` | Built into ASP.NET Core (`services.AddSession()`) |
| `Microsoft.CodeDom.Providers.DotNetCompilerPlatform` | Not needed — Roslyn is the default compiler in SDK projects |

### Drop
| Package | Reason |
|---|---|
| `Antlr` | Only used by WebGrease bundling pipeline — drop with bundling |
| `WebGrease` | Bundling framework — replaced by direct file references |
| `Microsoft.Web.Infrastructure` | ASP.NET 4.x only |
| `Microsoft.Net.Compilers` | Build-time Roslyn; not needed in SDK projects |
| `Microsoft.AspNet.Razor` | Included in `Microsoft.AspNetCore.Mvc` |
| `Microsoft.AspNet.WebPages` | Included in `Microsoft.AspNetCore.Mvc` |
| `bootstrap` (NuGet) | Already in `wwwroot/Content/` and `wwwroot/Scripts/` as static files |
| `jQuery`, `jQuery.Validation` | Already in `wwwroot/Scripts/` as static files |
| `Microsoft.jQuery.Unobtrusive.Validation` | Already in `wwwroot/Scripts/` as static file |
| `Modernizr`, `Respond`, `popper.js` | Already in `wwwroot/Scripts/` as static files |
| `Pipelines.Sockets.Unofficial`, `System.IO.Pipelines`, `System.Buffers`, `System.Memory`, `System.Numerics.Vectors`, `System.Runtime.CompilerServices.Unsafe`, `System.Threading.Channels`, `System.Threading.Tasks.Extensions` | Polyfills no longer needed on net461+ / .NET Core |
| `System.Diagnostics.DiagnosticSource`, `System.Diagnostics.PerformanceCounter` | Built into .NET Core |
| `System.IO.Compression`, `System.IO.Compression.ZipFile` | Built into .NET Core |

---

## 5. Static Assets

All static assets currently reside outside `wwwroot` and must be relocated. The modern ASP.NET Core static files middleware serves from `wwwroot/`.

| Current path | Target path |
|---|---|
| `src/eShopLegacyMVC/Content/` | `wwwroot/Content/` |
| `src/eShopLegacyMVC/Scripts/` | `wwwroot/Scripts/` |
| `src/eShopLegacyMVC/Images/` | `wwwroot/Images/` |
| `src/eShopLegacyMVC/fonts/` | `wwwroot/fonts/` |
| `src/eShopLegacyMVC/Pics/` | `wwwroot/Pics/` |
| `src/eShopLegacyMVC/favicon.ico` | `wwwroot/favicon.ico` |

**View references** to `~/Content/`, `~/Scripts/`, `~/Images/` will remain valid after the move because ASP.NET Core resolves `~/` relative to `wwwroot/`.

---

## 6. Obsolete or Removed APIs

| API | Status | Action |
|---|---|---|
| `System.Runtime.Serialization.Formatters.Binary.BinaryFormatter` | Deprecated (security risk); removed in .NET 7+. Available on net461. | On net461 target, compiles and runs but is a known deserialization vulnerability. Replace with `System.Text.Json` serialization if the target is ever raised above net461. |
| `System.Runtime.Remoting.Messaging.CallContext` | Not available in .NET Core. | The `using` in `BrandsController.cs` must be removed. (The API is imported but not actually called in that file.) |
| `System.Web.Hosting.HostingEnvironment` | Not available in .NET Core. | Replace with injected `IWebHostEnvironment` in `CatalogDBInitializer`. |
| `System.Web.*` namespaces | Not available in .NET Core. | All usings must be replaced with `Microsoft.AspNetCore.*` equivalents. |
| `System.Data.Entity.*` (EF6) | Not available in .NET Core; use `Microsoft.EntityFrameworkCore`. | Replace throughout. |
| `AppDomain.CurrentDomain.BaseDirectory` | Available in .NET Core but semantics differ. | Replace with `IWebHostEnvironment.ContentRootPath` for application-relative paths. |
| `Database.ExecuteSqlCommand()` | Deprecated in EF Core 3+; use `ExecuteSqlRaw()` or `ExecuteSqlInterpolated()`. | Replace in `CatalogDBInitializer.cs`. |
| `Database.SqlQuery<T>()` | Not available in EF Core. | Replace with `FromSqlRaw<T>()` in `CatalogItemHiLoGenerator.cs` and `CatalogDBInitializer.cs`. |

---

## 7. File-by-File Migration Plan

Execute in this order (dependency graph: foundation first, dependents last).

### Step 1 — eShopLegacy.Utilities project file
**File:** `eShopLegacy.Utilities/eShopLegacy.Utilities.csproj`  
**Change:** Replace legacy XML `.csproj` with SDK-style format targeting `net461`. Remove `packages.config` (none needed). Delete `Properties/AssemblyInfo.cs`.

### Step 2 — eShopLegacyMVC project file
**File:** `src/eShopLegacyMVC/eShopLegacyMVC.csproj`  
**Change:** Replace with `<Project Sdk="Microsoft.NET.Sdk.Web">` targeting `net461`. Add package references per the disposition table above. Remove `packages.config` reference.

### Step 3 — Configuration
**New file:** `appsettings.json`  
```json
{
  "ConnectionStrings": {
    "CatalogDBContext": "Data Source=(localdb)\\MSSQLLocalDB;Initial Catalog=Microsoft.eShopOnContainers.Services.CatalogDb;Integrated Security=True;MultipleActiveResultSets=True;"
  },
  "UseMockData": false,
  "UseCustomizationData": false
}
```
> Note: `\\` in the JSON string is the JSON-escaped form of a single `\` backslash at runtime (e.g., `(localdb)\MSSQLLocalDB`).
**New file:** `appsettings.Development.json` — empty or debug overrides.  
**Delete:** `Web.config`, `Web.Debug.config`, `Web.Release.config`, `ApplicationInsights.config`.

### Step 4 — Data Layer: DbContext
**File:** `Models/CatalogDBContext.cs`  
**Changes:**
- `using System.Data.Entity` → `using Microsoft.EntityFrameworkCore`
- Remove `using System.Data.Entity.ModelConfiguration`
- Base class: `DbContext` constructor → `DbContext(DbContextOptions options)`
- `OnModelCreating(DbModelBuilder)` → `OnModelCreating(ModelBuilder)`
- Replace inline fluent config with `builder.ApplyConfigurationsFromAssembly(Assembly.GetExecutingAssembly())`

**New files:** `Models/Config/CatalogTypeConfig.cs`, `Models/Config/CatalogBrandConfig.cs`, `Models/Config/CatalogItemConfig.cs`  
(Translate each `EntityTypeConfiguration<T>` block from the legacy `OnModelCreating` into a separate `IEntityTypeConfiguration<T>` class. See `eShopPorted/Models/Config/` for the exact pattern.)

### Step 5 — Data Layer: CatalogItemHiLoGenerator
**File:** `Models/CatalogItemHiLoGenerator.cs`  
**Changes:**
- Remove `using System.Web`
- `db.Database.SqlQuery<Int64>("SELECT NEXT VALUE FOR catalog_hilo;")` → use a raw ADO.NET scalar query via `db.Database.GetDbConnection()`:
  ```csharp
  var conn = db.Database.GetDbConnection();
  conn.Open();
  using var cmd = conn.CreateCommand();
  cmd.CommandText = "SELECT NEXT VALUE FOR catalog_hilo;";
  sequenceId = (int)(long)cmd.ExecuteScalar();
  ```

### Step 6 — Data Layer: CatalogDBInitializer
**File:** `Models/Infrastructure/CatalogDBInitializer.cs`  
**Changes:**
- Remove base class `CreateDatabaseIfNotExists<CatalogDBContext>` — this class is no longer an EF initializer
- Inject `IWebHostEnvironment` instead of using `HostingEnvironment.ApplicationPhysicalPath`
- Replace `AppDomain.CurrentDomain.BaseDirectory` with `IWebHostEnvironment.ContentRootPath`
- `context.Database.ExecuteSqlCommand(sql)` → `context.Database.ExecuteSqlRaw(sql)`
- `context.Database.SqlQuery<Int64>(sql)` → raw ADO.NET scalar query via `context.Database.GetDbConnection()`
- Remove `using System.Web`, `using System.Web.Hosting`
- Add `using Microsoft.AspNetCore.Hosting`

### Step 7 — Service Layer: CatalogService
**File:** `Services/CatalogService.cs`  
**Changes:**
- `using System.Data.Entity` → `using Microsoft.EntityFrameworkCore`
- `EntityState.Modified` → `Microsoft.EntityFrameworkCore.EntityState.Modified`

### Step 8 — Controllers: CatalogController
**File:** `Controllers/CatalogController.cs`  
**Changes:**
- `using System.Web.Mvc` → `using Microsoft.AspNetCore.Mvc`, `using Microsoft.AspNetCore.Mvc.Rendering`
- `new HttpStatusCodeResult(HttpStatusCode.BadRequest)` → `BadRequest()`
- `HttpNotFound()` → `NotFound()`
- `[Bind(Include = "...")]` → `[Bind("...")]`
- `this.Request.Url.Scheme` → `Request.Scheme`
- Remove `using System.Net`

### Step 9 — Controllers: PicController
**File:** `Controllers/PicController.cs`  
**Changes:**
- `using System.Web.Mvc` → `using Microsoft.AspNetCore.Mvc`, `using Microsoft.AspNetCore.Hosting`
- Add constructor parameter `IWebHostEnvironment env`
- `Server.MapPath("~/Pics")` → `Path.Combine(_env.WebRootPath, "Pics")`
- `new HttpStatusCodeResult(HttpStatusCode.BadRequest)` → `BadRequest()`
- `HttpNotFound()` → `NotFound()`

### Step 10 — Controllers: Api/CatalogController
**File:** `Controllers/Api/CatalogController.cs`  
**Changes:**
- `using System.Web.Mvc` → `using Microsoft.AspNetCore.Mvc`

### Step 11 — Controllers: WebApi/BrandsController
**File:** `Controllers/WebApi/BrandsController.cs`  
**Changes:**
- `using System.Web.Http` → `using Microsoft.AspNetCore.Mvc`
- Remove `using System.Runtime.Remoting.Messaging` (unused, not available in .NET Core)
- Remove `using System.Net`, `using System.Net.Http`
- `ApiController` → `ControllerBase`
- `IHttpActionResult` → `IActionResult`
- `ResponseMessage(new HttpResponseMessage(HttpStatusCode.NotFound))` → `NotFound()`
- `ResponseMessage(new HttpResponseMessage(HttpStatusCode.OK))` → `Ok()`

### Step 12 — Controllers: WebApi/FilesController
**File:** `Controllers/WebApi/FilesController.cs`  
**Changes:**
- `using System.Web.Http` → `using Microsoft.AspNetCore.Mvc`
- Remove `using System.Net`, `using System.Net.Http`
- `ApiController` → `ControllerBase`
- `HttpResponseMessage` return type → `IActionResult`
- Return `File(serializer.SerializeBinary(brands), "application/octet-stream")`

### Step 13 — DI Module
**File:** `Modules/ApplicationModule.cs`  
**Changes:**
- Remove `builder.RegisterControllers(thisAssembly)` and `builder.RegisterApiControllers(thisAssembly)` calls (not valid; controllers are auto-registered by ASP.NET Core)
- Keep all service registrations unchanged
- Remove `CatalogDBContext` registration (moved to `services.AddDbContext<>()` in Startup)

### Step 14 — Views: Layout
**File:** `Views/Shared/_Layout.cshtml`  
**Changes:**
- Replace `@Styles.Render("~/Content/css")` with:
  ```html
  <link rel="stylesheet" href="~/Content/bootstrap.css" />
  <link rel="stylesheet" href="~/Content/custom.css" />
  <link rel="stylesheet" href="~/Content/base.css" />
  <link rel="stylesheet" href="~/Content/Site.css" />
  ```
- Replace `@Scripts.Render("~/bundles/modernizr")` with `<script src="~/Scripts/modernizr-2.6.2.js"></script>`
- Replace `@Scripts.Render("~/bundles/jquery")` with `<script src="~/Scripts/jquery-3.3.1.js"></script>`
- Replace `@Scripts.Render("~/bundles/bootstrap")` with:
  ```html
  <script src="~/Scripts/bootstrap.js"></script>
  <script src="~/Scripts/respond.js"></script>
  ```
- Replace `HttpContext.Current.Session["MachineName"]` with `Context.Session.GetString("MachineName")`
- Replace `HttpContext.Current.Session["SessionStartTime"]` with `Context.Session.GetString("SessionStartTime")`
- Add `@using Microsoft.AspNetCore.Http` at top (or via `_ViewImports.cshtml`)

### Step 15 — Views: _ViewImports
**New file:** `Views/_ViewImports.cshtml`  
```cshtml
@using eShopLegacyMVC
@using eShopLegacyMVC.Models
@using eShopLegacyMVC.ViewModel
@using Microsoft.AspNetCore.Http
@addTagHelper *, Microsoft.AspNetCore.Mvc.TagHelpers
```

### Step 16 — Static Assets
Move (not copy) entire directories:
- `Content/` → `wwwroot/Content/`
- `Scripts/` → `wwwroot/Scripts/`
- `Images/` → `wwwroot/Images/`
- `fonts/` → `wwwroot/fonts/`
- `Pics/` → `wwwroot/Pics/`
- `favicon.ico` → `wwwroot/favicon.ico`

### Step 17 — Entry Point
**New file:** `Program.cs` — create `WebHost.CreateDefaultBuilder(args).UseStartup<Startup>().Build().Run()`  
**New file:** `Startup.cs` — configure services and middleware:
- `services.AddMvc()` 
- `services.AddSession()` + session middleware
- `services.AddDbContext<CatalogDBContext>()` when not using mock data
- Autofac `ContainerBuilder` with `ApplicationModule`
- `app.UseStaticFiles()`
- `app.UseSession()`
- `app.UseMvc()` with default route `{controller=Catalog}/{action=Index}/{id?}`
- `app.UseExceptionHandler("/Catalog/Error")` in non-development

### Step 18 — Delete Legacy Files
Delete in order (after build passes):
1. `Global.asax`, `Global.asax.cs`
2. `Web.config`, `Web.Debug.config`, `Web.Release.config`
3. `ApplicationInsights.config`
4. `App_Start/BundleConfig.cs`, `App_Start/FilterConfig.cs`, `App_Start/RouteConfig.cs`, `App_Start/WebApiConfig.cs`
5. `Properties/AssemblyInfo.cs`
6. `packages.config`
7. `Views/Web.config`

---

## 8. Eval Criteria Checklist

```
[ ] Phase 0: Analysis complete (this document)
[ ] Phase 1: Project files converted to SDK-style; dotnet restore succeeds
[ ] Phase 2: CatalogDBContext, entity configs, CatalogDBInitializer, CatalogItemHiLoGenerator compile
[ ] Phase 3: CatalogService, CatalogServiceMock, ICatalogService compile with only using-directive changes
[ ] Phase 4: All controllers compile; no System.Web references remain
[ ] Phase 5: Views compile; static assets in wwwroot; layout renders without bundling helpers
[ ] Phase 6: Program.cs + Startup.cs created; app starts without exceptions
[ ] Phase 7: All legacy files deleted; build still succeeds
[ ] Phase 8: dotnet clean && dotnet build → 0 errors; app starts; GET / returns HTTP 200
```
