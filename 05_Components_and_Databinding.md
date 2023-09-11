
# Components & Databinding

## Binding to Custom Properties

In **parent** component:

```html
<app-element *ngFor="let sElement of sElements" [element]="sElement"></app-element>
```

In **child** component:

```typescript
import { Input } from '@angular/core';

@Component({
  selector: 'app-element',
  ...
})
export class Element { ...
  @Input() element: {...};
  ...
}
```

It is possible to add **alias**: `@Input('serverElement')`, it will be accessible outside the component.

## Binding to Custom Events

If you want to inform parent component about some event we can create custom event.

In **parent** component:

```html
<app-cockpit (serverCreated)="onServerCreated($event)"></app-cockpit>
```

```typescript
onServerCreated(serverData: {name: string, content: string}) {}
```

In **child** component:

```typescript
import { EventEmmiter, Output } from '@angular/core';

@Output() serverCreated = new EventEmitter<{name: string, content: string}>();

onAddServer() {
  this.serverCreated.emit({name: this.name, content: this.content})
...}
```

It is possible to add **alias**: `@Output('serverAdded')`

## View Encapsulation

Defines if child view will inherit styles of parent component.

```typescript
@Component({
    encapsulation: ViewEncapsulation.Emulated  // Emulated, None or Native
})
```

## Local References to Templates

Works **only** inside template, not in TS code.

Template:
```html
<input type="text" #nameInput> ...
<button (click)="onAdd(nameInput)"> ...
```
Component:
```typescript
onAdd(nameInput: HTMLInputElement) {
  console.log(name.value);
}
```

## Template & DOM access with @ViewChild

Template:
```html
<input ... #serverNameInput> ...
```
Component:
```typescript
@ViewChild('serverNameInput') serverNameInput: ElementRef;

onSomeEvent() {
  serverName = this.serverNameInput.nativeElement.value;
}
```
You **should not change** element this way, only read it.

## Projecting Content into Components with ng-content

In **parent** template:

```html
<app-server-element ...>
  <!-- this will replace ng-content tag later in a child component -->
  <p #contentParagraph>{{ serverElement.content }}</p>
<app-server-element>
```
In **child** component:
```html
<div class="content-from-parent">
  <ng-content></ng-content><!-- will be replaced -->
  <!-- by the content from a parent template which goes betwen tags -->
</div>
```

## Getting Access to ng-content with @ContentChild

The same as @ViewChild but for content from ng-content. 

```typescript
@ContentChild('contentParagraph') paragraph: ElementRef;
```

## Component Lifecycle

* `ngOnChanges` called after a bound input property changes
* `ngOnInit` called once the component is initialized
* `ngDoCheck` called during every change detection run
* `ngAfterContentInit` called after content (ng-content) has been projected into view
* `ngAfterContentChecked` called every time the projected content has been checked
* `ngAfterViewInit` called after the component's view (and child views) has been initialized
* `ngAfterViewChecked` called every time the view (and child views) has been checked
* `ngOnDestroy` called once the component is about to be destroyed
