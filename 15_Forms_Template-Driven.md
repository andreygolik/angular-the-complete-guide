# Template-Driven Forms

**Template-Driven Forms**: Angular infers the Form Object from the DOM

## Creating a TD Form and Registering the Controls

* First of all you need to import FormsModule and add it to imports array in app.module.ts.

  ```typescript
  import { FormsModule } from '@angular/forms';
  ```

* Second, you have to place `ngModel` and specify name to each form element you want to control:

  ```html
  <input type="text" id="username" class="form-control" ngModel name="username">
  ```

* 3rd, add `ngSubmit`:

  ```html
  <form (ngSubmit)="onSubmit(f)" #f="ngForm">...</form>
  ```

  *#f trick - this is how you get access to the form object created by Angular automatically.*

  ```typescript
  import { NgForm } from '@angular/forms';
  
  onSubmit(form: NgForm) {
    console.log(form);  // NgForm
    console.log(form.value);  // {username: '', email: '', ...}
  }
  ```

### [Alternative Approach] Accessing the Form with @ViewChild

```html
<form (ngSubmit)="onSubmit()" #f="ngForm">...</form>
```

```typescript
import { Component, ViewChild } from '@angular/core';
import { NgForm } from '@angular/forms';

export class AppComponent {
  @ViewChild('f') signupForm: NgForm;
  
  onSubmit() {
    console.log(this.signup.form);
  }
}
```

## Validation

Check out the Validators class: <https://angular.io/api/forms/Validators> - these are all built-in validators, though that are the methods which actually get executed (and which you can add when using the reactive approach).

For the template-driven approach, you need the directives. You can find out their names, by searching for "validator" in the official docs: <https://angular.io/api?type=directive> - everything marked with "D" is a directive and can be added to your template.

Additionally, you might also want to enable HTML5 validation (by default, Angular disables it). You can do so by adding the `ngNativeValidate`  to a control in your template.

Directive examples:

```html
<form>
  <input type="name" ... required> <!-- Required -->
  <input type="email" ... email> <!-- Valid Email -->
</form>
```

Angular dynamically add some CSS classes giving us information about the state of individual control:

```html
<input _ngcontent-c0="" class="form-control ng-dirty ng-invalid ng-touched" email="" id="email" name="email" ngmodel="" type="email" ng-reflect-email="" ng-reflect-name="email" ng-reflect-model="">
```

### Using the Form State

Disable the button if form is invalid:

```html
<button class="btn btn-primary" type="submit" [disabled]="!f.valid">Submit</button>
```

Using CSS classes:

```css
input.ng-invalid.ng-touched {
  border: 1px solid red;
}
```

### Validation Error Messages

```html
<input type="email" id="email" class="form-control"
       ngModel
       name="email"
       email
       #email="ngModel"> <!-- exposes additional info about the control for us -->
<span class="help-block" *ngIf="!email.valid">Please enter a valid email!</span>
```

## ngModel Property Binding

Set default value:

```html
<select id="secret" class="form-control"
        [ngModel]="defaultSecret"
        name="secret">
  <option value="pet">Your first Pet?</option>
  <option value="teacher">Your first teacher?</option>
</select>
```

```typescript
defaultSecret = 'pet';
```

## ngModel Two-Way Binding

```html
<div class="form-group">
  <textarea id="answer" rows="3" class="form-control"
            [(ngModel)]="answer"
            name="questionAnswer"></textarea>
</div>
<p>Your reply: {{ answer }}</p>
```

## Grouping Form Controls

```html
<form>
	<div id="user-data" nfModelGroup="userData" #userData="ngModelGroup">
  	<input ... name="username">
    <input ... name="email">
  </div>
  <p *ngIf="!userData.valid">Invalid user data!</p>
</form>
```

NgForm.value.userData holds object {username: '', email: ''}.

## Radio Buttons

```html
<div class="radio" *ngFor="let gender of genders">
  <label>
    <input type="radio"
           name="gender"
           ngModel
           [value]="gender"
           required>
    {{ gender }}
  </label>
</div>
```

## Setting and Patching Form Values

```typescript
@ViewChild('f') signupForm: NgForm;

suggestUserName() {
  // setValue is not the best approach
  // because it overwrites all the values at once
  //
  // this.signupForm.setValue({
  //   userData: {
  //     username: suggestedName,
  //     email: '',
  //   },
  //   secret: 'pet',
  //   questionAnswer: '',
  //   gender: 'male',
  // });

  // patchValue overwtites provided value[s] only
  this.signupForm.form.patchValue({
    userData: {
      username: suggestedName,
    }
  });
}
```

## Using Form Data

```html
<div class="row" *ngIf="submitted">
  <div class="col-xs-12">
    <h3>User Data</h3>
    <p>Username: {{ user.username }}</p>
    <p>Mail: {{ user.email }}</p>
    <p>Secret Question: {{ user.secretQuestion }} </p>
    <p>Answer: {{ user.answer }} </p>
    <p>Gender: {{ user.gender }} </p>
  </div>
</div>
```

```typescript
onSubmit() {
  // console.log(this.signupForm);
  this.submitted = true;
  this.user.username = this.signupForm.value.userData.username;
  this.user.email = this.signupForm.value.userData.email;
  this.user.secretQuestion = this.signupForm.value.secret;
  this.user.answer = this.signupForm.value.questionAnswer;
  this.user.gender = this.signupForm.value.gender;
}
```

## Resetting Forms

```typescript
onSubmit() {
  ...
  this.signupForm.reset();
}
```

If you want, you can pass the same object as in setValue() to reset() which will then reset the form to specific values.
