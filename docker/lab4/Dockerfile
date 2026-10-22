#build
FROM maven:3.9.15-eclipse-temurin-17-alpine AS build
WORKDIR /app
COPY . .
RUN mvn package -DskipTests

#run
FROM eclipse-temurin:17-jre-alpine
WORKDIR /app
RUN adduser -D appuser
COPY --from=build /app/target/demo-0.0.1-SNAPSHOT.jar app.jar
USER appuser
EXPOSE 8080
CMD ["java", "-jar", "app.jar"]
