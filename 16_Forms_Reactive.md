# Reactive Forms

**Reactive Forms** are created programatically and synchronized with the DOM.

## Setup

You do not need **FormsModule** (it is required for template-driven approach), but instead you need to import **ReactiveFormsModule**:

```typescript
import { ReactiveFormsModule } from '@angular/forms';

@NgModule({
  imports: [ ReactiveFormsModule, ],  
  ...
})
```

## Creating a From and Syncing HTML and Code

```typescript
import { FormGroup, FormControl } from '@angular/forms';
...
export class AppComponent implements OnInit {
  signupForm: FormGroup;

  ngOnInit() {
    this.signupForm = new FormGroup({
      'username': new FormControl(null),
      'email': new FormControl(null),
      'gender': new FormControl('male'),
    });
  }
}
```

```html
<form [formGroup]="signupForm">
  <div class="form-group">
    <label for="username">Username</label>
    <input type="text" id="username" class="form-control"
           [formControlName]="'username'">
  </div>
  <div class="form-group">
    <label for="email">email</label>
    <input type="text" id="email" class="form-control"
           formControlName="email">
  </div>
  <div class="radio" *ngFor="let gender of genders">
    <label>
      <input type="radio" 
             formControlName="gender" [value]="gender">{{ gender }}
    </label>
  </div>
  <button class="btn btn-primary" type="submit">Submit</button>
</form>
```

The directive **formControlName** can bee specified in two ways:

* `formControlName="username"`
* `[formControlName]="'username'"`

## Submitting the Form

```html
<form [formGroup]="signupForm" (ngSubmit)="onSubmit()">...</form>
```

```typescript
onSubmit() {
  console.log(this.signupForm.value.username);
}
```

## Validation

You are not configuring your form in the template. You only synchronize it with your code with the directives **formControlName** and **formGroup**. Then you configure it in your TypeScript code.

Check out the Validators class: https://angular.io/api/forms/Validators - these are all built-in validators.


```typescript
import { FormGroup, FormControl, Validators } from '@angular/forms';
...
ngOnInit() {
  this.signupForm = new FormGroup({
    'username': new FormControl(null, Validators.required),
    'email': new FormControl(null, [ Validators.required, Validators.email ]),
    ...
  });
}
```

### Getting Access to Controls

```html
<span *ngIf="!signupForm.get('username').valid && signupForm.get('username').touched"
  class="help-block">Please enter a valid username!</span>

<span *ngIf="!signupForm.valid && signupForm.touched"
  class="help-block">Please enter a valid data!</span>
```

### Grouping Controls

```typescript
this.signupForm = new FormGroup({
  'userData': new FormGroup({  // userData form group
    'username': new FormControl(null, Validators.required),
    'email': new FormControl(null, [Validators.required, Validators.email]),
  }),
  'gender': new FormControl('male'),
});
```

```html
<form [formGroup]="signupForm" (ngSubmit)="onSubmit()">
  <div formGroupName="userData"> <!-- userData form group -->
    <div class="form-group">
      <label for="username">Username</label>
      <input type="text" id="username" class="form-control" [formControlName]="'username'">
      <span class="help-block"
            *ngIf="!signupForm.get('userData.username').valid && signupForm.get('userData.username').touched">
        Please enter a valid username!</span>
    </div>
...
```

### Custom Validators

```typescript
forbiddenUsernames = ['Chris', 'Anna'];

ngOnInit() {
  this.signupForm = new FormGroup({
    'username': new FormControl(null, [ Validators.required, this.forbiddenNames.bind(this) ]),
    ...
  });
}

forbiddenNames(control: FormControl): {[s: string]: boolean} {
  if (this.forbiddenUsernames.indexOf(control.value) !== -1) {
    return {'nameIsForbidden': true};
  }
  // if validation successful you should return nothing or null
  return null;
}
```

```html
<span class="help-block"
  *ngIf="!signupForm.get('userData.username').valid && signupForm.get('userData.username').touched">
  <span 
    *ngIf="signupForm.get('userData.username').errors['nameIsForbidden']">This name is invalid!</span>
  <span
    *ngIf="signupForm.get('userData.username').errors['required']">This field is invalid!</span>
</span>
```

### Custom Async Validator

```typescript
// passing async validators as 3rd argument
'email': new FormControl(null, [Validators.required, Validators.email], this.forbiddenEmail), 
...
forbiddenEmail(control: FormControl): Promise<any> | Observable<any> {
  const promise = new Promise<any>((resolve, reject) => {
    setTimeout(() => {  // it may be http request, for example
      if (control.value === 'test@test.com') {
        resolve({'emailIsForbidden': true});
      } else {
        resolve(null);
      }
    }, 1500);
  });
  return promise;
}
```

## Array of Form Controls

```typescript
import { FormGroup, FormControl, FormArray, Validators } from '@angular/forms';

this.signupForm = new FormGroup({
  ...
  'hobbies': new FormArray([]),
});

onAddHobby() {
  const control = new FormControl(null, Validators.required);
  (<FormArray>this.signupForm.get('hobbies')).push(control);
}
```

```html
<div formArrayName="hobbies">
  <h4>Your Hobbies</h4>
  <button class="btn btn-default" type="button"
    (click)="onAddHobby()">Add Hobby</button>
  <div class="form-group"
    *ngFor="let hobbyControl of signupForm.get('hobbies').controls; let i = index">
    <input type="text" class="form-control" [formControlName]="i">
  </div>
</div>
```

## Reacting to Status or Value Changes

```typescript
ngOnInit() {
  ...
  this.signupForm.valueChanges.subscribe(
    (value) => console.log(value),
  );
  this.signupForm.statusChanges.subscribe(
    (status) => console.log(status),
  );
}
```

## Setting and Patching Values

```typescript
this.signupForm.setValue({
  'userData': {
    'username': 'andrey',
    'email': 'andrey@test.com'
  },
  'gender': 'male',
  'hobbies': [],
});

this.signupForm.patchValue({
  'userData': {
    'username': 'Andrey',
  },
});

onSubmit() {
  this.signupForm.reset({
    'gender': 'female',  // setting default value
  });
}
```

## Complete Example

```html
<div class="container">
  <div class="row">
    <div class="col-xs-12 col-sm-10 col-md-8 col-sm-offset-1 col-md-offset-2">
      <form [formGroup]="signupForm" (ngSubmit)="onSubmit()">
        <div formGroupName="userData">
          <div class="form-group">
            <label for="username">Username</label>
            <input
              type="text"
              id="username"
              formControlName="username"
              class="form-control">
              <!--
                There are two ways to use formControlName directive:
                  formControlName="username"
                  [formControlName]="'username'"
              -->
            <!-- Getting Access to Controls -->
            <span
              *ngIf="!signupForm.get('userData.username').valid && signupForm.get('userData.username').touched"
              class="help-block">
              <!-- Differnet messages -->
              <span *ngIf="signupForm.get('userData.username').errors['nameIsForbidden']">
                This name is invalid!</span>
              <span *ngIf="signupForm.get('userData.username').errors['required']">
                This field is required!</span>
            </span>
          </div>
          <div class="form-group">
            <label for="email">email</label>
            <input
              type="text"
              id="email"
              formControlName="email"
              class="form-control">
            <span
              *ngIf="!signupForm.get('userData.email').valid && signupForm.get('userData.email').touched"
              class="help-block">Please enter a valid email!</span>
          </div>
        </div>
        <div class="radio"
          *ngFor="let gender of genders">
          <label>
            <input type="radio"
              formControlName="gender"
              [value]="gender">{{ gender }}
          </label>
        </div>
        <div formArrayName="hobbies">
          <h4>Your Hobbies</h4>
          <button
            class="btn btn-default"
            type="button"
            (click)="onAddHobby()">Add Hobby</button>
          <div
            class="form-group"
            *ngFor="let hobbyControl of signupForm.get('hobbies').controls; let i = index">
            <input type="text" class="form-control" [formControlName]="i">
          </div>
        </div>
        <span
          *ngIf="!signupForm.valid && signupForm.touched"
          class="help-block">Please enter valid data!</span>
        <button class="btn btn-primary" type="submit">Submit</button>
      </form>
    </div>
  </div>
</div>
```

```typescript
import { Component, OnInit } from '@angular/core';
import { FormArray, FormControl, FormGroup, Validators } from '@angular/forms';
import { Observable } from 'rxjs';

@Component({
  selector: 'app-root',
  templateUrl: './app.component.html',
  styleUrls: ['./app.component.css']
})
export class AppComponent implements OnInit {
  genders = ['male', 'female'];
  signupForm: FormGroup;
  forbiddenUsernames = ['Chris', 'Anna'];

  constructor() {}

  ngOnInit() {
    /* Syncing HTML and Code
    *  You are not configuring your form in the template.
    *  You only synchronize it with your code with directives
    *  formControlName and formGroup.
    *  Then you configure it in your TypeScript code.
    */
    this.signupForm = new FormGroup({
      userData: new FormGroup({  // Grouping form controls
        username: new FormControl(null,
          [Validators.required, this.forbiddenNames.bind(this)]),
        email: new FormControl(null,
          [Validators.required, Validators.email],
          this.forbiddenEmails),  // async validators passed as 3rd argument
      }),
      gender: new FormControl('male'),  // default value passed
      // FormArray
      hobbies: new FormArray([]),
    });

    // Reacting to Status or Value changes
    this.signupForm.valueChanges.subscribe(
      (value) => console.log(value)
    );

    this.signupForm.statusChanges.subscribe(
      (status) => console.log(status)
    );

    // Setting and Patching Values
    this.signupForm.setValue({
      userData: {
        username: 'Max',
        email: 'max@test.com'
      },
      'gender': 'male',
      'hobbies': []
    });
    this.signupForm.patchValue({
      userData: {
        username: 'Anna',
      }
      // changes only username
      // other controls stay untouched
    });
  }

  // Submitting the form
  onSubmit() {
    console.log(this.signupForm.value.username);

    // Resetting the form
    this.signupForm.reset(
      // pass an object if you wish to set dafault value[s]
      { 'gender': 'male', }
    );
  }

  // FormArray
  onAddHobby() {
    const control = new FormControl(null, Validators.required);
    (<FormArray>this.signupForm.get('hobbies')).push(control);
  }

  // Custom Validator
  forbiddenNames(control: FormControl): {[s: string]: boolean} {
    if (this.forbiddenUsernames.indexOf(control.value) !== -1) {
      return {'nameIsForbidden': true};
    }
    return null;
  }

  // Custom Async Validator
  forbiddenEmails(control: FormControl): Promise<any> | Observable<any> {
    const promise = new Promise<any>((resolve, reject) => {
      setTimeout(() => {  // here may be http request, for example
        if (control.value === 'test@test.com') {
          resolve({'emailIsForbidden': true});
        } else {
          resolve(null);  // you must return null or nothing (not an object)
        }
      }, 1500);
    });
    return promise;
  }
}
```
