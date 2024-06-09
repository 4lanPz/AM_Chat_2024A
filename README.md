# Chat usando Ionic

Hacer un chat básico utilizando Ionic, Visual Studio Code y Android Studio

## Clonar
```bash
Git clone https://github.com/4lanPz/AM_Chat_2024A
```

## Pasos

- 1 Pre requisitos

Tener instalado Node.JS y Npm
Tener un IDE para customizar nuestro proyecto en este caso Visual Studio Code
Tener una cuenta de Firebase Authenticacion y sus credenciales
Tener Android Studio configurado
La lógica del login es la misma del proyecto: 
```bash
https://github.com/4lanPz/AM_Login_2024A
```
Estos se pueden descargar desde la pagina web oficial de dependiendo de el OS que estes utilizando.

- 2 Empezar el proyecto

Para empezar el proyecto hay que ejecutar el siguiente comando

```bash
npx ionic Start nombreproyecto blank --type=angular
```

En este nombreproyecto es el nombre que le vamos a poner a nuestro proyecto, por lo que se puede poner el
que tu quieras para tu proyecto.

En este tambien debemos elegir que módulos vamos a ocupar, en este caso vamos a ocupar "NGModules"

Al finalizar no es necesario tener una cuenta de Ionic, asi que eso podemos indicar que no y con eso nuestro
proyecto se ha creado

- 3 Navegar al directorio e intalación dependencias
Utilizando la consola CMD podemos ir a el directorio de nuestro proyecto con 
```bash
cd nombreproyecto
```
Dentro de esta carpeta tendremos que instalar los módulos necesarios para que se ejecute nuestro proyecto:

```bash
npm install
```
Después de instalar los módulos del proyecto, es necesario instalar firebase authentication, ya que este nos dara los servicios necesarios para poder generar la logica del login y registro
Para ello necesitamos ejecutar el siguiente comando
```bash
npm install firebase @angular/fire
npx ionic g service services/
npx ionic g page login
npx ionic g services chat
```
Estos lo que nos ayudan es a configurar nuestro Firebase que si no tenemos cuenta debemos registrarnos, generar una apikey web en authentication con correo y contraseña que necesitamos para que diferentes personas puedan logearse o registrarse para poder enviar mensajes
Para iniciar la configuracion de nuestro Firebase necesitamos estos comandos.
```bash
firebase login // si aun no inicias sesión
firebase init // configuración de firebase para nuestra aplicación
```
Cuando generemos nuestro apikey nos debe entregar algo como esto
```bash
firebaseConfig: {
    apiKey: "TU_API_KEY",
    authDomain: "TU_AUTH_DOMAIN",
    projectId: "TU_PROJECT_ID",
    storageBucket: "TU_STORAGE_BUCKET",
    messagingSenderId: "TU_MESSAGING_SENDER_ID",
    appId: "TU_APP_ID",
    measurementId: "TU_MEASUREMENT_ID"
  }
```

- 4 Funcionalidad
Para empezar a generar el código necesitamos realizar una importación de los modulos de autenticación, ene ste caso no se volverá a hacer el código, solo se reutilizará.

Después de toda la lógica del login debemos realizar el código del servicio del chat, dentro del archivo "chat.service.ts" agregamos todo lo que es la funcionalidad del chat, de tomar las credenciales de la persona que esté logeada para utilizar su nombre como identificador para mandar los mensajes
```bash
export class ChatService {
  currentUser: User | null = null;
  constructor(private afAuth: AngularFireAuth, private afs: AngularFirestore) {
    this.afAuth.onAuthStateChanged((user) => {
      if (user) {
        this.currentUser = { uid: user.uid, email: user.email };
      } else {
        this.currentUser = null;
      }
    });
  }
  async signup({ email, password }: { email: string, password: string }): Promise<any> {
    const credential = await this.afAuth.createUserWithEmailAndPassword(
      email,
      password
    );
  
    if (credential.user) {
      const uid = credential.user.uid;
      return this.afs.doc(
        `users/${uid}`
      ).set({
        uid,
        email: credential.user.email,
      })
    } else {
      throw new Error('User creation failed');
    }
  }
  signIn({ email, password }: { email: string, password: string }) {
    return this.afAuth.signInWithEmailAndPassword(email, password);
  }
  signOut(): Promise<void> {
    return this.afAuth.signOut();
  }
  addChatMessage(msg: string) {
    if (this.currentUser && this.currentUser.uid) {
      return this.afs.collection('messages').add({
        msg: msg,
        from: this.currentUser.uid,
        createdAt: serverTimestamp()
      });
    } else {
      throw new Error('No user is currently signed in');
    }
  }
  getChatMessages() {
    let users: any[] = [];
    return this.getUsers().pipe(
      switchMap(res => {
        users = res;
        return this.afs.collection('messages', ref => ref.orderBy('createdAt')).valueChanges({ idField: 'id' }) as Observable<Message[]>;
      }),
      map(messages => {
        // Get the real name for each user
        for (let m of messages) {
          m.fromName = this.getUserForMsg(m.from, users);
          if (this.currentUser && this.currentUser.uid) {
            m.myMsg = this.currentUser.uid === m.from;
          }
        }
        return messages
      })
    )
  }
  private getUsers() {
    return this.afs.collection('users').valueChanges({ idField: 'uid' }) as Observable<User[]>;
  }
  private getUserForMsg(msgFromId: string, users: User[]): string {
    for (let usr of users) {
      if (usr.uid == msgFromId) {
        return usr.email ? usr.email : 'Deleted';
      }
    }
    return 'Deleted';
  }
}
```
Ahora ya con el servicio ddl chat ya generado podemos pasar a las funcionalidades que se van a mostrar, para ello debemos empezar a editar el archivo "chat.page.ts" en donde solo deberemos agregar las funciones para enviar mensajes y si la persona cierra sesión
```bash
export class ChatPage implements OnInit {
  @ViewChild(IonContent) content: IonContent;
  messages: Observable<any[]>;
  newMsg = '';
  constructor(private chatService: ChatService, private router: Router) { }
  ngOnInit() {
    this.messages = this.chatService.getChatMessages();
  }
  sendMessage() {
    this.chatService.addChatMessage(this.newMsg).then(() => {
      this.newMsg = '';
      this.content.scrollToBottom();
    });
  }
  signOut() {
    this.chatService.signOut().then(() => {
      this.router.navigateByUrl('/', { replaceUrl: true });
    });
  }
```
Con las funcionalidades del servicio del chat y la funcionalidad de los mensajes, ya podemos generar nuestro HTML en donde vamos a llamar a todas estas funcionalidades para que se muestren en nuestra aplicación.
```bash
<ion-header>
  <ion-toolbar color="primary">
    <ion-title>Abrir chat</ion-title>
    <ion-buttons slot="end">
      <ion-button fill="clear" (click)="signOut()">
        <ion-icon name="log-out" slot="icon-only"></ion-icon>
      </ion-button>
    </ion-buttons>
  </ion-toolbar>
</ion-header>
<ion-content class="ion-padding">
  <ion-grid>
    <ion-row *ngFor="let message of messages | async">
      <ion-col size="9" class="message"
        [offset]="message.myMsg ? 3 : 0"
        [ngClass]="{ 'my-message': message.myMsg, 'other-message': !message.myMsg }">
        <b>{{ message.fromName }}</b><br>
        <span>{{ message.msg }}
        </span>
        <div class="time ion-text-right"><br>{{ message.createdAt?.toMillis() | date:'short' }}</div>
      </ion-col>
    </ion-row>
  </ion-grid>
</ion-content>
<ion-footer>
  <ion-toolbar color="light">
    <ion-row class="ion-align-items-center">
      <ion-col size="10">
        <ion-textarea autoGrow="true" class="message-input" rows="1" maxLength="500" [(ngModel)]="newMsg" >
        </ion-textarea>
      </ion-col>
      <ion-col size="2">
        <ion-button expand="block" fill="clear" color="primary" [disabled]="newMsg === ''"
          class="msg-btn" (click)="sendMessage()">
          <ion-icon name="send" slot="icon-only"></ion-icon>
        </ion-button>
      </ion-col>
    </ion-row>
  </ion-toolbar>
</ion-footer>
```
- 5 Ejecución
Para poder ejecutar nuestro proyecto simplemente necesitamos ejecutar el comando
```bash
npx ionic s
```
Este comando ejecutará nuestra aplicación de forma web, entonces podremos comprobar que la aplicación es totalmente funcional.
si se quiere ejecutar en android se necesitan 2 comandos
```bash
npx ionic build android
npx ionic capacitor open android
```
El primero comando empezará a generar los archivos necesarios para que nuestra aplicación tambien funcione en android, loque genera una carpeta android.
El segundo comando lo que hace es abrir este proyecto de Android en Android Studio, esto para poder hacer la emulación de la aplicación en android o emulación directamente en nuestro dispositivo Android a través de Wifi.
Un problema que suele ocurrir al momento de generar el build de Android es que no suele encontrar las credenciales de Firebase, para solucionar esto hay que pasar las credenciales de enviroment.ts y copiarlas tambien en enviroment.prod.ts

# Capturas
### Web
![image](https://github.com/4lanPz/AM_Chat_2024A/assets/117743495/359d0f07-56c2-43e0-ac26-f4957dbcb2bd)
![image](https://github.com/4lanPz/AM_Chat_2024A/assets/117743495/18e21bab-0de7-49a0-bebf-30df87834727)


### Android
![image](https://github.com/4lanPz/AM_Chat_2024A/assets/117743495/12f2046f-ec96-4346-8166-c874f64c62e4)
![image](https://github.com/4lanPz/AM_Chat_2024A/assets/117743495/ac4859af-8503-496b-9819-8013da66912d)



