Spring XD spark streaming processor example
=================

### Setup and configure spark cluster

Spark streaming module can be run on any of the supporting cluster modes.
You can either setup spark `standalone` cluster or use `mesos`, `yarn`. For instructions, see [spark cluster overview] (https://spark.apache.org/docs/1.2.1/cluster-overview.html) and [spark standalone setup] (https://spark.apache.org/docs/1.2.1/spark-standalone.html).

For the development experience, you can use `local` mode as well.

Once the spark cluster is setup, you can configure the property `spark.master` in XD servers.yml to use the corresponding cluster manager URL.The `spark.master` URL property can be configured at the module implementation using `@SparkConfig` annotation that returns Spark configuration properties:

```
  @SparkConfig
  def properties : Properties = {
    val props = new Properties()
    props.setProperty("spark.master", "local[4]")
    props
  }
```

By default Spring XD is set to use `spark://localhost:7077`. Please note that if running Spring XD in standalone mode, it is required to run the Spark cluster in local mode, and therefore to set the `spark.master` property to `local[<n>]`, where `n` is a greater than 1 integer representing the worker threads of the cluster (for the purpose of this example `local[2]` should be sufficient).

### Upload the module

Spring XD provides uploading modules archive from the `shell` interface. We can use this approach to upload the word count example module into XD's module registry.

1. Run the build to generate the jar

  ```
    ./gradlew clean build
  ```
  This will generate the spark-streaming-wordcount-scala-processor-0.1.0.jar under build/libs.
  
2. Upload the generated jar into XD module registry

  ```
    module upload --file [path-to]path to]/spark-streaming-wordcount-scala-processor/build/libs/spark-streaming-wordcount-scala-processor-0.1.0.jar 
    --name scala-word-count --type processor
  ```
  
### Deploy the stream

1. Once the module is uploaded, we can start creating the stream

  ```
  stream create spark-streaming-word-count --definition "http | scala-word-count | log" --deploy
  ```
  
2. Post messages to the stream

  ```
    http post --message "foo foo foo"
  ```
  
3. The log sink module would show the word count results at the container log. This shows the word count computation happens at the spark cluster by the `java-word-count` module and the result is sent to the log module.

  ```
  .....  INFO xdbus.a1.1-1 sink.a1 - (foo,3)
  ```
