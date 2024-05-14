# InsideAirBnB Database met Entity Framework Core DB First

## Database opzetten

1. De database voor de demo is InsideAirBNB-Paris-2024.bacpac van onderwijs online
2. Importeer de bacpak bestanden in SQL Server Management Studio

## Maak een nieuwe .Net Core Class Library

1. Open Visual Studio.
2. Ga naar het menu Bestand, wijs Nieuw aan en klik vervolgens op Project. Het dialoogvenster Nieuw project wordt geopend.
3. Aan de linkerkant van het dialoogvenster selecteer je Geïnstalleerd -> Visual C# -> .NET Core.
4. Aan de rechterkant van het dialoogvenster selecteer je Klassenbibliotheek (.NET Core).
5. Selecteer het pad om het project op te slaan en voer de projectnaam in als "EfCoreDBFirst".
6. Klik op OK. Er wordt een nieuw project aangemaakt.

## Installeer NuGet-pakketten

Installeer de vereiste NuGet-pakketten om modellen te maken van een bestaande SQL Server-database. Klik op het menu Tools in Visual Studio -> NuGet Package Manager -> Package Manager Console. Voer de onderstaande commando's uit om de pakketten te installeren:

```bash
Install-package Microsoft.EntityFrameworkCore
Install-package Microsoft.EntityFrameworkCore.SqlServer
Install-package Microsoft.EntityFrameworkCore.Tools
Install-package Microsoft.EntityFrameworkCore.Design
```
# Modellen maken met Scaffold-DbContext

Maak een nieuwe map om de Entiteit en DBContext-map te plaatsen met de naam Models.

Voer op de Terminal uit op de library:

```bash
dotnet ef dbcontext scaffold "Server=localhost;Database=InsideAirBNB-Paris-2024;Trusted_Connection=True;MultipleActiveResultSets=true;Encrypt=False;TrustServerCertificate=true;" Microsoft.EntityFrameworkCore.SqlServer -o Models
```

Nu zien we onder de map Models nieuwe klassen staan, de InsideAirBnbParis2024Context klasse die erft van DbContext. Deze heeft een instantie van DbContextOptions die informatie over de configuratie bevat.

## Maak een nieuwe Web API aan

1. Ga naar solution explorer en selecteer "Add" op de solution en klik op "New Project". Het dialoogvenster "Nieuw project" wordt geopend.
2. Aan de rechterkant van het dialoogvenster selecteer je "ASP.NET Core Web Api".
3. Geef je project de naam "EfCoreDBFirst_Api" en selecteer de locatie waar je het wilt opslaan.
4. Klik op "Create".

# configurate DBContext


Voeg in de server "EfCoreDBFirst_Api" die je hebt aangemaakt via nuget packet NuGet Package Manager -> Package Manager Console

```bash
Install-package Microsoft.EntityFrameworkCore.SqlServer
```

Voeg daarna in de program.cs de volgende regel toe 

```bash

builder.Services.AddDbContext<InsideAirBnbParis2024Context>(options =>
    options.UseSqlServer(builder.Configuration.GetConnectionString("DefaultConnection")));

```

En in de appsettings in de server de volgende regel

    "ConnectionStrings": {  
      "DefaultConnection": "Server=localhost;Database=InsideAirBNB-Paris-2024;Trusted_Connection=True;MultipleActiveResultSets=true;Encrypt=False;TrustServerCertificate=true;",
    }

Voeg een referentie toe in je web api naar EfCoreDBFirst door op rechtermuis te klikken op dependencies.

# Maak een controller om te testen

Voeg in je server een map toe met de naam controller en voeg een empty API controller toe en voeg in de klass de volgende code toe

      private InsideAirBnbParis2024Context _context;
    
      public ValuesController(InsideAirBnbParis2024Context context)
      {
          _context = context;
      }
      [HttpGet]
      public async Task<ActionResult<List<Listing>>> GetAllListings()
      {
          var listings = await _context.Listings.ToListAsync();
          return Ok(listings);
    
      }


# Update de scaffold

Voor update gebruik de flag --force om te update. Onthoudt dat alles overschreden wordt dus verwijder opnieuw in de context de Onconfiguration met de database connectionstring

dotnet ef dbcontext scaffold "Server=localhost;Database=InsideAirBNB-Paris-2024;Trusted_Connection=True;MultipleActiveResultSets=true;Encrypt=False;TrustServerCertificate=true;" Microsoft.EntityFrameworkCore.SqlServer -o Models --force

# Extra: maake gebruik van de service met LINQ


voeg een nieuwe map toe in EfCoreDBFirst met de naam Services en voeg een Interface toe genaamd IListingService


    {
        public interface IListingService
        {
            Task<List<Listing>> GetAllListingsAsync();
        }
    }

Voeg daarna een nieuwe klas toe genaamd ListingsService:



    public class ListingService : IListingService
    {
        private readonly InsideAirBnbParis2024Context _context;
        public ListingService(InsideAirBnbParis2024Context context)
        {
            _context = context;
        }

        public async Task<List<Listing>> GetAllListingsAsync()
        {
            List<Listing> listings = await _context.Listings
                .Select(l => new Listing
                {
                    Id = l.Id,
                    Longitude = l.Longitude,
                    Latitude = l.Latitude
                })
                .ToListAsync();

            return listings;
        }
    }

Voeg vervolgend de Service toe aan de program.cs

    
    builder.Services.AddScoped<IListingService, ListingService>();
    



