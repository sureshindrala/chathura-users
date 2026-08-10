# base image
FROM eclipse-temurin:17-jre 
# creae a folder for jar file
RUN mkdir -p /opt/chathura/
# make work dir
WORKDIR /opt/chathura
# create an argument to retreive JAR
ARG JAR_SOURCE
# cOPY JAR SRC TO DEST
COPY ["${JAR_SOURCE}" , "/opt/chathura/"]
# Change permission to folder
RUN chmod 777 /opt/chathura
# Expose port number
EXPOSE 8232
# Command for run java n
CMD [ "java" , "-jar" ,"/opt/chathura/chathura-user-0.0.1-SNAPSHOT.jar" ]

#chathura-eureka-0.0.1-SNAPSHOT.jar