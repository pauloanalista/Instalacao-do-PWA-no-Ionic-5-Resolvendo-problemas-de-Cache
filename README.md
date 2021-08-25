# Criando-PWA-no-Ionic-5
Passos para transformar sua aplicação em um PWA usando Ionic Framework 5

## Instalação do PWA no Ionic 5
#### 1 - Leia a documentação (https://ionicframework.com/docs/angular/pwa)
#### 2 - Rode o comando abaixo para adicionar suporte ao PWA no Ionic 5
```sh
ng add @angular/pwa
```
Será adicionado alguns arquivos no seu projeto
- src/manifest.webmanifest
- assets/icons

#### 3 - Modifique os icones em assets/icons (modifique as imagens)
#### 4 - Altere o manifesto. ex:
```sh
{
  "name": "Manutenção Gps",
  "short_name": "Manutenção Gps",
  "theme_color": "#1976d2",
  "background_color": "#fafafa",
  "display": "fullscreen",
  "scope": "./",
  "start_url": "./",
  "lang": "pt",
  "orientation": "portrait",
  "description": "Controle de equipamentos gps",
  "icons": [
    {
      "src": "assets/icons/icon-72x72.png",
      "sizes": "72x72",
      "type": "image/png",
      "purpose": "maskable any"
    },
    {
      "src": "assets/icons/icon-96x96.png",
      "sizes": "96x96",
      "type": "image/png",
      "purpose": "maskable any"
    },
    {
      "src": "assets/icons/icon-128x128.png",
      "sizes": "128x128",
      "type": "image/png",
      "purpose": "maskable any"
    },
    {
      "src": "assets/icons/icon-144x144.png",
      "sizes": "144x144",
      "type": "image/png",
      "purpose": "maskable any"
    },
    {
      "src": "assets/icons/icon-152x152.png",
      "sizes": "152x152",
      "type": "image/png",
      "purpose": "maskable any"
    },
    {
      "src": "assets/icons/icon-192x192.png",
      "sizes": "192x192",
      "type": "image/png",
      "purpose": "maskable any"
    },
    {
      "src": "assets/icons/icon-384x384.png",
      "sizes": "384x384",
      "type": "image/png",
      "purpose": "maskable any"
    },
    {
      "src": "assets/icons/icon-512x512.png",
      "sizes": "512x512",
      "type": "image/png",
      "purpose": "maskable any"
    }
  ]
}
```
#### Gerando seu projeto com suporte PWA para produção
Rode o comando para gerar os arquivos de produção
```sh
ionic build --prod
```

#### Problemas com cache
É muito comum você ter problemas com o cache, ou seja, quando sobe uma nova versão não é atualizado para o cliente.

Para resolver este problema é necessário usar SwUpdate, com ele é possível você identificar uma nova atualização e notificar o cliente.
ex:
```typescript
import { Component, OnInit } from '@angular/core';
import { AlertController, NavController, Platform, ToastController } from '@ionic/angular';
import { SwUpdate } from '@angular/service-worker';

@Component({
  selector: 'app-root',
  templateUrl: 'app.component.html',
  styleUrls: ['app.component.scss'],
})
export class AppComponent implements OnInit {
  public appPages = [
    { title: 'Trocar Id Equipamento', url: 'trocar-id-equipamento', icon: 'bus' },
    { title: 'Consultar Comunicação', url: 'consultar-comunicacao', icon: 'radio' },
  ];
  public labels = ['Family', 'Friends', 'Notes', 'Work', 'Travel', 'Reminders'];
  private promptInstallEvent;
  private toast: HTMLIonToastElement;
  selectedIndex: any;
  constructor(
    private alertCtrl: AlertController,
    private navCtrl: NavController,
    private platform: Platform,
    private swUpdate: SwUpdate,
    private toastCtrl: ToastController,
  ) {
    this.initializeApp();

  }



  async sair() {
    const alert = await this.alertCtrl.create({
      cssClass: 'my-custom-class',
      header: 'Atenção!',
      message: 'Tem certeza que sair da aplicação?',
      buttons: [
        {
          text: 'Cancelar',
          role: 'cancel',
          cssClass: 'secondary',
          handler: (blah) => {

          }
        }, {
          text: 'Sair',
          handler: () => {
            sessionStorage.removeItem('token');
            this.navCtrl.navigateRoot('');
          }
        }
      ]
    });

    await alert.present();


  }

  initializeApp() {
    
  }
  async ngOnInit() {

    console.log(`Runing app ${this.isPWAInstalled ? 'standalone' : 'in browser'}`);

    this.swUpdate.available.subscribe(async event => {

      console.log('current version is', event.current);
      console.log('available version is', event.available);

      if (event.current !== event.available) {
        const alert = await this.alertCtrl.create({
          header: 'Oba, Temos Novidades!',
          subHeader: 'Há uma nova versão disponível da aplicação.',
          message: 'Deseja atualizar agora?',
          buttons: [
            {
              text: 'Instalar',
              handler: () => { this.swUpdate.activateUpdate(); }
            },
            'Mais tarde'
          ]
        });
        alert.present();
      }
    });

    this.swUpdate.activated.subscribe(event => {
      console.log('old version was', event.previous);
      console.log('new version is', event.current);
    });

    await this.platform.ready();

    if (!this.isMobile) {
      this.checkForUpdate();
      if (!this.isPWAInstalled) {
        this.listenForInstallEvent();
      }
    }

    console.log('swUpdate.isEnabled: ' + this.swUpdate.isEnabled);
    //this.swUpdate.checkForUpdate();
  }

  private listenForInstallEvent() {
    window.addEventListener('beforeinstallprompt', async (e) => {
      e.preventDefault();
      this.promptInstallEvent = e;

      setTimeout(() => {
        this.suggestInstall();
      }, 5000);
    });
  }
  private async suggestInstall() {
    this.toast = await this.toastCtrl.create({
      message: 'Você pode usar este aplicativo offline',
      buttons: [{
        text: 'Baixar',
        handler: () => { this.installPWA(); },
      }, {
        text: '',
        icon: 'close'
      }],
      duration: 0,
    });
    this.toast.present();
  }

  private installPWA() {
    this.toast.dismiss();
    // Show the prompt
    this.promptInstallEvent.prompt();
    // Wait for the user to respond to the prompt
    this.promptInstallEvent.userChoice
      .then((choiceResult) => {
        if (choiceResult.outcome === 'accepted') {
          console.log('User accepted the A2HS prompt');
        } else {
          console.log('User dismissed the A2HS prompt');
        }
        this.promptInstallEvent = null;
      });
  }

  get isMobile() {
    return this.platform.is('mobile');
  }
  get isPWAInstalled(): boolean {
    return window.matchMedia('(display-mode: standalone)').matches || (window.navigator as any).standalone;
  }

  async checkForUpdate() {
    console.log('Check for updates');
    try {
      await this.swUpdate.checkForUpdate();
    } catch (e) {
      console.debug('service worker not available');
    }
  }
}

```






