FROM maven:3.9.12-amazoncorretto-11
WORKDIR /app
COPY . .
RUN mvn clean package

#FROM tomcat:9.0-jdk11
FROM mayurthitame/tomcat:2
COPY --from=0 /app/target/student-reg-webapp.war /usr/local/tomcat/webapps/student-reg-webapp.war
