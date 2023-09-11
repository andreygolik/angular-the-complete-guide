# The Basics

## Component Selector

```typescript
# By element
selector: 'app-component' # <app-component></app-component>

# By atribute
selector: '[app-component]' # <div app-component></div>

# By class
selector: '.app-component' # <div class="app-component"></div>
```

## Databinding

* Output
  * String Interpolation `{{data}}`
  * Property Binding `[property]="expression|data"`
* Input
  * Event Binding `(event)="expression"`
* Two-Way Binding
  * `[(ngModel)]="data"`

### String Interpolation

```typescript
serverId: number = 10;
serverStatus: string = 'offline';

getServerStatus()
	return this.serverStatus;
}
```

```html
<p>{{ 'Server' }} with ID {{ serverId }} is {{ getServerStatus() }}</p>
```

### Property Binding

```html
<button class="btn btn-primary"
    [disabled]="!allowNewServer">Add Server</button>
```

### Event Binding

```html
<input type="text" class="form-control"
       (input)="onUpdateServerName($event)"></input>
<button class="btn btn-primary" [disabled]="!allowNewServer"
        (click)="onCreateServer()">Add Server</button>
```

```type
onCreateServer() {
    this.status = "created";
}

onUpdateServerName(event: Event) {
    this.serverName = (<HTMLInputElement>event.target).value;
}
```

### Two-Way Binding

* You need to enable the `ngModel`  directive. This is done by adding the `FormsModule`  to the `imports[]`  array in the AppModule.
* You then also need to add the import from `@angular/forms`  in the app.module.ts file:

```typescript
import { FormsModule } from '@angular/forms';
```

```html
<input type="text" class="form-control"
    [(ngModel)]="serverName"></input>
```

### ngIf

```html
<p *ngIf="serverCreated; else noServer">
    Server was created, server name is {{ serverName }}
</p>
<ng-template #noServer>
    <p>No server was created!</p>
</ng-template>
```

### ngStyle and ngClass

```html
<p [ngStyle]="{backgroundColor: getColor()}"
    [ngClass]="online: serverStatus === 'online'">
    {{ 'Server' }} with ID {{ serverId }} is {{ getServerStatus() }}
</p>
```

### ngFor

```html
<div *ngFor="let logItem of log; let i = index">
    {{ logItem }}
</div>
```
