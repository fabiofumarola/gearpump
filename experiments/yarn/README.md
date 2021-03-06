## Gearpump over YARN

### How to launch a Gearpump cluster on YARN

1. Upload the gearpump-${version}.zip to remote HDFS Folder, suggest to put it under /user/gearpump/gearpump-version.zip
2. Put the YARN configurations under classpath.
  Before calling "yarnclient launch", make sure you have put all yarn configuration files under classpath.
  Typically, you can just copy all files under $HADOOP_HOME/etc/hadoop from one of the YARN Cluster machine to conf/yarnconf of gearpump.
  $HADOOP_HOME points to the Hadoop installation directory. 
3. Launch the gearpump cluster on YARN
  ```bash
    yarnclient launch -package /user/gearpump/gearpump-version.zip
  ```
  If you don't specify package path, it will read default package-path from gear.conf(gearpump.yarn.client.package-path).
4. Before step 3, You can change gear.conf configuration section "gearpump.yarn" to config the cluster.
5. After launching, you can browser the Gearpump UI via YARN resource manager dashboard.

### How to submit a application to Gearpump cluster.

To submit the jar to the Gearpump cluster, we first need to know the Master address, so we need to get
a active configuration file first.

There are two ways to get an active configuration file:
1. Option 1: specify "-output" option when you launch the cluster.
  ```bash
    yarnclient launch -package /user/gearpump/gearpump-${version}.zip -output /tmp/mycluster.conf
   ```
   It will return in console like this:
   ```bash
   ================================================
   ==Application Id: application_1449802454214_0034
   ```

2. Option 2: Query the active configuration file
  ```bash
    yarnclient getconfig -appid <yarn application id> -output /tmp/mycluster.conf
  ```
  yarn application id can be found from the output of step1 or from YARN dashboard.
3. After you downloaded the configuration file, you can launch application with that config file.
  ```bash
    gear app -jar examples/wordcount-${version}.jar -conf /tmp/mycluster.conf
  ```
4. Now the application is running. To check this:
  ```bash
   gear info -conf /tmp/mycluster.conf
  ```

### How to add/remove machines dynamically.

Gearpump yarn tool allows to dynamically add/remove machines. Here is the steps:
 1. First, query to get active resources.
 ```bash
   yarnclient query -appid <yarn application id>
 ```
 The console output will shows how many workers and masters there are. For example, I have output like this:
 ```bash
 masters:
 container_1449802454214_0034_01_000002(IDHV22-01:35712)
 workers:
 container_1449802454214_0034_01_000003(IDHV22-01:35712)
 container_1449802454214_0034_01_000006(IDHV22-01:35712)
 ```

 2. To add a new worker machine, you can do:
 ```bash
   yarnclient addworker -appid <yarn application id> -count 2
 ```
 This will add two new workers machines. Run the command in first step to check whether the change is effective.
 3. To remove old machines, use:
 ```bash
   yarnclient removeworker -appid <yarn application id> -container <worker container id>
 ```
 The worker container id can be found from the output of step 1. For example "container_1449802454214_0034_01_000006" is a good container id.

### Other usage:
 1. To kill a cluster,
  ```bash
  yarnclient kill -appid <yarn application id>
  ```
 2. To check the Gearpump version
  ```bash
  yarnclient version -appid <yarn application id>
  ```
