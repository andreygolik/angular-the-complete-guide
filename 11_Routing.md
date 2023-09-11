# Routing

## Router Links

```html
<a routerLink="/">Home</a>
<a routerLink="/servers">Servers</a>
<a [routerLink]="['/users']">Users</a>
```

It is possible to use relative links: `routerLink="../servers"`

### routerLinkActive

```html
<li routerLinkActive="active"
  [routerLinkActiveOptions]="{exact: true}">
  <a routerLink="/">Home</a>
</li>
```

## Navigating Programmatically

```typescript
import { Router } from '@angular/router';

export class HomeComponent {
  constructor(private router: Router) {}

  onLoadServers() {
    ...
    this.router.navigate(['/servers']);
  }
}
```

Unlike routerLink router.navigate() doesn't know on which route you are currently on. To use relative path you have to use it this way:

```typescript
import { Router, ActivatedRoute } from '@angular/router';
...
constructor(
  private router: Router,
  private route: ActivatedRoute
) {}
...
this.router.navigate(['servers'], {relativeTo: this.route});
```

## Route Parameters

### Passing Parameters to Routes

```typescript
const appRoutes: Routes = [
  { path: 'users/:id/:name', component: UserComponent },
];
```

```html
<a [routerLink]="['/users', user.id, user.name]"
  *nfFor="let user of users" href="#">
  {{ user.name }}
</a>
```

### Fetching Route Parameters

```typescript
import { ActivatedRoute } from '@angular/router';

constructor(private route: ActivatedRoute) {}

onInit() {
  this.user.id = +this.route.snapshot.params['id'];
  // parameter will allways be 'string', you have to convert it to number if needed
  this.user.name = this.route.snapshot.params['name'];
}
```

### Fetching Route Parameters Reactively

You **should unsubscribe** if component could theoretically be destroyed.

```typescript
import { Component, OnInit, OnDestroy } from '@angular/core';
import { ActivatedRoute } from '@angular/router';
import { Subscription } from 'rxjs/Subscription';
...
export class UserComponent implements OnInit, OnDestroy {
  paramsSubscription: Subscription;

  constructor(private route: ActivatedRoute) {}

  ngOnInit() {
    this.user = {
      id: this.snapshot.params['id'],
      name: this.snapshot.params['name'],
    }

    this.paramsSubscription = this.route.params
      .subscribe(
        (params: Params) => {
          this.user.id = params['id'];
          this.user.name = params['name'];
        }
      );
  }

  ngOnDestroy() {
    this.paramsSubscription.unsubscribe();
  }
}
```

## Query Parameters and Fragments

### Passing

```typescript
const appRoutes: Routes = [
  { path: 'servers/:id/edit', component: ServerComponent },
];
```

#### From Template

```html
<a [routerLink]="['/servers', 5, 'edit']"
  [queryParams]="{allowEdit: '1'}"
  fragment="loading"
  href="#">
  {{ server.name }}
</a>
```

URL will be: `/servers/5/edit?allowEdit=1#loading`

#### Programmatically

```typescript
this.router.navigate(
  ['/servers', id, 'edit'],
  {
    queryParams: { allowEdit: '1' },
    fragment: 'loading',
  }
);
```

### Retrieving

```typescript
import { ActivatedRoute } from '@angular/router';
...
constructor(private route: ActivatedRoute) {}
...
ngOnInit() {
  this.queryParams = this.route.snapshot.queryParams;
  this.fragment = this.route.snapshot.fragment;

  // you can additianlly subscribe if needed
  this.route.queryParams.subscribe();
  this.route.fragment.subscribe();
  // do not forget to ubsubscribe in ngOnDestroy()
}
```

## Child (Nested) Routes

```typescript
const appRoutes: Routes = [
  { path: 'users', component: UsersComponent, children: [
    { path: ':id/:name', component: UsersComponent },
  ]},
  { path: 'servers', component: ServersComponent, children: [
    { path: ':id', component: ServerComponent },
    { path: ':id/edit', component: EditServerComponent },
  ]}, 
];
```

You should add `<router-outlet></router-outlet>` to the templates of UsersComponent and ServersComponent.

## Handling of Query Parameters

To preserve query parameters (like `?allowEdit=1#loading`) during navigation:

```typescript
this.router.navigate(
  ['edit'],
  {
    relativTo: this.route,
    queryParamsHanfling: 'preserve',
  },
)
```

## Redirecting

```typescript
const appRoutes: Routes = [
  { path: '', redirectTo: '/home', pathMatch: 'full'},
  // without pathMatch == 'full' it will redirect ANY route
  ...
  { path: '**', redirectTo: '/not-found' }
}
```

## Guards

### canActivate

*auth.guard.ts* (or *auth-guard.service.ts*)

```typescript
import { Injectable } from '@angular/core';
import {
  Router,
  CanActivate, 
  ActivatedRouterSnapshot,
  RouterStateSnapshot,
} from '@angular/router';

import { AuthService } from '../services/auth.service';

@Injectable()
export class AuthGuard implements CanActivate {
  constructor(
    private router: Router,
    private authService: AuthService) {}

  public canActivate(
    // you can skip this params if not needed and just use ()
    route: ActivatedRouteSnapshot,
    state: RouterStatusSnapshot,
  ): Observable<boolean> | Promise<boolean> | boolean {
    return this.authService.isAuthenticated()
      .then(
        (authenticated: boolean) => {
          if (authenticated) {
            return true;
          } else {
            this.router.navigate(['/login']);
            return false;
          }
        }
      );
  }
}
```

*auth.service.ts*

```typescript
// isAuthenticated() Promise
public isAuthenticated(): Promise<boolean> {
  const promise = new Promise<boolean>(
    (resolve, reject) => {
      resolve(this.isLoggedIn());
    }
  );
  return promise;
}
```

*app.routing.ts*

```typescript
import { AuthGuard } from './services/auth.guard';

const appRoutes: Routes = [
  { path: 'servers', canActivate: [AuthGuard], component: ServersComponent, ...},  
];
```

You have to **define providers** AuthGuard and AuthService in AppModule.

### canActivateChild

Guard should implement CanActivateChild. It haves the same format as CanActivate.

Route example:

```typescript
{ path: 'servers',
  canActivateChild: [AuthGuard],
  component: ServersComponent,
  children: [
    { path: ':id', component: ServerComponent },
    ...
  ],
},
```

### canDeactivate

*can-deactivate.guard.ts*

```typescript
import { CanDeactivate, ActivatedRouteSnapshot, RouterStateSnapshot } from '@angular/router';
import { Observable } from 'rxjs';

export interface CanComponentDeactivate {
  canDeactivate: () => Observable<boolean> | Promise<boolean> | boolean;
}

export class CanDeactivateGuard implements CanDeactivate<CanComponentDeactivate> {
  canDeactivate(component: CanComponentDeactivate,
    currentRoute: ActivatedRouteSnapshot,
    currentState: RouterStateSnapshot,
    nextState?: RouterStateSnapshot
  ): Observable<boolean> | Promise<boolean> | boolean {
    return component.canDeactivate();
  }
}
```

In component:

```typescript
import { CanComponentDeactivate } from './can-deactivate.guard';

export class EditServerComponent implements CanComponentDeactivate {
  canDeactivate(): Observable<boolean> | Promise<boolean> | boolean {
    if (this.changesSaved) {
      return true;
    }
    return false;
  }
}
```

## Passing Static Data

Route:

```typescript
{ path: 'not-found', component: ErrorComponent,
  data: { code: 404, message: 'The requested page was not found.' } },
```

Component:

```typescript
ngOnInit() {
  this.route.data.subscribe(
    (data: Data) => {
      this.code = data['code'];
      this.message = data['message'];
    }
  );
}
```

## Resolving Dynamic Data

server-resolver.service.ts:

```typescript
import { Injectable } from '@angular/core';
import { Resolve, ActivatedRouteSnapshot, RouterStateSnapshot } from '@angular/router';
import { Observable } from 'rxjs';

import { ServerService } from './server.service';

interface Server {
  id: number;
  name: string;
  status: string;
}

@Injectable()
export class ServerResolver implements Resolve<Server> {
  constructor(private serverService: ServerService) {}

  resolve(route: ActivatedRouteSnapshot, state: RouterStateSnapshot): Observable<Server> | Promise<Server> | Server {
    return this.serverService.getServer(+route.params['id']);
  }
}
```

Route:

```typescript
{ path: ':id', component: ServerComponent, resolver: {server: ServerResolver} },
```

Component:

```typescript
ngOnInit() {
  this.route.data.subscriber(
    (data: Data) => {
      this.server = data['server'];
    }
  );
}
```

Note: *data['**server**']* should match to *resolver: {**server**: ServerResolver}*.

## useHash

Http server (nginx) should be configured to return index.html instead of 404. If it is impossible there is another way to make router works: 

```ts
RouterModule.forRoot(appRoutes, {useHash: true})
```

It will add # between server name and route: ```localhost:4200/#/users```. Part after # will be ignored by http server.

## a href page reloading fix

To prevent page reloading when using *<a href="#" (click)=...* there are couple options:

- `href` can be replaced by `style="cursor: pointer;"`
- `false` can be added after `onClick` function: `(click)="onClick(); false"`

## Examples

### Routing Module

*app.routing.ts*

```typescript
import { NgModule } from '@angular/core';
import { RouterModule, Routes } from '@angular/router';

import { AuthGuard } from './services/auth.guard';
import { LoginComponent } from './pages/login/login.component';
import { ErrorComponent } from './pages/error/error.component';

const appRoutes: Routes = [
  // { path: '', redirectTo: 'somewhere', pathMatch: 'full' },
  // Layout
  { path: '', loadChildren: './layout/layout.module#LayoutModule', canActivate: [AuthGuard] },
  // Login
  { path: 'logout', redirectTo: 'login' },
  { path: 'login', component: LoginComponent },
  // Errors
  { path: 'not-found', component: ErrorComponent, 
    data: { code: 404, message: 'The requested page was not found.' } },
  { path: '**', redirectTo: '/not-found' }
];

@NgModule({
  imports: [
    RouterModule.forRoot(appRoutes, { useHash: false }),  // enableTracing: true, // <-- debugging purposes only
  ],
  exports: [ RouterModule ],
})
export class AppRoutingModule {}
```

*app.module.ts*

```typescript
import { AppRoutingModule } from './app.routing.ts';

@NgModule({
  imports: [
    AppRoutingModule,
  ]
  ...
})
...
```
