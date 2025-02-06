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




