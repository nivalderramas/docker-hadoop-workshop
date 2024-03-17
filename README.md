# **Prerequisites**

Have Git and Docker installed

## **1.** **Start the necessary containers using docker-compose**

Run the next command:

`[yourMachine]$ docker-compose up -d`

This will start the five necessary containers (if it is the first time you’re doing this, you’ll have to wait until the download finish)

Docker Compose allows us to run multi-container Docker applications and use multiple commands using only a YAML file.  
So calling the previous command will run all the lines in our docker-compose.yml file, and will download the necessary images, if needed.

You can check if all the five containers are running typing `docker ps` command.

## **2.** **Get in our master node “namenode”**

`[yourMachine]$ docker exec -it namenode bash`

We’re getting in our namenode container, which is the master node of our Hadoop cluster. It is basically a mini-linux. It will allow us to manage our HDFS file system.

## **3\. Create folder structure to allocate input files**

First, you can list all the files in our HDFS system

`[namenode]$ hdfs dfs -l /`

Now, we have to create a /user/root/ file, since hadoop works with this defined structure

`[namenode]$ hdfs dfs -mkdir -p /user/root`

We can verify if it was created correctly

`[namenode]$ hdfs dfs -ls /user/`

## **4.** **Download MapReduce script**

We will use a .jar file containing the classes needed to execute MapReduce algorithm. You can do this manually, compiling the .java files and zipping them. But we will download the .jar file ready to go

`[yourMachine]$ wget https://repo1.maven.org/maven2/org/apache/hadoop/hadoop-mapreduce-examples/2.7.1/hadoop-mapreduce-examples-2.7.1-sources.jar`

## **5\. Download or create the .txt file that you want to process**

Most of the time we will use Hadoop to process a fairly large file, maybe several gigabytes, but for this occasion we will use a 1MB file, the classic book “Don Quijote de La Mancha” as plain text, you can use any text you want.

You can download the text file I used here:

`[yourMachine]$ wget https://gist.github.com/jsdario/6d6c69398cb0c73111e49f1218960f79\#file-el_quijote-txt`

## **6\. Move our .jar & .txt file into the container**

First, move these from where you downloaded them and put them in the cloned repository folder. Then type the next command (you have to be in your normal terminal, not inside the namenode container, you can use “exit” command).

```
[yourMachine]$ docker cp hadoop-mapreduce-examples-2.7.1-sources.jar namenode:/tmp
# Do the same for the .txt file
[yourMachine]$ docker cp el_quijote.txt namenode:/tmp
```

## **7**. **Create the input folder inside our namenode container**

Get in the namenode container again

```
[yourMachine]$ docker exec -it namenode bash
[namenode]$ hdfs dfs -mkdir /user/root/input
```

## **8.** **Copy your .txt file from /tmp to the input file**

```
[namenode]$ cd /tmp
[namenode]$ hdfs dfs -put el_quijote.txt /user/root/input
```

## **8.1** **Visualize the dashboard in your localhost**

Acces to localhost:9870 in your browser
![Image showing the dashboard visualization](https://miro.medium.com/v2/resize:fit:1100/format:webp/1*J-zYfmksqkmfimryi7GSSA.png)

## **9\. Run MapReduce**

hadoop jar hadoop-mapreduce-examples-2.7.1-sources.jar org.apache.hadoop.examples.WordCount input output

You’ll see a large input, but if you have done the past steps right everything should be fine.

## **10.** **See the results**

`[namenode]$ hdfs dfs -cat /user/root/output/\*`

Depending on your text file, youll see something like this

![](https://miro.medium.com/v2/resize:fit:496/format:webp/1*SpbNlGrF_PHTV8LM_gyh4A.png)

You can check the results accesing to the output folder  
`[namenode]$ hdfs dfs -ls /user/root/putput`

![](https://miro.medium.com/v2/resize:fit:1100/format:webp/1*bOGOlxeXPxfnnCa3rPyGjA.png)

We’re interested in the part-r-00000 file, wich has our word count.  
We can export this file putting it in a txt file and moving to our base folder

```
[namenode]$ hdfs dfs -cat /user/root/output/part-r-00000 > /tmp/quijote_wc.txt
[namenode]$ exit
[yourMachine]$  docker cp namenode:/tmp/quijote_wc.txt .
```

Now you will have the text file in your repository folder

## **12\. Turn down the containers**

`[yourMachine]$ docker-compose down`
