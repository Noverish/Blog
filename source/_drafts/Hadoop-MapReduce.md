---
layout: post
title: Apache Hadoop MapReduce 사용해보기 (Java)
date: 2022-01-02 16:49:00 +0900
cover: /covers/hadoop.png
disqusId: a752514867afe499f2bbcb8dec6eec29382e3967
toc: true
category: Hadoop
tags:
- hadoop
- hdfs
- mapreduce
---

Java로 Apache Hadoop MapReduce를 사용해보는 법을 알아보겠습니다.

<!-- more -->

> Lorem ipsum dolor sit amet, consectetur adipiscing elit, sed do eiusmod tempor incididunt ut labore et dolore magna aliqua. Porttitor lacus luctus accumsan tortor posuere ac. Pellentesque sit amet porttitor eget. Integer enim neque volutpat ac tincidunt vitae semper quis lectus. Malesuada fames ac turpis egestas sed. Risus viverra adipiscing at in tellus integer. Viverra tellus in hac habitasse platea dictumst vestibulum. Phasellus vestibulum lorem sed risus ultricies tristique. Commodo odio aenean sed adipiscing diam. Ut eu sem integer vitae justo eget. Eget mauris pharetra et ultrices neque ornare. Lobortis scelerisque fermentum dui faucibus in. Fusce ut placerat orci nulla. Est pellentesque elit ullamcorper dignissim cras tincidunt lobortis feugiat vivamus.

```shell shell
# input.txt를 HDFS에 넣습니다.
$ hdfs dfs -copyFromLocal input.txt /

# 단어의 수를 카운트하는 MapReduce Job을 실행한 후 그 결과를 HDFS의 /output 경로에 넣습니다.
$ hadoop jar $HADOOP_HOME/share/hadoop/mapreduce/hadoop-mapreduce-examples-3.3.1.jar wordcount /input.txt /output

# 결과를 출력합니다.
$ hdfs dfs -cat /output/part-r-00000
Commodo	1
Eget	1
Est	1
Fusce	1
Integer	1
...
```

```java WordCount.java
import java.io.IOException;
import java.util.StringTokenizer;

import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.fs.Path;
import org.apache.hadoop.io.IntWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Job;
import org.apache.hadoop.mapreduce.Mapper;
import org.apache.hadoop.mapreduce.Reducer;
import org.apache.hadoop.mapreduce.lib.input.FileInputFormat;
import org.apache.hadoop.mapreduce.lib.output.FileOutputFormat;

public class WordCount {
	public static class WordCountMapper extends Mapper<Object, Text, Text, IntWritable> {
		private final static IntWritable one = new IntWritable(1);
		private final Text word = new Text();

		@Override
		public void map(Object key, Text value, Context context) throws IOException, InterruptedException {
			StringTokenizer tokenizer = new StringTokenizer(value.toString());
			while (tokenizer.hasMoreTokens()) {
				word.set(tokenizer.nextToken());
				context.write(word, one);
			}
		}
	}

	public static class WordCountReducer extends Reducer<Text, IntWritable, Text, IntWritable> {
		private final IntWritable result = new IntWritable();

		@Override
		public void reduce(Text key, Iterable<IntWritable> values, Context context) throws IOException, InterruptedException {
			int sum = 0;
			for (IntWritable val : values) {
				sum += val.get();
			}
			result.set(sum);
			context.write(key, result);
		}
	}

	public static void main(String[] args) throws Exception {
		String input = args[0];
		String output = args[1];

		Configuration conf = new Configuration();
		Job job = Job.getInstance(conf, "Word count");

		job.setJarByClass(WordCount.class);
		job.setMapperClass(WordCountMapper.class);
		job.setCombinerClass(WordCountReducer.class);
		job.setReducerClass(WordCountReducer.class);
		job.setOutputKeyClass(Text.class);
		job.setOutputValueClass(IntWritable.class);

		FileInputFormat.addInputPath(job, new Path(input));
		FileOutputFormat.setOutputPath(job, new Path(output));

		System.exit(job.waitForCompletion(true) ? 0 : 1);
	}
}
```

```shell shell
$ export HADOOP_CLASSPATH=${JAVA_HOME}/lib/tools.jar

# WordCount.java 파일을 컴파일합니다.
$ hadoop com.sun.tools.javac.Main WordCount.java

# 그러면 다음과 같이 class 파일이 3개 생깁니다.
-rw-rw-r-- 1 ubuntu ubuntu 1736 Jan  2 22:56 'WordCount$WordCountMapper.class'
-rw-rw-r-- 1 ubuntu ubuntu 1745 Jan  2 22:56 'WordCount$WordCountReducer.class'
-rw-rw-r-- 1 ubuntu ubuntu 1579 Jan  2 22:56  WordCount.class

# 이 3개의 class 파일을 jar 파일로 패키징합니다.
$ jar cf WordCount.jar WordCount*.class

# 패키징한 jar 파일을 실행합니다.
$ hadoop jar WordCount.jar WordCount /input.txt /output

# 결과를 출력합니다.
$ hdfs dfs -cat /output/part-r-00000
Commodo	1
Eget	1
Est	1
Fusce	1
Integer	1
...
```

```

```