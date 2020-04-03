Streaming Data Processing: Streaming Data Pipelines into Bigtable
=================================================================

Overview
--------

In this lab you will use Dataflow to collect traffic events from simulated traffic sensor data made available through Google Cloud PubSub, and write them into a Bigtable table.

At the time of this writing, streaming pipelines are not available in the DataFlow Python SDK. So the streaming labs are written in Java.

Objectives
----------

In this lab, you will perform the following tasks:

-   Launch Dataflow pipeline to read from Pub/Sub and write into Bigtable

- Open an HBase shell to query the Bigtable database

Task 1: Preparation
-------------------

You will be running a sensor simulator from the training VM. There are several files and some setup of the environment required.

### Open the SSH terminal and connect to the training VM

1. In the Console, on the Navigation menu ( ![7a91d354499ac9f1.png](https://cdn.qwiklabs.com/ITyhQn00xPCVZIksELAC4wax%2BQdPdrH2DNGLazXJKgw%3D)), click Compute Engine > VM instances.

2. Locate the line with the instance called training-vm.

3. On the far right, under Connect column, Click on SSH to open a terminal window.

4. In this lab you will enter CLI commands on the training-vm.

### Verify initialization is complete

1. The training-vm is installing software in the background. Verify that setup is complete by checking that the following directory exists. If it does not exist, wait a few minutes and try again.

```sh
ls /training
```

Wait until setup is complete before proceeding. You can verify the installation of maven with mvn -version and the JDK with java -version.

### Copy files

1. A repository has been downloaded to the VM. Copy the repository to your home directory.

```sh
cp -r /training/training-data-analyst/ .
```

### Set environment variables

1. On the training-vm SSH terminal enter the following:

```sh
source /training/project_env.sh
```

#### `/training/project_env.sh` code

```sh
#! /bin/bash

# Create the DEVSHELL_PROJECT_ID on a VM
curl "http://metadata.google.internal/computeMetadata/v1/project/project-id" -H "Metadata-Flavor: Google" > Project_ID
awk '{print "export DEVSHELL_PROJECT_ID=" $0, "\n" "export BUCKET=" $0, "\n" "export JAVA_HOME=/usr/lib/jvm/java-8-openjdk-amd64/jre" }' Project_ID > env.txt
source env.txt
echo $DEVSHELL_PROJECT_ID
```

This script sets the __$$DEVSHELL\_PROJECT\_ID__ and __$$BUCKET__ environment variables.

### Prepare HBase quickstart files

1.  In the training-vm SSH terminal run the script to download and unzip the quickstart files (you will later use these to run the HBase shell.)

```sh
cd ~/training-data-analyst/courses/streaming/process/sandiego
./install_quickstart.sh
```
-------------------------------

#### `./install_quickstart.sh` code:

<details><summary>Click to expand</summary>

```sh
curl -f -O https://storage.googleapis.com/cloud-bigtable/quickstart/GoogleCloudBigtable-Quickstart-0.9.5.1.zip
unzip GoogleCloudBigtable-Quickstart-0.9.5.1.zip
```

</details>

-------------------------------

Task 2: Simulate traffic sensor data into Pub/Sub
-------------------------------------------------

1.  In the training-vm SSH terminal, start the sensor simulator. The script reads sample data from a csv file and publishes it to Pub/Sub.

```sh
/training/sensor_magic.sh
```

This command will send 1 hour of data in 1 minute. Let the script continue to run in the current terminal.

-------------------------------
#### `/training/sensor_magic.sh` code:

<details><summary>Click to expand</summary>


```sh
#! /bin/bash

# User tasks:
#  1. copy repo to ~/training-data-analyst
#  2. create $DEVSHELL_PROJECT_ID

# Install PIP
# sudo apt-get install -y python-pip
# Use PIP to install pubsub API
# sudo pip install -U google-cloud-pubsub
# Download the data file
gsutil cp gs://cloud-training-demos/sandiego/sensor_obs2008.csv.gz ~/training-data-analyst/courses/streaming/publish/

# cd to directory
cd ~/training-data-analyst/courses/streaming/publish/

# Run sensor simulator
python ./send_sensor_data.py --speedFactor=60 --project $DEVSHELL_PROJECT_ID
```
</details>

-------------------------------

#### `./send_sensor_data.py` code 

<details><summary>Click to expand</summary>

```python
#!/usr/bin/env python

# Copyright 2018 Google Inc.
#
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
#     http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.

import time
import gzip
import logging
import argparse
import datetime
from google.cloud import pubsub

TIME_FORMAT = '%Y-%m-%d %H:%M:%S'
TOPIC = 'sandiego'
INPUT = 'sensor_obs2008.csv.gz'

def publish(publisher, topic, events):
   numobs = len(events)
   if numobs > 0:
       logging.info('Publishing {0} events from {1}'.format(numobs, get_timestamp(events[0])))
       for event_data in events:
         publisher.publish(topic,event_data)

def get_timestamp(line):
   ## convert from bytes to str
   line = line.decode('utf-8')

   # look at first field of row
   timestamp = line.split(',')[0]
   return datetime.datetime.strptime(timestamp, TIME_FORMAT)

def simulate(topic, ifp, firstObsTime, programStart, speedFactor):
   # sleep computation
   def compute_sleep_secs(obs_time):
        time_elapsed = (datetime.datetime.utcnow() - programStart).seconds
        sim_time_elapsed = (obs_time - firstObsTime).seconds / speedFactor
        to_sleep_secs = sim_time_elapsed - time_elapsed
        return to_sleep_secs

   topublish = list() 

   for line in ifp:
       event_data = line   # entire line of input CSV is the message
       obs_time = get_timestamp(line) # from first column

        # how much time should we sleep?
       if compute_sleep_secs(obs_time) > 1:
          # notify the accumulated topublish
          publish(publisher, topic, topublish) # notify accumulated messages
          topublish = list() # empty out list

          # recompute sleep, since notification takes a while
          to_sleep_secs = compute_sleep_secs(obs_time)
          if to_sleep_secs > 0:
             logging.info('Sleeping {} seconds'.format(to_sleep_secs))
             time.sleep(to_sleep_secs)
       topublish.append(event_data)

   # left-over records; notify again
   publish(publisher, topic, topublish)

def peek_timestamp(ifp):
   # peek ahead to next line, get timestamp and go back
   pos = ifp.tell()
   line = ifp.readline()
   ifp.seek(pos)
   return get_timestamp(line)


if __name__ == '__main__':
   parser = argparse.ArgumentParser(description='Send sensor data to Cloud Pub/Sub in small groups, simulating real-time behavior')
   parser.add_argument('--speedFactor', help='Example: 60 implies 1 hour of data sent to Cloud Pub/Sub in 1 minute', required=True, type=float)
   parser.add_argument('--project', help='Example: --project $DEVSHELL_PROJECT_ID', required=True)
   args = parser.parse_args()

   # create Pub/Sub notification topic
   logging.basicConfig(format='%(levelname)s: %(message)s', level=logging.INFO)
   publisher = pubsub.PublisherClient()
   event_type = publisher.topic_path(args.project,TOPIC)
   try:
      publisher.get_topic(event_type)
      logging.info('Reusing pub/sub topic {}'.format(TOPIC))
   except:
      publisher.create_topic(event_type)
      logging.info('Creating pub/sub topic {}'.format(TOPIC))

   # notify about each line in the input file
   programStartTime = datetime.datetime.utcnow() 
   with gzip.open(INPUT, 'rb') as ifp:
      header = ifp.readline()  # skip header
      firstObsTime = peek_timestamp(ifp)
      logging.info('Sending sensor data from {}'.format(firstObsTime))
      simulate(event_type, ifp, firstObsTime, programStartTime, args.speedFactor)
```

</details>

-------------------------------

### Open a second SSH terminal and connect to the training VM

1.  In the upper right corner of the training-vm SSH terminal, click on the gear-shaped button ( ![9649d58acf1c4e06.png](https://cdn.qwiklabs.com/8PohZ1UPqyMVJD2%2BJxouC1NkYuEP%2BOWgiZHoYAB5yWg%3D)) and select New Connection to training-vm from the drop-down menu. A new terminal window will open.

![new_connection_vm.png](https://cdn.qwiklabs.com/1MmpBnj3VNJzd961aSrYd4ypuKs%2F%2BBKuIq28Vpw3mRk%3D)

1.  The new terminal session will not have the required environment variables. Run the following command to set them.

2.  In the new training-vm SSH terminal enter the following:

```
source /training/project_env.sh

```

Click Check my progress to verify the objective.

Simulate traffic sensor data into Pub/Sub

Check my progress

Task 3: Launch Dataflow Pipeline
--------------------------------

1. In the second training-vm SSH terminal, navigate to the directory for this lab. Examine the script in Cloud Shell or using nano. Do not make any changes to the code.

```sh
cd ~/training-data-analyst/courses/streaming/process/sandiego

nano run_oncloud.sh
```

-------------------------------
#### `run_oncloud.sh` code

<details><summary>Click to expand</summary>

```sh

#!/bin/bash
if [ "$#" -lt 3 ]; then
echo "Usage:   ./run_oncloud.sh project-name bucket-name classname [options] "
echo "Example: ./run_oncloud.sh cloud-training-demos cloud-training-demos CurrentConditions --bigtable"
exit
fi
PROJECT=$1
shift
BUCKET=$1
shift
MAIN=com.google.cloud.training.dataanalyst.sandiego.$1
shift
echo "Launching $MAIN project=$PROJECT bucket=$BUCKET $*"
export PATH=/usr/lib/jvm/java-8-openjdk-amd64/bin/:$PATH
mvn compile -e exec:java \
-Dexec.mainClass=$MAIN \
    -Dexec.args="--project=$PROJECT \
    --stagingLocation=gs://$BUCKET/staging/ $* \
    --tempLocation=gs://$BUCKET/staging/ \
    --runner=DataflowRunner"
# If you run into quota problems, add this option the command line above
#     --maxNumWorkers=2 
# In this case, you will not be able to view autoscaling, however.

```

</details>

-------------------------------

What does the script do?

1. The script takes 3 required arguments: project id, bucket name, classname and possibly a 4th argument: options. In this part of the lab, we will use the `--bigtable` option which will direct the pipeline to write into Cloud Bigtable.

2. Run the following script to create the Bigtable instance.

```sh
cd ~/training-data-analyst/courses/streaming/process/sandiego

./create_cbt.sh
```
-------------------------------

#### `./create_cbt.sh` code

```sh
gcloud beta bigtable instances create sandiego --cluster=cpb210 --cluster-zone=us-central1-b --display-name=="San Diego Freeway data" --instance-type=DEVELOPMENT
```

-------------------------------

1. Run the Dataflow pipeline to read from PubSub and write into Cloud Bigtable.

```sh
cd ~/training-data-analyst/courses/streaming/process/sandiego

./run_oncloud.sh $DEVSHELL_PROJECT_ID $BUCKET CurrentConditions --bigtable

```

Example successful run:

```
[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
[INFO] Total time: 47.582 s
[INFO] Finished at: 2018-06-08T21:25:32+00:00
[INFO] Final Memory: 58M/213M
[INFO] ------------------------------------------------------------------------
```

Click Check my progress to verify the objective.

Launch Dataflow Pipeline

Check my progress

Task 4: Explore the pipeline
----------------------------

1.  Return to the browser tab for Console. On the Navigation menu ( ![7a91d354499ac9f1.png](https://cdn.qwiklabs.com/ITyhQn00xPCVZIksELAC4wax%2BQdPdrH2DNGLazXJKgw%3D)), click Dataflow and click on the new pipeline job. Confirm that the pipeline job is listed and verify that it is running without errors.

2.  Find the write:cbt step in the pipeline graph, and click on the down arrow on the right to see the writer in action. Click on given writer. Review the Bigtable Options in the Step summary.

Task 5: Query Bigtable data
---------------------------

1.  In the second training-vm SSH terminal, run the quickstart.sh script to launch the HBase shell.

```
cd ~/training-data-analyst/courses/streaming/process/sandiego/quickstart

./quickstart.sh

```

-------------------------------

>#### `./quickstart.sh` code
<details><summary>Click to expand</summary>

```sh
#!/bin/bash
#
#    Copyright 2015 Google, Inc.
#
#    Licensed under the Apache License, Version 2.0 (the "License");
#    you may not use this file except in compliance with the License.
#    You may obtain a copy of the License at
#
#        http://www.apache.org/licenses/LICENSE-2.0
#
#    Unless required by applicable law or agreed to in writing, software
#    distributed under the License is distributed on an "AS IS" BASIS,
#    WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
#    See the License for the specific language governing permissions and
#    limitations under the License.set -x -e

# quickstart.sh
#
# This will start hbase shell using the pom.xml, assuming you have:
# 1. gcloud auth login
# 2. either given --project NAME or gcloud config set project XXXXX
# 3. have created a Cloud Bigtable Instance

# Prequsites: gcloud, mvn, Java

# Allow overriding the date function for unit testing.
function my_date() {
  date "$@"
}

# Simple wrapper around "echo" so that it's easy to add log messages with a
# date/time prefix.
function loginfo() {
  echo "$(my_date): ${@}"
}

# Simple wrapper around "echo" controllable with ${VERBOSE_MODE}.
function logdebug() {
  if (( ${VERBOSE_MODE} )); then
    loginfo ${@}
  fi
}

# Simple wrapper to pass errors to stderr.
function logerror() {
  loginfo ${@} >&2
}

# Handler for errors occuring during the deployment to print useful info before
# exiting. The following global variables control whether handle_error() should
# actually process and consolidate a trapped error, or otherwise simply flip
# CAUGHT_ERROR to '1' without trying to consolidate logs or exiting in case
# the caller wants to simply continue on error.
SUPPRESS_TRAPPED_ERRORS=0
CAUGHT_ERROR=0
function handle_error() {
  # Save the error code responsible for the trap.
  local errcode=$?
  local bash_command=${BASH_COMMAND}
  local lineno=${BASH_LINENO[0]}

  CAUGHT_ERROR=1

  if (( ${SUPPRESS_TRAPPED_ERRORS} )); then
    loginfo "Continuing despite trapped error with code '${errcode}'"
    return
  fi

  # Wait for remaining async things to finish, otherwise our error message may
  # get lost among other logspam.
  wait
  logerror "Command failed: ${bash_command} on line ${lineno}."
  logerror "Exit code of failed command: ${errcode}"

  consolidate_error_logs
  exit ${errcode}
}

# Given $1 prints and reads a response from the console.
SKIP_PROMPT=0
function prompt() {
  trap handle_error ERR
  local msg="$1"

  
  read -p "${msg}" PROMPT_RESPONSE
}

# Test for java
hash java 2>/dev/null  || { echo >&2 'Java needs to be installed'; exit 1; }

# Test for Maven
hash mvn 2>/dev/null  || { echo >&2 'Apache Maven needs to be installed.'; exit 1; }

# Test for gcloud
hash gcloud 2>/dev/null  || { echo >&2 'gcloud needs to be installed from https://cloud.google.com/sdk/'; exit 1; }
NOTLOGGEDIN=$(gcloud auth list --format text 2>/dev/null | grep active_account | grep None)
if [[ -n "$NOTLOGGEDIN" ]]; then
  echo >&2 'Please login using: gcloud init'; exit 1;
fi

if [ "$1" == "--project" ]; then
  _projectID=$2
else
  # If possible set a default project
  _defProj=$(gcloud config list project 2>/dev/null | grep project)
  if [ $? -eq 0 ] && [ -n "${_defProj}" ]; then
    _projectID="${_defProj##project = }"

    HAVEPROJECT=$(gcloud projects list --format 'value (projectId)' 2>/dev/null | grep "${_projectID}")
    if [ $? -ne 0 ]; then
      { echo "Project ${_projectID} not found."; exit 1; }
    fi
  else
    ix=1;for item in $(gcloud projects list --format='value (projectId)' 2>/dev/null ); do PROJ[$ix]=$item; echo $ix. $item; ((ix++)); done
    prompt "Select the project? "
    _projectID=${PROJ[$PROMPT_RESPONSE]}
  fi
fi

# Test for api enabled

HAVECLCMD=$(gcloud beta bigtable instances list --project ${_projectID} 2>&1 1>/dev/null | grep ERROR)
if [[ $HAVECLCMD == ERROR* ]]; then
  echo "Project ID= ${_projectID}"
  prompt 'Instance ID= '
  _instanceID=$PROMPT_RESPONSE
else
  ix=0
  for item in $(gcloud beta bigtable clusters list  --format 'value (INSTANCE)'); do
    _c[$ix]=$item
    ((ix++))
  done
  if [ $ix -eq 1 ]; then
    _instanceID=${_c[0]}
  else
    echo "Please choose an Instance to work with."

    for ((i=0; i<$ix; i++)); do
      echo $i. ${_c[$i]}
    done
    prompt 'Instance ID= '
    _instanceID=${_c[$PROMPT_RESPONSE]}
  fi
fi

echo "Project ID = ${_projectID}"
echo "Instance ID= ${_instanceID}"

mvn clean package exec:java -Dbigtable.projectID=${_projectID} -Dbigtable.instanceID=${_instanceID}
```

</details>

-------------------------------

1.  If the script runs successfully, you would be in an HBase shell prompt that looks something like this:

```
hbase(main):001:0>
```

1.  At the HBase shell prompt, type the following query to retrieve 2 rows from your Bigtable table that was populated by the pipeline.

```
scan 'current_conditions', {'LIMIT' => 2}

```

1.  Review the output. Notice each row is broken into column, timestamp, value combinations.

-------------------------------
#### Output

<details><summary>Click to expand </summary>

```
hbase(main):001:0> scan 'current_conditions', {'LIMIT' => 2}
ROW                                                 COLUMN+CELL                                                                                                                                         
 15#S#1#9223370811328675807                         column=lane:direction, timestamp=1225526100, value=S                                                                                                
 15#S#1#9223370811328675807                         column=lane:highway, timestamp=1225526100, value=15                                                                                                 
 15#S#1#9223370811328675807                         column=lane:lane, timestamp=1225526100, value=1.0                                                                                                   
 15#S#1#9223370811328675807                         column=lane:latitude, timestamp=1225526100, value=32.723248                                                                                         
 15#S#1#9223370811328675807                         column=lane:longitude, timestamp=1225526100, value=-117.115543                                                                                      
 15#S#1#9223370811328675807                         column=lane:sensorId, timestamp=1225526100, value=32.723248,-117.115543,15,S,1                                                                      
 15#S#1#9223370811328675807                         column=lane:speed, timestamp=1225526100, value=71.3                                                                                                 
 15#S#1#9223370811328675807                         column=lane:timestamp, timestamp=1225526100, value=2008-11-01 07:55:00                                                                              
 15#S#1#9223370811328975807                         column=lane:direction, timestamp=1225525800, value=S                                                                                                
 15#S#1#9223370811328975807                         column=lane:highway, timestamp=1225525800, value=15                                                                                                 
 15#S#1#9223370811328975807                         column=lane:lane, timestamp=1225525800, value=1.0                                                                                                   
 15#S#1#9223370811328975807                         column=lane:latitude, timestamp=1225525800, value=32.706184                                                                                         
 15#S#1#9223370811328975807                         column=lane:longitude, timestamp=1225525800, value=-117.120565                                                                                      
 15#S#1#9223370811328975807                         column=lane:sensorId, timestamp=1225525800, value=32.706184,-117.120565,15,S,1                                                                      
 15#S#1#9223370811328975807                         column=lane:speed, timestamp=1225525800, value=68.5                                                                                                 
 15#S#1#9223370811328975807                         column=lane:timestamp, timestamp=1225525800, value=2008-11-01 07:50:00                                                                              
2 row(s) in 0.7350 seconds
```
</details>

-------------------------------

2.  Run another query. This time look only at the lane: speed column, limit to 10 rows, and specify rowid patterns for start and end rows to scan over.

```
scan 'current_conditions', {'LIMIT' => 10, STARTROW => '15#S#1', ENDROW => '15#S#999', COLUMN => 'lane:speed'}

```

1.  Review the output. Notice that you see 10 of the column, timestamp, value combinations, all of which correspond to Highway 15. Also notice that column is 
restricted to lane: speed.

-------------------------------
#### Output

<details><summary>Click to expand</summary>

```

hbase(main):002:0> scan 'current_conditions', {'LIMIT' => 10, STARTROW => '15#S#1', ENDROW => '15#S#999', COLUMN => 'lane:speed'}
ROW                                                 COLUMN+CELL                                                                                                                                         
 15#S#1#9223370811319375807                         column=lane:speed, timestamp=1225535400, value=70.4                                                                                                 
 15#S#1#9223370811319675807                         column=lane:speed, timestamp=1225535100, value=71.3                                                                                                 
 15#S#1#9223370811319975807                         column=lane:speed, timestamp=1225534800, value=72.2                                                                                                 
 15#S#1#9223370811320275807                         column=lane:speed, timestamp=1225534500, value=72.1                                                                                                 
 15#S#1#9223370811320575807                         column=lane:speed, timestamp=1225534200, value=76.5                                                                                                 
 15#S#1#9223370811320875807                         column=lane:speed, timestamp=1225533900, value=72.1                                                                                                 
 15#S#1#9223370811321175807                         column=lane:speed, timestamp=1225533600, value=75.8                                                                                                 
 15#S#1#9223370811321475807                         column=lane:speed, timestamp=1225533300, value=75.3                                                                                                 
 15#S#1#9223370811321775807                         column=lane:speed, timestamp=1225533000, value=74.3                                                                                                 
 15#S#1#9223370811322075807                         column=lane:speed, timestamp=1225532700, value=72.1                                                                                                 
10 row(s) in 0.1530 seconds
```
</details>

-------------------------------

2.  Feel free to run other queries if you are familiar with the syntax. Once you're satisfied, `quit` to exit the shell.

```sh
quit
```

Cleanup
-------

1. Run the script to delete your Bigtable instance. If prompted, press `Enter`.

```sh
cd ~/training-data-analyst/courses/streaming/process/sandiego
./delete_cbt.sh
```

#### `./delete_cbt.sh` code

`gcloud beta bigtable instances delete sandiego`


1.  On your Dataflow page in your Cloud Console, click on the pipeline job name and click the `Stop job` on the right panel.
2.  Go back to the first Cloud Shell tab with the publisher and type `Ctrl+C` to stop it.
3.  Go to the BigQuery console and delete the dataset demos.

End your lab
------------

When you have completed your lab, click End Lab. Qwiklabs removes the resources you've used and cleans the account for you.
