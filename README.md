> Je vous remercie tout d'abord pour cette oppotunité, MERCI A VOUS TOUS 👋  
> Ce document est visé uniquement la réponse aux questions envoyer par l'équipe Jcdecaux et de mettre les points sur le contenu du projet sujet de l'entretien. Je me permets de faire l'ordre suivant de mes réponses : Q3 - Q1 - Q2 selon l'ordre dans le mail.

---
---
---

### `QUESTION 3 (Q3): Dans le code fourni, je vois du graphql. A quoi sert cette partie ?`

#### Pour mettre en place l'architecture du projet, au premier moment j'ai décidé de mettre deux configuration côté client pour deux types d'endpoint : "GRAPHQL" avec Apollo et "REST" avec Retrofit, est c'est pour but de mettre le point sur le fait qu'on peut travailler avec plusieurs endpoints du type différent. Mais j'ai décidé de ne pas le faire au dernier moment comme le sujet est loin d'avoir ce besoin. Je m'excuse pour avoir oublié d'éliminer tout trace de graphql

---
---
---

### `QUESTION 1 (Q1): Visiblement les pratiques de codage de Aymen ne sont pas exactement celles utilisées par mon équipe. Mon équipe a réalisé des guidelines de développement (MVVM, Clean Architecture, découpage de projet, liste des librairies comme Hilt, Flow, Retrofit, Picasso, ContraintLayout …). Est-ce que Aymen pourra les suivre et s’y conformer ? L’évolution de ces guidelines est toujours possible mais doit se faire en équipe.`

#### Je ferai de mon mieux pour bien exprimer et repondre au differente point du question : 
- En ce qui concernant les technos et le stack technique choisi par votre équipe, j'ai été, je suis et je serai toujours ouvert vers tous les choix mis en place pour les produits Jcdecaux comme je suis loin d'avoir une vraie visibilité sur ce qui est vraiment nécessaire dans le stack technique pour mieux répondre aux besoins des projets mais en même temps je ne suis pas d'accord avec l'adaptation de tout ce qui est nouveauté technique qui n'a pas encore du support de la communauté google/Android, c'est un risque pour les projets et le processus de développement et les qualité de livrable.

- Je n'ai pas bien compri cette partie de la question comme les deux phrases me paraissent contradictoires `est-ce que Aymen pourront les suivre et s’y conformer ? L’évolution de ces guidelines est toujours possible mais doit se faire en équipe.` De ma part je ferai de mon mieux pour etre une asset motrice dans l'équipe et de bien `communiqué` avec toute l'équipe pour aboutir au but commun,  sauf si mon rôle est de développer les fonctionnalités que vous me l'affecter, j'ai besoin d'avoir plus d'infos sur ce point.

---
---
---

### `QUESTION 2 (Q2): Il y a eu bcp d’échanges autour du design pattern Singleton (= 1 seule instance d’une classe). Aymen a expliqué qu’il n’utilisait pas object de Kotlin et implémentait ce pattern comme en Java. Nous n’avons pas compris les raisons malgré ces explications. Peut-il nous traiter un article sur le web qui l’explique ou nous dire dans le code qu’il nous a envoyé (si c’est le cas) où nous pourrions comprendre ses arguments ?`

#### Avant d'essayé de mieux expliquer m'a point de vue concernant Kotlin singleton vs Java singleton, je tien à vous exprimer que je suis à 1000% avec l'idée d'utilisation kotlin object pour avoir directement le fameux Singleton sans avoir besoin de passer avec le process du creation selon le pattern Java. Mais je suppose que dans le projet Livetouch vous avez une seule endpoint Rest.

Je commence par l'exemple typique avec Retrofit, supposant qu'on a plusieurs endpoints (E1, E2, E3, ...) et que la création d'instance du Retrofit passe par son builder pattern dans un singleton qui retourne cette instance, comment je peux définir les endpoints et les interfaces relatives à chacune ??

- Si je l'implémente de manière classique, j'aurai dû faire plusieurs fonctions avec le même bout de code avec seulement du changement d'URL endpoint ou bien d'avoir une seule fonction paramétrée avec la définition de l'endpoint ==> ! de cette manière et comme Retrofit passe par son factory et builder pour la création d'instance j'aurai probablement pour chaque appel de remonte une nouvelle instance.

- On peut avoir des singletons pour chaque endpoint qui fournit une instance Retrofit relatif ==> combien d'objets class dois-je avoir!!

- Si on aura un objet relatif à chaque endpoint avec SEULEMENT le lien qui est different, c'est une sorte de duplication de code et configuration pour les instances de retrofit ce qui est contradictoire avec un clean code.

```
    fun buildRetrofitObject(
        okHttpClient: OkHttpClient
    ): Retrofit = Retrofit.Builder()
        .baseUrl(BuildConfig.E1)
        .client(okHttpClient)
        .addCallAdapterFactory(RxJava2CallAdapterFactory.create())
        .addConverterFactory(GsonConverterFactory.create())
        .callbackExecutor(Executors.newSingleThreadExecutor())
        .build()
```
- Par définition, un `class` déclaré comme `objet` ne peut pas avoir un constructeur ni `défaut` ni `paramètre` mais on peut faire recours à finit{} pour initialiser des attributs.  
De cette manière, on ne peut pas utiliser un class objet pour la création d'une seule instance (singleton) et du paramètre lors de créations pour avoir du singleton de point de vue endpoint. [Veuillez consulter la section `Considerations and Limitations`](https://medium.com/tompee/idiomatic-kotlin-object-and-singleton-183c3cfdbd26
)
- Dagger fournit une solution pour avoir plusieurs singletons du même class avec différent paramètre en combinant `@Named` et `@Singleton`, [plus d'information](https://stackoverflow.com/questions/50666942/dagger-2-injecting-two-retrofit-objects/52348744#52348744)

- Les points precedent, explique mon choix dans certaine cas de faire recours au process Java pour la creation des Singletons à fin d'avoir la main sur le parametrage de mes Singletons selon le besoin exacte du fontionnalité et de mieux respecter les normes de codage, qui est deja utilisé par Google dans par exemple la creation du [LocalBroadcastManager](https://developer.android.com/reference/android/support/v4/content/LocalBroadcastManager.html)
> LocalBroadcastManager.getInstance(context).sendBroadcast(intent)

- Pour conclure, Je vous communique un code snippet du d'un class générique pour la création du singleton en kot lin de même manière que java et en respectant la concurrence des threads. [lien](https://bladecoder.medium.com/kotlin-singletons-with-argument-194ef06edd9e)

```
open class GenericSingletonCreator<out T: Any, in A>(creator: (A) -> T) {
    private var creator: ((A) -> T)? = creator
    @Volatile private var instance: T? = null

    fun getInstance(arg: A): T {
        val i = instance
        if (i != null) {
            return i
        }

        return synchronized(this) {
            val i2 = instance
            if (i2 != null) {
                i2
            } else {
                val created = creator!!(arg)
                instance = created
                creator = null
                created
            }
        }
    }
}
```
