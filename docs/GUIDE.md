# Architecture

L'architecture est indépendante d'une OS spécifique. L'application devrait fonctionner sur Linux, Mac OS, Windows.

Les technologies suivantes ont été utilisées pour le **développement** de l'application:
- Frontend en Vue.js (TypeScript)
- Backend en Java 25 + Spring Boot + OpenAPI REST + ORM JPA
- Base de données relationelle en PostgreSQL

Les technologies suivantes ont été choisies pour le **déploiement** et l'**hébergement** de l'application:
- Docker pour la containerisation du backend, hébergé sur Render
- Frontend hébergé sur Vercel
- Database hébergé sur Supabase

# Application web

L'application web affiche la liste des membres de l'UMONS par faculté, département ou service. Par défaut on voit initialement uniquement les membres au contrat actif aujourd'hui, mais il y a des filtres par date. Un panel d'administration permet d'ajouter, de supprimer, ou de modifier les facultés, départements, etcetera. Les screenshots suivants donnent un aperçu partiel de l'application.

Page listant tous les membres:
![image3](images/image3.png)

Page pour un membre individuel:
![image4](images/image4.png)

Page administration:
![image5](images/image5.png)

#  Instructions d'installation et d'exécution

## Configuration de Spring Boot:

Allez sur `https://spring.io/quickstart` et utilisez `https://start.spring.io/` pour créer la structure de base de votre application Spring Boot.

Voici les paramètres recommandés pour la stack choisie:
  (Dans le project metadata, remplacez `example` et `demo`. Pour le choix de Configuration: Properties ou YAML n'a pas d'importance)

Ajoutez les dépendances suivantes : Spring Web, Spring Data JPA, PostgreSQL Driver.

![image6](images/image6.png)

Architecture avec SpringBoot :
```
Database
↓
Entity
↓
Repository
↓
Service
↓
Mapper
↓
Response DTO
↓
Controller
↓
JSON
↓
Frontend
```

## Hébergement de la base de données sur Supabase

Rendez-vous sur `https://supabase.com/` et suivez les étapes pour créer un projet en "free plan". Vous serez tout au début amenés à choisir un mot de passe pour ce projet, celui-ci correspond au mot de passe de votre DB gardez-le ! 

Comme c'est le backend en Java qui va directement se connecter à la base de données, choisissez la configuration suivante : 

![image7](images/image7.png)

(Vous pouvez aussi essayer `Direct connection`, mais par expérience cette option a tendance à ne pas fonctionner correctement avec SpringBoot)

Dans le code Java, allez dans le fichier de configuration `application.properties` ou `application.yaml` pour configurer la connection à la base de données.

Faites attention d'utiliser un fichier `.env` avec vos variables d'environnement, voir la section Remarques plus bas.

Fichier `application.yaml`:

![image8](images/image8.png)

Fichier `application.properties`:

```java
spring.application.name=demo
spring.datasource.url=${DB_URL}
spring.datasource.username=${DB_USERNAME}
spring.datasource.password=${DB_PASSWORD}
spring.jpa.hibernate.ddl-auto=update
```

### Relations de la base de données

La base de données contient les relations suivantes:
- **faculties → departments** : One-to-Many
- **departments → services** : One-to-Many
- **services ↔ members** : Many-to-Many
- **members → services** : One-to-Many (director)
- **members ↔ roles** : Many-to-Many

Voici le rendu visuel, venant de Supabase:

![image9](images/image9.png)


## Description OpenAPI

La spécification OpenAPI peut être généré de manière automatique du code source Java Spring Boot:

- Dans `build.gradle` ajoutez la dépendance `'org.springdoc:springdoc-openapi-starter-webmvc-ui:3.0.3'` qui génère  l'OpenAPI au lancement de votre code grâce aux @RestController.

![image10](images/image10.png)

- Vous devriez maintenant pouvoir accéder à la spécification de l'API sur:

  L'UI de Swagger: `http://localhost:8080/swagger-ui.html`

  En version JSON: `http://localhost:8080/v3/api-docs`

  En version YAML: `http://localhost:8080/v3/api-docs.yaml`


## Frontend Vue.js

*!!! Le frontend est à créer à la racine du projet, là où se trouve `build.gradle` !!!*

**Prérequis et création**

- Installez Node.js sur votre machine.
- Ouvrez le terminal et tapez : `npm init vue@latest`.
- Validez l'installation de l'outil si demandé.
- Choisissez le nom du projet (ie: frontend) et activez les options souhaitées (cochez : TypeScript, Router (et si vous souhaitez Prettier).

**Commandes de lancement**

- Entrez dans le dossier : `cd frontend`
- Installez les outils : `npm install`
- Lancez le serveur : `npm run dev`

## Installation et utilisation de Tailwind

*!!! À réaliser dans le dossier frontend !!!*

[Tailwind CSS](https://tailwindcss.com) est un framework CSS *utility-first* permettant de construire rapidement une interface à l'aide de classes CSS directement dans les composants Vue.

### Installation

Depuis le dossier `frontend` :

```bash
npm install tailwindcss @tailwindcss/vite
```

Modifiez ensuite `vite.config.js` :

```tsx
import { defineConfig } from 'vite'
import vue from '@vitejs/plugin-vue'
import tailwindcss from '@tailwindcss/vite'

export default defineConfig({
  plugins: [vue(), tailwindcss()],
})
```

Dans `src/assets/main.css` (créez le fichier s'il n'existe pas) :

```css
@import "tailwindcss";
```

### Utilisation

Tailwind permet ensuite d'utiliser directement des classes dans les templates Vue :

```tsx
<button class="rounded-xl bg-blue-600 px-4 py-2 font-semibold text-white hover:bg-blue-700">
  Enregistrer
</button>
```

Quelques exemples :

- `bg-blue-600` → couleur de fond
- `text-white` → couleur du texte
- `p-4` → espacement intérieur
- `rounded-xl` → coins arrondis
- `font-semibold` → texte semi-gras

## Tests

Les tests de l'API utilisent **JUnit + Spring Boot Test + MockMvc** avec une base de données **H2 en mémoire**. Cela permet de tester les endpoints sans modifier la base Supabase.

### Configuration

Ajoutez H2 dans les dependencies de `build.gradle` :

```
testRuntimeOnly 'com.h2database:h2'
```

Créez une configuration de test dans :

```
src/test/resources/application.yaml (ou application.properties) 
```

```yaml
spring:
  datasource:
    url: jdbc:h2:mem:testdb
    driver-class-name: org.h2.Driver
    username: sa
    password:

  jpa:
    hibernate:
      ddl-auto: create-drop
      
  web:
    error:
      include-message: always
```

La base H2 est créée automatiquement au lancement des tests puis supprimée à la fin.

### Organisation

Les tests sont placés dans :

```
src/
└── test/
    ├── java/
    │   └── com/example/demo/
    │       └── FacultyControllerTest.java
    │
    └── resources/
        └── application.yaml
```

Les tests de contrôleurs utilisent `MockMvc` :

```
@SpringBootTest
@AutoConfigureMockMvc
class FacultyControllerTest {
    // ...
}
```

On peut ensuite lancer tous les tests avec :

```
./gradlew test
```

ou uniquement une classe :

```
./gradlew test --tests "com.example.demo.FacultyControllerTest"
```

Chaque test dispose d'une base H2 isolée et peut créer directement les entités nécessaires à son scénario. **Aucune donnée du seed Supabase n'est nécessaire pour les tests.**

## Déploiement

### Conteneuriser l'API avec Docker

L'API Spring Boot est conteneurisée avec **Docker** afin de faciliter son déploiement sur différentes plateformes. Le conteneur contient tout ce qui est nécessaire pour compiler et exécuter l'API, sans devoir installer Java ou Gradle sur le serveur de déploiement.

> **Docker n'est pas nécessaire pour développer ou lancer le projet normalement en local.** Il est principalement utilisé ici pour le déploiement. L'installation de Docker en local est donc uniquement nécessaire si vous souhaitez **tester le conteneur avant de le déployer**.
>

**Première et dernière étape créez un** `Dockerfile` **à la racine du projet**

Le `Dockerfile` utilise une **construction en deux étapes** :

1. Une image Java 25 sert à compiler l'application avec le **Gradle Wrapper** du projet.
2. Une image Java 25 plus légère sert uniquement à exécuter le `.jar` généré.

```docker
# 1.
FROM eclipse-temurin:25-jdk AS builder

WORKDIR /app

COPY . .

RUN ./gradlew clean bootJar -x test

# 2.
FROM eclipse-temurin:25-jre

WORKDIR /app

COPY --from=builder /app/build/libs/*.jar app.jar

EXPOSE 8080

ENTRYPOINT ["java", "-jar", "app.jar"]
```

Le projet utilise son **Gradle Wrapper** (`./gradlew`) plutôt qu'une installation de Gradle dans l'image Docker. La version de Gradle utilisée reste donc celle définie par le projet.

#### Installation de Docker pour tester en local (facultatif au déploiement)

Si vous souhaitez tester le `Dockerfile` et lancer l'API dans un conteneur en local, il faut installer Docker :

- **Linux** → [Docker engine](https://docs.docker.com/engine/install)
- **macOS** → [Docker Desktop](https://docs.docker.com/desktop/install/mac-install/)
- **Windows** → [Docker Desktop](https://docs.docker.com/desktop/install/windows-install/)

#### Construire l'image

À la racine du backend (là où se trouve le `Dockerfile`) :

```bash
docker build -t demogl-api .
```

#### Lancer l'API

Les informations de connexion à la base de données peuvent être fournies directement depuis le `.env` :

```bash
docker run --env-file .env -p 8080:8080 demogl-api
```

L'API est alors accessible sur `http://localhost:8080`.

**Notez comme il est simple de gérer le .env lorsqu'on lance l'API en local avec Docker ! (contrairement à ... voir 6.1)**

En production, une plateforme comme **Render** peut construire automatiquement cette image à partir du `Dockerfile` et exécuter l'API dans un conteneur. Docker n'a donc pas besoin d'être installé sur la machine de développement pour utiliser ou déployer normalement le projet.

### Déploiement de l'API sur [Render](https://render.com) à l'aide de Docker

1. Connectez-vous sur le compte GitHub où se trouve le repo du projet
2. Créez un nouveau projet Render
3. Dans ce projet créez un nouveau service
4. Choisissez "Web Services"

   ![image11](images/image11.png)


5. Sélectionnez le repo GitHub du projet

   ![image12](images/image12.png)


6. Sélectionnez Docker, la branche que vous voulez déployer ainsi que le root directory (ici la branche reste master, et la racine du backend correspond à la racine du repo donc on ne change rien)

   ![image13](images/image13.png)

7. Choisissez l'instance Free et chargez le fichier `.env` du backend (les informations de la base de données) dans les Environment Variables

   ![image14](images/image14.png)

8. Cliquez sur "Deploy Web Service" (cela peut prendre quelques minutes), une fois déployé, vous aurez accès à l'adresse de votre API en ligne.

   ![image15](images/image15.png)


### Déploiement du frontend sur [Vercel](https://vercel.com)

1. Connectez-vous sur le compte GitHub où se trouve le repo du projet
2. Créez un nouveau projet et sélectionnez le repo du projet que vous souhaitez déployer
3. Sélectionnez comme root directory le dossier où se trouve votre frontend (dans notre cas `/frontend`) → Vite sera détecté automatiquement

   ![image16](images/image16.png)

4. Pour finir ajoutez le `.env` du frontend en remplaçant l'adresse locale de votre API (http://localhost:8080) par l'adresse de votre API déployée récupérée à l'étape 8. du point 5.6.2

   ![image17](images/image17.png)

5. Cliquez sur "Deploy", une fois déployé vous aurez accès à l'adresse de votre frontend déployé

   ![image18](images/image18.png)


### Dernière étape - le CorsConfig

Actuellement l'API bloque les requêtes du frontend car elle ne connaît pas son adresse. Pour régler ça il faut aller dans le code de l'API dans `/config/CorsConfig`  et ajouter l'adresse du frontend déployé.

```java
@Configurationpublic classCorsConfig {

    @BeanpublicWebMvcConfigurer corsConfigurer() {
        return newWebMvcConfigurer() {
            @Overridepublic voidaddCorsMappings(CorsRegistry registry) {
                registry.addMapping("/**")
                        .allowedOrigins("http://localhost:5173", 
                        "https://demogl-frontend.vercel.app") // ici
                        .allowedMethods("GET", "POST", "PATCH", "DELETE", "OPTIONS")
                        .allowedHeaders("*");
            }
        };
    }
}
```

Après le push de cette modification Render va automatiquement récupérer les changements sur la branche Master et redéployer !

# Remarques

## ATTENTION .env dans IIU (SpringBoot)**

- Pour **Spring Boot,** IntelliJ IDEA Ultimate (IIU) ne gère pas automatiquement les .env. Il faut donc mettre le plugin EnvFile by Borys Pierov, qui permet d'ajouter manuellement à un setup de run un .env:

  **AJOUTEZ .env DANS LE .gitignore !!!!!**

  ![image1](images/image1.png)

    - cochez "Enable EnvFile"
    - cliquez sur le + et ajoutez le `.env`

  ![image2](images/image2.png)

  Après ça, IIU injecte bien automatiquement le .env à l'exécution par Spring Boot dans l'IDE.

- Pour **gradlew** (`./gradlew clean, ./gradlew build, ./gradlew bootRun`). Il faudra, dans la console où vous lancez les commandes, exporter manuellement le `.env`, soit ligne par ligne (ex : `export DB_URL='....'` pour Linux/macOS et `$env:DB_URL="...."` pour PowerShell Windows) soit avec la commande `export $(cat .env | xargs)` pour Linux/macOS. (à faire à chaque nouveau terminal ou redémarrage)

  Les commandes `export -p` pour Linux/macOS et `Get-ChildItem env:` pour PowerShell Windows affichent les variables d'environnement actives.


## .env dans frontend Vue.js

Pour Vue.js, comme il fonctionne avec Vite, les .env sont nativement supportés.

1. À la racine de votre dossier `frontend`, créez un fichier `.env` et mettez-y : `VITE_API_URL=http://localhost:8080` ou l'adresse https de votre API si celle-ci est déployée
2. Ajoutez `.env` dans le `.gitignore`
3. Créez un dossier `config` à la racine du dossier `frontend` et créez un fichier `api.ts` contenant ceci :
   `export const API_URL = import.meta.env.VITE_API_URL ?? 'http://localhost:8080'`
4. À chaque fois que vous utilisez `“http://localhost:8080/...”` pour vos appels API dans votre code faites : `import { API_URL } from "@/config/api.ts";`  et remplacez-le par :         ``${API_URL}/....``

## Essayer les endpoints de votre API à la main (post, patch, delete)

Vous pouvez pour ça utiliser des outils freemium comme Insomnia ou Postman, cependant dans IntelliJ IDEA Ultimate (la version payante d'IntelliJ mais gratuite grâce à votre adresse student UMONS) il est possible de manière native et intégrée à IIU d'essayer vos endpoints. Pour ça, il suffit d'aller dans l'onglet Endpoints, soit en y naviguant grâce à Alt+Tab (ou Alt+E), soit en allant dans la barre du haut dans :      View→Tool Windows→Endpoints

![image19](images/image19.png)

**ATTENTION : vérifiez que ces plugins soient bien activés dans votre IntelliJ. (sans Spring Web, Endpoints ne détectera pas les endpoints de vos @RestController)**

![image20](images/image20.png)

## Inclure les messages d'erreurs dans les réponses de l'API

Dans votre fichier application.yaml (ou équivalent application.properties) ajoutez `include-message: always` comme montré ci-dessous

```yaml
spring:
  application:
    name: demo

  datasource:
    url: ${DB_URL}
    username: ${DB_USERNAME}
    password: ${DB_PASSWORD}

  jpa:
    hibernate:
      ddl-auto: update
  web:
    error:
      include-message: always
```

Cela permet d'avoir, en réponse des requêtes faites à votre API, le message que vous avez configuré dans le service associé en plus du simple code `400 Bad request`

![image21](images/image21.png)

## Logs dans l'API

Pour activer les logs de l'API, il est possible d'activer le mode DEBUG :

```yaml
spring:
  application:
    name: demo

  datasource:
    url: ${DB_URL}
    username: ${DB_USERNAME}
    password: ${DB_PASSWORD}

  jpa:
    hibernate:
      ddl-auto: update
  web:
    error:
      include-message: always
      
logging:
  level:
    org.springframework.web: DEBUG
```

## Remplir la base de données

Une fois votre API lancée, dans un autre terminal, vous pouvez utiliser le script `seed.ts` situé dans `frontend/src/config/`
(L'API doit être lancée et la base de données doit être vide)

Lancer le script :
```bash
node seed.ts
```
