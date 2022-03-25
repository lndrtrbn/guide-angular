# Monter en compétence sur Angular

_Auteur : Landry Trebon_

*v1.0 - Guide rédigé sur la version 10 d'Angular*

L'objectif du document est d'expliquer aussi succinctement que possible les fondamentaux d'[Angular](https://angular.io/) pour être prêt à développer rapidement. J'aborde également des points de bonnes pratiques basés sur mon expérience personnelle.

> Angular est bien plus riche que ce je présente dans ce document. Je donne ici uniquement les bases pour commencer à développer dans de bonnes conditions.

En complément de ce guide, j'ai fait un projet exemple proposant une architecture pouvant évoluer et grossir facilement : https://github.com/lndrtrbn/npod

## Sommaire

- Angular, c'est quoi ?
- Les prérequis pour bien débuter
  - Langage de programmation : Typescript
  - La programmation réactive avec RxJS
  - L'injection de dépendances
- Les blocs principaux d'Angular
  - Les composants : IHM de notre application
  - Les services : tout ce qui n'est pas IHM
  - Les modules : le ciment entre les briques
  - Le router : naviguer entre les pages
  - Les guards : contrôler avant d'accéder
  - Les resolvers : récupérer avant d'accéder
  - Les directives : enrichir les éléments existants
  - Les pipes : modifier l'affichage de la donnée
  - Les reactive-forms : les formulaires efficaces et puissants
- Sujets avancés
  - Les composants de présentation VS les composants métier
  - Comment communiquer entre les composants ?
  - Le store pour partager de la donnée
  - Un exemple d'architecture

## Angular, c’est quoi

Angular est une **librairie Javascript**. Un framework dont l'objectif est de pouvoir développer des applications accessibles depuis un navigateur (comme React, Vue, ...). Il apporte un ensemble de fonctionnalités permettant de mieux structurer son code et de s'épargner de tâches récurrentes et complexes.

> On me demande de temps en temps _est-ce qu'on peut faire ci ou ça avec Angular ? Est-ce qu'on peut implémenter telle fonctionnalité ?_ Angular ne va pas limiter ce qu'on peut faire, notre application reste du **code Javascript** avant d'être du code Angular. Même si Angular ne prévoit pas la demande en question, une autre librairie peut être ajoutée au projet pour le faire, ou simplement un bout de code Javascript maison. La limitation ne viendra pas d'Angular mais du navigateur ou du langage (sans compter les contraintes de temps imposées par le projet biensûr).

## Les prérequis pour bien débuter

Si les sujets suivants n'ont pas de secret pour vous alors vous pouvez passer à la section suivante.

- Javascript et Typescript,
- Programmation réactive avec RxJS,
- L'injection de dépendances.

### Langage de programmation : Typescript

Avec Angular on fait des applications web exécutées dans un navigateur. Sans surprise c'est donc Javascript qui est utilisé, plus précisément Typescript. Je ne ferai pas de cours sur Javascript ici, on trouve une tonne de ressources sur internet.

Typescript, c'est avant tout du Javascript. Du Javascript avec une notion de POO. Ce qui est intéressant avec Typescript (et qui doit être utilisé systématiquement si on veut garder un code lisible) : **le typage**.

> Autrement dit : pas de _any_ dans le code merci

```typescript
// === TYPAGE === \\

// --- En javascript
var nbTweets = 5; // typage dynamique
nbTweets = "5"; // OK

// --- En typescript
let nbTweets: number = 5; // typage statique explicite
nbtweets = "5"; // Erreur à la compilation
let nbComments = 10; // typage statique implicite
nbComments = "10"; // Erreur à la compilation
```

Typescript permet de faire des interfaces, des classes, de l'héritage, on essaiera donc de toujours modéliser le format des objets que l'on manipule.

```typescript
// Interface pour représenter le format du body
// d'une requête /login à une API.
interface LoginDto {
  login: string;
  pwd: string;
}

// Classe pour modéliser un utilisateur d'une app.
class User {
  firstname: string;
  lastname: string;
  role: Role; // "Role" peut être une interface, une classe, une énumération...
}
```

> Mes exemples d'utilisation des interfaces et classes sont très basiques, Typescript apporte bien plus, je vous laisse aller voir la [documentation officielle](https://www.typescriptlang.org/docs/home.html). Ce qui est important de comprendre est le fait de pouvoir modéliser nos données pour être raccord avec l'API, ce qui n'était pas possible en Javascript (car pas de typage).

### La programmation réactive avec RxJS

Angular utilise énormément la programmation réactive. Il est donc intéressant de savoir ce que c'est avant d'écrire sa première application. Le concept derrière la programmation réactive est que des **sources émettent des données qui seront récupérées par des consommateurs**, les consommateurs ne savent pas quand ils vont recevoir des données, mais savent qu'ils les recevront dans l'ordre dans lesquelles elles ont été émises. On est donc ici face à un type de programmation qui **n'est pas séquentiel**. La programmation réactive se base sur le patron de conception [Observer](https://refactoring.guru/design-patterns/observer).

> Avec [RxJS](https://www.learnrxjs.io/) on fait donc du code asynchrone un peu comme avec les Promises en Javascript.

Et au niveau du code ça se passe comment ? Comme on utilise le patron de conception _Observer_ on aura des objets qui observent, et d'autres qui sont observés. Avec RxJS, la notion _d'objets qui observent_ est représentée par des fonctions qui sont appelées à chaque fois que l'objet observé change.

```typescript
// === Programmation réactive avec RxJS, la base === \\

// Création d'un objet qui est observé.
// La méthode of(args...) permet de créer un observable qui émettra
// toutes les valeurs qu'on lui a donné les unes après les autres.
const observable = of(1, 2, 3, 4, 5);

// Création de note fonction qui sera exécutée à chaque
// changement de notre observable.
const observer = (value) => console.log(value);

// On utilise la fonction subscribe pour commencer à
// observer notre observable.
observable.subscribe(observer); // Sortie : 1 2 3 4 5
```

Les observables seront utilisés pour récupérer les données depuis une API, ou pour souscrire aux modifications de l'URL pour en récupérer les paramètres par exemple.

### L’injection de dépendances

L'objectif derrière l'injection de dépendances est de réaliser une **séparation des responsabilités** (separation of concerns en anglais) et d'avoir des bouts de code les moins couplés entre eux possible. Autrement dit, chaque bout de code est responsable d'une tâche qui lui est propre et doit pouvoir l'exécuter sans avoir à se soucier de comment les autres tâches autour de la sienne sont réalisées.

Ce n'est donc pas le bout de code qui se charge de gérer les dépendances dont il a besoin pour fonctionner. On lui les donne, il ne sait pas qui lui les donne mais il sait qu'au moment où il en aura besoin elles seront là.

Avec Angular, l'injection de dépendances se fait à travers les paramètres du constructeur. Imaginons que notre code ai besoin d'accéder au _router_ Angular pour récupérer l'id en paramètre de l'URL. La dépendance du _router_ sera donnée en paramètre du constructeur de notre classe.

```typescript
class Composant {
  ...
  // Notre composant se voit donner la dépendance
  // et se fiche de savoir comment il l'a reçu, la
  // seule chose qui compte est qu'il peut l'utiliser.
  constructor(
    private readonly router: Router
  ) {}
  ...
}
```

[Un autre exemple](https://stackoverflow.com/questions/3058/what-is-inversion-of-control/3140#3140) d'injection de dépendance, peut-être mieux expliqué.

## Les blocs principaux d’Angular

Angular met à disposition un ensemble d'éléments pour nous aider à réaliser nos applications web. Dans cette section nous allons parcourir les principaux éléments qu'il faut connaître pour pouvoir utiliser le framework correctement.

### Les composants : IHM de notre application

Les composants sont les éléments qui nous serviront à faire notre interface graphique. Un composant a pour objectif d'**être rendu dans le DOM**. On peut faire un composant pour notre page entière, pour le haut de page, pour notre menu, pour une liste, un élément de la liste, pour un formulaire... ll n'y a pas de contrainte sur ce que représente un composant. La seule chose est que c'est un élément graphique.

On peut imaginer faire un seul composant qui contient tout le code de notre application. A moins qu'elle soit minimaliste ce n'est pas, **du tout**, une bonne idée. Il faut donc se poser la question de comment notre application peut être découpée en plusieurs composants. Supposons que notre application soit, pour être original, une liste de tâche à réaliser. On peut imaginer le découpage suivant :

- un composant qui sera notre page entière contenant :
  - un composant avec un formulaire pour ajouter une tâche,
  - un composant avec la liste des tâches à effectuer, lui même contenant :
    - X composants représentant chacun une tâche.

Un composant Angular est composé de plusieurs fichiers :

- un fichier Typescript contenant une classe avec l'annotation `@Component()`,
- un fichier pour le style CSS (ou SCSS de préférence),
- un fichier pour le template HTML,
- un fichier pour les tests unitaires (oui c'est important).

Reprenons notre todo list, le composant pour afficher **une tâche** pourrait ressembler à ce qui suit.

```typescript
// task.component.ts

// Annotation Angular spécifique à un composant. On y définit
// le nom du sélecteur html, le fichier de template et de style.
@Component({
  selector: "app-task",
  templateUrl: "./task.component.html",
  styleUrls: ["./task.component.scss"],
})
export class TaskComponent {
  // La tâche à effectuer, ici juste un String pour
  // la simplicité de l'exemple. La notion d'Input()
  // sera abordée juste après.
  @Input() task: string;
}
```

```html
<!-- task.component.html -->

<!-- On affiche le contenu de la tâche -->
<p class="ma-super-tache">{{ task }}</p>
```

```scss
// task.component.scss

.ma-super-tache {
  color: yellow;
  background: red;
  font-size: 100px;
}
```

> Si certains bouts de code semblent confus, pas de panique, les explications sont dans les paragraphes qui suivent.

#### Le data-binding

Le data-binding c'est ce qui nous permet de faire le pont entre notre fichier de script et notre fichier de template. Il existe plusieurs façon de le faire et cela va notamment dépendre du sens dans lequel on fait le binding (script vers template ou l'inverse).

Du script vers le template, on utilise les accolades, à l'intérieur on peut y mettre une variable, un appel de fonction, une expression.

```html
<!-- Affichera le contenu de la tâche -->
<p>{{ task }}</p>
```

On peut aussi utiliser les crochets si le data-binding se fait dans un attribut d'une balise HTML.

```html
<!-- Le champ sera rempli avec la valeur de la tâche dans les 2 cas -->
<input type="text" [value]="task" />
<input type="text" value="{{ task }}" />
```

Dans le sens template vers script c'est différent. On fait forcément appel à une fonction en réaction à quelque chose qui s'est produit (souvent une interaction utilisateur). On utilise dans ce cas les parenthèses.

```html
<!-- Au clic sur le bouton on fait appel à la méthode submitForm()
qui est définie dans notre script -->
<button (click)="submitForm()">Envoyer</button>
```

#### La manipulation du template

Angular apporte la possibilité de modifier dynamiquement le template avec des conditions, des boucles. Ceci grâce à des directives. On reparlera des directives un peu plus tard, ce sont des éléments qui permettent d'ajouter des fonctionnalités au niveau du template.

On peut afficher ou masquer certaines parties du template selon des conditions. Par exemple on peut imaginer afficher un message à la place de notre liste de tâche si aucune tâche n'a été créée.

```html
<!-- Le texte s'affichera uniquement si le tableau de tâches est vide -->
<p *ngIf="tasks.length == 0">Plus rien à faire ! C'est repos</p>
```

De manière un peu similaire on peut ajouter des classes dynamiquement selon des conditions.

```html
<!-- La tâche aura la classe "high-priority" si la condition est vraie -->
<p [class.high-priority]="task.isPrio">{{ task.content }}</p>
```

> Il existe d'autres moyens d'ajouter des classes, parfois utile quand on a plusieurs classes à gérer dynamiquement. Plus d'informations sur [cette réponse](https://stackoverflow.com/a/41974490) stackoverflow complète.

On peut également faire des boucles, utiles par exemple quand on a une liste de tâches à afficher.

```html
<!-- On aura un paragraphe pour chaque tâche de notre tableau tasks -->
<p *ngFor="let task of tasks">{{ task.content }}</p>
```

#### Le cycle de vie d’un composant

Un composant naît, vie, et meurt. C'est la vie ¯\\_(ツ)_/¯

Et pour chaque étape, on peut utiliser un hook pour exécuter du code si nécessaire :

- `ngOnInit()` est appelée quand le composant est initialisé,
- `ngAfterViewInit()` est appelée quand le template du composant est prêt (ie. quand le DOM est rendu pour la première fois),
- `ngOnDestroy()` est appelée juste avant de détruire le composant.

```typescript
@Component({
  ...
})
export class TaskComponent implements OnInit, OnDestroy {
  @Input() task: string;

  // Appelée à l'initialisation du composant.
  ngOnInit() {
    console.log("N'oublie pas de : " + task);
  }

  // Appelée quand le composant est détruit
  ngOnDestroy() {
    console.log("Tâche terminée!");
  }
}
```

> Il existe d'autres hooks, voici [la liste complète](https://angular.io/guide/lifecycle-hooks).

#### Les inputs et les outputs

Un composant peut recevoir des messages venant d'autres composants :

- Le composant parent peut envoyer de la donnée à ses enfants avec un `@Input()`,
- Un composant enfant peut émettre un évènement à son parent avec un `@Output()`.

```html
<!-- Utilisation d'un composant qui a un input et un output -->

...
<app-super-button [label]="data" (onClicked)="buttonClicked($event)">
  <!-- $event contient ce qui est donné en paramètre du output -->
  <!-- ici "hello", cf le script du composant juste en dessous -->
</app-super-button>
...
```

```typescript
// Script du composant enfant utilisé ci-dessus

...
export class SuperButtonComponent {
  @Input() label: string; // Le label du bouton à afficher
  @Output() onClicked = new EventEmitter<string>();

  // Quand cette méthode est appelée, elle émet un évènement 'onClicked'
  // avec en paramètre "hello" qui sera capturé par le parent.
  sendEvent() {
    this.onClicked.emit("hello");
  }
}
```

```html
<!-- Template du composant enfant -->

<button (click)="sendEvent()">{{ label }}</button>
```

Reprenons l'exemple de notre liste de tâches. Nous avons un composant pour lister les tâches et un composant pour afficher le détail d'une tâche. On peut alors utiliser un `@Input()` pour, depuis le composant _liste_, envoyer la tâche au composant affichant son détail. Et on peut utiliser un `@Output()` depuis le composant de détail vers le composant liste quand on veut supprimer la tâche par exemple.

Voici ce que notre code de todo list pourrait donner :

```typescript
// Notre modèle de tâche, j'ai choisi de faire une
// interface plutôt qu'une classe car de toute façon
// je ne compte pas créer d'instances.
export interface Task {
  id: number;
  content: string;
  isPrio: boolean;
}
```

```typescript
// Composant pour lister les tâches

@Component({ ... })
export class TodoListComponent implements OnInit {
  // Notre liste de tâche qui sera initialisée dans le hook.
  tasks: Task[] = [];

  ngOnInit() {
    // Dans une vraie application on irait chercher les données depuis une API.
    this.tasks = [
      { id: 1, content: "Faire les courses", isPrio: true },
      { id: 2, content: "Donner des nouvelles à maman", isPrio: false }
    ];
  }

  // Méthode appelée quand on reçoit un évènement de l'enfant.
  deleteTask(taskId: number) {
    // On enlève la tâche correspondant à l'id donné.
    this.tasks = this.tasks.filter(task => task.id !== taskId);
  }
}
```

```html
<!-- Template du composant liste -->

<div class="my-todoes">
  <app-task
    *ngFor="let task of tasks"
    [task]="task"
    (onDelete)="deleteTask($event)"
  >
  </app-task>
</div>
```

```typescript
// Composant pour le détail d'une tâche

@Component({ ... })
export class TodoTaskComponent {
  // La tâche reçue depuis la liste de tâches.
  @Input() task: Task;

  // On émet un évènement avec l'id de la tâche à supprimer.
  @Output() onDelete = new EventEmitter<number>();

  // Appelée lors du clic sur le bouton de suppression.
  delete() {
    this.onDelete.emit(this.task.id);
  }
}
```

```html
<!-- Template du composant de détails -->

<p [class.high-prio]="task?.isPrio">{{ task?.content }}</p>
<button (click)="delete()">X</button>
```

> Je n'ai pas représenté les fichiers de style qui ne sont pas vraiment spécifiques à Angular. Ni les fichiers de test unitaire qui feront l'objet d'un sujet à part entière.

Cet exemple de Todolist fait le tour de ce qu'il est important de savoir sur les composants. Vous êtes encore là ? Bravo ! Rassurez-vous les parties suivantes ne sont pas aussi longues.

### Les services : tout ce qui n’est pas IHM

Les services sont des classes qui pourront être injectées dans d'autres services ou composants via le système d'injection de dépendances. Il n'y a aucune notion d'éléments graphiques avec un service, il n'y a pas de DOM. Un service peut avoir de nombreuses utilisations différentes. Par exemple dans mes projets Angular j'ai le plus souvent :

- Des services dédiés aux appels HTTP de l'API (et uniquement ça),
- Des services pour externaliser du code métier qui allongerait le code d'un composant (dans l'idée mon composant ne doit pas faire plus de 200/250 lignes, commentaires compris),
- Des services pour partager de la donnée à travers l'application (cf les sujets avancés plus loin).

Un service c'est deux fichiers : un fichier pour le code du service qui contient une classe Typescript avec une annotation `@Injectable()`, et un autre fichier pour les tests unitaires. L'annotation permet de dire à Angular "ce service doit pouvoir être injecter par injection de dépendances".

Reprenons l'exemple de notre Todolist, dans le code de notre composant on avait mis une liste de tâches en dur à l'initialisation du composant. Dans un cas réel on aurait un service qui va se charger de récupérer nos tâches auprès d'une API. Le service pourrait ressembler à :

```typescript
// Annotation pour déclarer le service pour qu'il soit utilisable
// via le système d'injection de dépendances.
@Injectable({ providedIn: "root" })
export class TodoHttpService {
  // Dans un cas réel l'url à l'API sera stockée ailleurs
  // Dans les fichiers environments par exemple.
  todoUrl: string = "https://.../mon-api/todoes";

  // Injection du service Angular pour faire des requêtes HTTP.
  constructor(private readonly http: HttpClient) {}

  // Pour récupérer l'ensemble des tâches.
  getAll(): Observable<Todo[]> {
    return this.http.get<Todo[]>(this.todoUrl);
  }

  // Pour récupérer une tâche par son id.
  get(todoId: number): Observable<Todo> {
    return this.http.get<Todo>(`${this.todoUrl}/${todoId}`);
  }

  // Pour créer une nouvelle tâche.
  create(todo: TodoCreateDto): Observable<Todo> {
    return this.http.post<Todo>(this.todoUrl, todo);
  }

  // Pour modifier une tâche.
  update(todoId: number, todo: TodoUpdateDto): Observable<Todo> {
    return this.http.put<Todo>(`${this.todoUrl}/${todoId}`, todo);
  }

  // Pour supprimer une tâche par son id.
  delete(todoId: number): Observable<boolean> {
    return this.http.delete<boolean>(`${this.todoUrl}/${todoId}`);
  }
}
```

Ici l'idée est de faire un service dédié aux requêtes API pour la ressource _Todo_. L'objectif étant toujours d'avoir un service pour **un seul but**. Si on avait plusieurs ressources, on ferait un service par ressource.

A noter également pour les méthodes `create` et `update` le typage via des interfaces `*Dto` . J'ai l'habitude de créer une interface par requête nécessitant un body afin de mieux représenter ce qui est envoyé. Même si ce qui est envoyé est identique à une entité `Todo` de base, si à l'avenir cela devait changer, ça ne posera pas de soucis.

On peut aussi imaginer avoir des services métier qui contiennent de la logique spécifique au projet et qu'on ne veut pas mettre dans un composant pour ne pas alourdir le code de ce composant. Ou encore un service pour partager de la donnée entre des composants sans avoir à passer par les **@Input()** et **@Output()** (sujet abordé plus tard).

Finalement, les services sont juste des classes Typescript bénéficiant du système d'injection de dépendances d'Angular. Libre à vous d'en faire ce que vous voulez ! Pensez juste au fait qu'ils nous aide à découper et partager notre code.

### Les modules : le ciment entre les briques

Les modules vont nous permettre d'encapsuler notre application en plusieurs parties. Chaque module sera un dossier qui peut contenir des composants, des services, des pipes, d'autres modules, etc, bref tout ce qu'on peut faire avec Angular. C'est un sujet à la fois simple pour la mise en place, mais complexe si on veut creuser la manière dont ils fonctionnent, comment on peut les optimiser. Moi même j'en connais les rudiments mais certains points sont encore assez obscures. Je vais donc faire au mieux.

Ce qu'il faut retenir c'est que les modules sont là pour découper notre application en gros morceaux, principalement découpés par thème.

Voici à quoi ressemble la déclaration d'un module, il est tiré de mon projet d'exemple NPOD :

```typescript
@NgModule({
  // "declarations" va contenir les composants, pipes,
  // (tout ce qui est lié au DOM) du module.
  declarations: [
    PicturesComponent,
    PodComponent,
    PicturePreviewComponent
  ],
  // "imports" va importer d'autres modules si besoin.
  imports: [
    CommonModule,
    ApodRoutingModule,
    SharedModule
  ],
  // "exports" est un sous-ensemble de "declarations" qui va contenir
  // ce qui doit être accessible depuis les autres modules qui
  // ont importé ce module.
  exports: []
})
export class ApodModule {}
```

#### Découpage en module

Prenons un exemple d'une application qui ferait le suivit d'un athlète. On pourrait imaginer que l'application lui permet de suivre son entraînement, maintenir une hygiène de vie correcte et d'organiser ses compétitions. On pourrait donc avoir un module `training`, un autre `nutrition`, encore un `competition` et enfin un module `auth` pour qu'il puisse se connecter à son appli. Chacun de ces modules auraient des composants, des pages, des services probablement. Mais on peut imaginer aussi que certains composants ou services soient réutilisés à travers plusieurs de ces modules, on ne va pas les réécrire plusieurs fois. On peut donc avoir un module partagé `shared` également.

Concrètement, dans une application Angular on va toujours avoir un module `app` qui sera la racine de tout le reste, c'est lui qui sera chargé au lancement de l'application. Dans dans ce module `app` on y trouvera un module `shared` et un `core` pour partager les éléments communs à l'application. Puis ce qu'on appelle des `features` modules, ce sont les modules `training`, `nutrition`, `competition`.

> *Attends un peu, c'est quoi ça `core module` ? Tu as parlé de `shared` mais pas de `core`, quelle différence ?*
>
> C'est vrai, et la différence entre les deux n'est pas si facile à percevoir ou à expliquer. Dans tous les cas ici on rentre plus dans une question de convention / best pratice plutôt qu'une question technique Angular. Angular ne dit pas il faut un `core` et un  `shared`, c'est plutôt une architecture établie par des gens expérimentés  afin de répondre à des contraintes courantes.
>
> Le module `shared` va contenir des éléments réutilisables au sein de l'application : des composants, des directives et des pipes principalement. Le module `core` va lui plutôt contenir les éléments globaux à l'application : des services (http ou autre), la modélisation du domaine. On va importer le module `shared` dans chaque `feature` module qui a besoin de son contenu. A l'inverse, `core` a une portée globale et sera donc importé une **et une seule** fois dans `app` module et pas ailleurs.

Mes connaissances sur les modules se basent principalement sur [cet article](https://medium.com/@michelestieven/organizing-angular-applications-f0510761d65a) que je vous invite vivement à lire.

#### Lazy loading

Derniers points sur les modules. Ils permettent de faire du lazy laoding. Ca veut dire qu'au premier chargement, toute l'application ne sera pas chargée. Seulement la partie (module) affichée. Et les parties (modules) seront chargées au fur et à mesure que l'utilisateur le demande en naviguant sur l'application. L'objectif étant de limiter le temps de chargement initial qui peut être important. Le lazy loading sur les modules n'est pas obligatoire et il se peut même que dans certains cas on préfère ne pas l'utiliser, mais pour une application standard, c'est plutôt pratique.

Pour pouvoir faire du lazy loading, on va associer certaines URLs de notre application à des modules, et ces modules seront chargés quand on navigue sur ces URLs. Pour cela on va utiliser le **Router**.

### Le router : quand afficher quoi

On a vu qu'on pouvait créer des composants, les organiser en modules, très bien. Mais comment faire pour dire "tiens si tu vas sur telle URL alors tu affiches tel composant" ? C'est le rôle du `Router`.

On va faire notre routing principal au niveau du composant `app` à la racine du projet grâce au fichier `app-routing.module.ts`. Avec le nom du fichier on peut comprendre qu'on va faire du routing pour chaque module. C'est en effet comme ça qu'on fera du lazy loading plus tard, chaque module définit ses propres routes et seront chargées dynamiquement au besoin.

**Utilisation simple**

Mais commençons d'abord simplement à faire nos routes sans lazy loading. On suppose que notre application n'est pas très grosse, il n'y a pas beaucoup de pages, on peut faire toutes nos routes au niveau de `app`.

Si on prend l'exemple de notre application d'athlète, notre router (`app-routing.module.ts`) pourrait ressembler à ça :

````typescript
const ROUTES: Routes = [
	{ path: '', component: HomeComponent },
	{ path: 'nutrition', component: NutritionComponent },
	{ path: 'training', component: TrainingComponent },
	{
        path: 'competitions',
        children: [
            {
                path: '',
                component: CompetitionsComponent
            },
            {
                path: ':id',
                component: CompetitionDetailComponent
            },
        ]
    },
    // Default route.
    { path: '**', redirectTo: '' }
];

@NgModule({
	imports: [RouterModule.forRoot(ROUTES)],
	exports: [RouterModule]
})
export class AppRoutingModule {}
````

Ici tout est définit au niveau de notre composant `app` et donc tous les modules seront chargés au moment de lancer l'application. Si l'application est petite ce n'est pas vraiment un problème. Parfois, même avec de grosses applications, on veut que tout se charge au début, dans ce cas on fait en sorte de mettre une jolie page de chargement histoire de ne pas trop frustrer l'utilisateur...

**Lazy loading**

Mais on pourrait aussi faire en sorte que chaque module gère ses routes et quelles soient chargées dynamiquement quand on en a besoin :

````typescript
// app-routing.module.ts

const ROUTES: Routes = [
	{ path: '', component: HomeComponent },
	{
        path: 'nutrition',
         loadChildren: () => import("./nutrition/nutrition.module").then(m => m.NutritionModule)
    },
    {
        path: 'training',
         loadChildren: () => import("./training/training.module").then(m => m.TrainingModule)
    },
    {
        path: 'competitions',
         loadChildren: () => import("./competition/competition.module").then(m => m.CompetitionModule)
    },
    // Default route.
    { path: '**', redirectTo: '' }
];

@NgModule({
	imports: [RouterModule.forRoot(ROUTES)],
	exports: [RouterModule]
})
export class AppRoutingModule {}
````

````typescript
// nutrition-routing.module.ts

const routes: Routes = [
  {
    path: '',
    component: NutritionComponent
  }
];

@NgModule({
  imports: [RouterModule.forChild(routes)],
  exports: [RouterModule]
})
export class NutritionRoutingModule {}
````

````typescript
// training-routing.module.ts

const routes: Routes = [
  {
    path: '',
    component: TrainingComponent
  }
];

@NgModule({
  imports: [RouterModule.forChild(routes)],
  exports: [RouterModule]
})
export class TrainingRoutingModule {}
````

````typescript
// competition-routing.module.ts

const routes: Routes = [
  	{
      	path: '',
      	component: CompetitionsComponent
  	},
    {
        path: ':id',
        component: CompetitionDetailComponent
    },
];

@NgModule({
  imports: [RouterModule.forChild(routes)],
  exports: [RouterModule]
})
export class CompetitionRoutingModule {}
````

Alors oui, c'est un peu plus de travail, mais là on a du lazy loading, et nos routes sont découpées dans les modules proprement. Ce qui est intéressant si l'application grossit.

Sur ces routes on peut y appliquer des contrôles, faire des vérifications, même y ajouter des données, c'est le sujet des chapitres suivants.

### Les guards : contrôler avant d’accéder

Le rôle d'une guard est assez simple. On va affecter une guard à une ou plusieurs routes et ces routes seront accessibles par l'utilisateur uniquement si la guard est respectée. Le cas typique serait une guard qui vérifie que l'utilisateur est admin pour lui laisser accéder à un dashboard que seul un admin peut consulter.

Une guard c'est une classe Typescript avec une méthode qui retourne vrai ou faux selon si la ressource doit être accessible par l'utilisateur ou non.

Exemple d'une guard pour vérifier que l'utilisateur est connecté (exemple tiré d'un projet personnel) :

```typescript
@Injectable({ providedIn: "root" })
export class AuthGuard implements CanActivate {
  constructor(
    private readonly router: Router,
    private readonly authService: AuthService
  ) {}

  /**
   * Cannot activate if no token for the user (not logged in)
   * @returns True if logged in, false otherwise
   */
  canActivate(): Observable<boolean> {
    return this.authService.isAuthenticated().pipe(
      map(authenticated => {
        if (!authenticated) this.router.navigate(["auth"]);
        return authenticated;
      })
    );
  }
}
```

La classe implémente l'interface `CanActivate` qui définit la méthode `canActivate()` contenant la logique pour déterminer si on passe la guard ou non.

> Comme les services, les guards utilisent l'annotation `Injectable`.

Cette guard est ensuite ajoutée au niveau des routes à contrôler :

````typescript
{
    path: "me",
    canActivate: [AuthGuard], // On peut mettre plusieurs guards.
    loadChildren: "./user/user.module#UserModule"
},
````

### Les resolvers : récupérer avant d’accéder

De la même manière que les guards, les resolvers sont appliqués sur les routes. Au lieu de contrôler l'accès à certaines routes, les resolvers vont permettrent de faire un traitement avant d'accéder à la route en question. Le chargement de la page associée à la route ne se fera pas tant que le resolver n'a pas fini son traitement.

Un exemple d'utilisation des resolvers pourrait être le suivant : reprenons notre application de suivi d'athlète. Sur son planning des compétitions un athlète peut cliquer sur une compétition pour arriver sur la page de détails de celle-ci. On peut imaginer que le design de l'application fait qu'on ne veut pas charger la page de détails tant qu'on n'a pas récupérer les informations de la compétition depuis l'API. Dans ce cas on peut faire un resolver associé à la route de détails d'une compétition dont le traitement serait de contacter l'API pour récupérer les informations de la compétition.

Un resolver est une classe Typescript implémentant l'interface `Resolve` qui contient une méthode `resolve`.

Exemple pour notre athlète :

````typescript
@Injectable({ providedIn: 'root' })
export class CompetitionResolver implements Resolve<Competition> {
  constructor(
    private readonly httpCompet: CompetitionHttpService
  ) {}

  /**
   * Call the http service to get the data of the competition.
   * @params route To get the id of the competition in the URL.
   * @returns The competition from the API.
   */
  resolve(route: ActivatedRouteSnapshot): Observable<Competition> {
    return this.httpCompet.getOne(route.paramMap.get('id'));
  }
}
````

On ajoute ensuite ce resolver au niveau de la route en question dans une propriété `competition` qui sera accessible dans le composant de la page :

````typescript
{
	path: 'competition/:id',
	component: CompetitionDetailComponent,
	resolve: {
		competition: CompetitionResolver
	}
}
````

Parfois les resolvers ne peuvent pas être utilisés pour récupérer de la donnée, si on veut faire du "skeleton loading" par exemple, où la page doit s'afficher malgré l'absence de contenu chargé. Il faut donc étudier le fonctionnement de l'application avant de s'engager dans l'utilisation de resolver.

### Les directives : enrichir les éléments existants

Les directives vont permettre d'ajouter des comportements à des éléments du DOM, n'importe lesquels, que se soit des composants Angular ou des éléments natifs HTML comme des `div` par exemple. Pour cela les directives ont un nom qui sera mis en tant qu'attribut de l'élement à enrichir.

Par exemple : `<p app-highlight>...</p>` si on imagine qu'on a créé une directive `Highlight` qui, on suppose, permettrait de surligner du texte avec une couleur un peu flashy.

Pour créer une directive on fait une classe Typescript avec l'annotation `@Directive` en lui spécifiant un sélecteur qui sera le nom de l'attribut à utiliser dans le DOM.

Un exemple complet issu d'un de mes projets perso permettant de gérer un appui long sur un bouton : 

````typescript
@Directive({ selector: '[appLongPress]' })
export class LongPressDirective {
  pressTimer: any;

  @Input('appLongPress') timer: number; // Duration of the press.
  @Output() success: EventEmitter<void> = new EventEmitter();

  // Create a timer on click on the button.
  // After an amount of time send an output to notify.
  @HostListener('mousedown') onMouseEnter() {
    const native: HTMLElement = this.el.nativeElement;
    native.classList.add('press');
    this.pressTimer = setTimeout(() => this.success.emit(), this.timer);
  }
  
  // Reset the timer when the button is released.
  @HostListener('mouseup') onMouseLeave() {
    const native: HTMLElement = this.el.nativeElement;
    native.classList.remove('press');
    clearTimeout(this.pressTimer);
  }

  /**
   * @param el To add class on the button when pressed.
   */
  constructor(private readonly el: ElementRef) {}
}
````

````html
<button
   class="btn btn--long"
   [appLongPress]="2200"
   (success)="end()">
   Abandon
</button>
````

Ici on applique la directive avec l'attribut `appLongPress` et on spécifie quelle doit être la durée de l'appuie pour que l'évènement `success` soit émit. Après avoir appuyé pendant 2.2 secondes sur le bouton la méthode `end()` sera appelée.

Angular arrive avec tout un ensemble de directives déjà prêtes, qu'on a déjà vu pour certaines  comme `*ngIf` ou `*ngFor`. La liste complète des directives [built-in](https://angular.io/guide/built-in-directives).

Les directives sont très intéressantes quand on veut un comportement qui peut être dupliqué un peu partout dans l'application sans avoir à en faire un composant. Ici la directive `longPress` peut être utilisée partout sans que j'ai à redéfinir à chaque fois comment cela fonctionne.

### Les pipes : transformer la façon dont afficher la donnée

Les pipes permettent de modifier l'affichage de la donnée depuis le template HTML. Par exemple pour formatter une date ouformatter une somme d'argent en fonction de la monnaie choisie, etc.

Un exemple d'utilisation de pipe built-in d'Angular :

````html
<p>The hero's birthday is {{ birthday | date:"MM/dd/yy" }} </p>
````

La documentation officiel sur les [pipes](https://angular.io/guide/pipes#transforming-data-using-pipes) avec notamment les pipes built-in.

Pour créer son propre pipe on créé une classe Typescript qui implémente l'interface `PipeTransform` et avec l'annotation `@Pipe` pour définir le nom du pipe.

Un exemple de pipe qui écrirait deux fois ce qu'on lui passe (oui ça ne sert pas à grand chose) : 

````typescript
@Pipe({ name: 'doubleStr' })
export class DoubleStrPipe implements PipeTransform {
  transform(value: string): string {
    return `${value} ${value}`;
  }
}
````

 On l'utiliserait comme suit :

````html
<p>{{ 'hello' | doubleStr }}</p>
<!-- résultat : <p>hello hello</p> -->
````

Je ne vais pas rentrer plus en détails sur l'utilisation des pipes qui n'arrive pas si fréquemment que ça. La documentation donnée plus haut explique tout très bien si vous en avez besoin.

### Les reactive-forms : les formulaires efficaces et puissants

Avec Angular il existe deux moyens de faire des formulaires : les **Template-driven forms** et les **Reactive forms**.

Pas grand chose à dire sur les Template-driven forms, on met de la directive `ngModel` sur tous les inputs pour les associer à une variable et c'est tout. C'est sûrement l'approche la plus simple quand on débute. Cependant cette solution n'est pas celle à privilégier, sauf si vous avez un petit formulaire avec un seul champ à la limite.

Je ne vais pas rentrer dans les détails de pourquoi les **Reactive forms** sont plus efficaces. Sachez juste que les Reactive forms sont en grande partie gérés côté script avec des **Observables** plutôt que dans le template ce qui offre des possibilités très intéressantes. Si vous avez besoin de plus que ça avant de continuer je vous invite à aller faire vos propres recherches ou alors tester les deux et comparer. Vous pouvez creuser le sujet [ici](https://guide-angular.wishtack.io/angular/formulaires).

Donc faisons les choses bien et commençons directement par la bonne solution : les Reactive forms. L'objectif principal de ces formulaires est de modéliser la tête de notre formulaire dans notre code Typescript ce qui permet d'avoir une vision claire de à quoi ressemble le formulaire, on peut y ajouter ainsi facilement des vérifications sur les champs, des contraintes, manipuler les champs, observer les changements sur les champs, etc.

Quand on parle de Reactive forms il faut avoir en tête ces éléments : `FormControl`, `FormGroup`, `FormArray`, `AbstractControl` et `FormBuilder`.

**AbstractControl**

Abstraction qui regroupe les `FormControl`, `FormGroup` et `FormArray`.

**FormControl**

Représente un input de notre formulaire, à l'initialisation on va lui spécifier une valeur par défaut et des contraintes de validation. C'est l'élément atomique des reactive forms.

Exemple :

```typescript
new FormControl('', Validators.required);
```

**FormGroup**

Représente un ensemble de `AbstractControl` identifiés par des clés, comme un objet.

Exemple :

````typescript
new FormGroup({
    firstName: new FormControl('', Validators.required),
    lastName: new FormControl(''),
});
````

**FormArray**

Représente un tableau de `FormControl`. Pas le plus intéressant des trois, mais peut être utile parfois donc il faut savoir qu'il existe. Dans la plupart des cas on utilisera uniquement les deux premiers.

**FormBuilder**

C'est une classe pour nous aider à construire nos Reactive forms. Si on reprend l'exemple du `FormGroup` mais en le faisant avec le `FormBuilder` ça donenrait ça :

````typescript
formBuilder.group({
	firstname: ['', Validators.required],
	lastName: ['']
})
````

C'est un peu moins verbeux.

**Un exemple plus qu'un beau discours**

Pour bien comprendre comment ces éléments fonctionnent entre eux nous allons prendre un exemple concret. Et quoi de mieux que notre athlète qui souhaite ajouter un exercice à son programme d'entraînement. Supposons que pour ajouter un exercice nous avons besoin du nom de l'exercice, une description de ce qui doit être fait, puis soit une durée, soit un nombre de répétition.

On va imaginer que nous faisons un composant dédié juste pour ce formulaire. On va se consacrer sur la partie Typescript et template, pour le style et les tests on repassera.

````typescript
// Represents the format of the data in the form.
export interface ExerciseFormData {
  name: string;
  description: string;
  withReps: boolean;
  duration?: number;
  nbReps?: number;
}
````

````typescript
@Component({
  selector: 'app-exercise-form',
  templateUrl: './exercise-form.component.html',
  styleUrls: ['./exercise-form.component.css'],
})
export class ExerciseFormComponent implements OnInit {
  form: FormGroup; // The reactive form.

  // Emitted when the form is submitted.
  @Output() submitted: EventEmitter<ExerciseFormData> = new EventEmitter();

  /**
   * @param fb To build the reactive form.
   */
  constructor(private readonly fb: FormBuilder) {}

  ngOnInit(): void {
    this.form = this.createForm();
    // Listen for changes on type of exercise.
    this.form.get('withReps').valueChanges.subscribe((value: boolean) => {
      if (value) {
        this.form.get('duration').disable();
        this.form.get('nbReps').enable();
      } else {
        this.form.get('duration').enable();
        this.form.get('nbReps').disable();
      }
    });
  }

  /**
   * Creates the initial form group.
   * @returns The initial form group.
   */
  createForm(): FormGroup {
    return this.fb.group({
      name: ['', Validators.required],
      description: ['', Validators.required],
      withReps: [false, Validators.required],
      duration: [0, Validators.required],
      // Disabled by default because withReps is false
      nbReps: [{ value: 0, disabled: true }, Validators.required],
    });
  }

  /**
   * Emits the data of the form.
   */
  submit(): void {
    if (this.form.valid) {
      this.submitted.emit(this.form.value);
    }
  }
}
````

````html
<form [formGroup]="form">
  <input type="text" formControlName="name" />
  <textarea cols="30" rows="10" formControlName="description"></textarea>
  <input type="checkbox" formControlName="withReps" />
  <input type="number" formControlName="duration" />
  <input type="number" formControlName="nbReps" />
  <button [disabled]="form.invalid" (click)="submit()">Submit</button>
</form>
````

Je pense que l'exemple parle de lui même. Le template est volontairement ultra simplifié pour ne garder que l'essentiel pour les Reactive forms c'est-à-dire pas grand chose :

- L'attribut `formGroup` pour associé notre formulaire à la balise HTML,
- Les attributs `formControlName` pour associer le nom de la propriété dans le reactive form,
- Le bouton de soumission qui n'est pas actif tant que le formulaire n'est pas valide.

Au niveau du Typescript rien de bien spécial non plus. Juste à noter qu'on souscrit aux changements de valeur de la propriété `withReps` afin d'activer le bon champ du formulaire. Si on a des répétitions alors on désactive le champ de durée et on active le champ pour spécifier le nombre de répétitions et vis-versa.

Je suis resté sur les bases des Reactve forms pour permettre de bien commencer le sujet sans s'y perdre. On peut pousser bien plus loin : créer nos propres validateurs personnalisés, imbriquer des `FormGroup` entre eux, créer nos propres champs personalisés (sujet très intéressant ça, cf [cet article](https://www.digitalocean.com/community/tutorials/angular-custom-form-control)).

Pour plus d'informations sur les Reactive forms rendez-vous sur la [doc officielle](https://angular.io/guide/reactive-forms#grouping-form-controls).

## Sujets avancés

Les différents éléments qui composent Angular ont été vu. Cette partie ne va pas présenter de nouveaux outils mais plutôt la façon dont utiliser ce que nous avons déjà vu. On va parler de bonnes pratiques.

### Les composants de présentation VS les composants métier

Angular ne propose qu'un seul type de composant. Un composant c'est un composant. Mais on peut découper les composants en deux catégories en fonction de à quoi ils vont servir. Certains, ne vont faire que de l'affichage et ne contiennent pas de logique, on les appelle des **composants de présentation**. D'autres, vont contenir de la logique, communiquer avec des API, etc, on les appelle des **composants métier**.

Faire cette différenciation est utile afin de maintenir un code facile à lire et plus évolutif. L'objectif étant d'avoir un maximum de composants de présentation et le moins de composants métier possible. Comme les composants de présentation n'ont aucune logique particulière ils sont aussi beaucoup plus facilement réutilisable. Limiter le nombre de composant métier permet également de centraliser la logique métier et de ne pas en avoir un peu partout ce qui peut complexifier la lisibilité du code.

**Composant de présentation**

Au niveau du code, un composant de présentation ne va pas s'occuper d'aller récupérer de la donnée. On lui la donne via des `@Input` et il peut émettre des évènements via des `@Output`. Un composant de présentation a égélement très peu, voire pas du tout, de dépendances. Il fonctionne de lui même.

**Composant métier**

A l"inverse un composant métier va aller chercher de la donnée, utiliser des services pour faire des requêtes HTTP, etc. Une fois la donnée récupérée, ce composant métier va pouvoir la fournir à ses composants enfants qui sont des composants de présentation.

**Découpage concret**

De manière générale dans mon code : les composants qui sont associés à une route (que j'appelle des composants "pages") sont toujours des composants métier. Ensuite j'essaie de découper ma page en différent sous-composants de taille correcte (max 250 lignes) qui eux doivent être au plus possible des composants de présentation. Une exception assez courante : les composants qui englobent un reactive form. J'ai l'habitude de faire un composant dédié à un formulaire, ce composant n'est pas vraiment de présentation car il contient la logique du formulaire, mais pas vraiment métier non plus car il ne gère pas la donnée mais juste le formulaire.

> On a déjà vu dans les exemples précédents ce que j'explique ici. Si on reprend ma todo list de la fin de la partie sur les composants. Alors on peut constater que le `TodoListComponent` est un composant métier et `TodoTaskComponent` est un composant de présentation.

Mais faisons un autre exemple pour bien visualiser le concept. Toujours notre application pour athlètes : supposons que dans la partie nutrition, l'athlète accède à un ensemble de recettes de cuisine via une page de l'application. Le code pourrait ressembler à ce qui suit :

````typescript
export interface Recipe {
  name: string;
  preparationTime: number;
  ingredients: { name: string; qty: number }[];
  steps: string[];
}
````

````typescript
// recipes.component.ts

@Component({
  selector: 'app-recipes',
  templateUrl: './recipes.component.html',
  styleUrls: ['./recipes.component.css']
})
export class RecipesComponent implements OnInit {
  recipes: Recipe[] = [];

  constructor(private readonly recipeHttp: RecipeHttpService) {}

  ngOnInit(): void {
    this.recipeHttp.getAll().subscribe(r => this.recipes = r);
  }
}
````

````html
<!-- recipes.component.html -->

<div class="recipes">
    <app-recipe
        *ngFor="let recipe of recipes"
        [recipe]="recipe">
    </app-recipe>
</div>
````

````typescript
// recipe.component.ts

@Component({
  selector: 'app-recipe',
  templateUrl: './recipe.component.html',
  styleUrls: ['./recipe.component.css'],
})
export class RecipeComponent {
  @Input() recipe: Recipe;
}
````

````html
<!-- recipe.component.html -->

<h4 class="recipe__name">{{ recipe?.name }}</h4>
<span class="recipe__time">{{ recipe?.preparationTime }}</span>

<ul class="recipe__ingredients">
  <li class="recipe__ingredient" *ngFor="let ingredient of recipe?.ingredients">
    {{ ingredient.name }} / {{ ingredient.qty }}
  </li>
</ul>

<div class="recipe__steps">
  <p class="recipe__step" *ngFor="let step of recipe?.steps">
    {{ step }}
  </p>
</div>
````

L'exemple est ultra simple donc on peut se dire qu'on aurait pu tout mettre dans un seul composant. C'est vrai, l'idée ici était de montrer la différence entre composants métier et présentation.

On pourrait imaginer que l'athète peut noter la recette avec des étoiles, on aurait alors un système avec des étoiles au niveau du composant de présentation qui, quand l'athlète clique sur une étoile, remonte un évènement via un `@Output` au composant métier qui lui se chargera de faire un appel à l'API pour prendre en compte le vote, etc.

Comme d'habitude voici [un article](https://medium.com/@jtomaszewski/how-to-write-good-composable-and-pure-components-in-angular-2-1756945c0f5b) si vous voulez compléter les informations sur ce sujet.

> Avec un peu d'expérience cette distinction se fera naturellement et vous ferez vos découpages dans votre tête facilement avant d'entammer la moindre ligne de code pour déjà visualiser comment le tout va s'articuler.

### Comment communiquer entre les composants

On a vu que découper notre application en composants était une bonne idée. Mais si nous avons pleins de composants, il va bien falloir trouver un moyen de les faire communiquer entre eux. Dans un premier temps on peut se dire qu'on sait déjà faire communiquer les composants entre eux.

**Input et Output**

Avec des `@Input` dans un sens et des `@Output` dans l'autre les composants s'échangent des informations. C'est vrai, et pour des communications directes de parent à enfant c'est un système qui fonctionne très bien, simple à mettre en place. Cependant (représentons les liens entre composants sous forme d'arbre) si on souhaite faire communiquer des composants qui ne sont pas directement parent et enfant, s'il y a des composants intermédiaires, ou pire, s'ils ne sont même pas sur la même lignée. Alors il faudra faire des séries de `@Input` et `@Ouput` à travers tous les composants intermédiaires pour arriver à nos fin. Autant dire que niveau maintenabilité et lisibilité du code on n'y est pas du tout.

> De manière générale j'opte pour l'option `@Input` `@Ouput` uniquement si le lien de parenté est direct, à la limite s'il y a un composant intermédiaire. Pas plus.

Alors comment partager facilement de la donnée peut importe où se trouve les composants ? Avec les services.

**Services**

En effet, les services sont accessibles partout via le système d'injection de dépendances. De cet fait la contrainte de parentalité des composants n'existe plus. Mais les services à eux seuls ne résolvent pas tout. Comment faire, avec un service, pour que dès que la donnée est modifiée, tous les composants utilisant cette donnée soient mis au courant ? En utilisant les observables de RxJS. En couplant les services à la programmation réactive on permet d'avoir un système de gestion de la donnée qui a une portée globale et qui réagit aux changements en direct.

Prenons un exemple plutôt parlant : les données utilisateur. Notre athlète se connecte sur son application. A la connexion on récupère ses informations. Potentiellement les informations sont utilisées à plusieurs endroits dans l'application : dans le header pour afficher son nom, dans la page compétition pour afficher son classement, sur son dashboard d'accueil où on affiche également son classement, etc.

Une (bonne) solution serait de faire un service dédié au partage de ces informations, on pourrait ainsi avoir un code qui ressemble à ça :

````typescript
@Injectable({ providedIn: 'root' })
export class UserDataService {
  // The object that contains the current value of user.
  private readonly _user: BehaviorSubject<User>;
  // The object that will be observed.
  readonly user: Observable<User>;

  constructor() {
    this._user = new BehaviorSubject(null);
    this.user = this._user.asObservable();
  }

  /**
   * Updates the value of the user.
   * @param user The new value of user.
   */
  setUser(user: User): void {
    this._user.next(user);
  }
}
````

Grâce à RxJS on englobe notre donnée dans un `BehaviorSubject` qui est un objet que l'on va pouvoir modifier avec la méthode `next` et qui notifiera tous les observateurs.

> Avec un `Observable` on peut juste souscire aux changements. Un `BehaviorSubject` on peut souscrire aux changement ET modifier la valeur courante. Comme on ne veut pas que n'importe qui puisse modifier la valeur, on met notre `BehaviorSubject` en privé et on créé un `Observable` associé.

Partout où on a besoin des données utilisateurs pour pourra la récupérer avec :

````typescript
class X {
    constructor(private readonly userData: UserDataService) {
        userData.user.subscribe(u => /* do something */)
    }
}
````

Comprendre le fonctionnement de ces services de partage de données est très important car il permet d'améliorer grandement la lisibilité du code du fait que ces services sont accessibles de partout.

**Pour résumé : lien de parenté direct un `@Input`/`@Ouput` fonctionne très bien, sinon un service à part est une meilleure solution.**

Ces services globaux dédiés au partage de la donnée nous amène doucement vers la notion de **store**. Et les stores, on pourrait en dire beaucoup, en parler longtemps. C'est un sujet qui peut avoir son guide à part entière, je vais rester en surface de ce sujet en proposant une solution qui se veut la plus simple et rapide à mettre en place qui suffit dans la plupart des situations.

### Le store pour partager de la donnée

**Un peu d'histoire**

Il y a quelques années de cela maintenant, des développeurs d'une petite application sociale du nom de Facebook se sont retrouvés face à un problème : nos avons plusieurs composants qui affichent les messages (barre de notifications et fenêtres en bas à droite) comment faire pour que ces composants soient toujours synchronisés ? C'est comme ça que l'architecture Flux est née.

Flux c'est une architecture, un patern, organisant le flux de donnée de manière uni-directionnelle. On a un **store** qui centralise toute la donnée, puis des **vues** qui vont observer l'état de ce store, et enfin des **actions** qui vont permettre de modifier ce store. Quand on lance une action, le store est modifié, les vues qui observent le store se mettent à jour. Bon, je simplifie, tout est mieux expliqué [dans l'article de Facebook](https://facebook.github.io/flux/docs/in-depth-overview).

On dit que Flux c'est du "state management". L'objectif est de contrôler et manipuler l'état de la donnée.

> *Mais dis moi Jamy, ce store qui centralise la donnée, ça ressemble beaucoup aux services avec RxJS qu'on vient de voir ?*
>
> Exact, en effet les services qu'on a vu juste avant peuvent s'apparenter aux **Stores**, et les méthodes du service qui modifient la donnée aux **Actions**. Les composants qui souscrivent sont eux, les **Vues**.
>
> Cependant Flux c'est un peu plus que de simples services et certaines personnes ont fait des bibliothèques permettant d'implémenter l'architecture Flux dans Angular.

**NGRX et NGXS**

[NGRX](https://ngrx.io/) et [NGXS](https://www.ngxs.io/) sont toutes les deux des bibliothèques Angular de state management s'inspirant de l'architecture Flux. Elles sont évidemment plus complètes que juste des services et permettent de créer rapidements des Stores, des Actions, des Dispatcher, bref tout ce qui fait le state management vu par Flux.

Et il y en a d'autres : mobx, akita... Le state management est à la mode.

A première vue quand on découvre ces outils (en tout cas c'était mon cas) on se dit que c'est vraiment génial, on va pouvoir gérer notre donnée super facilement, bref c'est top. 

Eeeeet en fait oui, et non. En effet c'est pratique, parfois, mais comme à chaque fois quand on rajoute une couche d'abstraction s'en suit forcément du code pour la mettre en place, plus de complexité dans le code, besoin de plus de connaissance pour rentrer dans le projet.

Finalement, à moins que l'application soit très grosse et qu'il y ai énormément de partage de données un peu partout dans les composants, on se rend compte que le jeu n'en vaut pas forcément la chandelle. Mais si j'en parle quand même, c'est parce que c'est intéressant de savoir que ça existe, et aussi comment ça marche.

> "Flux libraries are like glasses: you’ll know when you need them." Dan Abramov, un des gars de chez Facebook qui a pensé l'architecture Flux.
>
> Donc si vous ne vous êtes pas dis jusque là que vous aviez besoin de quelque chose pour gérer vos états, c'est probablement que vous n'en avez pas besoin. Et si vous vous êtes déjà posé la question, peut-être que la solution que je vous présente ci-dessous vous suffira.

> *Bon alors finalement on reste sur nos services RxJS partagés ?*
>
> C'est l'idée, mais j'aimerai vous présenter une version améliorée de ces services qui est comme un mini NGRX ou NGXS mais sans avoir besoin d'installer d'autres bibliothèques !

**Du state management sans bibliothèque**

Le soucis avec les services qu'on a vu c'est que cela peut vite devenir verbeux, et on peut se mettre à répéter beaucoup  de code. L'objectif est donc de centraliser la partie "store" dans une classe que l'on pourra réutiliser. Je me suis hautement inspiré de [cet article](https://georgebyte.com/state-management-in-angular-with-observable-store-services/) qui commence à se faire un peu vieux mais bon, c'est toujours aussi efficace.

Notre classe de store :

````typescript
/**
 * Abstract class representing a store in the app. The goal of a store
 * is to centralize the data at one point in the app.
 *
 * The generic type T represents the state of store.
 */
export abstract class Store<T> {
  // Name of the store.
  protected abstract name: string;
  // Attribute that contains the data.
  protected readonly _state: BehaviorSubject<T>;

  /**
   * Initializes the state with the value given in parameter.
   * @param initialState The initial state.
   */
  protected constructor(initialState: T) {
    this._state = new BehaviorSubject(initialState);
  }

  /**
   * Gets a snapshot of the state at a current time.
   * @returns The state at a specific point of time.
   */
  get snapshot(): T {
    return this._state.getValue();
  }

  /**
   * Sets the new value of the state.
   * @param nextState The new state.
   */
  protected setState(nextState: T): void {
    this._state.next(nextState);
  }

  /**
   * Patches the state partially with new values.
   * @param partialState The new partial state.
   */
  protected patchState(partialState: Partial<T>): void {
    this._state.next({ ...this.snapshot, ...partialState });
  }
}
````

Et c'est tout ! Cette classe abstraite n'a plus qu'à être implémentée en classe concrète à travers un service représentant la donnée à mettre dans le store. Ci-dessous un exemple de store pour les informations utilisateur pour un de mes projets persos :

````typescript
export class UserState {
  token: Token;
  userInfo: User;
}

@Injectable({ providedIn: "root" })
export class UserStore extends Store<UserState> {
  name = "UserStore";

  // ===== STATE GETTERS
  // ===================

  token: Observable<Token> = this._state.pipe(
    map(state => state.token),
    // distinctUntilChanged permet d'émettre uniquement si
	// la partie en question (ici token et pas userInfo) a changé
    distinctUntilChanged()
  );

  userInfo: Observable<User> = this._state.pipe(
    map(state => state.userInfo),
    distinctUntilChanged()
  );

  constructor() {
    super(new UserState());
  }

  // ===== ACTIONS
  // =============

  setToken(token: Token) {
    this.patchState({ token });
  }

  setUserInfo(user: User) {
    this.patchState({ userInfo: user });
  }
}
````

Et voilà, notre service délimite bien les parties visibles de la donnée à observer avec les "getters" et les actions possibles sur le state avec les méthodes. On utilise ensuite ce service comme n'importe quel autre service Angular comme on a déjà vu précédemment.

### Un exemple d'architecture

En complément de ce guide, j'ai fait un projet exemple proposant une architecture se basant sur tout ce que j'ai expliqué dans ce guide : https://github.com/lndrtrbn/npod

Après avoir lu ce guide, vous devriez être capable de comprendre assez facilement la structure et le contenu de l'application du projet d'exemple.

Le mot (ou plutôt l'article) de la fin, je viens de tomber sur cet [article](https://georgebyte.com/scalable-angular-app-architecture/) en essayant de retrouver l'article sur les stores RxJS de la partie précédente. Je ne l'ai pas encore lu mais il a l'air super intéressant.
