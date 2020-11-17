---
layout: post
title: Important tips while working with Microsoft Blazor
description: Working with Blazor, EF Core, please see some important tips
image: 
---

## Some command line (if change model before dotnet run; need to remove all from Migrations first and delete changed tables or the whold database)
### Add migrations, change model
```console
add-migration Initial
```

### Reverse Engineer (no needs in this project), it will generate classes and DbContext for tables in Ms. SQL Server
```console
Scaffold-DbContext "Server=.\;Database=db;Trusted_Connection=True;" Microsoft.EntityFrameworkCore.SqlServer -OutputDir Entities
```

### App.razor
```console
<CascadingAuthenticationState>
    <Router AppAssembly="@typeof(Program).Assembly">
        <Found Context="routeData">
            <!--
                <RouteView RouteData="@routeData" DefaultLayout="@typeof(MainLayout)" />
            -->
            <AuthorizeRouteView RouteData="@routeData" DefaultLayout="@typeof(MainLayout)">
                <NotAuthorized>
                    <h1>Sorry</h1>
                    <p>You are not authorized to reach this page.</p>
                    <p>You may need to login with an authorized user.</p>
                </NotAuthorized>
                <Authorizing>
                    <h1>Authenticate in progress</h1>
                </Authorizing>
            </AuthorizeRouteView>
        
        </Found>
            <NotFound>
                <LayoutView Layout="@typeof(MainLayout)">
                    <p>Sorry, there's nothing at this address.</p>
                </LayoutView>
            </NotFound>
        </Router>
    </CascadingAuthenticationState>

```

### Tracking page, Authorized pages to a specific role

```console
@attribute [Authorize(Roles = "Manager")]
```

### Authorize to all users
```console
@attribute [Authorize]
```

### Mix display according to roles
```console
<AuthorizeView Roles="Administrator, Manager">
    <li class="nav-item px-3">
        <NavLink class="nav-link" href="BusinessProfile">
            <span class="oi oi-list-rich" aria-hidden="true"></span> Business Information
        </NavLink>
    </li>
</AuthorizeView>

```
### Login page
```console
@page "/"
@inject IUserService UsrService
@inject ILogger<Login> Logger

@using Microsoft.AspNetCore.Identity
<h2>MyService</h2>
<hr />
<AuthorizeView>
    <Authorized Context="auth">
        <div class="display-4">Hi, @auth.User.Identity.Name </div>
    </Authorized>
    <NotAuthorized Context="auth">
        <EditForm Model="@loginViewModel" OnValidSubmit="MyLogin">
            <FluentValidator TValidator="LoginValidator" />
            <div class="row">
                <div class="col-md-8">
                    <div class="form-group">
                        <label for="Email" class="control-label">Email</label>
                        <InputText id="Email" class="form-control" @bind-Value="@loginViewModel.Email" />
                        <ValidationMessage For="@(() => loginViewModel.Email)" />
                    </div>
                    <div class="form-group">
                        <label for="Password" class="control-label">Password</label>
                        <InputText type="password" id="Password" class="form-control" @bind-Value="@loginViewModel.Password" />
                        <ValidationMessage For="@(() => loginViewModel.Password)" />
                    </div>
                </div>

            </div>
            <div class="row">
                <div class="col-md-4">
                    <div class="form-group">
                        <button type="submit" class="btn btn-primary">Log in</button>
                        <a href="/addUser" class="btn btn-secondary">Register</a>

                    </div>
                    @if (showErrorLogin)
                    {
                        <div class="text-danger validation-summary-errors">
                            Invalid login attempt.
                        </div>
                    }
                </div>
            </div>
        </EditForm>
    </NotAuthorized>
</AuthorizeView>

@code{


    private bool showErrorLogin = false;
    private User usr { get; set; } = new User();

    LoginViewModel loginViewModel = new LoginViewModel();

    protected async Task MyLogin()
    {
        try
        {
            usr = UsrService.Authenticate(loginViewModel.Email, loginViewModel.Password);

            //Console.WriteLine(SignInManager.IsSignedIn(usr));

            if (usr != null)
            {
                UserLogin usrLogin = new UserLogin();
                usrLogin.Id = usr.Id;
                usrLogin.Email = usr.Email;
                usrLogin.Roles = usr.Roles;

                await ServiceProvider.Get<LoginService>().LoginAsync(usrLogin);
                NavigationManager.NavigateTo("/");
            }
            else
            {
                showErrorLogin = true;
            }
        }
        catch (Exception ex)
        {
            Logger.LogInformation(ex.StackTrace);
        }
    }
    void Cancel()
    {
        NavigationManager.NavigateTo("/");
    }
}
```

### ENUMS

```console
using System;
using System.Collections.Generic;
using System.Linq;
using System.Threading.Tasks;

namespace MyService.Web.Helpers
{
    [Flags]
    public enum Roles
    {
        //None = 0b_0000_0000,  // 0
        User = 0b_0000_0001,  // 1
        Customer = 0b_0000_0010,  // 2
        Advisor = 0b_0000_0100,  // 4
        Supervisor = 0b_0000_1000,  // 8
        Administrator = 0b_0001_0000,  // 16
        Manager = 0b_0010_0000,  // 32

    }

    public class Role
    {
        public string Name { get; set; }
        public int Value { get; set; }
    }
}

```

### Enum operation
<section>
     <a href="https://docs.microsoft.com/en-us/dotnet/csharp/language-reference/builtin-types/enum" target="_blank">
        How to use Enum in C#
    </a>
</section>
