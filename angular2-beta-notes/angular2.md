Angular 2-beta
=========

## Using TypeScript

### Architecture overview

![Angular 2 Architecture](https://angular.io/resources/images/devguide/architecture/overview2.png)

1. Module
2. Component
3. Template
4. Metadata
5. Data Binding
6. Service
7. Directive
8. Dependency Injection

> Main building block is **module**

* Angular apps are composed of modules.
* Modules export things — classes, function, values — that other modules import.
* We prefer to write our application as a collection of modules, each module exporting one thing.


For every **component**, which role is to prepare and serve view a templates we import it from core

```javascript
import {Component} from 'angular2/core';
```

The `angular2/core` library is the primary Angular library module from which we get most of what we need.

There are other important Angular library modules too such as `angular2/common`, `angular2/router`, and `angular2/http`

![Library Module](https://angular.io/resources/images/devguide/architecture/library-module.png)

More about [modules, barrels and bundles](https://github.com/angular/angular/blob/master/modules/angular2/docs/bundles/overview.md)

___
### What your app should have?

Every application have this class `app/app.componet.ts`:
```javascript
export class AppComponent { }
```

It would be nice to make it available to the browser using `app/main.ts`

```javascript
import {bootstrap} from 'angular2/platform/browser'
import {AppComponent} from './app.component';

bootstrap(AppComponent);
```

#### Component

The *Component* controls a patch of screen real estate that we could call a *view*. The shell at the application root with navigation links, that list of heroes, the hero editor ... they're all **views controlled by Components**.

We define a Component's application logic - what it does to support the view - inside a class. The class interacts with the view through an API of properties and methods.

Another Component:

```javascript
export class HeroListComponent implements OnInit {
  constructor(private _service: HeroService){ }
  heroes:Hero[];
  selectedHero: Hero;
  ngOnInit(){ // optional Lifecycle Hook
    this.heroes = this._service.getHeroes();
  }
  selectHero(hero: Hero) { this.selectedHero = hero; }
}
```

Now the template of `HeroListComponent` using *template syntax*:
```html
<h2>Hero List</h2>
<p><i>Pick a hero from the list</i></p>
<div *ngFor="#hero of heroes" (click)="selectHero(hero)">
  {{hero.name}}
</div>
<hero-detail *ngIf="selectedHero" [hero]="selectedHero"></hero-detail>
```

The `<hero-detail>` tag is a custom element representing the `HeroDetailComponent`

![Templates](https://angular.io/resources/images/devguide/architecture/component-tree.png)

#### Angular Metadata

> Metadata tells Angular how to process a class.

We tell Angular that `HeroListComponent` is a component by attaching *metadata* to the class.

```javascript
@Component({
  selector:    'hero-list',
  templateUrl: 'app/hero-list.component.html',
  directives:  [HeroDetailComponent],
  providers:   [HeroService]
})
export class HeroesComponent { ... }
```

* `selector` - a css selector `<hero-list></hero-list>` - where Angular inserts an instance of the `HeroListComponent` view between tags.

* `templateUrl` - where the template is or `template` and `styles` when it is within `@Component` metadata

* `directives` - an array of the Components or Directives that this template requires.

* `providers` - an array of dependency injection providers for services that the component requires.

We apply other metadata decorators in a similar fashion to guide Angular behavior. The `@Injectable`, `@Input`, `@Output`, `@RouterConfig` are a few of the more popular decorators we'll master as our Angular knowledge grows.

#### Data Binding

Example of data binding:

```html
<div>{{hero.name}}</div> <!-- interpolation -->
<hero-detail [hero]="selectedHero"></hero-detail> <!-- property binding -->
<div (click)="selectHero(hero)"></div> <!-- event binding -->

<input [(ngModel)]="hero.name"> <!-- two-way data binding -->
```

*Data Binding pyramid*:

![Data Binding](https://angular.io/resources/images/devguide/architecture/databinding.png)

![Data Binding, Template to Component](https://angular.io/resources/images/devguide/architecture/component-databinding.png)

![Data Binding, parent and child components](https://angular.io/resources/images/devguide/architecture/parent-child-binding.png)

#### The Directive

##### Structural

*Directive alter layout* by adding, removing, and replacing elements in DOM.

```html
<div *ngFor="#hero of heroes"></div> <!-- repeats div for every hero -->
<hero-detail *ngIf="selectedHero"></hero-detail> <!-- display element if is true -->
```

##### Attribute directive

The `ngModel` directive, which implements two-way data binding, is an example of an **attribute directive**.

```html
<input [(ngModel)]="hero.name">
```

It modifies the behavior of an existing element (typically an `<input>`) by setting its *display value* property and responding to *change events*.

Angular ships with a small number of other directives that either alter the layout structure (e.g. `ngSwitch`) or modify aspects of DOM elements and components (e.g. `ngStyle` and `ngClass`).

#### The Service

"Service" is a broad category encompassing any value, function or feature that our application needs.

Almost *anything can be a service*. A service is typically a class with a narrow, well-defined purpose. It should *do something specific* and do it well.

Examples include:

* logging service
* data service
* message bus
* tax calculator
* bapplication configuration

Some of the services can be loggers like `app/logger.service.ts`

```javascript
export class Logger {
  log(msg: any)   { console.log(msg); }
  error(msg: any) { console.error(msg); }
  warn(msg: any)  { console.warn(msg); }
}
```

Or fetching data, `app/hero.service.ts`

```javascript
export class HeroService {
  constructor(
    private _backend: BackendService,
    private _logger: Logger) { }

  private _heroes:Hero[] = [];

  getHeroes() {
    this._backend.getAll(Hero).then( (heroes:Hero[]) => {
      this._logger.log(`Fetched ${heroes.length} heroes.`);
      this._heroes.push(...heroes); // fill cache
    });
    return this._heroes;
  }
}
```

> Services are everywhere.

Our components are big consumers of services. They depend upon services to handle most chores. They don't fetch data from the server, they don't validate user input, they don't log directly to the console. They delegate such tasks to services.

A component's job is to enable the user experience and nothing more. It mediates between the view (rendered by the template) and the application logic (which often includes some notion of a "model"). A good component presents properties and methods for data binding. It delegates everything non-trivial to services.

Angular doesn't enforce these principles. It won't complain if we write a "kitchen sink" component with 3000 lines.

Angular does help us follow these principles ... by making it easy to factor our application logic into services and make those services available to components through dependency injection.

#### Dependency Injection

![Dependency Injection](https://angular.io/resources/images/devguide/architecture/dependency-injection.png)

"Dependency Injection" is a way to supply a new instance of a class with the fully-formed dependencies it requires. Most dependencies are services. *Angular* uses *dependency injection* to provide new *components* with the services they need.


In *TypeScript*, *Angular* can tell which *services* a *component* needs by looking at the types of its *constructor parameters*. For example, the constructor of our `HeroListComponent` needs the `HeroService`:

`app/hero-list.component.ts` constructor:

```javascript
constructor(private _service: HeroService){ }
```

> Avra Cadavra

When Angular creates a component, it first asks an Injector for the services that the component requires.

An Injector maintains a container of service instances that it has previously created. If a requested service instance is not in the container, the injector makes one and adds it to the container before returning the service to Angular. When all requested services have been resolved and returned, Angular can call the component's constructor with those services as arguments. This is what we mean by dependency injection.

The process of HeroService injection looks a bit like this:

![Dependency Injection Process](https://angular.io/resources/images/devguide/architecture/injector-injects.png)

> Recipe

If the Injector doesn't have a HeroService, how does it know how to make one?

In brief, we must have previously registered a provider of the HeroService with the Injector. A provider is something that can create or return a service, typically the service class itself.

We can register providers at any level of the application component tree. We often do so at the root when we bootstrap the application so that the same instance of a service is available everywhere.

Code at `app/main.ts`:

```javascript
bootstrap(AppComponent, [BackendService, HeroService, Logger]);
```

Or code at component level `app/hero-list.component.ts`:

```javascript
@Component({
  providers:   [HeroService]
})
export class HeroesComponent { ... }
```

... in which case we get a *new instance of the service* with *each new instance of that component*.

The points to remember are:

* dependency injection is wired into the framework and used everywhere.
* the Injector is the main mechanism.
 * an injector maintains a container of service instances that it created.
 * an injector can create a new service instance using a provider.
* a provider is a recipe for creating a service.
* we register providers with injectors.

---

### View on [Angular.io](https://angular.io/docs/ts/latest/guide/architecture.html)

1. [Module](https://angular.io/docs/ts/latest/guide/architecture.html#module)
2. [Component](https://angular.io/docs/ts/latest/guide/architecture.html#component)
3. [Template](https://angular.io/docs/ts/latest/guide/architecture.html#template)
4. [Metadata](https://angular.io/docs/ts/latest/guide/architecture.html#metadata)
5. [Data Binding](https://angular.io/docs/ts/latest/guide/architecture.html#data-binding)
6. [Service](https://angular.io/docs/ts/latest/guide/architecture.html#service)
7. [Directive](https://angular.io/docs/ts/latest/guide/architecture.html#directive)
8. [Dependency Injection](https://angular.io/docs/ts/latest/guide/architecture.html#dependency-injection)

---

### Animations

A forthcoming animation library makes it easy for developers to animate component behavior without deep knowledge of animation techniques or css.

---

### Bootstrap

A method to configure and launch the root application component.

---

###Change Detection

Learn how Angular decides that a component property value has changed and when to update the screen. Learn how it uses zones to intercept asynchronous activity and run its change detection strategies.

---

### Component Router

With the Component Router service, users can navigate a multi-screen application in a familiar web browsing style using URLs.

[View on Angular.io](https://angular.io/docs/ts/latest/guide/router.html)

---

### Events

The DOM raises events. So can components and services. Angular offers mechanisms for publishing and subscribing to events including an implementation of the [RxJS Observable](https://github.com/zenparsing/es-observable) proposal.

---

### Forms

Support complex data entry scenarios with HTML-based validation and dirty checking.

[View on Angular.io]([View on Angular.io](https://angular.io/docs/ts/latest/guide/router.html)

---

### HTTP

Communicate with a server to get data, save data, and invoke server-side actions with this Angular HTTP client.

---

### Lifecycle Hooks

We can tap into key moments in the lifetime of a component, from its creation to its destruction, by implementing the "Lifecycle Hook" interfaces.

[View on Angular.io](https://angular.io/docs/ts/latest/guide/lifecycle-hooks.html)

---

### Pipes

Services that transform values for display. We can put pipes in our templates to improve the user experience. For example, this currency pipe expression,

[View on Angular.io](https://angular.io/docs/ts/latest/guide/pipes.html)

```javascript
price | currency:'USD':true
displays a price of "42.33" as $42.33.
```

---

### Testing

Angular provides a testing library for "unit testing" our application parts as they interact with the Angular framework.

[View on Angular.io](https://angular.io/docs/ts/latest/testing/index.html)

---
> Created by [neosb](mailto:simone@neosb.net) using [Angular.io](https://angular.io) docs developed by [Google, Inc.](https://google.com) <br>
Copyrights &copy; 2016 under [CC 4.0](http://creativecommons.org/licenses/by/4.0/) license by neosb and Google