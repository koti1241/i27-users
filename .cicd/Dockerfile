# Instructions

# This is the official image as a base
FROM eclipse-temurin:17

# Create a directory called /opt/i27  in which we store our artifacts or other read only data.
RUN mkdir -p /opt/i27

# Setting the working directory to /opt/i27
WORKDIR /opt/i27

# Define a build time variable called JAR_SOURCE to specify the path format
ARG Jar_Source

COPY ["${Jar_Source}", "/opt/i27/i27-user.jar"]

# Grant permissions to the Workdir 
RUN chmod 777 /opt/i27/


# Expose port 8761 to allow access to application
EXPOSE 8232

# Run the process in the background 
CMD ["java", "-jar", "/opt/i27/i27-user.jar"]