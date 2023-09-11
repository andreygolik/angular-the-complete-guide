# The HttpClient

[HttpClient Guide](https://angular.io/guide/http)

## Migration

Instead of:

```ts
// in app.module.ts
import { HttpModule } from '@angular/http';

// in component/service
import { Http } from '@angular/http';
constructor(private http: Http, ...);

this.httpClient.get().map(
    (response: Response) => {
        const recipes: Recipe[] = response.json();
        ...
    });
```

use:

```ts
// in app.module.ts
import { HttpClientModule } from '@angular/common/http';

// in component/service
import { HttpClient } from '@angular/common/http';
constructor(private httpClient: HttpClient, ...);

// HttpClient automatically extracts body, also
// methods (get/etc.) can be used as a generic methods
// we can tell it which type we are getting back
this.httpClient.get<Recipe[]>(url).pipe(map(
    (recipes) => {
        ...
    }));
```

## Requesting Response

```ts
this.httpClient.get(url, {
    // default values
    observe: 'body',  // 'response' to get full response
    responseType: 'json', // text, blob, arraybuffer, etc.
});
```

## Requesting Events

```ts
import { HttpEvent, HttpEventType } from '@angular/common/http';

this.httpClient.put(url, data, {
    observe: 'events',
}).subscribe(
    (response: HttpEvent<Object>) => {
        // type == 0: http sent event
        // type == 4: a normal response
        console.log(response.type === HttpEventType.Response);
    }
);
```

## Headers

```ts
import { HttpClient, HttpHeaders } from '@angular/common/http';

this.httpClient.get(url, {
    headers: new HttpHeaders()
        .set('Authorization', 'Bearer some-token')
        .append('some-header', 'some-header-value')
});
```

## Http Parameters

```ts
import { HttpClient, HttpParams } from '@angular/common/http';

this.httpClient.get(url, {
    // ?auth=' + token
    params: new HttpParams().set('auth', token)
});
```

## Progress

```ts
import { HttpClient, HttpRequest } from '@angular/common/http';

const req = new HttpRequest('PUT', url, data, {
    reportProgress: true,  // will give you progress feedback (events)
});

// More advanced way to make request
return this.httpClient.request(req);
```

## Interceptors

### Basic Interceptor

*auth.interceptor.ts*

```ts
import { HttpInterceptor, HttpRequest, HttpHandler, HttpEvent } from '@angular/common/http';
import { Observable } from 'rxjs';

export class AuthInterceptor implements HttpInterceptor {
  intercept(req: HttpRequest<any>, next: HttpHandler): Observable<HttpEvent<any>> {
    console.log('Intercepted!', req);
    return next.handle(req);
  }
}
```

Provide it in a module this way:

```ts
import { HTTP_INTERCEPTORS } from '@angular/common/http';

import { AuthInterceptor } from '../shared/auth.interceptor';

@NgModule({
  providers: [
    {provide: HTTP_INTERCEPTORS, useClass: AuthInterceptor, multi: true},
    // {provide: HTTP_INTERCEPTORS, useClass: LoggingInterceptor, multi: true},
    // the order you setup here is the order by which the request will travel
  ]
})
```

### Modifying Requests

**By default the requests are immuttable!**
You have to clone it before you edit it.

*auth.interceptor.ts*

```ts
import { HttpInterceptor, HttpRequest, HttpHandler, HttpEvent } from '@angular/common/http';
import { Injectable } from '@angular/core';
import { Observable } from 'rxjs';

import { AuthService } from '../auth/auth.service';

@Injectable()
export class AuthInterceptor implements HttpInterceptor {
  constructor(private authService: AuthService) {}

  intercept(req: HttpRequest<any>, next: HttpHandler): Observable<HttpEvent<any>> {
    console.log('Intercepted!', req);
    const copiedReq = req.clone({
      params: req.params.set('auth', this.authService.getToken()),
      // headers: req.headers.append('', ''),
    });
    return next.handle(req);
  }
}
```

### Capturing Response

*loggin.interceptor.ts*

```ts
import { HttpInterceptor, HttpRequest, HttpHandler, HttpEvent } from '@angular/common/http';
import { Observable } from 'rxjs';
import { tap } from 'rxjs/operators';

export class LogginInterceptor implements HttpInterceptor {
  intercept(req: HttpRequest<any>, next: HttpHandler): Observable<HttpEvent<any>> {
    return next.handle(req).pipe(tap(
      event => {
        console.log('Logging interceptor:', event);
      }
    ));
  }
}
```

## Complete Example

*data-storage.service.ts*

```ts
import { Injectable } from '@angular/core';
import { HttpClient, HttpHeaders, HttpParams, HttpRequest } from '@angular/common/http';
import { map } from 'rxjs/operators';

import { RecipeService } from '../recipes/recipe.service';
import { Recipe } from '../recipes/recipe.model';
import { AuthService } from '../auth/auth.service';

@Injectable()
export class DataStorageService {
  private baseUrl = 'https://project-647283321345.firebaseio.com';

  constructor(private httpClient: HttpClient,
              private recipeService: RecipeService,
              private authService: AuthService) {
  }

  storeRecipesPut() {
    const token = this.authService.getToken();
    // const headers = new HttpHeaders()
    //   .set('Authorization', 'Bearer some-token').append('some', 'value');

    return this.httpClient.put(this.baseUrl + '/recipes.json',
      this.recipeService.getRecipes(),
      {
        params: new HttpParams().set('auth', token),  // ?auth=' + token
        // headers: headers,
        observe: 'body',
      }
    );
  }

  // More advanced way to make request
  storeRecipes() {
    const token = this.authService.getToken();

    const req = new HttpRequest('PUT', this.baseUrl + '/recipies.json',
      this.recipeService.getRecipes(),
      {
        params: new HttpParams().set('auth', token),  // ?auth=' + token
        reportProgress: true,  // will give you progress feedback (events)
      }
    );
    return this.httpClient.request(req);
  }

  getRecipes() {
    const token = this.authService.getToken();

    this.httpClient.get<Recipe[]>(this.baseUrl + '/recipes.json',
      {
        params: new HttpParams().set('auth', token),  // ?auth=' + token
        observe: 'body',
        responseType: 'json',
      }
    ).pipe(map(
        (recipes) => {
          for (const recipe of recipes) {
            if (!recipe['ingredients']) {
              recipe['ingredients'] = [];
            }
          }
          return recipes;
        }
      ))
      .subscribe(
        (recipes: Recipe[]) => {
          this.recipeService.setRecipes(recipes);
        }
      );
  }
}
```
