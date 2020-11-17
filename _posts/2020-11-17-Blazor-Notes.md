---
layout: post
title: Important tips while working with Microsoft Blazor
description: Working with Blazor, EF Core, please see some important tips
image: 
---

# Some command line (if change model before dotnet run; need to remove all from Migrations first and delete changed tables or the whold database)
## Add migrations, change model
```console
add-migration Initial
```

## Reverse Engineer (no needs in this project), it will generate classes and DbContext for tables in Ms. SQL Server
```console
Scaffold-DbContext "Server=.\;Database=ComplizeWeb;Trusted_Connection=True;" Microsoft.EntityFrameworkCore.SqlServer -OutputDir Entities
```

# Tracking page, Authorized pages to a specific role

```console
@attribute [Authorize(Roles = "Manager")]
```

# Authorize to all users
```console
@attribute [Authorize]
```

# Mix display according to roles
```console
<AuthorizeView Roles="Administrator, Manager">
    <li class="nav-item px-3">
        <NavLink class="nav-link" href="BusinessProfile">
            <span class="oi oi-list-rich" aria-hidden="true"></span> Business Information
        </NavLink>
    </li>
</AuthorizeView>

```
```console
@page "/"
@inject IUserService UsrService
@inject ILogger<Login> Logger

@using Microsoft.AspNetCore.Identity
<h2>Complize</h2>
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


