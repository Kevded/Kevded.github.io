---
description: Dev tips in Stencil.js
---

# Ionic 4 & Stencil.js

## Use Ionic 4 Controller in Stencil.js

```
 @Prop({ connect: 'ion-toast-controller' }) toastCtrl: HTMLIonToastControllerElement;
 
 async showMessage(){
   const toast = await this.toastCtrl.create({
     message: 'Hello World',
     showCloseButton: true,
     closeButtonText: 'Close',
     position: 'middle'
     });
     await toast.present();
 }
```

Example: [https://github.com/ionic-team/ionic-stencil-conference-app](https://github.com/ionic-team/ionic-stencil-conference-app)

