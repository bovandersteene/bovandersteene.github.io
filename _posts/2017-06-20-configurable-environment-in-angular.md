---
title:  "Configure environment variable in Angular"
published: true
comments: true
author: Bo Vandersteene
categories: [angular]
tags: [angular, angular2, environment, configuration]
---

Your environment variables need to come from a different place than environment.prod.js or environment.dev.js, because they are variable.
The endpoints depends on data that is configured on the different backend systems.


# Add a config service
We need a config service, so we can send a request to an url, to provide us the information about the configuration.

```
@Injectable()
export class ConfigService { 
  config: Configuration;

  constructor(private http: Http) {
  }

  load() {
    return new Promise((resolve, reject) => {
          this.http.get('/assets/environment.json')
            .subscribe(config => {
                this.config = config.json();
                resolve(true);
              },
              _ => {
                console.error('Configuration failed to load');
                reject(true);
              });
        });
  }


}
```
The response of the configuration file needs to be of the type *application/json*.
Important here is to use **Promise**, in the configuration in the next section, we use **APP_INITIALIZER**, which need an Promise, Observable will not work in this case.

# Let your application use the service
In your *app.module.ts* you provide the information on the service.

```
export function loadConfig(config: ConfigService) {
  return () => config.load();
}

@NgModule({
    // Other module configuration
  providers: [
    ConfigService,
    {
      provide: APP_INITIALIZER,
      useFactory: loadConfig,
      deps: [ ConfigService, Http ],
      multi: true
    },
    // Other providers
]})
export class AppModule {
}
```
The configuration is loaded at App initialization time. Nothing else will be done until the configuration have been loaded successfull.

### APP_INITIALIZER and Promise
When you provide a loader on app initialization, which can causes delay f.e. a http call, where you need to wait unti the call is finished.
You need to use promises, to define when your call is finished or not. Angular does not provide app initialization with observables.

# Create a custom http service that use your configuration
You have configured custom endpoints on your server. These endpoints depends on the server you are working on.

```
@Injectable()
export class HttpService {

  constructor(private http: Http, 
              private configService: ConfigService) {
  }

  get(url: string): any { 
    return this.http.get(`${this.configService.config.api_url}${url}`)
      .map((response: Response) => response.json());
  }

```
You can use the configuration also in other services. Depending on what you provide in your configuration files.


# Versions
* Angular 4.2.2
* Typescript > 2