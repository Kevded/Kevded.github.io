---
description: 'Advanced State management without Redux, Mobs, or Stencil State Tunnel.'
---

# StencilJS State Management Tips

Example with Stencil.js v0.16.x .

## Root Component

```typescript
import { Component } from '@stencil/core';

export interface User {
  name: string;
}

@Component({
  tag: 'app-root'
})
export class AppRoot {

  @State() user: User;

  signOut = () => {
      this.user = null;
    }
    
  signIn = () => {
      this.user = { name: "Pablo" }
    }
    
  render() {
    return (
      <div>
        <header>
          <h1>Stencil App Starter</h1>
        </header>

        { this.user && <span> Your are connected {this.user.name}. </span> }
        <main>
          <stencil-router>
            <stencil-route-switch scrollTopOffset={0}>
              <stencil-route url='/' component='app-home' exact={true} />
              <stencil-route url='/profile' component='app-profile' componentsProps={{user: this.user, signIn : this.signIn, signOut : this.signout }}/>
            </stencil-route-switch>
          </stencil-router>
        </main>
      </div>
    );
  }
}

```

## Profile Page

```text

import { Component, Prop } from '@stencil/core';

@Component({
  tag: 'app-profile'
})
export class AppProfile {
  @Prop() user: User;
  @Prop() signIn: Function;
  @Prop() signOut: Function;

  render() {
      return (
        <div class="app-profile">
          <p>
            Hello! My name is {this.user && this.user.name}. My name was passed in
            through a @Prop!
          </p>
          
          <button onClick={this.signIn}>Sign In</button>
          <button onClick={this.signOut}>Sign Out</button>
        </div>
      );
  }
}
```



Resources :

Real world example application built with Stencil.js :

[https://github.com/hcavalieri/stencil-realworld-app](https://github.com/hcavalieri/stencil-realworld-app)

