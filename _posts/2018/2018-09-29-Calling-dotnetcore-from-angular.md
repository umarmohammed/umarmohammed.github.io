---
layout: post
title: Calling a .NET Core endpoint from angular
date: 2018-09-29
comments: true
categories:
- Mobile
tags:
- angular
- dotnetcore
---

This post shows how to call a .Net Core endpoint from an Angular client application.

<!-- more -->

Versions used in this post

- Angular CLI: 6.2.3
- Node: 8.9.4
- Angular: 6.1.9
- .Net Core 2.1

### .Net Core Web Application

Create a new empty .Net Core Web Application in Visual Studio. 

Edit the Startup class to use the MVC middleware. For this demo we also allow the application to allow any CORS for the preflight request sent by the Angular app. See [here](https://docs.microsoft.com/en-us/aspnet/core/security/cors?view=aspnetcore-2.1) for more details.

```csharp
public class Startup
{
  public void ConfigureServices(IServiceCollection services)
  {
    services.AddMvc();
  }

  public void Configure(
    IApplicationBuilder app, 
    IHostingEnvironment env)
  {
    if (env.IsDevelopment())
    {
        app.UseDeveloperExceptionPage();
    }

    app.UseCors(options =>
    {
      options.AllowAnyHeader();
      options.AllowAnyMethod();
      options.AllowAnyOrigin();`
    });

    app.UseMvc();
  }
}
```

Add a Controller and Action to return some values

```csharp
[Route("api/[controller]")]
[ApiController]
public class ValuesController : ControllerBase
{
  public ActionResult Get()
  {
    return Ok(new []{ "Values1", "Values2" });
  }

}
```

### Angular Client

Create a new Angular project using the Angular CLI.

```
ng new angular 
```

Add a service to call the Values Controller

```
ng g service services/values
```

```ts
import { Injectable } from '@angular/core';
import { HttpClient } from '@angular/common/http';

@Injectable({
  providedIn: 'root'
})
export class ValuesService {

  constructor(private http: HttpClient) { }


  private getValues(): Promise<string[]> {
    return this.http.get<string[]>('https://localhost:44321/api/values')
      .toPromise();
  }

}
```

Call the service from the app component.

```ts
import { Component, OnInit } from '@angular/core';
import { ValuesService } from './services/values.service';

@Component({
  selector: 'app-root',
  templateUrl: './app.component.html',
  styleUrls: ['./app.component.css']
})
export class AppComponent implements OnInit {
  
  values: string[] = [];


  constructor(private valuesService: ValuesService) {}


  ngOnInit(): void {
    this.loadValues();
  }


  private async loadValues() {
    this.values =  await this.valuesService.getValues();
  }

}
```

```
<ul>
  <li *ngFor = "let value of values">
    {% raw %}{{ value }}{% endraw %}
  </li>
</ul>
```

Edit the AppModule to import the HttpClientModule and declare the ValueService as a provider.

```ts
import { BrowserModule } from '@angular/platform-browser';
import { NgModule } from '@angular/core';
import { HttpClientModule } from '@angular/common/http';

import { AppComponent } from './app.component';
import { ValuesService } from './services/values.service';

@NgModule({
  declarations: [
    AppComponent
  ],
  imports: [
    BrowserModule,
    HttpClientModule
  ],
  providers: [
    ValuesService
  ],
  bootstrap: [AppComponent]
})
export class AppModule { }

```