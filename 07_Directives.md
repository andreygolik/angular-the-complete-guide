# Directives

## Introduction

### Attribute Directives

* Look like a normal HTML Attribute (possibly with databinding or event binding)
* Only affect/change the element they are added to

### Structural Directives

* Looks like a normal HTML Attribute bat have a leading *
* Affect a whole area in the DOM (elements get added/removed)

## ngFor & ngIf

```html
<li class="list-group-item"
  *ngFor="let number of numbers; let i = index">
  {{ number }}
</li>

<li ... *ngIf="number % 2 === 0">
```

## ngSwitch

```html
<div [ngSwitch]="value">
  <p *ngSwitchCase="5">Value is 5</p>
  <p *ngSwitchCase="10">Value is 10</p>
  <p *ngSwitchDefault>Value is Default</p>
</div>
```

## ngClass & ngStyle

```html
<li ... [ngClass]="{red-text: odd % 2 !== 0}">

<li ... [ngStyle]="{color: odd % 2 !== 0 ? 'red' : 'black'}">
```

## Attribute Directives

### Basic Attribute Directive

*basic-highlight.directive.ts*

```typescript
import { Directive, ElementRef, OnInit } from '@angular/core';

@Directive({
  selector: '[appBasicHighlight]'
})
export class BasicHighlightDirective implements OnInit {
  constructor(private elementRef: ElementRef) {}

  ngOnInit() {
    this.elementRef.nativeElement.style.backgroundColor = 'green';
    // not the best way
    // use renderer or host binding from examples below
  }
}
```

You need to declare it in a module:
```typescript
import { BasicHighlightDirective} from ...

@NgModule({
  declarations: [ BasicHighlightDirective, ... ]
```

Use in a template:
```typescript
<p appBasicHighlightDirective>Style me!</p>
```

### Better Attribute Directive

You should use the **Renderer** for any DOM manipulations. 

[Renderer2 Documentation](https://angular.io/api/core/Renderer2)

Creating directive with CLI:

```bash
ng c d better-highlight
```

*basic-highlight.directive.ts*

```typescript
import { Directive, OnInit, ElementRef, Renderer2 } from '@angular/core';

@Directive({
  selector: '[appBetterHighlight]'
})
export class BetterHighlightDirective implements OnInit {
  constructor(private elRef: ElementRef, private renderer: Renderer2) {}

  ngOnInit() {
    this.renderer.setStyle(this.elRef.nativeElement, 'background-color', 'blue');
  }
}
```

## HostListener

Host listener is used for listening Host Events.

Inside directive:

```typescript
@HostListener('mouseenter') mouseover(eventData: Event) {
  this.renderer.setStyle(this.elRef.nativeElement, 'background-color', 'red');
}
@HostListener('mouseleave') mouseleave(eventData: Event) {
  this.renderer.setStyle(this.elRef.nativeElement, 'background-color', 'green');
}
```

## HostBinding

Host binding is used to bind to Host Properties.

Inside directive:

```typescript
@HostBinding('style.backgroundColr') backgroundColr: string = 'transparent';
```
Then in @HostListener (for example) simply set the property value:
```typescript
this.backgroundColor = 'blue';
```

## Binding to Directive Properties

In directive:

```typescript
// Default values, can be overwriten outside
@Input() dafaultColor: string = 'transparent';
@Input() highlightColor: string = 'blue';
@HostBinding('style.backgroundColor') backgroundColor: string;
...
ngOnInit() {
  this.backgroundColor = this.defaultColor;
}

@HotListener('mouseenter') mouseover(eventData: Event) {
  this.backgroundColor = this.highlightColor;
}
```

In template:

```html
<p appBetterHighlight [dafaultColor]="'yellow'" [highlightColor]="'red'">
<!-- this will override @Input variables -->
```

Note it uses single quotes inside double quotes. It is possible to remove brackets and single quotes, like this:

```html
<p appBetterHighlight ... dafaultColor="yellow">
```

But you need to be careful not to interfere with the real HTML attribute.

Using @Input alias:

```typescript
@Input('appBetterHighlight') highlightColor: string;
```

```html
<p [appBetterHighlite]="'red'">
```

## Behind the Scenes on Structural Directives

```html
<div *ngIf="!onlyOdd">...</div>
```

Behind the scenes transforms to: 
```html
<ng-template [ngIf]="!onlyOdd">...</ng-template>
```

## Structural Directive

*unless.directive.ts*

```typescript
import { Directive, Input, TemplateRef, ViewContainerRef } from '@angular/core';

@Directive({
  selector: '[appUnless]'
})
export class UnlessDirective {
  @Input() set appUnless(condition: boolean) {
    if (!condition) {
      this.vcRef.createEmbeddedView(this.templateRef);
    } else {
      this.vcRef.clear();
    }
  }

  constructor(
    private templateRef: TemplateRef<any>,
    private vcRef: ViewContainerRef
  ) {}
}
```

Template:

```html
<div *appUnless="onlyOdd">...</div>
```

## Examples

### dropdown.directive.ts

```typescript
import { Directive, HostListener, HostBinding } from '@angular/core';

@Directive({
  selector: '[appDropdown]'
})
export class DropdownDirective {
  @HostBinding('class.open') isOpen = false;
  @HostListener('click') toggleOpen() {
    this.isOpen = !this.isOpen;
  }
}
```
Do not forget to add it to the declarations in the module.

Template:
```html
<li class="dropdown" appDropdown>...
```
