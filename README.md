# Angular Event Plugins

[![npm version](https://img.shields.io/npm/v/@tinkoff/ng-event-plugins.svg)](https://npmjs.com/package/@tinkoff/ng-event-plugins)
[![npm bundle size](https://img.shields.io/bundlephobia/minzip/@tinkoff/ng-event-plugins)](https://bundlephobia.com/result?p=@tinkoff/ng-event-plugins)
[![Build Status](https://travis-ci.com/TinkoffCreditSystems/ng-event-plugins.svg?branch=master)](https://travis-ci.com/TinkoffCreditSystems/ng-event-plugins)
[![Coverage Status](https://coveralls.io/repos/github/TinkoffCreditSystems/ng-event-plugins/badge.svg?branch=master)](https://coveralls.io/github/TinkoffCreditSystems/ng-event-plugins?branch=master)
[![angular-open-source-starter](https://img.shields.io/badge/made%20with-angular--open--source--starter-d81676?logo=angular)](https://github.com/TinkoffCreditSystems/angular-open-source-starter)

**@tinkoff/ng-event-plugins** is a tiny (1KB gzip) library for
optimizing change detection cycles for performance sensitive events
(such as _touchmove_, _scroll_, _drag_ etc.) and declarative
_preventDefault()_ and _stopPropagation()_.

## How to use

1. Add new providers to your app module:

    ```typescript
    import {NgModule} from '@angular/core';
    import {NG_EVENT_PLUGINS} from '@tinkoff/ng-event-plugins'; // <-- THIS

    @NgModule({
        bootstrap: [
            /*...*/
        ],
        imports: [
            /*...*/
        ],
        declarations: [
            /*...*/
        ],
        providers: NG_EVENT_PLUGINS, // <-- GOES HERE
    })
    export class AppModule {}
    ```

2. Use new modifiers for events in templates and in `@HostListener`:

    - `.stop` to call stopPropagation() on event
    - `.prevent` to call preventDefault() on event
    - `.silent` to call event handler outside Angular's `NgZone`

    For example:

    ```html
    <div (mousedown.prevent)="onMouseDown()">
        Clicking on this DIV will not move focus
    </div>
    ```

    ```html
    <div (click.stop)="onClick()">
        Clicks on this DIV will not bubble up
    </div>
    ```

    ```html
    <div (mousemove.silent)="onMouseMove()">
        Callbacks to mousemove will not trigger change detection
    </div>
    ```

3. You can also re-enter `NgZone` and trigger change detection, using `@shouldCall` decorator
   that takes a predicate function as argument:

```html
<div (scroll.silent)="onScroll($event.currentTarget)">
    Scrolling this DIV will only trigger change detection and onScroll callback if it is
    scrolled to bottom
</div>
```

```typescript
import {HostListener} from '@angular/core';
import {shouldCall} from '@tinkoff/ng-event-plugins';

export function scrollFilter(element: HTMLElement): boolean {
    return element.scrollTop === element.scrollHeight - element.clientHeight;
}

// ...

@shouldCall(scrollFilter)
@HostListener('init.onScroll', ['$event'])
onScroll(_element: HTMLElement) {
    this.someService.requestMoreData();
}
```

**IMPORTANT** You must couple `@shouldCall` with `@HostListener` for `init` event
as shown above until Angular 10 and **@tinkoff/ng-event-plugins** v.2.0 are released

> All examples above work the same when used with `@HostListener` and `CustomEvent`

## Important notes

-   Predicate is called with the same arguments as the decorated method and
    in the context of class instance (has access to `this`)

-   Decorated method will be called and change detection triggered if predicate
    returns `true`.

-   Predicates must be exported named function for AOT, arrow
    functions will trigger build error.

-   `.silent` modifier will not work with built-in keyboard pseudo-events,
    such as `keydown.enter` or `keydown.arrowDown` since Angular re-enters `NgZone`
    inside internal handlers.

## Demo

You can try this [interactive demo](https://codesandbox.io/s/github/TinkoffCreditSystems/ng-event-plugins/tree/master/projects/demo)

You can also read a this [detailed article](https://indepth.dev/supercharge-event-management-in-your-angular-application/) 
explaining how this library works

## Open-source

Do you also want to open-source something, but hate the collateral work?
Check out this [Angular Open-source Library Starter](https://github.com/TinkoffCreditSystems/angular-open-source-starter)
we’ve created for our projects. It got you covered on continuous integration,
pre-commit checks, linting, versioning + changelog, code coverage and all that jazz.
