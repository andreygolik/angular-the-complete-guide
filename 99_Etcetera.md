# CLI

Create new *clean* project: 

```bash
sudo npm i -g @angular/cli

ng new --skip-git --skip-install --skip-tests --style scss --routing <project-name>
```

Generate component/service/etc.:

```bash
# component
ng g c auth/signup --spec false

# service
ng g s my-service
```

Build for Prod:

```bash
ng build --prod --aot
```

# Debuging

https://augury.angular.io/ - a Google Chrome Dev Tools extension for debugging Angular applications.
