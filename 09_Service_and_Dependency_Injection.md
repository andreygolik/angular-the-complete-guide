# Services & Dependency Injection

## Simple Service

*logging.service.ts*

```typescript
export class LoggingService {
  logStatus(status: string) {
    console.log('Status: ' + status);
  }
}
```

*app.module.ts*

```typescript
import { LoggingService } from './logging.service';

@NgModule({
  ...
  providers: [ LoggingService ]
})
```

*my.component.ts*

```typescript
import { LoggingService } from '../loggin.service';

// !!! WRONG WAY !!!
  const service = new LoggingService();
// You have to define provider and use constructor.

// Right Way
@Component({
  providers: [ LoggingService ], 
  // service can be provided in the component,
  // but it also can be defined a module (for application-wide access).
})
...
constructor(private loggingService: LoggingService) {}
...
loggingService.logStatus('new status');
```

## Hierarchical Injector

* **AppModule** - Same Instance of Service is available **Application-wide**
* **AppComponent** - Same Instance of Service is available for **all Components** (but **not for other Services**)
* **Any other Component** - Same Instance of Service is available for **the Component and all its child components**

## Injecting Service into Service

*accounts.service.ts*

```typescript
import { Injectable } from '@angular/core';
import { LoggingService } from './logging.service';

@Injectable  // something can be injected in there.
// if you are not going to inject anything TO THIS service you do not need this metadata.
export class AccountsService {
  constructor(private loggingService: LoggingService) {}

  updateStatus(status: string) {
    this.loggingService.logStatus(status);
  }
}
```


## providedIn

*Added in Angular 6.*

Instead of adding your service to `providers` array in a module now you can also use a different approach:

```ts
@Injectable({
  providedIn: 'root'
})
export class MyService {...}
```

## Cross-Component Communication

*app.service.ts*

```typescript
import { EventEmitter } from '@angular/core';

export class AppService {
  statusUpdated = new EventEmitter<string>();
}
```

*first.component.ts*

```typescript
onSomething(status: string) {
  this.accountService.statusUpdated.emit(status);
}
```

*second.component.ts*

```typescript
constructor(private appService: AppService) {
  this.appService.statusUpdated.subscribe(
    (status: string) => alert('New Status: ' + status)
  );
}
```

## Examples

### Access to Data Reference

```typescript
import { AccountsService } from './accounts.service';

export class AppComponent implements OnInit {
  accounts: {name: string, status: string}[] = [];

  constructor(private accountsService: AccountService) {}

  ngOnInit() {
    this.account = this.accountsService.accounts;
    // since accounts is an array it is a reference tipe.
    // by setting it equal here you will have access to the same array
    // stored in AccountsService.
  }
}
```
