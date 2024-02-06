# Étape 1: Utiliser une image Maven pour construire le code source
FROM maven:3.8.4-openjdk-17 AS build
WORKDIR /app
# Copie le pom.xml en prenant en compte les spécificités du projet
COPY pom.xml .
# Télécharge les dépendances pour tirer parti de la mise en cache des couches Docker
RUN mvn dependency:go-offline
# Copie le reste du projet
COPY src ./src
# Construit l'application sans exécuter les tests pour accélérer le processus
RUN mvn clean package -DskipTests

# Étape 2: Utiliser une image OpenJDK pour exécuter l'application compilée
FROM openjdk:17
WORKDIR /app
# Copie le JAR à partir de l'étape de construction
COPY --from=build /app/target/*.jar app.jar
# Exécute l'application
ENTRYPOINT ["java", "-jar", "app.jar"]
