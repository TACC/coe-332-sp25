Frequently Encountered Problems
===============================

Find solutions to common problems below.

.. dropdown:: Connect VS Code to Jetstream VM

   1. In VS code, download the Remote - SSH plugin
   2. On your local laptop, edit the file ``~/.ssh/config`` to include the following:
   
   .. code-block:: console
      
      Host student-login
          HostName student-login.tacc.utexas.edu
          User taccusername
      Host coe332-vm
          User ubuntu
          HostName 129.xx.xx.xx
          ProxyCommand ssh -o "ForwardAgent yes" student-login "ssh-add && nc %h %p"

   3. Replace ``taccusername`` with your TACC username
   4. Replace the IP address I have above (129.xx.xx.xx) with the IP address of your
      private Jetstream VM. An easy way to do this is to login to your Jetstream VM,
      type “ip addr” and look for the address that begins 129.xx.xx.xx

   To verify that this is working, try to do “ssh coe332-vm” from your local laptop.
   It should ask you for your student-login username and password, then it should log
   you directly into ubuntu@coe332-vm (the Jetstream VM).

   If that is working, try connecting in VS Code directly to “coe332-vm” - it will ask
   you for your username and password. You don’t need to establish a separate connection
   to student-login first.
   


.. dropdown:: Cannot push to GitHub

   The most common issue is an improperly configured upstream on your local
   repo. If you try to ``git push`` and you are prompted for your username and
   password, then are presented with an error like:

   .. code-block:: console
      
      fatal: Authentication failed for 'https://github.com/username/reponame.git/'
   
   This is because you are using the wrong upstream URL. To solve:

   1. Make sure you upload public SSH key to GitHub settings
   2. If using a non-standard public key name, modify your ``~/.ssh/config``
      accordingly
   3. Fix the upstream URL by executing the following in the git repo directory:

   .. code-block:: console
      
      git remote set-url origin git@github.com/username/reponame.git

   Replacing 'username' with your GitHub username and 'reponame' with the name
   of the repo as it appears on GitHub. 


.. dropdown:: Cannot login to Jetstream VM

   If you try to ssh to coe332-vm and it gives any sort of error or if it asks
   you for a password, it is likely that you removed some necessary lines from
   your ~/.ssh/config on student-login. Contact one of the course instructors
   and we will restore it for you.


.. dropdown:: How do I copy data from my local laptop to coe332-vm?

   Assuming you have a file like 'data.csv' on your local laptop and you want to
   copy it to coe332-vm. It will require two hops: first copy from laptop to
   student-login, then from student-login to coe332-vm:

   .. code-block:: console

      [local]$ ls
      data.csv
      [local]$ scp data.csv username@student-login.tacc.utexas.edu:~/
      Password:
      TACC_Token:

   The file 'data.csv' is now copied to your home directory on student-login,
   so ssh to student-login and from there copy it to coe332-vm:

   .. code-block:: console

      [local]$ ssh username@student-login.tacc.utexas.edu
      Password:
      TACC_Token:
      [student-login]$ ls
      data.csv
      [student-login]$ scp data.csv coe332-vm:~/
      # no password or token prompt 


.. dropdown:: How do I copy an image from coe332-vm to my local laptop?

   Assuming you have a file like 'output.png' on coe332-vm that you want to copy
   to your local laptop. It will require two hops: first copy from coe332-vm
   to student-login, then from student-login to your laptop:

   .. code-block:: console

      [coe332-vm]$ pwd
      /path/where/data/is
      [coe332-vm]$ ls
      output.png
      [coe332-vm]$ logout

      [student-login]$ pwd
      /home/username
      [student-login]$ scp coe332-vm:/path/where/data/is/output.png ./

   This will copy 'output.png' from coe332-vm to your home directory on
   student-login. Next, logout of student-login and copy the file to your 
   local laptop:

   .. code-block:: console

      [student-login]$ logout
      [local]$ scp username@student-login.tacc.utexas.edu:~/output.png ./
      Password:
      TACC_Token:


.. dropdown:: How do I find the closest time to "now" in a list of epochs?

   The process is fairly straightforward -

   * Find the time right "now" in UTC time zone
   * Iterate over each epoch in the ISS data
   * Compare the difference in time between "now" and each epoch
   * Return the epoch with the minimum difference

   The difficult part is working with time stamps. Python3 has a few libraries
   to make it easier to work with time data including ``time`` and ``datetime``.
   Either will work for the above, but here is an example using the ``time``
   library:


   You will need to import the standard library ``time``:

   .. code-block:: python3

      import time

   Find the time right "now" in UTC time zome, and put in a format that is easier
   to compare to other times:

   .. code-block:: python3

      >>> time.gmtime()
      time.struct_time(tm_year=2025, tm_mon=2, tm_mday=19, tm_hour=14, tm_min=55, tm_sec=33, tm_wday=2, tm_yday=50, tm_isdst=0)
      >>> time.mktime(time.gmtime())
      1739998535.0

   The first method ``gmtime()`` returns the current UTC time as a 'struct_time'
   object. Then using the ``mktime()`` method, that object can be converted into
   seconds since the Unix epoch (Jan 1, 1970, 00:00:00 UTC), which is represented
   as a float and very easy to compare to other time stamps in the same format.

   The next step is to iterate over all the state vectors in the ISS data, pull
   out the timestamps, convert them to Unix epoch time, and compare them to the
   time we got above. An example time stamp from the XML data looks like this:
   ``2025-043T12:16:00.000Z``. Consider the following code for parsing this time
   stamp:

   .. code-block:: python3

      >>> time.strptime('2025-043T12:16:00.000Z', '%Y-%jT%H:%M:%S.000Z')
      time.struct_time(tm_year=2025, tm_mon=2, tm_mday=12, tm_hour=12, tm_min=16, tm_sec=0, tm_wday=2, tm_yday=43, tm_isdst=-1)
      >>> time.mktime(time.strptime('2025-043T12:16:00.000Z', '%Y-%jT%H:%M:%S.000Z'))
      1739384160.0

   The first method, ``strptime()``, takes two arguments: a string (timestamp), and a
   format parameter, which uses a combination of directives and characters to
   extract the time data from the string and convert it into a 'struct_time' object.
   Then again, we can use the ``mktime()`` method to convert that object to
   a Unix epoch time stamp. 

   Iterating and finding the minimum difference can be solved using standard
   programming approaches.

   Find more information about the Python3 ``time`` library
   `here <https://docs.python.org/3/library/time.html>`_


.. dropdown:: How do I transform state vectors to latitude, longitude, and altitude?

   Check out the example code block below. Assumes you have a single state vector
   ``sv`` as input following a specific schema. You may need to modify a few of the
   lines below to make sure you are grabbing the right x, y, and z values from the
   state vector.

   If you use this approach, don't forget to add ``astropy`` to your list of
   requirements.

   .. code-block:: python3
   
      import time
      from astropy import coordinates
      from astropy import units
      from astropy.time import Time
      
      def compute_location_astropy(sv):
          x = float(sv['X']['#text'])
          y = float(sv['Y']['#text'])
          z = float(sv['Z']['#text'])
      
          # assumes epoch is in format '2024-067T08:28:00.000Z'
          this_epoch=time.strftime('%Y-%m-%d %H:%M:%S', time.strptime(sv['EPOCH'][:-5], '%Y-%jT%H:%M:%S'))
          
          cartrep = coordinates.CartesianRepresentation([x, y, z], unit=units.km)
          gcrs = coordinates.GCRS(cartrep, obstime=this_epoch) 
          itrs = gcrs.transform_to(coordinates.ITRS(obstime=this_epoch))
          loc = coordinates.EarthLocation(*itrs.cartesian.xyz)
          
          return loc.lat.value, loc.lon.value, loc.height.value


.. dropdown:: How do I find the nearest geolocation given latitude and longitude?

   Once you have latitude and longitude, you can try to plug it in to the ``GeoPy``
   python library (which needs to be pip installed). Here are a few key lines of
   code you could use:

   .. code-block:: python3

      from geopy.geocoders import Nominatim
      geocoder = Nominatim(user_agent='iss_tracker')
      geoloc = geocoder.reverse((lat, lon), zoom=15, language='en')

   Try playing around with different ``zoom`` levels. When the ISS is over the
   ocean, it will usually return ``None`` instead of a geolocation unless you
   zoom way out. You will need to also play around with the ``geoloc`` object
   to figure out how to access the information inside. Once you do, you can
   start to associate places with the latitude and longitude values:

   .. code-block:: json

      {
        "lat": -0.718275860472717,
        "actual_lat": "-1.8219",
        "lon": 106.88329187186568,
        "actual_lon": "105.4535",
        "alt": 422.2579992310011,
        "epoch_timestamp": "03/05/2024, 01:36:00",
        "now_timestamp": "03/04/2024, 19:35:59",
        "geoloc": "Bangka-Belitung Islands, Sumatra, Indonesia"
      }

   I recommend checking a site like this one to confirm whether this meets your
   expectations:

   https://www.n2yo.com/?s=90027

   Geopy supports a bunch of other different geocoders. Nominatim is not very good,
   but it is free and does not require setting up an API key. Some others are very
   powerful and can provide very detailed location information. However, many of
   the services require API keys, which you would need to sign up for / obtain
   from a third party. I chose Nominatim when I did this just because it was free
   and easy. You can chose others, though. More details here: 

   https://geopy.readthedocs.io/en/stable/#module-geopy.geocoders



.. dropdown:: ERROR: Error during data fetching: MISCONF Redis is configured to save RDB snapshots, but it's currently unable to persist to disk

   This is a very common error when working with Redis databases. The error 
   indicates that Redis is trying to save a snapshot of the database to disk, 
   usually to a folder called "/data", but it is unable to do so. The most 
   frequent cause of this problem is that the Docker daemon created that folder
   for you. If Docker makes that folder for you, it inherits improper permissions
   that make it difficult to work with. The solution is to remove the bad folder
   that Docker made, and make a new folder yourself with the correct permissions.

   Navigate to where your code is, listing the files to confirm that there is
   a folder called “data”, and remove the folder like this:

   .. code-block:: console

      sudo rm -rf data

   Then remake the folder yourself. This way it will have proper permissions:
   
   .. code-block:: console

      mkdir data

   Re-launch your containers and then Redis should be able to write to the database.
