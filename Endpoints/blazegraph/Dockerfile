FROM dockerbase/java7

RUN mkdir /db
WORKDIR /db
ADD blazegraph.jar /db
CMD java -jar blazegraph.jar