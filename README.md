# Word-counting using Spark

A simple running-through on Ubuntu to count words in the book "The Mysterious Affair at Styles" by Agatha Christie

###1. Install JRE, Scala, and Spark
Run the following commands directly on bash prompt (or by saving into a shell script and running it)
```
# Installing JRE
  sudo apt-get -y install default-jre
  #or sudo apt-get install openjdk-7-jdk

# Installing Scala
  wget -qO- http://downloads.lightbend.com/scala/2.11.8/scala-2.11.8.tgz | sudo tar -xz -C /usr/local &&
  sudo mv /usr/local/scala* /usr/local/scala &&
  sudo chown -R root:root /usr/local/scala

# Installing Spark
  wget -qO- http://apache.mirror.cdnetworks.com/spark/spark-1.6.2/spark-1.6.2-bin-hadoop2.6.tgz | sudo tar -xz -C /usr/local &&
  sudo mv /usr/local/spark* /usr/local/spark &&
  sudo chown -R root:root /usr/local/scala

# Configuring Spark (to lower the log level to WARN)
  sudo cp /usr/local/spark/conf/log4j.properties.template /usr/local/spark/conf/log4j.properties
  sudo sed -i '/log4j.rootCategory=/s/INFO/WARN/' /usr/local/spark/conf/log4j.properties

# Setting up the environment for Spark
  sed -i '$a\
\
export PATH=$PATH:/usr/local/scala/bin:/usr/local/spark/bin' ~/.bashrc
```

###2. Prepare the novel
Run this command or download it from a web browser
```
wget http://www.gutenberg.org/files/863/863-0.txt
```

###3. Prepare the Python script as `my_script.py`
```python
#!/usr/local/spark/bin/spark-submit

# reference:
# - "Learning Spark" (O'Reilly, 2015), Standalone Applications

from pyspark import SparkConf, SparkContext
import string   # for string.punctuation

conf = SparkConf().setMaster("local").setAppName("test")
    # By "local", Spark runs on one thread on the local machine

sc = SparkContext(conf = conf)  # initialize a SparkContext

lines = sc.textFile("863-0.txt")
words = ( lines
    .flatMap(lambda line: line.split())     
    .map(lambda word: word.strip(string.punctuation))   # strip punctuation characters
    .filter(lambda word: word != '')        # filter out null words
    #.map(lambda word: word.lower())        # to lower case
    .map(lambda word: (word, 1))   
    .reduceByKey(lambda a, b: a+b)
    .sortBy(lambda (word, count): -count)   # descending sort by count
)

#words.saveAsTextFile("output") # output the whole dataset into a directory

print "%d words in total." % words.count()
print "The top 20 of most frequent words are:"
print words.take(20)
```

###3. Run it
```
chmod +x ./my_script.py
./my_script.py
```

```
6728 words in total.
The top 20 of most frequent words are:
[(u'the', 2559), (u'I', 1662), (u'to', 1389), (u'of', 1344), (u'a', 1185), (u'and', 1050), (u'was', 919), (u'that', 887), (u'it', 841), (u'in', 811), (u'you', 783), (u'is', 548), (u'he', 514), (u'not', 512), (u'had', 489), (u'her', 449), (u'his', 392), (u'at', 385), (u'Poirot', 382), (u'me', 382)]
```
