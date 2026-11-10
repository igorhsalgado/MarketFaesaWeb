# Imagem da API para o Render (Web Service com runtime Docker).
# Build: docker build -t marketfaesa-api backend

# Etapa 1: compila o jar com o Maven Wrapper
FROM eclipse-temurin:21-jdk AS build
WORKDIR /src

# Dependencias em camada separada: so baixa de novo quando o pom.xml muda
COPY .mvn/ .mvn/
COPY mvnw pom.xml ./
RUN chmod +x mvnw && ./mvnw -B -q dependency:go-offline

COPY src/ src/
RUN ./mvnw -B -q package -Dmaven.test.skip=true \
    && cp target/*.jar /src/app.jar

# Etapa 2: so o JRE e o jar, rodando com usuario sem privilegios
FROM eclipse-temurin:21-jre
RUN groupadd --system app && useradd --system --gid app --no-create-home app
WORKDIR /app
COPY --from=build --chown=app:app /src/app.jar /app/app.jar
USER app

EXPOSE 8080
# O Render define PORT; fora dele a API usa a 8080
ENTRYPOINT ["sh", "-c", "exec java -Dserver.port=${PORT:-8080} -jar /app/app.jar"]
