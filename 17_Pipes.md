# Using Pipes to Transform Output

You can use CLI to generate custom pipe: `npx ng generate pipe filter`

*app.component.html*

```html
<div class="container">
  <div class="row">
    <div class="col-xs-12 col-sm-10 col-md-8 col-sm-offset-1 col-md-offset-2">
      <input type="text" [(ngModel)]="filteredStatus"><hr>
      <ul class="list-group">
        <li class="list-group-item"
          *ngFor="let server of servers | filter:filteredStatus:'status'"
          [ngClass]="getStatusClasses(server)"> <!-- custom filter pipe -->
          <span class="badge">{{ server.status }}</span>
          <strong>{{ server.name | shorten:15 }}</strong>  <!-- using our custom pipe -->
          | {{ server.instanceType | uppercase }}  <!-- using built-in pipe -->
          | {{ server.started | date:'fullDate' | uppercase }}  <!-- parametrizing and chaining -->
        </li>
      </ul>
    </div>
  </div>
</div>
```

*shorten.pipe.ts*

```typescript
import { Pipe, PipeTransform } from '@angular/core';

@Pipe({
  name: 'shorten'
})
export class ShortenPipe implements PipeTransform {

  transform(value: any, limit: number) {
    return value.length > limit ? value.substr(0, limit) + '...' : value;
  }
}
```

*filter.pipe.ts*

```typescript
import { Pipe, PipeTransform } from '@angular/core';

@Pipe({
  name: 'filter',
  pure: false  // enables recalculation if input data changed
    // disabled by default, because can lead to performance issue
})
export class FilterPipe implements PipeTransform {

  transform(value: any, filterString: string, propName: string): any {
    if (value.length === 0 || filterString === '') {
      return value;
    }
    const resultArray = [];
    for (const item of value) {
      if (item[propName] === filterString) {
        resultArray.push(item);
      }
    }
    return resultArray;
  }
}
```

*app.module.ts*

```typescript
import { AppComponent } from './app.component';
import { ShortenPipe } from './shorten.pipe';
import { FilterPipe } from './filter.pipe';

@NgModule({
  declarations: [
    AppComponent,
    ShortenPipe,
    FilterPipe,
  ],
...
```

## AsyncPipe

Unwraps a value from an asynchronous primitive.

The async pipe subscribes to an Observable or Promise and returns the latest value it has emitted. When a new value is emitted, the async pipe marks the component to be checked for changes. When the component gets destroyed, the async pipe unsubscribes automatically to avoid potential memory leaks.

```html
<h2>App Status: {{ appStatus | async }}</h2>
```
