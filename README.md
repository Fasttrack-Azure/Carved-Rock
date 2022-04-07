# Carved Rock - Docker 
This repo contains some simple code that illustrates .NET applications targeting 
Docker containers.

## Initial Creation Notes
The project was created by using the ASP.NET Web API (.NET 5 - C#) template and including Docker Support.

It also added the `.vscode` folder to support running and debugging the project within VS Code.

No other initial changes were made for the initial commit of the repo.

## Inject ILogger to ProductsController
```
private readonly ILogger _logger;

        public ProductsController(ILogger<ProductsController> logger, IProductLogic productLogic)
        {
            _productLogic = productLogic;
            _logger = logger;
        }
```

## DO the same for QuickOrderController

## Validate logger injection in Domain classes
* ProductLogic.cs
* QuickOrderLogic.cd

## Validate startup.cs for dependency injection
```
services.AddScoped<IProductLogic, ProductLogic>();
services.AddScoped<IQuickOrderLogic, QuickOrderLogic>();
```

## Notice there is no try catch block in the controller logic
* Navigate to the startup.cs file to validate the call to middleware for exception handling 
* Validate the ProductLogic.cs code for the validation logic that throws Application exception or a technical exception

## Add support for Serilog
```
cd .\CarvedRock.Api\
dotnet add package Serilog.AspNetCore --version 5.0.0
dotnet add package Serilog.Enrichers.Environment
```

## Update the main method
```
public static int Main(string[] args)
    {
        Log.Logger = new LoggerConfiguration()
            .MinimumLevel.Override("Microsoft", LogEventLevel.Information)
            .Enrich.FromLogContext()
            .WriteTo.Console()
            .CreateLogger();

        try
        {
            Log.Information("Starting web host");
            CreateHostBuilder(args).Build().Run();
            return 0;
        }
        catch (Exception ex)
        {
            Log.Fatal(ex, "Host terminated unexpectedly");
            return 1;
        }
        finally
        {
            Log.CloseAndFlush();
        }
```

## Code Cleanup
* Add .UseSerilog()
```
 public static IHostBuilder CreateHostBuilder(string[] args) =>
		    // http://bit.ly/aspnet-builder-defaults
            Host.CreateDefaultBuilder(args)
                .UseSerilog()
                .ConfigureWebHostDefaults(webBuilder =>
                {
                    webBuilder.UseStartup<Startup>();
                });
```

* Remove Logging information from appsettings
```
{
  "Logging": {
    "LogLevel": {
      "Default": "Information",
      "Microsoft": "Warning",
      "Microsoft.Hosting.Lifetime": "Information"
    }
  },
  "AllowedHosts": "*"
}
```

## Add log enrichment and custom property
```
 var name = typeof(Program).Assembly.GetName().Name;

            Log.Logger = new LoggerConfiguration()
                .MinimumLevel.Override("Microsoft", LogEventLevel.Information)
                .Enrich.FromLogContext()
                .Enrich.WithMachineName()
                .Enrich.WithProperty("Assembly", name)
                .WriteTo.Console()
                .CreateLogger();
```

## Remove dependency injection from ProductsController
```
//_logger.LogInformation("Starting controller action GetProducts for {category}");
Log.Information("Starting controller action GetProducts for {category}");
```

## Add support for Seq
```
dotnet add package Serilog.Sinks.Seq

.WriteTo.Seq(serverUrl: "http://host.docker.internal:5341")

//Optionally, Avoid including category in the log message text for easier log search
Log.ForContext("Category", category)
    .Information("Starting controller action GetProducts");
```

## Update startup.cs file to exclude health check logs and summarize HTTP requests
```
app.UseCustomRequestLogging();
```
