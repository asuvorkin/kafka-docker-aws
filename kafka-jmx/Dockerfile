# Builds an image for Apache Kafka 0.8.1.1 from binary distribution.
#
# The netflixoss/java base image runs Oracle Java 7 installed atop the
# ubuntu:trusty (14.04) official image. Docker's official java images are
# OpenJDK-only currently, and the Kafka project, Confluent, and most other
# major Java projects test and recommend Oracle Java for production for optimal
# performance.

FROM netflixoss/java:8

RUN echo "debconf debconf/frontend select noninteractive" | debconf-set-selections
RUN wget http://central.maven.org/maven2/org/jmxtrans/jmxtrans/259/jmxtrans-259.deb
RUN dpkg -i jmxtrans-259.deb
RUN apt-get install sudo

#JMXTRANS part

ENV JMX_PORT=7203 TRANS_JMX_PORT=7201

# default json confs dir
VOLUME ["/var/lib/jmxtrans"]

#RUN wget https://raw.githubusercontent.com/softwaremill/confluent-playground/master/metrics/kafka/kafka.json -P /var/lib/jmxtrans

ADD kafka.json /var/lib/jmxtrans/kafka.json

ADD run-jmxtrans.sh /usr/bin/run-jmxtrans.sh
RUN chmod +x /usr/bin/run-jmxtrans.sh

# The Scala 2.11 build is currently recommended by the project.
ENV KAFKA_VERSION=0.10.0.1 KAFKA_SCALA_VERSION=2.11 
ENV KAFKA_RELEASE_ARCHIVE kafka_${KAFKA_SCALA_VERSION}-${KAFKA_VERSION}.tgz

RUN mkdir /kafka /data /logs

RUN apt-get update && \
  DEBIAN_FRONTEND=noninteractive apt-get install -y \
    ca-certificates

# Download Kafka binary distribution
ADD http://www.us.apache.org/dist/kafka/${KAFKA_VERSION}/${KAFKA_RELEASE_ARCHIVE} /tmp/
ADD https://dist.apache.org/repos/dist/release/kafka/${KAFKA_VERSION}/${KAFKA_RELEASE_ARCHIVE}.md5 /tmp/

WORKDIR /tmp

# Check artifact digest integrity
RUN echo VERIFY CHECKSUM: && \
  gpg --print-md MD5 ${KAFKA_RELEASE_ARCHIVE} 2>/dev/null && \
  cat ${KAFKA_RELEASE_ARCHIVE}.md5

# Install Kafka to /kafka
RUN tar -zx -C /kafka --strip-components=1 -f ${KAFKA_RELEASE_ARCHIVE} && \
  rm -rf kafka_*

ADD config /kafka/config
ADD start.sh /start.sh
RUN chmod +x /start.sh

ADD kafka-start.sh /kafka-start.sh
RUN chmod +x /kafka-start.sh

# Set up a user to run Kafka
RUN groupadd kafka && \
  useradd -d /kafka -g kafka kafka && \
  chown -R kafka:kafka /kafka /data /logs && \
  chown -R kafka:kafka /var/log/jmxtrans /var/lib/jmxtrans && \
  usermod -aG sudo kafka && \
  echo "kafka:kafka" | chpasswd
  
USER kafka
ENV PATH /kafka/bin:$PATH
WORKDIR /kafka

# broker, jmx
EXPOSE 9092 ${JMX_PORT} ${TRANS_JMX_PORT}
VOLUME [ "/data", "/logs" ]

# default json confs dir
VOLUME ["/var/lib/jmxtrans"]

# default logs dir
VOLUME ["/var/log/jmxtrans"]

WORKDIR /
CMD /start.sh
#CMD /usr/bin/run-jmxtrans.sh
