---
title: Sample Flink Code that reads from Kafka and Writes to Cassandra
date: 2016-12-12
tags: 
- Azure
- IoT
- BigData
---

## The context

Last summer, I started an open source project called **boontadata-streams**.
It is an environment where one can compare big data streaming engines like Apache Flink, Apache Storm or Apache Spark Streaming to name a few.

You can find the project at <https://github.com/boontadata/boontadata-streams/>.

As of this writing, the project contains an implementation of the scenario with Flink. 

The principle is to have the following components: 
- Some Python code simulates IOT objects which
    - sends events to Kafka
    - sends its view of the truth to an Apache Cassandra database
- Apache Flink consumes the events from Kafka. It calculates aggregates and writes them to the Cassandra database
- Some other Python code reads the Cassandra database to compare the results between the IOT simulator and Flink

For instance, here is what we get when time windows are calculated based on **processing time**:

```
Comparing ingest device and flink for m1_sum
--------------------------------------------------
25 exceptions out of 25

Exceptions are:
            window_time                             device_id category  m1_sum_ingest_devicetime  m1_sum_flink  delta_m1_sum_ingestdevice_flink
0   2016-12-05 14:51:55  90fddddf-c42c-4418-8133-fb3866fe826c    cat-1                     459.0           NaN                              NaN
1   2016-12-05 14:52:00  90fddddf-c42c-4418-8133-fb3866fe826c    cat-1                     909.0         459.0                            450.0
2   2016-12-05 14:52:05  90fddddf-c42c-4418-8133-fb3866fe826c    cat-1                     448.0         942.0                           -494.0
3   2016-12-05 14:52:10  90fddddf-c42c-4418-8133-fb3866fe826c    cat-1                    1087.0         402.0                            685.0
4   2016-12-05 14:52:15  90fddddf-c42c-4418-8133-fb3866fe826c    cat-1                     681.0        1203.0                           -522.0
5   2016-12-05 14:52:20  90fddddf-c42c-4418-8133-fb3866fe826c    cat-1                       NaN         681.0                              NaN
6   2016-12-05 14:51:55  90fddddf-c42c-4418-8133-fb3866fe826c    cat-2                     162.0           NaN                              NaN
7   2016-12-05 14:52:00  90fddddf-c42c-4418-8133-fb3866fe826c    cat-2                     901.0         162.0                            739.0
8   2016-12-05 14:52:05  90fddddf-c42c-4418-8133-fb3866fe826c    cat-2                     998.0         920.0                             78.0
9   2016-12-05 14:52:10  90fddddf-c42c-4418-8133-fb3866fe826c    cat-2                     928.0        1033.0                           -105.0
10  2016-12-05 14:52:15  90fddddf-c42c-4418-8133-fb3866fe826c    cat-2                     720.0         928.0                           -208.0
11  2016-12-05 14:52:20  90fddddf-c42c-4418-8133-fb3866fe826c    cat-2                       NaN         768.0                              NaN
12  2016-12-05 14:51:55  90fddddf-c42c-4418-8133-fb3866fe826c    cat-3                     325.0           NaN                              NaN
13  2016-12-05 14:52:00  90fddddf-c42c-4418-8133-fb3866fe826c    cat-3                     857.0         227.0                            630.0
14  2016-12-05 14:52:05  90fddddf-c42c-4418-8133-fb3866fe826c    cat-3                     767.0         939.0                           -172.0
15  2016-12-05 14:52:10  90fddddf-c42c-4418-8133-fb3866fe826c    cat-3                     632.0         783.0                           -151.0
16  2016-12-05 14:52:15  90fddddf-c42c-4418-8133-fb3866fe826c    cat-3                     415.0         632.0                           -217.0
17  2016-12-05 14:52:20  90fddddf-c42c-4418-8133-fb3866fe826c    cat-3                       NaN         514.0                              NaN
18  2016-12-05 14:49:50  90fddddf-c42c-4418-8133-fb3866fe826c    cat-4                      49.0           NaN                              NaN
19  2016-12-05 14:51:55  90fddddf-c42c-4418-8133-fb3866fe826c    cat-4                     589.0           NaN                              NaN
20  2016-12-05 14:52:00  90fddddf-c42c-4418-8133-fb3866fe826c    cat-4                    1078.0         589.0                            489.0
21  2016-12-05 14:52:05  90fddddf-c42c-4418-8133-fb3866fe826c    cat-4                    1310.0        1156.0                            154.0
22  2016-12-05 14:52:10  90fddddf-c42c-4418-8133-fb3866fe826c    cat-4                     905.0        1238.0                           -333.0
23  2016-12-05 14:52:15  90fddddf-c42c-4418-8133-fb3866fe826c    cat-4                     693.0        1079.0                           -386.0
24  2016-12-05 14:52:20  90fddddf-c42c-4418-8133-fb3866fe826c    cat-4                       NaN         820.0                              NaN
```


and here is what we get when extracting time from the event data (**event time**):

```
Comparing ingest device and flink for m1_sum
--------------------------------------------------
5 exceptions out of 21

Exceptions are:
            window_time                             device_id category  m1_sum_ingest_devicetime  m1_sum_flink  delta_m1_sum_ingestdevice_flink
4   2016-12-05 14:53:35  b8c0c5c4-9189-4a0d-a96b-55ca4adafe14    cat-1                       388           NaN                              NaN
9   2016-12-05 14:53:35  b8c0c5c4-9189-4a0d-a96b-55ca4adafe14    cat-2                       496           NaN                              NaN
14  2016-12-05 14:53:35  b8c0c5c4-9189-4a0d-a96b-55ca4adafe14    cat-3                       296           NaN                              NaN
15  2016-12-05 14:51:10  b8c0c5c4-9189-4a0d-a96b-55ca4adafe14    cat-4                        49           NaN                              NaN
20  2016-12-05 14:53:35  b8c0c5c4-9189-4a0d-a96b-55ca4adafe14    cat-4                       537           NaN                              NaN
```

this second result is better. All the differences can be explained: 
those with the `2016-12-05 14:53:35` time window happen because no further event was sent that could trigger the calculation of this time Window; 
the `2016-12-05 14:51:10` time window for `cat4` corresponds to an event that was sent too late by the IOT simulator (on purpose) so that the streaming engine cannot take it into account.

You can see the [full execution log in GitHub](https://github.com/boontadata/boontadata-streams/raw/7a09d63b5b73edb792f0821f8aa2408a770326d9/doc/sample_execution_log.md). 

Here are the versions used in the current implementation: 

---------------------------------------
| Component | version | comments
------------|---------|----------------
| Python | Python 3.5.2 | Anaconda 4.2.0 (64-bit)
| Apache Kafka | 0.10 | 
| Apache Flink | 1.1.3 |
| Apache Cassandra | 3.9 |
| Apache Zookeeper | 3.4.9 |

The project leverages Docker containers, it can run on a single Ubuntu VM with ~14 GB of RAM.
Of course, you are welcome to contribute. We may even let you use one of our Azure VMs while you develop. You can contact me: `contact à boontadata.io`. 

## The pom and the code

In order to write that, I relied on sample code on the Internet about Flink consuming Kafka and Flink writing to Cassandra.
Still, it took me some time to put everything together. 

What I consumed most of my time on was the Maven configuration file: [pom.xml](https://github.com/boontadata/boontadata-streams/blob/master/code/flink/master/code/pom.xml).

I'm not a Java specialist, so I gave up on the JAR size optimization and preferred to have a [uber-jar](http://stackoverflow.com/questions/11947037/what-is-an-uber-jar).

I couldn't make the Kafka 0.10 client working so I used Kafka 0.8.2. This version of the client needs to communicate to Zookeeper. Still, it can read from Kafa 0.10.


In terms of [Flink code](https://github.com/boontadata/boontadata-streams/tree/master/code/flink/master/code/src/main/java/io/boontadata/flink1), I created a pipeline that does the following: 
- reads from Kafka
- parses message
- assigns timestamp and watermarks from event data
- creates a time window in order to remove duplicates (the key is the `message id`)
- creates a time window in order to aggregate (based on `device id`, `message category`)
- sends some debugging information to Cassandra `debug` table about events and their time windows
- aggredates data
- writes results to Cassandra `agg_events` table

Feel free to visit the [GitHub repo](https://github.com/boontadata/boontadata-streams), or use the code as samples, or fix it and create a pull request when you find ways to improve it, or add your own implementation on other streaming engines like Storm, Spark streaming or others.

## A copy of the most significant pieces of code

The rest of this post is a copy of the most significant pieces of code

pom.xml:
```
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
	<modelVersion>4.0.0</modelVersion>
	
	<groupId>io.boontadata.flink1</groupId>
	<artifactId>flink1</artifactId>
	<version>0.1</version>
	<packaging>jar</packaging>

	<name>Flink flink1 Job</name>
	<url>http://boontadata.io</url>

	<properties>
		<project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
		<flink.version>1.1.3</flink.version>
	</properties>

	<repositories>
		<repository>
			<id>apache.snapshots</id>
			<name>Apache Development Snapshot Repository</name>
			<url>https://repository.apache.org/content/repositories/snapshots/</url>
			<releases>
				<enabled>false</enabled>
			</releases>
			<snapshots>
				<enabled>true</enabled>
			</snapshots>
		</repository>
	</repositories>
	
	<dependencies>
		<dependency>
			<groupId>org.apache.flink</groupId>
			<artifactId>flink-java</artifactId>
			<version>${flink.version}</version>
			<scope>provided</scope>
		</dependency>
		<dependency>
			<groupId>org.apache.flink</groupId>
			<artifactId>flink-clients_2.11</artifactId>
			<version>${flink.version}</version>
			<scope>provided</scope>
		</dependency>
		<dependency>
			<groupId>org.apache.flink</groupId>
			<artifactId>flink-streaming-java_2.11</artifactId>
			<version>${flink.version}</version>
			<scope>provided</scope>
		</dependency>
		<dependency>
			<groupId>org.apache.flink</groupId>
			<artifactId>flink-connector-kafka-0.8_2.11</artifactId>
			<version>${flink.version}</version>
			<scope>provided</scope>
		</dependency>
		<dependency>
			<groupId>org.apache.flink</groupId>
			<artifactId>flink-connector-cassandra_2.11</artifactId>
			<version>${flink.version}</version>
			<scope>provided</scope>
		</dependency>
	</dependencies>

	<build>
		<plugins>
			<plugin>
				<groupId>org.apache.maven.plugins</groupId>
				<artifactId>maven-dependency-plugin</artifactId>
				<version>2.9</version>
				<executions>
					<execution>
						<id>unpack</id>
						<!-- executed just before the package phase -->
						<phase>prepare-package</phase>
						<goals>
							<goal>unpack</goal>
						</goals>
						<configuration>
							<artifactItems>
								<artifactItem>
									<groupId>org.apache.flink</groupId>
									<artifactId>flink-clients_2.11</artifactId>
									<version>${flink.version}</version>
									<type>jar</type>
									<overWrite>true</overWrite>
									<outputDirectory>${project.build.directory}/classes</outputDirectory>
									<includes>org/apache/flink/client/**</includes>
								</artifactItem>
								<artifactItem>
									<groupId>org.apache.flink</groupId>
									<artifactId>flink-streaming-java_2.11</artifactId>
									<version>${flink.version}</version>
									<type>jar</type>
									<overWrite>true</overWrite>
									<outputDirectory>${project.build.directory}/classes</outputDirectory>
									<includes>org/apache/flink/streaming/**</includes>
								</artifactItem>
								<artifactItem>
									<groupId>org.apache.flink</groupId>
									<artifactId>flink-connector-kafka-base_2.11</artifactId>
									<version>${flink.version}</version>
									<type>jar</type>
									<overWrite>true</overWrite>
									<outputDirectory>${project.build.directory}/classes</outputDirectory>
									<includes>org/apache/flink/streaming/**</includes>
								</artifactItem>
								<artifactItem>
									<groupId>org.apache.flink</groupId>
									<artifactId>flink-connector-kafka-0.8_2.11</artifactId>
									<version>${flink.version}</version>
									<type>jar</type>
									<overWrite>true</overWrite>
									<outputDirectory>${project.build.directory}/classes</outputDirectory>
									<includes>org/apache/flink/streaming/connectors/kafka/**</includes>
								</artifactItem>
								<artifactItem>
									<groupId>org.apache.flink</groupId>
									<artifactId>flink-streaming-java_2.11</artifactId>
									<version>${flink.version}</version>
									<type>jar</type>
									<overWrite>true</overWrite>
									<outputDirectory>${project.build.directory}/classes</outputDirectory>
									<includes>org/apache/flink/**</includes>
								</artifactItem>
								<artifactItem>
									<groupId>org.apache.kafka</groupId>
									<artifactId>kafka_2.11</artifactId>
									<version>0.8.2.2</version>
									<type>jar</type>
									<overWrite>true</overWrite>
									<outputDirectory>${project.build.directory}/classes</outputDirectory>
									<includes>kafka/**</includes>
								</artifactItem>
								<artifactItem>
									<groupId>com.101tec</groupId>
									<artifactId>zkclient</artifactId>
									<version>0.8</version>
									<type>jar</type>
									<overWrite>true</overWrite>
									<outputDirectory>${project.build.directory}/classes</outputDirectory>
									<includes>org/I0Itec/zkclient\/**</includes>
								</artifactItem>
								<artifactItem>
									<groupId>org.apache.flink</groupId>
									<artifactId>flink-connector-cassandra_2.11</artifactId>
									<version>${flink.version}</version>
									<type>jar</type>
									<overWrite>true</overWrite>
									<outputDirectory>${project.build.directory}/classes</outputDirectory>
									<includes>org/apache/flink/**,com/datastax/driver/**</includes>
								</artifactItem>
								<artifactItem>
									<groupId>com.yammer.metrics</groupId>
									<artifactId>metrics-core</artifactId>
									<version>2.2.0</version>
									<type>jar</type>
									<overWrite>true</overWrite>
									<outputDirectory>${project.build.directory}/classes</outputDirectory>
									<includes>com/yammer/metrics/**</includes>
								</artifactItem>
								<artifactItem>
									<groupId>org.apache.servicemix.bundles</groupId>
									<artifactId>org.apache.servicemix.bundles.kafka-clients</artifactId>
									<version>0.8.2.2_1</version>
									<type>jar</type>
									<overWrite>true</overWrite>
									<outputDirectory>${project.build.directory}/classes</outputDirectory>
									<includes>org/apache/kafka/**</includes>
								</artifactItem>
							</artifactItems>
						</configuration>
					</execution>
				</executions>
			</plugin>					
			<plugin>
				<groupId>org.apache.maven.plugins</groupId>
				<artifactId>maven-compiler-plugin</artifactId>
				<version>3.1</version>
				<configuration>
					<source>1.8</source> <!-- If you want to use Java 8, change this to "1.8" -->
					<target>1.8</target> <!-- If you want to use Java 8, change this to "1.8" -->
				</configuration>
			</plugin>
		</plugins>

		<!-- If you want to use Java 8 Lambda Expressions uncomment the following lines -->
		<pluginManagement>
			<plugins>
				<plugin>
					<artifactId>maven-compiler-plugin</artifactId>
					<configuration>
						<source>1.8</source>
						<target>1.8</target>
						<compilerId>jdt</compilerId>
					</configuration>
					<dependencies>
						<dependency>
							<groupId>org.eclipse.tycho</groupId>
							<artifactId>tycho-compiler-jdt</artifactId>
							<version>0.21.0</version>
						</dependency>
					</dependencies>
				</plugin>
				<plugin>
					<groupId>org.eclipse.m2e</groupId>
					<artifactId>lifecycle-mapping</artifactId>
					<version>1.0.0</version>
					<configuration>
						<lifecycleMappingMetadata>
							<pluginExecutions>
								<pluginExecution>
									<pluginExecutionFilter>
										<groupId>org.apache.maven.plugins</groupId>
										<artifactId>maven-assembly-plugin</artifactId>
										<versionRange>[2.4,)</versionRange>
										<goals>
											<goal>single</goal>
										</goals>
									</pluginExecutionFilter>
									<action>
										<ignore/>
									</action>
								</pluginExecution>
								<pluginExecution>
									<pluginExecutionFilter>
										<groupId>org.apache.maven.plugins</groupId>
										<artifactId>maven-compiler-plugin</artifactId>
										<versionRange>[3.1,)</versionRange>
										<goals>
											<goal>testCompile</goal>
											<goal>compile</goal>
										</goals>
									</pluginExecutionFilter>
									<action>
										<ignore/>
									</action>
								</pluginExecution>
							</pluginExecutions>
						</lifecycleMappingMetadata>
					</configuration>
				</plugin>
			</plugins>
		</pluginManagement>
	</build>


</project>
```

StreamingJob.java:
```
package io.boontadata.flink1;

import com.datastax.driver.core.Cluster;
import java.lang.Double;
import java.lang.Long;
import java.text.Format;
import java.text.SimpleDateFormat;
import java.time.Instant;
import java.util.Date;
import java.util.Iterator;
import java.util.Properties;
import org.apache.flink.streaming.api.environment.StreamExecutionEnvironment;

import org.apache.flink.api.common.functions.MapFunction;
import org.apache.flink.api.common.functions.ReduceFunction;
import org.apache.flink.api.java.functions.KeySelector;
import org.apache.flink.api.java.tuple.Tuple;
import org.apache.flink.api.java.tuple.Tuple2;
import org.apache.flink.api.java.tuple.Tuple5;
import org.apache.flink.api.java.tuple.Tuple6;
import org.apache.flink.streaming.api.datastream.DataStream;
import org.apache.flink.streaming.api.datastream.WindowedStream;
import org.apache.flink.streaming.api.environment.StreamExecutionEnvironment;
import org.apache.flink.streaming.api.functions.sink.SinkFunction;
import org.apache.flink.streaming.api.functions.source.RichParallelSourceFunction;
import org.apache.flink.streaming.api.functions.windowing.WindowFunction;
import org.apache.flink.streaming.api.TimeCharacteristic;
import org.apache.flink.streaming.api.windowing.time.Time;
import org.apache.flink.streaming.api.windowing.windows.Window;
import org.apache.flink.streaming.api.windowing.windows.TimeWindow;
import org.apache.flink.streaming.connectors.cassandra.CassandraTupleSink;
import org.apache.flink.streaming.connectors.cassandra.ClusterBuilder;
import org.apache.flink.streaming.connectors.kafka.FlinkKafkaConsumer082;
import org.apache.flink.util.Collector;
import org.apache.flink.streaming.util.serialization.SimpleStringSchema;

import static java.util.concurrent.TimeUnit.MILLISECONDS;

/**
 * Skeleton for a Flink Streaming Job.
 *
 * For a full example of a Flink Streaming Job, see the SocketTextStreamWordCount.java
 * file in the same package/directory or have a look at the website.
 *
 * You can also generate a .jar file that you can submit on your Flink
 * cluster.
 * Just type
 * 		mvn clean package
 * in the projects root directory.
 * You will find the jar in
 * 		target/quickstart-0.1.jar
 * From the CLI you can then run
 * 		./bin/flink run -c io.boontadata.flink1.StreamingJob target/quickstart-0.1.jar
 *
 * For more information on the CLI see:
 *
 * http://flink.apache.org/docs/latest/apis/cli.html
 */
public class StreamingJob {
	private static final String VERSION = "161205a";

	private static final Integer FIELD_MESSAGE_ID = 0;
	private static final Integer FIELD_DEVICE_ID = 1;
	private static final Integer FIELD_TIMESTAMP = 2;
	private static final Integer FIELD_CATEGORY = 3;
	private static final Integer FIELD_MEASURE1 = 4;
	private static final Integer FIELD_MESAURE2 = 5; 

	public static void main(String[] args) throws Exception {
		String timeCharacteristic = "EventTime";
		if (args.length > 0) {
			timeCharacteristic = args[0];
		}

		// set up the streaming execution environment
		final StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();
		env.enableCheckpointing(5000); // checkpoint every 5000 msecs
		env.setParallelism(2);
		
		if (timeCharacteristic.equals("EventTime")) {
			env.setStreamTimeCharacteristic(TimeCharacteristic.EventTime);
		} else if (timeCharacteristic.equals("ProcessingTime")) {
			env.setStreamTimeCharacteristic(TimeCharacteristic.ProcessingTime);
		} else if (timeCharacteristic.equals("IngestionTime")) {
			env.setStreamTimeCharacteristic(TimeCharacteristic.IngestionTime);
		}

		Format windowTimeFormat = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");

		// properties about Kafka
		Properties kProperties = new Properties();
		kProperties.setProperty("bootstrap.servers", "ks1:9092,ks2:9092,ks3:9092");
		kProperties.setProperty("zookeeper.connect", "zk1:2181");
		kProperties.setProperty("group.id", "flinkGroup");


		// get data from Kafka, parse, and assign time and watermarks
		DataStream<Tuple6<String, String, Long, String, Long, Double>> stream_parsed_with_timestamps = env 
			.addSource(new FlinkKafkaConsumer082<>(
                                "sampletopic",
                                new SimpleStringSchema(),
                                kProperties))
			.rebalance()
			.map(
				new MapFunction<String, 
					Tuple6<String, String, Long, String, Long, Double>>() {
					private static final long serialVersionUID = 34_2016_10_19_001L;

					@Override
					public Tuple6<String, String, Long, String, Long, Double> map(String value) throws Exception {
						String[] splits = value.split("\\|");
						return new Tuple6<String, String, Long, String, Long, Double>(
							splits[FIELD_MESSAGE_ID], 
							splits[FIELD_DEVICE_ID],
							Long.parseLong(splits[FIELD_TIMESTAMP]),
							splits[FIELD_CATEGORY],
							Long.parseLong(splits[FIELD_MEASURE1]),
							Double.parseDouble(splits[FIELD_MESAURE2])
						);
					}
				}
			)
			.assignTimestampsAndWatermarks(new BoundedOutOfOrdernessGenerator());

		// deduplicate on message ID
		WindowedStream stream_windowed_for_deduplication = stream_parsed_with_timestamps
			.keyBy(FIELD_MESSAGE_ID)
			.timeWindow(Time.of(5000, MILLISECONDS), Time.of(5000, MILLISECONDS));

		DataStream<Tuple6<String,String,Long,String,Long,Double>> stream_deduplicated = stream_windowed_for_deduplication			
			.apply(new WindowFunction<Tuple6<String, String, Long, String, Long, Double>, 
				Tuple6<String, String, Long, String, Long, Double>, Tuple, TimeWindow>() {
				// remove duplicates. cf http://stackoverflow.com/questions/35599069/apache-flink-0-10-how-to-get-the-first-occurence-of-a-composite-key-from-an-unbo
				
				@Override
				public void apply(Tuple tuple, TimeWindow window, Iterable<Tuple6<String, String, Long, String, Long, Double>> input, 
					Collector<Tuple6<String, String, Long, String, Long, Double>> out) throws Exception {
					out.collect(input.iterator().next());
				}
			});

		// Group by device ID, Category
		WindowedStream stream_windowed_for_groupby = stream_deduplicated
			.keyBy(FIELD_DEVICE_ID, FIELD_CATEGORY)
			.timeWindow(Time.of(5000, MILLISECONDS), Time.of(5000, MILLISECONDS));

		// add debug information on stream_windowed_for_groupby
		stream_windowed_for_groupby
			.apply(new WindowFunction<Tuple6<String, String, Long, String, Long, Double>, 
				Tuple2<String, String>, Tuple, TimeWindow>() {

				@Override
				public void apply(Tuple keyTuple, TimeWindow window, Iterable<Tuple6<String, String, Long, String, Long, Double>> input, 
					Collector<Tuple2<String, String>> out) throws Exception {

					for(Iterator<Tuple6<String, String, Long, String, Long, Double>> i=input.iterator(); i.hasNext();) {
                                                Tuple6<String, String, Long, String, Long, Double> value = i.next();

						out.collect(new Tuple2<String, String>(
							"v" + VERSION + "- stream_windowed_for_groupby - " + Instant.now().toString(), 
							"MESSAGE_ID=" + value.getField(0).toString() + ", "
							+ "DEVICE_ID=" + value.getField(1).toString() + ", "
							+ "TIMESTAMP=" + value.getField(2).toString() + ", "
							+ "time window start=" + (new Long(window.getStart()).toString()) + ", "
							+ "time window end=" + (new Long(window.getEnd()).toString()) + ", "
							+ "CATEGORY=" + value.getField(3).toString() + ", "
							+ "M1=" + value.getField(4).toString() + ", "
							+ "M2=" + value.getField(5).toString()
						));
					}
				}
			})
			.addSink(new CassandraTupleSink<Tuple2<String, String>>(
                                "INSERT INTO boontadata.debug"
                                        + " (id, message)"
                                        + " VALUES (?, ?);",
                                new ClusterBuilder() {
                                        @Override
                                        public Cluster buildCluster(Cluster.Builder builder) {
                                                return builder
                                                        .addContactPoint("cassandra1").withPort(9042)
                                                        .addContactPoint("cassandra2").withPort(9042)
                                                        .addContactPoint("cassandra3").withPort(9042)
                                                        .build();
                                        }
                                }));


		// calculate sums for M1 and M2
		DataStream<Tuple5<String, String, String, Long, Double>> stream_with_aggregations = stream_windowed_for_groupby
			.apply(new WindowFunction<Tuple6<String, String, Long, String, Long, Double>, 
				Tuple5<String, String, String, Long, Double>, Tuple, TimeWindow>() {
			        // sum measures 1 and 2

				@Override
				public void apply(Tuple keyTuple, TimeWindow window, Iterable<Tuple6<String, String, Long, String, Long, Double>> input, 
					Collector<Tuple5<String, String, String, Long, Double>> out) throws Exception {

					long window_timestamp_milliseconds = window.getEnd();
					String device_id=keyTuple.getField(0); // DEVICE_ID
					String category=keyTuple.getField(1); // CATEGORY
					long sum_of_m1=0L;
					Double sum_of_m2=0.0d;

					for(Iterator<Tuple6<String, String, Long, String, Long, Double>> i=input.iterator(); i.hasNext();) {
                                                Tuple6<String, String, Long, String, Long, Double> item = i.next();
						sum_of_m1 += item.f4; // FIELD_MEASURE1
						sum_of_m2 += item.f5; // FIELD_MESAURE2
					}

					out.collect(new Tuple5<String, String, String, Long, Double>(
								windowTimeFormat.format(new Date(window_timestamp_milliseconds)),
								device_id, 
								category,
								sum_of_m1,
								sum_of_m2
							));
				}
			});

		// send aggregations to destination
		stream_with_aggregations
			.addSink(new CassandraTupleSink<Tuple5<String, String, String, Long, Double>>(
                                "INSERT INTO boontadata.agg_events"
                                        + " (window_time, device_id, category, m1_sum_flink, m2_sum_flink)"
                                        + " VALUES (?, ?, ?, ?, ?);",
                                new ClusterBuilder() {
                                        @Override
                                        public Cluster buildCluster(Cluster.Builder builder) {
                                                return builder
                                                        .addContactPoint("cassandra1").withPort(9042)
                                                        .addContactPoint("cassandra2").withPort(9042)
                                                        .addContactPoint("cassandra3").withPort(9042)
                                                        .build();
                                        }
                                }));

		// execute program
		env.execute("io.boontadata.flink1.StreamingJob v" + VERSION + " (" + timeCharacteristic + ")");
	}
}
```

BoundedOutOfOrdernessGenerator.java:
```
package io.boontadata.flink1;

import org.apache.flink.streaming.api.environment.StreamExecutionEnvironment;

import org.apache.flink.api.common.functions.ReduceFunction;
import org.apache.flink.api.java.functions.KeySelector;
import org.apache.flink.api.java.tuple.Tuple;
import org.apache.flink.api.java.tuple.Tuple5;
import org.apache.flink.api.java.tuple.Tuple6;
import org.apache.flink.streaming.api.datastream.DataStream;
import org.apache.flink.streaming.api.environment.StreamExecutionEnvironment;
import org.apache.flink.streaming.api.functions.AssignerWithPeriodicWatermarks;
import org.apache.flink.streaming.api.functions.sink.SinkFunction;
import org.apache.flink.streaming.api.functions.source.RichParallelSourceFunction;
import org.apache.flink.streaming.api.functions.windowing.WindowFunction;
import org.apache.flink.streaming.api.windowing.time.Time;
import org.apache.flink.streaming.api.windowing.windows.Window;
import org.apache.flink.util.Collector;
import org.apache.flink.streaming.api.watermark.Watermark;

import static java.util.concurrent.TimeUnit.MILLISECONDS;

// cf https://ci.apache.org/projects/flink/flink-docs-release-1.1/apis/streaming/event_timestamps_watermarks.html
/**
 * This generator generates watermarks assuming that elements come out of order to a certain degree only.
 * The latest elements for a certain timestamp t will arrive at most n milliseconds after the earliest
 * elements for timestamp t.
 */
public class BoundedOutOfOrdernessGenerator implements AssignerWithPeriodicWatermarks<Tuple6<String, String, Long, String, Long, Double>> {
    private final long maxOutOfOrderness = 1_000L; // 1 second
    private long currentMaxTimestamp = 0;

    @Override
    public long extractTimestamp(Tuple6<String, String, Long, String, Long, Double> element, long previousElementTimestamp) {
        long timestamp = element.f2; // get processing timestamp from current event
        currentMaxTimestamp = Math.max(timestamp, currentMaxTimestamp);
        return timestamp;
    }

    @Override
    public Watermark getCurrentWatermark() {
        // return the watermark as current highest timestamp minus the out-of-orderness bound
        return new Watermark(currentMaxTimestamp - maxOutOfOrderness);
    }
}
```


:-) b
