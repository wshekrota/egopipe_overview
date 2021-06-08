# egopipe_overview
## The Big Picture
I've had  a number of small inquiries that seemed uncertain as to function so I wanted to 
provide some thinking alignment. I am not changing Elastic or providing deployment or configuration 
of their product.

Consider Egopipe an extension of logstash. Deployment will still be relatively the same. I provide 
a template logstash pipeline.conf which you can see just launches my pipe. So as I say in my setup 
we concern ourselves with executable, config file, logstash yaml file and pipeline.conf all pretty 
barebones. (4 files) Deployment is only placing those files.

Of course I am  assuming that filebeat is likely already configured and running? What kinds of logs are 
being sent? It is best for test purposes that you only use one type so you know the data layout.

The executable is the result of either copying a your_pipe_code.go file in the egopipe directory
or writing your own. Then compile or 'go build ./...'. Then you have an egopipe executable.

What does deployment look like? That depends on your elastic setup. Basically you will copy 
those simple files to the proper logstash directories. 

Here is what I do in the test Dockerfile.. If you use these Dockerfiles in a purely test environment
they will place all files for you. They just need to be current in the Dockerfile directory.
```
WORKDIR /etc/logstash/conf.d
RUN mkdir /var/log/logstash/ego
RUN mkdir /etc/logstash/conf.d/ego
COPY pipeline.conf .
COPY egopipe.cfg ego
COPY egopipe ego
COPY logstash.yml /etc/logstash
RUN chmod 777 /etc/logstash/logstash.yml
WORKDIR /usr/share/logstash
RUN bin/logstash -f /etc/logstash/conf.d --path.settings=/etc/logstash
```

so 
```
pipeline.conf ->  /etc/logstash/conf.d
egopipe       ->  /etc/logstash/conf.d/ego
egopipe.cfg   ->  /etc/logstash/conf.d/ego
logstash.yml  ->  /etc/logstash
```
Once in place the application logstash may be started. Please be aware this is still beta.

Either create your own placement scripts or manually copy files.
Or use the Dockerfiles I created. They assume those 4 files are in the current directory.

If you just want to try out egopipe I advise you do the following. Operations will be quite simple 
and you can do a recompile and deploy it by simply control-C out of logstash.
- copy the new executable
- docker build . 

Each elastic application has a Dockerfile directory. You can cd to it and start it with a 'docker build'.

#### clones

https://github.com/wshekrota/egopipe_containers.git

https://github.com/wshekrota/egopipe.git


cd to the appropriate apps directory where you cloned {where clone is}/egopipe_containers/apps/elastic

likely you can use the config as-is

start that app 'docker build .'

try it 'curl -i http://172.17.0.2:9200' (should return version)

In this test caintainer instance if you control-C and docker build again the database will be reset so remember that.


so now lets do logstash

In another window cd {where clone is}/egopipe_containers/apps/logstash

If you look at those files you have a Dockerfile and the 4 files it will use. You need to update the executable at the minimum. (so you have your code)

For your first test maybe just add a test field to the docs? These lines are added to the your_pipe_code.go. Change to the egopipe repo and 

Edit the your_pipe_code.go adding the following lines.

---
            // Create a field in the doc
            h["test"] = "this is a test"
   
---

Then 'go build ./...' will compile the new egopipe. Copy it to {where clone is}/egopipe_containers/apps/logstash

Change to that directory.

Then 'docker build .' should start logstash. 

Rinse and repeat .... do the same thing for Kibana!
Then they should all be running.
Rewriting the transform stage or the your_pipe_code.go file is then as simple as control-C, copy and rebuild.
