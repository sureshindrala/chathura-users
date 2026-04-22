# This is offical image
FROM eclipse-temurin:17-jre 
# Creat a folder for jar file
RUN mkdir -p /opt/chathura/
# Make work dir 
WORKDIR /opt/chathura/
# create an argument to retrieve JAR
ARG JAR_SOURCE
# copy Jar src to dest
COPY ["${JAR_SOURCE}", "/opt/chathura/"]
# change permission of folder
RUN chmod 777 /opt/chathura/
# expose port number
EXPOSE 8232
# command for run java
CMD ["java" , "-jar" , "/opt/chathura/i27-users-0.0.1-SNAPSHOT.jar"]