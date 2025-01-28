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

--------------------------------------------------------------------------------------------------------------------------------------------------------

**Via Helm**


   cd helm/java-jar
   helm create java-jar

**deployment.yaml**

    apiVersion: apps/v1
    kind: Deployment
    metadata:
      name: {{ .Release.Name }}-app
      labels:
        app: {{ .Release.Name }}-app
        chart: "{{ .Chart.Name }}-{{ .Chart.Version }}"
        release: {{ .Release.Name }}
        heritage: {{ .Release.Service }}
    spec:
      replicas: {{ .Values.replicaCount }}
      revisionHistoryLimit: {{ .Values.revisionHistoryLimit }}
      selector:
        matchLabels:
          app: {{ .Release.Name }}-app
      template:
        metadata:
          labels:
            app: {{ .Release.Name }}-app
        spec:
          containers:
          - name: {{ .Release.Name }}-app
            image: "{{ .Values.image.repository }}:{{ .Values.image.tag }}"
            imagePullPolicy: {{ .Values.image.pullPolicy }}
            env:
            {{- toYaml .Values.env | nindent 8 }}
            ports:
              - containerPort: {{ .Values.containerPort }}
            resources:
              {{- toYaml .Values.resources | nindent 10 }}    


**service.yaml**

    apiVersion: v1
    kind: Service
    metadata:
      name: {{ .Release.Name }}-srv
      labels:
        app: {{ .Release.Name }}-srv
        chart: "{{ .Chart.Name }}-{{ .Chart.Version }}"
        release: {{ .Release.Name }}
        heritage: {{ .Release.Service }}
    spec:
      ports:
      - port: {{ .Values.servicePort }}
        protocol: TCP
        targetPort: {{ .Values.containerPort }}
      selector:
        app: {{ .Release.Name }}-app
      type: {{ .Values.serviceType }}
    status:
      loadBalancer: {}

**ingress.yaml**

    apiVersion: networking.k8s.io/v1
    kind: Ingress
    metadata:
      name: {{ .Release.Name }}-ingress
      creationTimestamp: null
    spec:
      rules:
      - host: {{ .Values.domen }}
        http:
          paths:
          - backend:
              service:
                name: {{ .Release.Name }}-srv
                port:
                  number: {{ .Values.servicePort }}
            path: /
            pathType: {{ .Values.pathType }}
    status:
      loadBalancer: {}

**values.yaml**

    #Replicas
    replicaCount: 1
    revisionHistoryLimit: 1

    #Image
    image:
      repository: ulugbekit94/fraud
      tag: latest
      pullPolicy: IfNotPresent
      pullSecretName: my-pull-secret

    #Ports
    containerPort: 1919

    #ENV-ConfigMap
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

    #Service-Params
    servicePort: 80
    #targetPort: 80
    serviceType: ClusterIP

    #Ingress-Params
    domen: kubs.uz
    pathType: Prefix

**commond**

helm template java-jar ./java-jar/ --set "application.reloader=true" --debug > app.yaml
helm install java-jar ./java-jar/
