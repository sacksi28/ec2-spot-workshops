---
title: "Launch a cluster - Step 1"
weight: 60
---

In this step first you will launch EMR cluster that runs on a mix of On-Demand and Spot Instances. Next, you will add step that runs a simple wordcount Spark application against a public dataset of Amazon product reviews located in an Amazon S3 bucket in the N. Virginia region. 

{{% notice note %}}
Ideally your dataset on S3 would be located in the same region where you run EMR clusters. In this workshop you might run EMR in a different region than N. Virginia, however Spark application will work without any issues with the dataset located in S3 bucket in the N. Virginia region. If you want to know more about the Amazon Customer Reviews Dataset, [click here] (https://s3.amazonaws.com/amazon-reviews-pds/readme.html)
{{% /notice %}}

To launch the cluster, follow these steps:

1. [Open the EMR console](https://console.aws.amazon.com/elasticmapreduce/home) in the region where you are looking to launch your cluster.  
1. Click "**Create cluster**"  
1. Click "**Go to advanced options**"  
1. Select the latest EMR 5.x.x release (the console will default to it), and in the list of components, only select **Hadoop**, **Spark** and **Ganglia**. You will use Ganglia to monitor our cluster resource utilization.  
1. Skip to "**Steps (Optional)**" section and add a step as per below instructions:

* For the option **After last step completes** select **Clusters enters waiting state**. Since you will be examining the cluster during and after the Spark application, you can terminate the cluster at the end of the workshop manually instead of EMR auto-terminating immediately after the completion of step.

 {{% notice note %}} 
 **Cluster auto-terminates** cluster after the last step is completed is a powerful EMR feature that is used for running transient clusters. This is an effective model for clusters that perform periodic processing tasks, such as a daily data processing run, event-driven ETL workloads, etc.
 You will not be running a transient cluster, since it might terminate before you complete some of the next steps in the workshop.
 {{% /notice %}}

* In the **Step type** drop down menu, select **Spark application** and click on **Add step**, then add the following details in the **Add step** dialog window:  

  * **Spark-submit options**: here you configure the memory and core count for each executor, as described in the previous section. Use these settings (make sure you have two '-' chars):  
  ```
  --executor-memory 18G --executor-cores 4
  ```
  * **Application location**: here you configure the location of Spark application. Save the following python code to a file (or download it from the Attachment box) and upload it to your S3 bucket using the AWS management console. You can refer to the [S3 Getting Started guide](https://docs.aws.amazon.com/AmazonS3/latest/gsg/PuttingAnObjectInABucket.html) for detailed instructions

    ```python
    import sys
    from pyspark.sql import SparkSession
    spark = SparkSession.builder.appName('Amazon reviews word count').getOrCreate()
    df = spark.read.parquet("s3://amazon-reviews-pds/parquet/")
    df.selectExpr("explode(split(lower(review_body), ' ')) as words").groupBy("words").count().write.mode("overwrite").parquet(sys.argv[1])
    exit()
    ```
    {{%attachments style="orange" /%}}


  * Next add the location of the file under the **Application location** field, i.e: s3://\<your-bucket-name\>/script.py

      * **Arguments**: Here you will configure the location of where Spark will write the results of the job. Enter: s3://\<your-bucket-name\>/results/  
      * **Action on failure**: Leave this as **Continue** 
      


    ![sparksubmit](/images/running-emr-spark-apps-on-spot/sparksubmitstep1.png)

  * Click **Add** to save the step.

Click **Next** to continue setting up the EMR cluster and move from "**Step 1: Software and steps**"" to "**Step 2: Hardware**".



