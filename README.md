# Angular ClientPanel App
This project is the result of my code-along to the ClientPanel App Project in [Angular 4 Front to Back](https://www.udemy.com/angular-4-front-to-back/) by *Brad Traversy*.  He is an excellent teacher, and I highly recommend taking his course to learn Angular!

<p align="center">
    <img width="500" height="332" src="./src/assets/img/homepage.png"><br>
</p>

## Versions Used
* Angular CLI v1.6.3
* Angular v5.1.3
* Angular Fire 2 v5.0.0-rc.4
* Firebase CLI v3.16.0
* Firebase v4.8.0

# Changes to the Original Project
### Major
1. `Cloud Firestore` used in place of the `Realtime Database` (see details below)

### Minor
1. Modified `updateBalance()` in `client-details.component.ts`.  When editing the balance, black text will now show up instead of red text if the user has no balance after pressing the `Update` button.  In the original project, the color was updated only after the user refreshed the page.
    ```typescript
    updateBalance(id: string) {
      if (this.client.balance <= 0) this.hasBalance = false;  // this line was added
      this.clientService.updateClient(this.id, this.client);
      this.flashMessagesService.show('Balance Updated', { cssClass: 'alert-success', timeout: 4000 });
      this.router.navigate([`/client/${this.id}`]);
    }
    ```

1. Rearranged the file structure of components, services, models, and guards into four separate modules: `Clients`, `Core`, `Routing`, and `Shared`.

## Cloud Firestore (Database)
Cloud Firestore is a flexible, scalable NoSQL cloud database that is used in this project, replacing the Realtime Database used in the original course.  The database is structured as a collection that contains individual documents, and is used 1. to display all clients and 2. to implement the ability to create, read, update, and delete an individual client, also known as CRUD functionality.  Cloud Firestore is accessed using the assistance of Angular Fire 2.

### Setting up Cloud Firestore in the Angular application
1. Import `AngularFire2`, `AngularFirestore`, and the environment file containing the `firebase environment` into `app.module.ts` (configured into `shared.module.ts` in this application)
    ```typescript
    ...
    import { AngularFireModule } from 'angularfire2';
    import { AngularFirestoreModule } from 'angularfire2/firestore';
    import { environment } from '../environments/environment';
    ...

    @NgModule({
      imports: [
        ...
        AngularFireModule.initializeApp(environment.firebase, 'clientpanel'),
        AngularFireAuthModule,
        AngularFirestoreModule,
        ...
      ],
    ```

2. Import `AngularFirestore`, `AngularFirestoreCollection`, and `AngularFirestoreDocument` into `client.service.ts`
    ```typescript
    import { AngularFirestore, AngularFirestoreCollection, AngularFirestoreDocument } from 'angularfire2/firestore';

    @Injectable()
    export class ClientService {

      clientsCollection: AngularFirestoreCollection<Client>;
      clientDoc: AngularFirestoreDocument<Client>;
      clients: Observable<Client[]>;
      client: Observable<Client>;

      constructor(private afs: AngularFirestore) {
        this.clientsCollection = afs.collection<Client>('clients');
      }
    ```

### Implementing CRUD Functionality
* Display all clients
    ```typescript
      // clients.component.ts
      ngOnInit() {
          this.clientService.getClients().subscribe((clients: Client[]) => {
            this.clients = clients;
            ...
          });
        }
    ```

    ```typescript
      // client.service.ts
      getClients(): Observable<Client[]> {
        this.clients = this.clientsCollection.snapshotChanges().map(actions => {
          return actions.map(a => {
            const data = a.payload.doc.data() as Client;
            const id = a.payload.doc.id;
            return { id, ...data };
          });
        });
        return this.clients;
      }
    ```

* Create a client
    ```typescript
      // add-client.component.ts
      onSubmit({value, valid}: {value: Client, valid: boolean}) { // Uses value and valid from template-driven form
        ...
        if (!valid) {
          ... // Runs if form is not valid
        } else {
          this.clientService.newClient(value);
          ...
        }
      }
    ```

    ```typescript
      // client.service.ts
      newClient(client: Client) {
        this.clientsCollection.add(client);
      }
    ```

* Read a client (Get Client Details)
    ```typescript
      // client-details.component.ts
      ngOnInit() {
        this.id = this.route.snapshot.params['id'];
        this.clientService.getClient(this.id).subscribe((client) => {
          if (client === null) return;    // Handles console.log error when the client is deleted
          if (client.balance > 0) this.hasBalance = true;
          this.client = client;
        });
      }
    ```

    ```typescript
      // client.service.ts 
      getClient(id: string): Observable<Client> {
        this.clientDoc = this.afs.doc<Client>(`clients/${id}`);
        this.client = this.clientDoc.valueChanges();
        return this.client;
      }
    ```

* Update a client
    ```typescript
      // edit-client.component.ts
      onSubmit({ value, valid }: { value: Client, valid: boolean }) { // Uses value and valid from template-driven form
        if (!valid) {
          ... // Runs if form is not valid
        } else {
          this.clientService.updateClient(this.id, value);
          ...
        }
    ```

    ```typescript
      // client.service.ts
      updateClient(id: string, client: Client) {
        this.clientDoc = this.afs.doc<Client>(`clients/${id}`);
        this.clientDoc.update(client);
      }
    ```

* Delete a client
    ```typescript
      // clients.component.ts
      onDeleteClick() {
        if (confirm('Are you sure to delete?')) {
          this.clientService.deleteClient(this.id);
          ...
        }
    ```

    ```typescript
      // client.service.ts
      deleteClient(id: string) {
        this.clientDoc = this.afs.doc<Client>(`clients/${id}`);
        this.clientDoc.delete();
      }
    ```

# Cloning the Project for Personal Use
## Installing the Project
To begin working with this project, perform the following tasks:

1. Clone this repo: `https://github.com/Stanza987/angular-client-panel.git`
1. `cd` into the folder of the cloned repo
1. Run `yarn install` to install dependencies
1. Add your Firebase configuration to `environment.ts` and `environment.prod.ts`

    ```typescript
    export const environment = {
        production: false, //change to true for environment.prod.ts
        firebase: {
            apiKey: '<your-key>',
            authDomain: '<your-project-authdomain>',
            databaseURL: '<your-database-URL>',
            projectId: '<your-project-id>',
            storageBucket: '<your-storage-bucket>',
            messagingSenderId: '<your-messaging-sender-id>'
        }
    };
    ```

## Deploying to Firebase
1. Run `ng build --prod`
1. Run `firebase init` and choose `Hosting`, follow the on-screen prompts.
1. Delete the `public` directory automatically generated by the Firebase CLI
1. Change `firebase.json` to
    ```
    {
        "hosting": {
            "public": "dist",
            "ignore": [
                "firebase.json",
                "**/.*",
                "**/node_modules/**"
            ],
            "rewrites": [
                {
                    "source": "**",
                    "destination": "/index.html"
                }
            ]
        }
    }
    ```
1. Run `firebase deploy`

## Further help
* [Angular CLI README](https://github.com/angular/angular-cli/blob/master/README.md)
* [Angular Fire 2 README](https://github.com/angular/angularfire2/blob/master/README.md)
* [Firebase Cloud Storage Docs](https://firebase.google.com/docs/storage/web/start)
* [Firebase Docs](https://firebase.google.com/docs/web/setup)
