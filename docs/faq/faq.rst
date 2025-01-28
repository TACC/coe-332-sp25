Frequently Encountered Problems
===============================

Find solutions to common problems below.

.. dropdown:: Cannot push to GitHub

   The most common issue is an improperly configured upstream on your local
   repo. If you try to ``git push`` and you are prompted for your username and
   password, then are presented with an error like:

   ``fatal: Authentication failed for 'https://github.com/username/reponame.git/'``
   
   This is because you are using the wrong upstream URL. To solve:

   1. Make sure you upload public SSH key to GitHub settings
   2. If using a non-standard public key name, modify your ``~/.ssh/config``
      accordingly
   3. Fix the upstream URL by executing the following in the git repo directory:

   ``git remote set-url origin git@github.com/username/reponame.git'``

   Replacing 'username' with your GitHub username and 'reponame' with the name
   of the repo as it appears on GitHub. 


