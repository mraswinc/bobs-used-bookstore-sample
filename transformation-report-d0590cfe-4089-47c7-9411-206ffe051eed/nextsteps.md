# Next Steps

## Issues resolved
- Transformed Bookstore.Domain.csproj to net8.0
- Transformed Bookstore.Data.csproj to net8.0
- Transformed Bookstore.Web.csproj to net8.0
- Transformed Bookstore.Cdk.csproj to net8.0
- Transformed Bookstore.Domain.Tests.csproj to net8.0

## Overview

The solution has been transformed with no build errors across all projects:

- `Bookstore.Data`
- `Bookstore.Domain.Tests`
- `Bookstore.Cdk`
- `Bookstore.Web`
- `Bookstore.Domain`

Since no build errors were detected, the focus should be on validating correctness, running tests, and preparing for deployment.

---

## 1. Restore Dependencies

Before running or testing the solution, ensure all NuGet packages are properly restored.

```bash
dotnet restore
```

Review the output for any warnings related to deprecated packages or version conflicts and resolve them if present.

---

## 2. Build the Solution

Perform a full solution build to confirm the error-free state is consistent across all configurations.

```bash
dotnet build --configuration Release
```

Verify that the `Release` configuration builds cleanly, as the transformation may have only been validated under `Debug`.

---

## 3. Run Unit Tests

Execute the test project to validate that domain logic behaves as expected after the transformation.

```bash
dotnet test app/Bookstore.Domain.Tests/Bookstore.Domain.Tests.csproj --configuration Release --verbosity normal
```

- Review test output for any failures or skipped tests.
- If tests were written against legacy .NET Framework behaviors (e.g., specific exception types, serialization behavior, or culture-sensitive formatting), they may require updates to align with cross-platform .NET behavior.

---

## 4. Validate Runtime Behavior of the Web Project

Run the web application locally to confirm it starts and operates correctly.

```bash
dotnet run --project app/Bookstore.Web/Bookstore.Web.csproj --configuration Release
```

Check the following:
- Application starts without runtime exceptions.
- All routes and pages load as expected.
- Database connectivity functions correctly if `Bookstore.Data` involves Entity Framework or another ORM.
- Any configuration values previously stored in `Web.config` have been correctly migrated to `appsettings.json`.

---

## 5. Validate Data Layer

If `Bookstore.Data` uses Entity Framework Core, verify that migrations are up to date and the schema is correct.

```bash
dotnet ef migrations list --project app/Bookstore.Data/Bookstore.Data.csproj
dotnet ef database update --project app/Bookstore.Data/Bookstore.Data.csproj
```

Confirm that:
- All migrations apply cleanly.
- Seed data, if any, is correctly populated.
- Connection strings in `appsettings.json` point to the correct database instance.

---

## 6. Review the CDK Project

Inspect `Bookstore.Cdk` to ensure all infrastructure definitions are accurate for the target environment.

- Confirm that environment-specific values (e.g., region, account IDs, resource names) are correctly configured.
- Run a CDK synthesis to validate the infrastructure output without deploying.

```bash
cdk synth
```

Review the synthesized CloudFormation template for correctness.

---

## 7. Cross-Platform Behavior Checks

Since this project was migrated from legacy .NET Framework, review the following areas that commonly differ in cross-platform .NET:

- **File paths**: Ensure no hardcoded backslash (`\`) path separators exist. Use `Path.Combine` instead.
- **Registry access**: Remove or replace any `Microsoft.Win32.Registry` usage, as it is not supported on Linux/macOS.
- **`System.Drawing`**: If used, replace with a supported cross-platform alternative such as `SkiaSharp` or `ImageSharp`.
- **`HttpContext` and session handling**: Confirm these are correctly configured for ASP.NET Core conventions.
- **Global.asax / OWIN startup**: Verify these have been replaced with the ASP.NET Core `Program.cs` and `Startup.cs` (or minimal API) pattern.

---

## 8. Deploy the Application

Once all validation steps pass, publish the application for deployment.

```bash
dotnet publish app/Bookstore.Web/Bookstore.Web.csproj --configuration Release --output ./publish
```

Verify the contents of the `./publish` directory and deploy to the target environment according to your infrastructure setup defined in `Bookstore.Cdk`.