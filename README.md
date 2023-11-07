# code
BlogPost.cs
---------------
namespace CodePulse.API.Models.Domain
{
    public class BlogPost
    {
        public Guid Id { get; set; }
        public string Title { get; set; }
        public string ShortDescription { get; set; }
        public string Content { get; set; }
        public string FeaturedImageUrl { get; set; }
        public string UrlHandle { get; set; }
        public DateTime PublishedDate { get; set; }
        public string Author { get; set; }
        public bool IsVisible { get; set; }

    }
}

Category.cs
--------------
﻿namespace CodePulse.API.Models.Domain
{
    public class Category
    {
        public Guid Id { get; set; }
        public string Name { get; set; }
        public string UrlHandle { get; set; }

    }
}

ApplicationDbContext.cs
----------------------------

﻿using CodePulse.API.Models.Domain;
using Microsoft.EntityFrameworkCore;

namespace CodePulse.API.Data
{
    public class ApplicationDbContext : DbContext
    {
        public ApplicationDbContext(DbContextOptions<ApplicationDbContext> options) : base(options)
        {
        }

        public DbSet<BlogPost> BlogPosts { get; set; }
        public DbSet<Category> Categories { get; set; }
        
    }
}

appsettings.json
------------------
{
  "Logging": {
    "LogLevel": {
      "Default": "Information",
      "Microsoft.AspNetCore": "Warning"
    }
  },
  "AllowedHosts": "*",
  "ConnectionStrings": {
    "CodePulseConnectionString": "Server=localhost\\MSSQLSERVER01;Database=CodePulseDb;TrustServerCertificate=True;Trusted_Connection=True"
  }
}

program.cs
------------

builder.Services.AddDbContext<ApplicationDbContext>(options =>
{
    options.UseSqlServer(builder.Configuration.GetConnectionString("CodePulseConnectionString"));
});

var app = builder.Build();

CategoriesController.cs
------------------------

using Microsoft.AspNetCore.Http;
using Microsoft.AspNetCore.Mvc;

namespace CodePulse.API.Controllers
{
    // https://localhost:xxxx/api/categories
    [Route("api/[controller]")]
    [ApiController]

    public class CategoriesController : ControllerBase
    {
    }



 CreateCategoryRequestDto.cs
---------------------------------
﻿namespace CodePulse.API.Models.DTO
{
    public class CreateCategoryRequestDto
    {
        public string Name { get; set; }
        public string UrlHandle { get; set; }
    }
}

CategoryDto.cs
----------------
﻿namespace CodePulse.API.Models.DTO
{
    public class CategoryDto
    {
        public Guid Id { get; set; }
        public string Name { get; set; }
        public string UrlHandle { get; set; }
    }
}
CategoriesController.cs
------------------------

using Microsoft.AspNetCore.Http;
using Microsoft.AspNetCore.Mvc;

namespace CodePulse.API.Controllers
{
    // https://localhost:xxxx/api/categories
    [Route("api/[controller]")]
    [ApiController]

    public class CategoriesController : ControllerBase
    {	private readonly ApplicationDbContext dbContext;
	
	public CategoriesController(ApplicationDbContext dbContext)
	{
	     this.dbContext= dbContext;
	}
    

[HttpPost]
        public async Task<IActionResult> CreateCategory(CreateCategoryRequestDto request)
        {  // include breakpoint here
            // Map DTO to Domain Model
            var category = new Category
            {
                Name = request.Name,
                UrlHandle = request.UrlHandle
            };

            await dbContext.Categories.AddAsync(category);
  	    await dbContext.SaveChangesAsync();

            // Domain model to DTO
            var response = new CategoryDto
            {
                Id = category.Id,
                Name = category.Name,
                UrlHandle = category.UrlHandle
            };

            return Ok(response);
		}
	}
}

-> run the application


ICategoryRepository.cs
------------------------
﻿using CodePulse.API.Models.Domain;

namespace CodePulse.API.Repositories.Interface
{
    public interface ICategoryRepository
    {
        Task<Category> CreateAsync(Category category);
	}
}


CategoryRepository.cs
----------------------
﻿using CodePulse.API.Data;
using CodePulse.API.Models.Domain;
using CodePulse.API.Repositories.Interface;
using Microsoft.EntityFrameworkCore;

namespace CodePulse.API.Repositories.Implementation
{
    public class CategoryRepository : ICategoryRepository
    {
        private readonly ApplicationDbContext dbContext;
	
	public CategoryRepository(ApplicationDbContext dbContext)
        {
            this.dbContext = dbContext;
        }

        public async Task<Category> CreateAsync(Category category)
        {
		           await dbContext.Categories.AddAsync(category);
  	    await dbContext.SaveChangesAsync();
	    
            return category;
           
        }

program.cs
-----------

builder.Services.AddDbContext<ApplicationDbContext>(options =>
{
    options.UseSqlServer(builder.Configuration.GetConnectionString("CodePulseConnectionString"));
});

builder.Services.AddScoped<ICategoryRepository, CategoryRepository>();

CategoriesController.cs
-------------------------

  public class CategoriesController : ControllerBase
    { private readonly ICategoryRepository categoryRepository;
	
	public CategoriesController(ICategoryRepository categoryRepository)
	{
	     this.categoryRepository= categoryRepository;
	}

[HttpPost]
        public async Task<IActionResult> CreateCategory(CreateCategoryRequestDto request)
        {  // include breakpoint here
            // Map DTO to Domain Model
            var category = new Category
            {
                Name = request.Name,
                UrlHandle = request.UrlHandle
            };
             await categoryRepository.CreateAsync(category);

