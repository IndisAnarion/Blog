# Projenin Adımları

## Projenin oluşturulması

```powershell
dotnet new webapi -o Blog
```

## Kurulacak Paketler

```powershell
dotnet add package Microsoft.EntityFrameworkCore.Tools -v 7.0.0
```

```powershell
dotnet add package Microsoft.EntityFrameworkCore.SqlServer -v 7.0.0
```

```powershell
dotnet add package Microsoft.AspNetCore.Identity.EntityFrameworkCore -v 7.0.0
```

## Adımlar

### Connection String ayarlanması

appsettings.json içerisine connectionString aşağıdaki gibi eklenir.(Değişkenlik gösterebilir)

```json
  "ConnectionStrings": {
    "DefaultConnection": "Data Source=172.18.112.1;Initial Catalog=MiddleEarth;Application Name=MiddleEarth;User ID=sa;Password=Sa-252Wer; MultipleActiveResultSets=True;TrustServerCertificate=true;"
  }
```

### Context oluşturma

MiddleEarthContext.cs aşağıdaki gibi oluşturulur.

```csharp
public class MiddleEarthContext : IdentityDbContext
{
    public MiddleEarthContext(DbContextOptions<MiddleEarthContext> options) : base(options)
    {
    }
}
```

### Identity ve veri tabanı ayarlamaları

Program.cs sınıfı içerisinde aşağıdaki kod bloğu eklenir.

```csharp
builder.Services.AddDbContext<MiddleEarthContext>(options =>
    options.UseSqlServer(builder.Configuration.GetConnectionString("DefaultConnection")));

builder.Services.AddIdentity<IdentityUser, IdentityRole>()
    .AddEntityFrameworkStores<MiddleEarthContext>();
```

### Context ayarlamaları

İlk olarak Models klasörü açılır ardından içerisine IdentityUser nesnesinden inherit alan [User.cs](/../Blog/Models/User.cs) eklenir.

```csharp
public class User : IdentityUser<Guid>
{
    // Ek özelliklerinizi buraya ekleyin. Örneğin:
    // public string FullName { get; set; }
}
```

Bir sonraki adımda Context e bu özelleştirmeyi belirtmek için Context tanımı aşağıdaki gibi güncellenir.

```csharp
using Microsoft.AspNetCore.Identity.EntityFrameworkCore;
using Microsoft.EntityFrameworkCore;

public class MiddleEarthContext : IdentityDbContext<User,IdentityRole<Guid>, Guid>
{
    public MiddleEarthContext(DbContextOptions<MiddleEarthContext> options) : base(options)
    {
    }

        protected override void OnModelCreating(ModelBuilder builder)
    {
        base.OnModelCreating(builder);

        // Identity tablolarının adlarını özelleştirme
        builder.Entity<User>().ToTable("Users");
        builder.Entity<IdentityRole<Guid>>().ToTable("Roles");
        builder.Entity<IdentityUserRole<Guid>>().ToTable("UserRoles");
        builder.Entity<IdentityUserClaim<Guid>>().ToTable("UserClaims");
        builder.Entity<IdentityRoleClaim<Guid>>().ToTable("RoleClaims");
        builder.Entity<IdentityUserLogin<Guid>>().ToTable("UserLogins");
        builder.Entity<IdentityUserToken<Guid>>().ToTable("UserTokens");
    }
}
```

Son olarakda Program.cs içinde identity için yaptığımız ayarlar aşağıdaki gibi güncellenir.

```csharp
builder.Services.AddIdentity<User, IdentityRole<Guid>>()
    .AddEntityFrameworkStores<MiddleEarthContext>();
```

### Diğer entiy sınıfları oluşturulur

Models klasörü içerisine entity nesneleri oluşturulur ve Context içerisinde belirtilir.

### Migration oluşturma

Öncelikle yeni bir migration oluşturulur.

```bash
dotnet ef migrations add InitDatabase
```

Ardından migration kontrol edilir. Eğer bir sorun gözlemlenirse migration kaldırılır.

```bash
dotnet ef migrations remove
```

Bir sorun gözlemlenmezse eğer migrationın db ye yansıması için update database işlemi yapılır.

```bash
dotnet ef database update
```
