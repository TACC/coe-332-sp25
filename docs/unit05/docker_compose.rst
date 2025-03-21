Docker Compose
==============

Up to this point, we have been looking at single-container applications - small
units of code that are containerized, executed *ad hoc* to generate or read a
JSON file, then exit on completion. But what if we want to do something more
complex? For example, what if our goal is to orchestrate a multi-container
application consisting of, e.g., a Flask app, a database, a message queue, an
authentication service, and more.

**Docker compose** is a tool for managing multi-container applications. A YAML
file is used to define all of the application service, and a few simple commands
can be used to spin up or tear down all of the services.

In this module, we will get a first look at Docker compose. Later in this course
we will do a deeper dive into advanced container orchestration. After going
through this module, students should be able to:

* Translate Docker run commands into YAML files for Docker compose
* Run commands inside *ad hoc* containers using Docker compose
* Manage small software systems composed of more than one script, and more than
  one container
* Copy data into and out of containers as needed


Another Script, Another Container
---------------------------------

We have been working a lot with a script for reading in and analyzing a
JSON file of meteorite landing data. Let's quickly write a new script to
generate that data, then we will package it into its own container. Consider the
following script for **generating** sample Meteorite Landing data like we have
been working with:

.. code-block:: python3
    :linenos:
   
    #!/usr/bin/env python3
    import json
    import random
    import sys
    import names
    
    NUM = 10
    CLASSES = ['CI1', 'CR2-an', 'CV3', 'EH4', 'H4', 'H5', 'H6', 'L5', 'L6', 'LL3-6', 'LL5']
    
    def generate_meteors(num:int):
        """
        Generate and return randomly generated meteors in list of dicts format
        """
        meteors = []
        for i in range(num):
            meteor = {}
            rand_lat = '{:.4f}'.format(random.uniform(-90.0000, 90.0000))
            rand_lon = '{:.4f}'.format(random.uniform(-90.0000, 90.0000))
            meteor['name'] = names.get_first_name()
            meteor['id'] = str(10000 + 1 + i)
            meteor['recclass'] = random.choice(CLASSES)
            meteor['mass (g)'] = str(random.randrange(1, 10000))
            meteor['reclat'] = rand_lat
            meteor['reclong'] = rand_lon
            meteor['GeoLocation'] = f'({rand_lat}, {rand_lon})'
            meteors.append(meteor)
        return meteors
    
    def main():
        data = {'meteorite_landings': generate_meteors(NUM)}
        with open(sys.argv[1], 'w') as o:
            json.dump(data, o, indent=2)
            print(f'Data written to {sys.argv[1]}!')
    
    if __name__ == '__main__':
        main()



Copy that into a file called ``gen_ml_data.py``, 
(also available at `this link <https://raw.githubusercontent.com/tacc/coe-332-sp25/main/docs/unit05/scripts/gen_ml_data.py>`_)
save it, make it executable, and test it. You'll find that this script requires a
**command line argument**. Meaning we have to invoke it AND pass some information
on the command line in order to get it to work. In this case, it is expecting the
name of the output file.

.. code-block:: console

   # copy contents into file called ``gen_ml_data.py`` and save
   [coe332-vm]$ chmod +rx gen_ml_data.py
   [coe332-vm]$ ./gen_ml_data.py
   Traceback (most recent call last):
     File "./gen_ml_data.py", line 29, in <module>
       main()
     File "./gen_ml_data.py", line 25, in main
       with open(sys.argv[1], 'w') as o:
   IndexError: list index out of range

   # need to provide output filename on command line
   [coe332-vm]$ ./gen_ml_data.py data.json
   Data written to data.json!
   [coe332-vm]$ ls
   data.json  Dockerfile  gen_ml_data.py  Meteorite_Landings.json  ml_data_analysis.py
   [coe332-vm]$ head -n11 data.json
   {
     "meteorite_landings": [
       {
         "name": "Sandra",
         "id": "10001",
         "recclass": "EH4",
         "mass (g)": "4119",
         "reclat": "73.8716",
         "reclong": "14.8207",
         "GeoLocation": "(73.8716, 14.8207)"
       },

Containerizing this script should be easy enough - we already worked through
containerizing another very similar script. Let's say for this new script we do
not need the ``pytest`` dependency, because there is not really anything
interesting to test. But, we do need a different dependency: the Python3
``names`` library.

To make things a little more clear, rename the existing Dockerfile as
``Dockerfile-analysis``, and make a copy of it called ``Dockerfile-gen``.

.. code-block:: console

   [coe332-vm]$ mv Dockerfile Dockerfile-analysis
   [coe332-vm]$ cp Dockerfile-analysis Dockerfile-gen
   [coe332-vm]$ ls
   data.json       Dockerfile-analysis      Dockerfile-gen
   gen_ml_data.py  Meteorite_Landings.json  ml_data_analysis.py

Edit ``Dockerfile-gen`` as follows:

.. code-block:: Dockerfile
   :linenos:
   :emphasize-lines: 3,5,7

   FROM python:3.12

   RUN pip3 install pytest==8.3.4 names==0.3.0

   COPY gen_ml_data.py /code/gen_ml_data.py

   RUN chmod +rx /code/gen_ml_data.py

   ENV PATH="/code:$PATH"


Now that we have a Dockerfile named something other than the default name, we
need to modify our command line a little bit to build it:

.. code-block:: console

   [coe332-vm]$ docker build -t username/gen_ml_data:1.0 -f Dockerfile-gen ./

After the image is successfully built, change directories to a new folder just
to be sure you are not running the local scripts or looking at the local data.
Then, test the container as follows:

.. code-block:: console

   [coe332-vm]$ mkdir test
   [coe332-vm]$ cd test
   [coe332-vm]$ ls
   [coe332-vm]$ docker run --rm username/gen_ml_data:1.0 gen_ml_data.py ml.json
   Data written to ml.json!

If you list your local files, can you find ``ml.json``? No! This is because
whatever data generated inside the container is lost when the container
completes its task. What we need to do is use the ``-v`` flag to mount a directory
somewhere inside the container, write data into that directory, then the data will
be captured after the container exists. For example:

.. code-block:: console

   [coe332-vm]$ docker run --rm -v $PWD:/data username/gen_ml_data:1.0 gen_ml_data.py /data/ml.json
   Data written to ml.json!

.. note::

   To reiterate, because we mounted our current location as a folder called "/data"
   (``-v $PWD:/data``), and we made sure to write the output file to that location in
   the container (``gen_ml_data.py /data/ml.json``), then we get to keep the file
   after the container exits, and it shows up in our current location (``$PWD``).

Alas, there is one more issue to address. The new file is owned by root, simply
because it is root who created the file inside the container. This is one minor
Docker annoyance that we run in to from time to time. The simplest fix is to use
one more ``docker run`` flag (``-id``)to specify the user and group ID namespace
that should be used inside the container.

.. code-block:: console

   [coe332-vm]$ ls -l
   total 4
   -rw-r--r--. 1 root root 2098 Feb 21 22:39 ml.json
   [coe332-vm]$ rm ml.json
   rm: remove write-protected regular file ml.json’? y
   [coe332-vm]$ docker run --rm -v $PWD:/data -u $(id -u):$(id -g) username/gen_ml_data:1.0 gen_ml_data.py /data/ml.json
   Data written to /data/ml.json!
   [coe332-vm]$ ls -l
   total 4
   -rw-r--r--. 1 ubuntu ubuntu 2098 Feb 21 22:41 ml.json



EXERCISE
~~~~~~~~

Spend a few minutes testing both containers. Be sure you can generate data with
one container, then read in and analyze the same data with the other. Data needs
to persist outside the containers in order to do this.



Write a Compose File
--------------------

Docker compose works by interpreting rules declared in a YAML file (typically
called ``docker-compose.yml``). The rules we will write will replace the
``docker run`` commands we have been using, and which have been growing quite
complex. For example, the commands we used to run our JSON parsing scripts in a
container looked like the following:

.. code-block:: console

   [coe332-vm]$ docker run --rm -v $PWD:/data -u $(id -u):$(id -g) username/gen_ml_data:1.0 gen_ml_data.py /data/ml.json
   [coe332-vm]$ docker run --rm -v $PWD/ml.json:/data/ml.json username/ml_data_analysis:1.0 ml_data_analysis.py /data/ml.json

The above ``docker run`` commands can be loosely translated into a YAML file.
Navigate to the folder that contains your Python scripts and Dockerfiles, then
create a new empty file called ``docker-compose.yml``:

.. code-block:: console

   [coe332-vm]$ pwd
   /home/ubuntu/coe-332/docker-exercise
   [coe332-vm]$ touch docker-compose.yml
   [coe332-vm]$ ls
   docker-compose.yml  Dockerfile-analysis  Dockerfile-gen  gen_ml_data.py  ml_data_analysis.py  data/


Next, open up ``docker-compose.yml`` with your favorite text editor and type /
paste in the following text:

.. code-block:: yaml
   :linenos:
   :emphasize-lines: 9,12,18

   ---
   services:
       gen-data:
           build:
               context: ./
               dockerfile: ./Dockerfile-gen
           image: username/gen_ml_data:1.0
           volumes:
               - ./data:/data
           user: "1000:1000"
           command: gen_ml_data.py /data/ml.json
       analyze-data:
           build:
               context: ./
               dockerfile: ./Dockerfile-analysis
           depends_on:
               - gen-data
           image: username/ml_data_analysis:1.0
           volumes:
               - ./data:/data
           command: ml_data_analysis.py /data/ml.json

.. warning::

   The highlighted lines above may need to be edited with your username / userid /
   groupid in order for this to work. See instructions below.

The ``services`` section defines the configuration of individual container
instances that we want to orchestrate. In our case, we define two called
``gen-data`` for the gen_ml_data functionality, and ``analyze-data`` for
the ml_data_analysis functionality.

Each of those services is configured with its own Docker image,
a mounted volume (equivalent to the ``-v`` option for ``docker run``), a user
namespace (equivalent to the ``-u`` option for ``docker run``), and a default
command to run.

Please note that the image name above should be changed to use your image. Also,
the user ID / group ID are specific to ``ubuntu`` - to find your user and group
ID, execute the Linux commands ``id -u`` and ``id -g``.

.. note::

   The top-level ``services`` keyword shown above is just one important part of
   Docker compose. Later in this course we will look at named volumes and
   networks which can be configured and created with Docker compose.


Running Docker Compose
----------------------

The Docker compose command line tool follows the same syntax as other Docker
commands:

.. code-block:: console

   docker compose <verb> <parameters>

Just like Docker, you can pass the ``--help`` flag to ``docker compose`` or to
any of the verbs to get additional usage information. To get started on the
command line tools, try issuing the following two commands:

.. code-block:: console

   [coe332-vm]$ docker compose version
   [coe332-vm]$ docker compose config

The first command prints the version of Docker compose installed, and the second
searches your current directory for ``docker-compose.yml`` and checks that it
contains only valid syntax.

To run one of these services, use the ``docker compose run`` verb, and pass the
name of the service as defined in your YAML file:

.. code-block:: console

   [coe332-vm]$ ls test/     # currently empty
   [coe332-vm]$ docker compose run gen-data
   Data written to /data/ml.json!
   [coe332-vm]$ ls test/
   ml.json               # new file!
   [coe332-vm]$ docker compose run analyze-data
   6004.5
   Southern & Eastern
   ... etc.


Now we have an easy way to run our *ad hoc* services consistently and
reproducibly. Not only does ``docker-compose.yml`` make it easier to run our
services, it also represents a record of how we intend to interact with this
container.



Essential Docker Compose Command Summary
----------------------------------------

+------------------------+------------------------------------------------+
| Command                | Usage                                          |
+========================+================================================+
| docker compose version | Print version information                      |
+------------------------+------------------------------------------------+
| docker compose config  | Validate docker-compose.yml syntax             |
+------------------------+------------------------------------------------+
| docker compose up      | Spin up all services                           |
+------------------------+------------------------------------------------+
| docker compose down    | Tear down all services                         |
+------------------------+------------------------------------------------+
| docker compose build   | Build the images listed in the YAML file       |
+------------------------+------------------------------------------------+
| docker compose run     | Run a container as defined in the YAML file    |
+------------------------+------------------------------------------------+


Additional Resources
--------------------

* `Docker Compose Docs <https://docs.docker.com/compose/>`_
