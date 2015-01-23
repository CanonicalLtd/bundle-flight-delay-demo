## Machine learning workshop demo

This demo is part of a 'Machine Learning with Hadoop' webinar.
The webinar recording and slides are available at http://hortonworks.com/partners/learn

It has been ported to Ubuntu and Juju as part of an exercise to demonstrate how Juju can make it much easier to research Data Science by providing reproducible working environments. 

## Predicting Flight Delays 

Every year approximately 20% of airline flights are delayed or cancelled, resulting in significant costs to both travelers and airlines. 
As our example use-case, we will build a supervised learning model that predicts airline delay from historical flight data and weather information.
Currently there are two versions of this demo available: one with Python/Scikit-learn and one with Spark/Scala (the R demo is being developed)

More details can be found on the below Hortonworks blog posts:
- http://hortonworks.com/blog/data-science-apacheh-hadoop-predicting-airline-delays/
- http://hortonworks.com/blog/data-science-hadoop-spark-scala-part-2/

To get a better understanding of machine learning and how the models below work:
- https://www.coursera.org/course/ml

## Data Scientist experience

When connecting to the blog posts, there is a feeling of urgency as anyone wants to use the ipython-notebook and start hacking into the data immediately. 

Unfortunately, the notebook itself doesn't contain much information technically speaking, out of mentioning that the backend will be based on Hadoop, Pig and Spark, and use the Python and Scala APIs of the later. 

Moreover, unless having a personal interest in long and complex deployments, trying to deploy a Hadoop cluster made of 4 nodes (quad core, 16GB RAM) is not what one would call a painless experience. 40 pages of documentation for the *automated* installer, 232 for the manual installation. Outch! 

That is where Juju excels. As soon as one has already done it in the past, it becomes available to the masses in the time required to spin VMs on your favorite cloud or even on your local machine. 

This bundle will give you a 4-node Hortonworks HDP cluster matching the above requirement. A fifth node welcomes the YARN master, along with Apache Pig and the iPython Notebook preinstalled. 

You get out of the box access to the first  part of the blog post by connecting on the HTTPS address of the iPython Notebook.

When you are ready to move to the second part (Spark / Scala), you just have to change the configuration of the flight-delay-demo charm to tell it to switch to spark. Wait for about 10sec and refresh the Notebook page. 

## Demo Setup 

#### Setup Environment and deploy with with Juju GUI

Make sure you have Juju installed and an environment properly configured on your computer. You can find information on https://juju.ubuntu.com/docs/

Then bootstrap your environment

```
juju bootstrap
juju deploy --to 0 juju-gui
juju expose juju-gui
URL=$(juju api-endpoints | cut -f1 -d":")
PASS=$(cat ~/.juju/environments/ENV_NAME.yaml | grep pass | cut -f2 -d":")
echo "Connect on https://${URL}. Password is \"${PASS}\""
```

Now search the bundle in the GUI and deploy it. Wait for all icons to be green... This can take a little while. 

Now you can click on the iPython-notebook unit and connect on the port 443 interface... Default password is "secret", you can change that in the configuration of the charm. 

#### Setup Environment and deploy with with Juju CLI (short version)

Make sure you have Juju installed and an environment properly configured on your computer. You can find information on https://juju.ubuntu.com/docs/
Now also add the juju-quickstart option as explained on https://launchpad.net/juju-quickstart

Download the bundle 

```
git clone https://github.com/SaMnCo/bundle-flight-delay-demo.git flight-delay-demo 
cd flight-delay-demo
juju-quickstart -n flight-delay-demo bundles.yaml
```

Note that you don't have to bootstrap in that case as juju-quickstart will do that for you. Don't forget to switch to the correct environment though!
Now wait until the deployment is done. You can check for it regularly with the command

```
juju status
```

Now you can connect on the ipython-notebook HTTPS URL and start playing. Default password will be "secret" but you can change it in the configuration. 

If you don't know how to get the URL, you can use 

```
juju stat | python -c 'import sys, yaml, json; json.dump(yaml.load(sys.stdin), sys.stdout, indent=4)' | jq '.services."ipython-notebook".units."ipython-notebook/0"'
{
  "agent-state": "started",
  "machine": "XX",
  "public-address": "THIS IS THE URL YOU'RE LOOKING FOR",
  "agent-version": "1.20.14",
  "open-ports": [
    "443/tcp"
  ]
}
```

When you want to change and use the Spark version, 

```
juju set flight-delay profile="spark"
```

#### Setup Environment with Juju CLI (Manual / Long version)

Make sure you have Juju installed and an environment properly configured on your computer. You can find information on https://juju.ubuntu.com/docs/

Then bootstrap your environment

```
juju bootstrap
juju deploy --to 0 juju-gui
juju expose juju-gui
```

Then deploy an Hortonworks demo cluster as below (or use the 00-deploy script provided)

```
juju deploy --constraints "mem=16G cpu-cores=4 root-disk=128G" hdp-hadoop yarn-master
juju deploy --constraints "mem=16G cpu-cores=4 root-disk=128G" hdp-hadoop compute-node
juju set-constraints --service compute-node mem=16G cpu-cores=4 root-disk=128G

juju add-relation yarn-master:namenode compute-node:datanode
juju add-relation yarn-master:resourcemanager compute-node:nodemanager

juju add-unit -n3 compute-node

sleep 60 # This is to make sure YARN starts to deploy and there is a machine available for the notebook. 

TARGET_MACHINE=$(juju stat | python -c 'import sys, yaml, json; json.dump(yaml.load(sys.stdin), sys.stdout, indent=4)' | jq '.services."yarn-master".units."yarn-master/0".machine' | tr -d "\"" )
echo $TARGET_MACHINE
juju deploy --to ${TARGET_MACHINE} cs:~samuel-cozannet/trusty/ipython-notebook
juju expose ipython-notebook

juju deploy --to $TARGET_MACHINE hdp-pig hdp-pig
juju add-relation hdp-pig:namenode yarn-master:namenode
juju add-relation hdp-pig:resourcemanager yarn-master:resourcemanager

juju deploy cs:~samuel-cozannet/trusty/flight-delay-demo flight-delay
juju add-relation ipython-notebook flight-delay 

```

Now wait until the deployment is done. You can check for it regularly with the command

```
juju status
```

Now you can connect on the ipython-notebook HTTPS URL and start playing. Default password will be "secret" but you can change it in the configuration. 

If you don't know how to get the URL, you can use 

```
juju stat | python -c 'import sys, yaml, json; json.dump(yaml.load(sys.stdin), sys.stdout, indent=4)' | jq '.services."ipython-notebook".units."ipython-notebook/0"'
{
  "agent-state": "started",
  "machine": "XX",
  "public-address": "THIS IS THE URL YOU'RE LOOKING FOR",
  "agent-version": "1.20.14",
  "open-ports": [
    "443/tcp"
  ]
}
```

When you want to change and use the Spark version, 

```
juju set flight-delay profile="spark"
```

## Demo setup instructions

Now you can connect on the ipython-notebook HTTPS URL and start playing. Default password will be "secret" but you can change it in the configuration. 

If you don't know how to get the URL, you can use 

```
juju stat | python -c 'import sys, yaml, json; json.dump(yaml.load(sys.stdin), sys.stdout, indent=4)' | jq '.services."ipython-notebook".units."ipython-notebook/0"'
{
  "agent-state": "started",
  "machine": "XX",
  "public-address": "THIS IS THE URL YOU'RE LOOKING FOR",
  "agent-version": "1.20.14",
  "open-ports": [
    "443/tcp"
  ]
}
```

## Author's notes

First of all, most the credit is to give to Ofer Mendelevitch and Beau Plath @Hortonworks who did all the Data Science. I would not have ported it to Ubuntu and Juju if they add not produced it in the first place. I hope this work will also be interesting to them. 

I hope you will enjoy Juju as a great tool to create demos and Data Science hackathons. It's a great tool to reproduce complex, distributed environments in a cloud agnostic manner. This results in being perfect to setup hackathon VMs so that all players are on the base (deploy XX units of ipython-notebook on a public cloud) and/or client demos. 





Enjoy! 
