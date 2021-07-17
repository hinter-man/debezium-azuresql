FROM debezium/connect:1.6
ENV KAFKA_CONNECT_JDBC_DIR=$KAFKA_CONNECT_PLUGINS_DIR/kafka-connect-jdbc

ARG KAFKA_JDBC_VERSION=5.3.2

# Deploy Kafka Connect JDBC
RUN mkdir $KAFKA_CONNECT_JDBC_DIR && cd $KAFKA_CONNECT_JDBC_DIR &&\
	curl -sO https://packages.confluent.io/maven/io/confluent/kafka-connect-jdbc/$KAFKA_JDBC_VERSION/kafka-connect-jdbc-$KAFKA_JDBC_VERSION.jar

# Deploy MSSQL JDBC Driver
RUN cd /kafka/libs && curl -sO https://repo1.maven.org/maven2/com/microsoft/sqlserver/mssql-jdbc/7.0.0.jre8/mssql-jdbc-7.0.0.jre8.jar

