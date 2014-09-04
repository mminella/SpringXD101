### Spring XD 101
---
This repository is the code and presentation for my introduction to Spring XD talk.  Below are the commands used in the talk.  The below commands assume that you've installed Spring XD and have configured both HDFS and Twitter.  To read more about the Spring XD installation process and configuration options see the documentation here: [http://projects.spring.io/spring-xd/](http://projects.spring.io/spring-xd/).

For the dashboard in the tap section to work, you'll need to clone the Spring XD samples repository here: [https://github.com/spring-projects/spring-xd-samples](https://github.com/spring-projects/spring-xd-samples) and install the spring command line tool found here: [http://docs.spring.io/spring-boot/docs/current-SNAPSHOT/reference/htmlsingle/#getting-started-installing-the-cli](http://docs.spring.io/spring-boot/docs/current-SNAPSHOT/reference/htmlsingle/#getting-started-installing-the-cli)

#### Twitter to File

1. Create twitter to file stream (output is in /tmp/xd/output/<STREAM_NAME>)

     ```
     xd:> stream create --definition "twittersearch --query='#icloud' | file" --name icloud --deploy
     ```
2. delete twitter to file stream

    ```
    xd:> stream destroy icloud
    ```

#### Twitter to HDFS

1. Create twitter to HDFS stream

    ```
	xd:> stream create --definition "twittersearch --query='#icloud' | hdfs" --name icloud --deploy
	```
2. View file is there: 

    ```
    $ hadoop fs -ls -h /xd/icloud
    ```
3. View file contents: 

    ```
    $ hadoop fs -tail /xd/icloud/icloud-0.log
    ```
    
4. Cleanup in HDFS

    ```
	$ hadoop fs -rmr /xd
	```

#### Tap Stream
	
1. create tap that counts hashtags

    ```
    xd:> stream create --name icloudCount --definition "tap:stream:icloud > field-value-counter --fieldName=entities.hashtags.text --name=tags" --deploy
    ```

2. View counters: 

    ```
    xd:> fieldvaluecounter display --name tags
    ```
    
3. Display dashboard from the root of the `spring-xd-samples/analytics-dashboard` directory

    ```
	$ spring run dashboard.groovy
	```
4. Direct a browser to: http://localhost:9889/dashboard.html
5. Cleanup:

    ```
    xd:> stream destroy icloudCount
    xd:> stream destroy icloud
    xd:> fieldvaluecounter delete tags
    ```

#### Custom Job

1. Build job.  From the root of the custom-xd-job module in this repository:

    ```
    $ mvn clean package
    ```
2.  Unzip the resulting zip file (in custom-xd-job/target) in the $XD_HOME/modules/job directory.
3.  Restart Spring XD
4.  View the module information/options

    ```
    xd:> module list
    xd:> module info job:customXdJob
    ```
    
5.  Deploy the job

    ```
    xd:> job create --name hiJob --definition "customXdJob" --deploy
    ```
6. Launch job (view the output in the log of Spring XD)

    ```
    xd:> job launch hiJob
    ``` 
7. Deploy job with options

    ```
    xd:> job create --name hiJobWithParams --definition "customXdJob --message \"Hello to me!\"" --deploy
    ```
8. Launch job with options (view the output in the log of Spring XD)

    ```
    xd:> job launch hiJobWithParams
    ```
9. Cleanup

    ```
    xd:> job destroy hiJob
    xd:> job destroy hiJobWithParams
    ```
