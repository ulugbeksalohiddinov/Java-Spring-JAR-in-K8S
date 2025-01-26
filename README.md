**Struction app**

    java-jar.yaml
    application.yaml
    Dockerfile
    
**Write Dockerfile**

    FROM openjdk:19-jdk-alpine3.16
    WORKDIR /app
    COPY java-jar.yaml java-jar.yaml
    COPY application.yaml application.yaml
    EXPOSE 1919
    CMD ["java", "-jar", "java-jar.yaml","--spring.config.location=file:./application.yaml"]

**Write application.yaml**
