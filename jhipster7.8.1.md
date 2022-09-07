<img src="https://www.pnglib.com/wp-content/uploads/2020/08/jhipster-logo_5f33f6a91ec80.png" align="right" style="height: 64px"/>

# jHipster 
## Sommaire
### 1. Qu'est que c'est ?
### 2. L'installation
### 3. Génération de projet
### 4. Ajout d'entités
### 5. Coder

<img src="https://www.pnglib.com/wp-content/uploads/2020/08/jhipster-logo_5f33f6a91ec80.png" align="right" style="height: 64px"/>

# 1. Qu'est que c'est ? 
En soit jHipster est une plateforme qui permet de génération le squellette d'une application:

* Par exemple la génération d'une application avec spring boot et angular accompagné de bootstrap

* Il y a plusieurs autres possibilité mais pour le cours suivant nous allons se concentrer sur le contexte cité précédément


# 2. L'installation 

Installer chocolatey à partir de  [chocolatey.org/install](https://chocolatey.org/install)

```bash
choco install temurin11 –y
choco install nodejs-lts –y --version=16.15.0
choco install git -y
choco install docker-desktop -y
choco install heroku-cli -y

choco install intellijidea-ultimate –y
choco install eclipse –y
choco install vscode -y

npm i generator-jhipster@7.8.1
```

# 3. Génération de projet 

Pour générer un projet jHipster, il faut crée un dossier portant le nom simple de l'application à générer et ensuite suivre les instructions en lançant la commande suivante dans le dossier créé.

```bash
jhipster
```
![step 1](./image1.png) 

![step 2](./image2.png) 

Pour lancer l'application générer, il faut lancer la commande suivante: 

```bash
./mvnw
```

![step 3](./image3.png) 

Quand l'application à fini de charger, il faut aller sur  [localhost:8080](https://localhost:8080) pour voir l'application généré.

![step 4](./image4.png) 


# 4. Ajout d'entités

![step 5](./image5.png)

```bash
blog.jdl

entity Blog {
  name String required minlength(3)
  handle String required minlength(2)
}

entity Post {
  title String required
  content TextBlob required
  date Instant required
}

entity Tag {
  name String required minlength(2)
}

relationship ManyToOne {
  Blog{user(login)} to User
  Post{blog(name)} to Blog
}

relationship ManyToMany {
  Post{tag(name)} to Tag{entry}
}

paginate Post, Tag with infinite-scroll
```

Pour importer un fichier jdl pour créer les entitées à travers [JDL Studio : https://www.jhipster.tech//jdl-studio/](https://www.jhipster.tech//jdl-studio/)
```bash
jhipster jdl blog.jdl
```

À la suite il sera demandé d’écraser le fichier suivant : 

```bash
src/main/webapp/i18n/en/global.json
```

Il faudra alors appuyer sur la touche 

```bash
a
```

pour en faire de même pour tous les autres fichiers

![step 6](./image6.png)

![step 7](./image7.png)

Puis il faudra relancer l'application générer avec la commande suivante: 

```bash
./mvnw
```
![step 8](./image8.png)


# 5. Coder

Dans le fichier 

```bash
src/main/resources/config/application-dev.yml
```

Il faut désactiver le mode faker en dev pour pouvoir coder correctement

```bash
liquibase:
 # Add 'faker' if you want the sample data to be loaded automatically
 contexts: dev
```

Pour plus de confidentialité nous allons remplacer le code dans le fichier  **BlogResource.java**

```java
src/main/java/org/jhipster/blog/web/rest/BlogResource.java;
return blogRepository.findAllWithEagerRelationships();	
à:
return blogRepository.findByUserIsCurrentUser();

```

![step 9](./image9.png)

Pour plus de confidentialité nous allons remplacer le code dans le fichier  **PostResource.java** et **PostRepository.java** 

```java
src/main/java/org/jhipster/blog/web/rest/PostResource.java;
if (eagerload) {    
    page = postRepository.findAllWithEagerRelationships(pageable);
    
} else {
    page = postRepository.findAll(pageable);
    
}	
    à:
    
page = postRepository.findAllByBlogUserLoginOrderByDateDesc(SecurityUtils.getCurrentUserLogin().orElse(null), pageable);

if(page.getTotalElements()==0 && page.stream().allMatch(post -> post.getBlog()==null)){ /* When no username is set with faked data as example */
    page = postRepository.findAllByBlogUserLoginOrderByDateDesc(null, pageable); 
}

```
![step 10](./image10.png)

Nous allons intégrer du code html à la volé nous allons modifier du code dans le fichier  **post.component.html**

```html
src/main/webapp/app/entities/post/list/post.component.html

<div class="table-responsive" id="entities" *ngIf="posts && posts.length > 0">
    <div infinite-scroll (scrolled)="loadPage(page + 1)" [infiniteScrollDisabled]="page >= links['last']" [infiniteScrollDistance]="0">
        <div *ngFor="let post of posts; trackBy: trackId">
            <a [routerLink]="['/post', post.id, 'view']">
                <h2>{{ post.title }}</h2>
            </a>
            <div [innerHTML]="post.content"></div>
                <small>Ecrit le {{ post.date | formatMediumDatetime }} par {{ post.blog?.user?.login }}</small>
                <div class="btn-group mb-2 mt-1">
                    <button type="submit" [routerLink]="['/post', post.id, 'edit']" class="btn btn-primary btn-sm" data-cy="entityEditButton">
                         <fa-icon icon="pencil-alt"></fa-icon>
                               <span class="d-none d-md-inline" jhiTranslate="entity.action.edit">Edit</span>
                    </button>
                    <button type="submit" (click)="delete(post)" class="btn btn-danger btn-sm" data-cy="entityDeleteButton">
                         <fa-icon icon="times"></fa-icon>
                              <span class="d-none d-md-inline" jhiTranslate="entity.action.delete">Delete</span>
                    </button>
                </div>
            </div>
        </div>
    </div>
</div>

```

![step 11](./image11.png)

