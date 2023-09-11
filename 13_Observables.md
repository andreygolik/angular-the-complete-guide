# Observables

## Examples

```typescript
import { Observable } from 'rxjs/Observable';
import { Subscription } from 'rxjs/Subscription';

... {
  subscription: Subscription;

  ngOnInit() {
    const myNumbers = Observable.interval(1000);
    this.subscription = myNumbers.subscribe(
      (number: number) => {
        console.log(number);
      }
    );
  }

  ngOnDestroy() {
    this.subscription.unsubscribe();
  }
}
```

```typescript
const myObservable = Observable.create((observer: Observer<string>) => {
  setTimeout(() => observer.next('first package'), 2000);
  setTimeout(() => observer.next('second package'), 4000);
  setTimeout(() => observer.error('this does not work'), 5000);
  setTimeout(() => observer.complete(), 6000);  // will not arive, because of error before
});

const subscription = myObservable.subscribe(
  (data: string) => console.log(data),
  (error: string) => console.error(error),
  () => console.log('completed')
);
```

## Subjects

If you want to implement cross-component communication you can use subject instead of event emitter.

*user.service.ts*

```typescript
import { Subject } from "rxjs/Subject";

export class UserService {
  userActivated = new Subject();
}
```

*user.component.ts*

```typescript
onActivate() {
  this.userService.userActivated.next(this.id);
}
```

*app.component.ts*

```typescript
ngOnInit() {
  this.userService.userActivated.subscribe(
    (id: number) => console.log(id)
  )
}
```

## Operators

The Map operator applies a function to each item emitted by the source Observable, and returns an Observable that emits the results of these function applications.

```typescript
import { map } from 'rxjs/operators';

const myNumbers = Observable.interval(1000)
  .pipe(
    map((x: number) => 10 * x)
  );
```

## RxJS 6 without rxjs-compat

### Imports

```typescript
import { Observable, Observer, Subscription, interval } from 'rxjs';
import { map } from 'rxjs/operators';
```

### Operators

You have to use operators like map() differently:

Instead of 

```typescript
....map(...)
```

use

```typescript
....pipe(
  map(...)
)
```

### Errors

Instead of 

```typescript
import 'rxjs/Rx'; 
import { Observable } from 'rxjs/Observable'; 

....catch(error => {
  return Observable.throw(...);
})
```

use
```typescript
import { catchError } from 'rxjs/operators';
import { throwError } from 'rxjs';

....pipe(
  catchError(error => {
    return throwError(...);
  })
)
```
