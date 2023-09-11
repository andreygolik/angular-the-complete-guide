# Http Requests [DEPRECATED]

***Angular 6 is currently the latest version of Angular and it deprecates the Http-access method.***

*server.service.ts*

```typescript
import { Injectable } from '@angular/core';
import { Http, Headers, Response } from '@angular/http';
import { Observable, throwError } from 'rxjs';
import { map, catchError } from 'rxjs/operators';

@Injectable({
  providedIn: 'root'
})
export class ServerService {
  baseUrl = 'https://sandbox-f02c8.firebaseio.com';

  constructor(private http: Http) {}

  saveServers(servers: any[]): Observable<Response> {
    const url = this.baseUrl + '/servers.json';
    const headers = new Headers({ 'Content-Type': 'application/json' });
    return this.http.put(url, servers, { headers });
  }

  getServers(): Observable<any[]> {
    const url = this.baseUrl + '/servers.json';
    return this.http.get(url).pipe(
      map((response: Response) => {
        const data = response.json();
        // Example of data transformation
        for (const server of data) {
          server.name = 'FETCHED_' + server.name;
        }
        return data;
      }),
      catchError((error) => {
        console.log(error);
        return throwError('Something went wrong');
      })
    );
  }

  getAppName(): Observable<string> {
    const url = this.baseUrl + '/appName.json';
    return this.http.get(url).pipe(
      map((response: Response) => response.json())
    );
  }
}
```

*app.component.ts*

```typescript
import { Component } from '@angular/core';
import { ServerService } from './server.service';
import { Response } from '@angular/http';

@Component({
  selector: 'app-root',
  templateUrl: './app.component.html',
  styleUrls: ['./app.component.css']
})
export class AppComponent {
  appName = this.serverService.getAppName();
  // you have to use it in template with AsyncPipe
  // <h1>{{ appName | async }}</h1>

  servers = [
    {
      name: 'Testserver',
      capacity: 10,
      id: this.generateId()
    },
    {
      name: 'Liveserver',
      capacity: 100,
      id: this.generateId()
    }
  ];

  onAddServer(name: string) {
    this.servers.push({
      name: name,
      capacity: 50,
      id: this.generateId()
    });
  }

  constructor(private serverService: ServerService) {}

  private generateId() {
    return Math.round(Math.random() * 10000);
  }

  onSave() {
    this.serverService
      .saveServers(this.servers)
      .subscribe(
        (response: Response) => console.log(response),
        (error) => console.error(error),
      );
  }

  onGet() {
    this.serverService
      .getServers()
      .subscribe(
        (servers: any[]) => this.servers = servers,
        (error) => console.error(error),
      );
  }
}
```