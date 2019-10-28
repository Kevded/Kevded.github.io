---
description: >-
  Attendre que la callback enregistrer dans onAuthStateChanged soit appelé avant
  d'initialiser l'application. Permet d'éviter le déclenchement trop tôt d'une
  routeGuard redirection si User non connecté.
---

# Firebase Auth onAuthStateChanged avec Stenciljs

Date publication : 28/10/2019 

Solution pour éviter qu'au rechargement de la page que l'utilisateur soit rediriger vers la page de connexion. Alors que l'utilisateur est encore connecté. On attends juste que la callback soit au moins appelé une fois pour vérifier que l'utilisateur est bien à null ou présent. 

{% code-tabs %}
{% code-tabs-item title="app-root.tsx" %}
```typescript
import { Component, State, h, Prop, } from '@stencil/core';
import { User } from '../models/user';
import { PrivateRoute } from '../commons/utils';

declare var firebase: firebase.app.App

@Component({
  tag: 'app-root'
})
export class AppRoot {
  @State() user: User;
  
  componentWillLoad() {
    return new Promise((resolve) => {
      firebase.auth().onAuthStateChanged((user) => {
        this.injectUser(user);
        resolve();
      });
    })
  }

  injectUser = (user) => {
    if (user) {
      this.defaultProps = Object.assign({}, { user: user })

    } else {
      this.defaultProps =Object.assign({ user: undefined });
    }
  }

  render() {
    return (
      [
          <stencil-route-switch >
            <stencil-route componentProps={{user: this.user}} url='/' component='app-home' exact={true} />

            <stencil-route componentProps={{user: this.user}} url='/sign-in' component='app-sign-in' />
            <stencil-route componentProps={{user: this.user}} url='/sign-up' component='app-sign-up' />
          
            <PrivateRoute url='/profile' componentProps={{user: this.user}} component='app-profile' />

          </stencil-route-switch>
        </stencil-router>
      ]
    );
  }
}


```
{% endcode-tabs-item %}
{% endcode-tabs %}

{% code-tabs %}
{% code-tabs-item title="utils.tsx" %}
```typescript
import { h } from "@stencil/core";

/**
 * https://github.com/ionic-team/stencil-router/wiki/Enforce-authentication-for-some-routes 
*/
export const PrivateRoute = ({ component, ...props}: { [key: string]: any}) => {
    const Component = component;
    const redirectUrl = props.failureRedirect || '/sign-in';
  
    return (
      <stencil-route {...props} routeRender={
        (props: { [key: string]: any}) => {
          if (props.user) {
            return <Component {...props} {...props.componentProps}></Component>;
          }
          return <stencil-router-redirect url={redirectUrl}></stencil-router-redirect>
        }
      }/>
    );
  }
```
{% endcode-tabs-item %}
{% endcode-tabs %}



