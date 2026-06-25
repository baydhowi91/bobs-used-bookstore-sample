# Next Steps

## Issues resolved
- Transformed Bookstore.Domain.csproj to net8.0
- Transformed Bookstore.Data.csproj to net8.0
- Transformed Bookstore.Web.csproj to net8.0
- Transformed Bookstore.Cdk.csproj to net8.0
- Transformed Bookstore.Domain.Tests.csproj to net8.0

## Summary

The transformation appears to have completed successfully. No build errors were detected across any of the projects in the solution:

- `Bookstore.Data`
- `Bookstore.Domain.Tests`
- `Bookstore.Cdk`
- `Bookstore.Web`
- `Bookstore.Domain`

The following steps outline how to validate, test, and deploy the migrated solution.

---

## 1. Restore Dependencies

Run a full NuGet package restore to ensure all dependencies are resolved correctly in the new target framework:

```bash
dotnet restore
```

Review the output for any warnings related to package compatibility or deprecated packages that may need to be updated.

---

## 2. Build the Solution

Perform a full solution build to confirm there are no compilation issues:

```bash
dotnet build --configuration Release
```

Address any warnings that surface during the build, particularly those related to nullable reference types or obsolete APIs, as these can indicate areas that may cause runtime issues.

---

## 3. Run the Unit Tests

Execute the test project to verify that existing functionality behaves as expected under the new framework:

```bash
dotnet test app/Bookstore.Domain.Tests/Bookstore.Domain.Tests.csproj --configuration Release --verbosity normal
```

Review the test results carefully. Any failing tests should be investigated before proceeding. If tests were previously passing on the legacy framework, failures here indicate regressions introduced during migration.

---

## 4. Verify Runtime Behavior of the Web Project

Run the web application locally to confirm it starts and operates correctly:

```bash
dotnet run --project app/Bookstore.Web/Bookstore.Web.csproj --configuration Release
```

Manually verify the following:
- Application starts without exceptions.
- Key pages and routes load correctly.
- Any database connections defined in `Bookstore.Data` are functioning (check connection strings in `appsettings.json` or equivalent configuration files).
- Authentication and authorization flows work as expected, if applicable.

---

## 5. Validate the Data Layer

Confirm that `Bookstore.Data` is operating correctly against the target database:

- If the project uses Entity Framework Core, verify that migrations are up to date:

```bash
dotnet ef migrations list --project app/Bookstore.Data/Bookstore.Data.csproj --startup-project app/Bookstore.Web/Bookstore.Web.csproj
```

- If migrations are missing or out of sync, generate a new migration to capture any model changes:

```bash
dotnet ef migrations add PostMigration --project app/Bookstore.Data/Bookstore.Data.csproj --startup-project app/Bookstore.Web/Bookstore.Web.csproj
```

- Apply pending migrations to the target database:

```bash
dotnet ef database update --project app/Bookstore.Data/Bookstore.Data.csproj --startup-project app/Bookstore.Web/Bookstore.Web.csproj
```

---

## 6. Review the CDK Project

The `Bookstore.Cdk` project likely defines infrastructure. Review its contents to confirm:

- Any target environment references (region, account, resource names) are correctly configured for the intended deployment target.
- The CDK version being used is compatible with the current AWS CDK library NuGet packages.

Synthesize the CDK stack to validate the infrastructure definition:

```bash
dotnet run --project app/Bookstore.Cdk/Bookstore.Cdk.csproj
```

Or if using the CDK CLI directly:

```bash
cdk synth --app "dotnet run --project app/Bookstore.Cdk/Bookstore.Cdk.csproj"
```

Review the synthesized CloudFormation template for correctness before deploying.

---

## 7. Deploy the Application

Once all validation steps above pass, deploy using the CDK project:

```bash
cdk deploy --app "dotnet run --project app/Bookstore.Cdk/Bookstore.Cdk.csproj"
```

After deployment, perform a smoke test against the deployed environment to confirm the application is functioning as expected in the target infrastructure.