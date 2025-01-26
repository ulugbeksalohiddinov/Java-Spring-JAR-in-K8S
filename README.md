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

    server:
      port: 1919
      servlet:
        context-path: /

    #DATABASE CONFIGURATION
    spring:
      servlet:
        multipart:
          max-file-size: 1572864
      datasource:
        url: ${DB_URL}
        username: ${DB_USERNAME}
        password: ${DB_PASSWORD}
      jpa:
        open-in-view: true
        generate-ddl: true
        show-sql: false
        hibernate:
          ddl-auto: update
      devtools:
        livereload:
          enabled: false
      jackson:
        serialization:
          fail-on-empty-beans: false
      main:
        allow-bean-definition-overriding: true

    #PROXY SETTINGS
    proxy:
      host: 10.5.49.110
      port: 2002
    #SWAGGERs
    springdoc:
      swagger-ui:
        path: /swagger-ui/
      api-docs:
        path: /v3/api-docs

**Login DockerHub**
    
    docker login --username asdxxyy

**Create docker image and push DockerHub**

    docker build -t asdxxyy/-javajar .

    docker pull asdxxyy/-javajar

**Create Secret in Kubernetes**

    kubectl create secret generic postgres-credentials --from-literal db.username=dbuser --from-literal db.password=dbpass123 --from-literal db.url=jdbc:postgresql://192.168.182.151:5432/javadb 

**Create Deployment in Kubernetes**

      containers:
      - image: asdxxyy/-javajar
        env:
        - name: DB_USERNAME
          valueFrom:
            secretKeyRef:
              name: postgres-credentials
              key: db.username
        - name: DB_PASSWORD
          valueFrom:
            secretKeyRef:
              name: postgres-credentials
              key: db.password
        - name: DB_URL
          valueFrom:
            secretKeyRef:
              name: postgres-credentials
              key: db.url
        name: jar
        ports:
        - containerPort: 1919
        



