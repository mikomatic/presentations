---
theme: gaia
_class: lead
paginate: true
backgroundColor: #fff
backgroundImage: url('./assets/hero-background.svg')
---

<style>
img[alt~="center"] {
  display: block;
  margin: 0 auto;
}
</style>
# Construire nos images Java sans Docker

---
# Sommaire

1. Introduction 
2. Les bonnes pratiques
3. JIB
4. Alternatives

---
## Intro - Processus

![width:950px center](assets/docker-java/docker_build_flow.png)

* Dockerfile
* Installation Docker

--- 
## Intro - Dockerfile

```dockerfile
FROM openjdk:17-jdk-alpine

MAINTAINER miguel.ortega@jesuisundev.sh

COPY target/myMegaApp.jar
ENTRYPOINT ["java","-jar","/myMegaApp.jar"]
```

- quid de la sécurité ?
- quid du layering ?
- quid du versioning ?

--- 
![bg left:65%](assets/docker-java/java_docker_layer_1.svg)

<!--
- Chaque commande dans notre Docker file correspond à un layer/couche
- Chaque layer/couche est une image intermédiaire
- Le but de cette notion et gagner en temps (pour construire une image, la publier) -->

- Chaque version de l'image va générer une nouvelle couche
- Sur la taille finale, une grosse majorité n'a pas bougé
* Coût network, disque, temps !

---

![width:950px center](assets/docker-java/fat-jar-content-eo.svg)

_le contenu de notre fat jar ne change presque pas, pourtant il va être copié encore et encore_

--- 
# Meilleure approche

```dockerfile
FROM eclipse-temurin:11-jre

# Multiple copy statements are used to break the app into layers,
# allowing for faster rebuilds after small changes
COPY dependencyJars /app/libs
COPY resources /app/resources
COPY classFiles /app/classes

ENTRYPOINT ["java", "$JVM_OPTIONS", "-cp", "/app/resources:/app/classes:/app/libs/*","com.example.Main.class"]
CMD ["--some-args"]
```

- un layer pour nos dépendances 
- un layer pour nos ressources 
- un layer pour nos classes project
- un entry point custom

--- 
![bg left:65%](assets/docker-java/java_docker_layer_2.svg)

- On réutilise plus souvent le "cache"
- Reduction du "prix" network, disque, temps

---

## Un bon Dockerfile, pas une mince affaire !

_Finalement, je voudrais juste intégrer cela à mon build maven..._

* On a toujours besoin d'un env Docker (pénible des fois pour un env de CI/CD)
* il nous reste cette histoire de versioning

---
## JIB

![width:950px center](assets/docker-java/jib_build_flow.png)

- pas besoin de docker ✅
- pas (forcément) besoin d'un Dockerfile 🤔
- pas d'image "locale", publication directe (configurable)
- via maven 🚀🎉

---
## JIB Maven Plugin

```xml
<project>
  <build>
    <plugins>
      <plugin>
        <groupId>com.google.cloud.tools</groupId>
        <artifactId>jib-maven-plugin</artifactId>
        <version>3.2.1</version>
        <configuration>
          <to>
            <image>custom.registry/my-app:${project.version}</image>
          </to>
        </configuration>
      </plugin>
    </plugins>
  </build>
</project>
```

<!-- 
- par défaut, une image jdk est utilisé
- plein de customization (utilisateur, arguments de la JVM)
-->

---
## JIB Maven Plugin

```bash
mvn compile jib:build
```

- Configurations:
  - base images 
  - main class
  - jvm options
- Push vers tous les registry: artifactory, gcp, aws, ...
- On peut lier cette commande à une phase maven (_e.g. `deploy`_)
---
## JIB - Equivalent

```dockerfile
# Jib uses Adoptium Eclipse Temurin (formerly AdoptOpenJDK).
FROM eclipse-temurin:11-jre

# Multiple copy statements are used to break the app into layers,
# allowing for faster rebuilds after small changes
COPY dependencyJars /app/libs
COPY snapshotDependencyJars /app/libs
COPY projectDependencyJars /app/libs
COPY resources /app/resources
COPY classFiles /app/classes

# Jib's extra directory ("src/main/jib" by default) is used to add extra, non-classpath files
COPY src/main/jib /

# Jib's default entrypoint when container.entrypoint is not set
ENTRYPOINT ["java", jib.container.jvmFlags, "-cp", "/app/resources:/app/classes:/app/libs/*", jib.container.mainClass]
CMD [jib.container.args]
```

---

### JIB, c'est bien :thumbsup:

➕
- construit une image Docker de qualité, sans docker
- customizable à souhait
- s'intègre dans votre outil de build préféré

⚠️
- vous aurez certainement besoin d'un Dockerfile pour votre image de base (certificats, spécificités client)

---
### References

- don't put fat jar in docker images: https://phauer.com/2019/no-fat-jar-in-docker-image/
- jib maven plugin: https://github.com/GoogleContainerTools/jib/tree/master/jib-maven-plugin 