# Angular Block

[![npm version](https://badge.fury.io/js/%40efk3%2Fangular-block.svg)](https://badge.fury.io/js/%40efk3%2Fangular-block)
[![Build Status](https://travis-ci.org/Efk3/angular-block.svg?branch=master)](https://travis-ci.org/Efk3/angular-block)
[![codecov](https://codecov.io/gh/Efk3/angular-block/branch/master/graph/badge.svg)](https://codecov.io/gh/Efk3/angular-block)

Simple and customizable Angular block module for block/busy effects/loading indicators.

## Installation

Install package:

`npm install @efk3/angular-block`

Set up in your NgModule:

```typescript
import { NgModule } from '@angular/core';
import { BlockModule, BLOCK_DEFAULT_GLOBAL_COMPONENT, BLOCK_DEFAULT_SPECIFIED_COMPONENT } from '@efk3/angular-block';

@NgModule({
  imports: [BlockModule.forRoot()],
  declarations: [YourBlockerComponent, YourTargetSpecificBlockerComponent],
  entryComponents: [YourBlockerComponent, YourTargetSpecificBlockerComponent],
  providers: [
    { provide: BLOCK_DEFAULT_GLOBAL_COMPONENT, useValue: YourBlockerComponent },
    { provide: BLOCK_DEFAULT_SPECIFIED_COMPONENT, useValue: YourTargetSpecificBlockerComponent },
  ],
})
export class YourModule {}
```

## Configuration

The module is configurable by providers.

The component provided in `BLOCK_DEFAULT_GLOBAL_COMPONENT` will be used as default blocker component when the target is not specified.

The component provided in `BLOCK_DEFAULT_SPECIFIED_COMPONENT` will be used as default blocker component when the target is specified. If you do not provide this then the `BLOCK_DEFAULT_GLOBAL_COMPONENT` will be used.

If there is no default component provided and the `block` method called without `component` property then the service will throw an error.

## Usage

You can block the application or an element manually, by subscription, by promise and by observable.

If the target is not specified then the `ApplicationRef` will be used as the target and the blocker component will be appended to the `document.body`.

If you call `block` multiple times for the same target then the component will be appended only once but the service will store the number of blocks. The number of blocks is grouped by the target. The appended component will be removed when the block count is zero.

### Manual block example

```typescript
import { BlockService } from '@efk3/angular-block';

@Component({...})
export class MyComponent implements OnInit {
  constructor(private blockService: BlockService) {}

  public ngOnInit(): void {
    this.blockService.block();

    setTimeout(() => {
      this.blockService.unblock();
    }, 3000);
  }
}
```

### Block by a subscription example

The block will be initialized instantly and will be removed when the unsubscription happens.

```typescript
import { BlockService } from '@efk3/angular-block';

@Component({...})
export class MyComponent implements OnInit {
  constructor(private blockService: BlockService, private http: HttpClient) {}

  public ngOnInit(): void {
    const subscription = this.http.get('...').subscribe();
    this.blockService.block(subscription);
  }
}
```

### Block by an observable example

The block will be initialized when the first subscription happens to the observable. The `block` method will wrap the original observable so you have to use the returned observable to make the service work. The block will be removed automatically when the observable completes or throws an error.

```typescript
import { BlockService } from '@efk3/angular-block';

@Component({...})
export class MyComponent implements OnInit {
  constructor(private blockService: BlockService) {}

  public ngOnInit(): void {
    const subject = new Subject();
    this.blockService.block(subject.asObservable()).subscribe(() => {
      // logic
    });
  }
}
```

### Block by a promise example

The block will be initialized instantly and will be removed when the promise returns or throws an error.  
The `block` method returns the original promise so you can chain the call if you want.

```typescript
import { BlockService } from '@efk3/angular-block';

@Component({...})
export class MyComponent implements OnInit {
  constructor(private blockService: BlockService) {}

  public ngOnInit(): void {
    const promise = new Promise<any>(() => {
      // logic
    });

    this.blockService.block(promise);
    promise.then(() => {
      // logic
    });
  }
}
```

### Directive driven block

You can set up the block by using the block directive.

```typescript
<div [k3Block]="trigger" [k3BlockerComponent]="BlockerComponent" [k3BlockData]="blockData">
</div>
```

The trigger can be a subscription, an observable or a promise. The blocker component can be specified with the `k3BlockerComponent` input. The data for blocker component can be specified with the `k3BlockData` input.

**Important** to know that the block will be triggered every time when the `k3Block` input changes. If you want to replace e.g. an observable as a trigger then first you have to make the current observable complete or the block will not be removed.

### Block input

The `block` method overloaded with multiple types for easier work.

If you call it only with a trigger then it works like you don't specified anything but the `trigger` in the input and returns the given `trigger`.

The other call version is for more specified configuration, in this version you can use the following properties:

* `trigger` - You can use types described above as a trigger.
* `target` - The blocker component will be appended to the given target(s). Simple and multiple targets (as an array) are accepted. `ApplicationRef`, `ViewContainerRef` and `ElementRef` types are supported as target.
* `data` - This object will be injected into the blocker component.
* `component` - You can specify the blocker component. If you do not set this property then the globally configured components will be used.

In this method also the given trigger will be returned.

### Blocker component

You have to create your own blocker component. There is no requirement for this component but you have to add it to the `entryComponents` too in your module because it will be created at runtime.

The blocker component will receive the following custom injections.

#### Get content of the `data` property of input

```typescript
import { BLOCK_DATA } from '@efk3/angular-block'

@Inject(BLOCK_DATA) public data: any
```

#### Get the number of blockers for target:

```typescript
import { BLOCK_BLOCKER_COUNT } from '@efk3/angular-block';

@Inject(BLOCK_BLOCKER_COUNT) public blockerCount: Observable<number>
```

#### Get the target of block

```typescript
import { BLOCK_TARGET, Target } from '@efk3/angular-block';

@Inject(BLOCK_TARGET) public target: Target
```

### Get the blocker count

You can get the blocker count for a target with this method:

```typescript
getBlockerCount(target?: Target): Observable<number>
```

### Get the blocker component

You can get the blocker component for a target with this method:

```typescript
getBlockerComponent(target?: Target): ComponentRef<any>
```

### Clear blocks

If you have to manage the blocks manually and you don't want to care about how much time you used the block then you can use the `clearBlock`
method which removes every block from the given target(s).

```typescript
resetBlock(target?: Target): void
```

If you call this method then all of the blocks which are started by a trigger will be invalidated.
Event if they expire they will not unblock the target(s) if you blocked them again in the meantime.

## Example

You can see a full example on [stackblitz][1].

[1]: https://stackblitz.com/edit/efk3-angular-block-example
