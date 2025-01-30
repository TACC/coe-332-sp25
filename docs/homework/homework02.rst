Homework 02
===========

**Due Date: Thursday, Feb 7, by 11:00am central time**

A Fall of Meteordust
--------------------

Our in-class examples so far have used a modified and abridged Meteorite Landings
data set. For this homework, we will work with the real Meteorite Landings data
set from NASA while exercising our Python3 best practices.


PART 1
~~~~~~

Write two Python3 scripts for reading and summarizing the Meteorite Landings data
from NASA. The requirements are as follows:

* Scripts are organized following guidelines in Unit 03
* Primary script reads in CSV-formatted 
  `Meteorite Landings data <https://data.nasa.gov/Space-Science/Meteorite-Landings/gh4g-9sfh/about_data>`_
* Primary script contains at least three functions for parsing the data and
  printing or plotting some summary statistics to screen or file. These cannot
  be the same functions shown in the class readthedocs (i.e. not ``compute_average_mass``,
  ``check_hemisphere``, or ``count_classes``)
* At least one function must make use of the great-circle distance algorithm to
  calculate the distance between landing sites
* The great-circle distance algorithm must be provided as a standalone script / 
  function that is imported into your primary script
* All functions must contain appropriate doc strings and type hints
* All functions must contain corresponding unit tests in an appropriately-named 
  files. Unit tests must be compatible with ``pytest``
* Scripts must support logging, and include a mix of log statements (focus on 
  ``DEBUG``, ``WARNING``, and ``ERROR`` where appropriate)
* Scripts must use appropriate error handling if, e.g., a null value is present
  in the input data


PART 2
~~~~~~

Your homework 02 files must be within a new subdirectory called ``homework02`` in
your COE 332 homeworks repository on GitHub. The directory should contain your primary
Python3 script, the secondary script containing the great-circle distance function,
two unit test scripts, and a ``README.md`` file. The README should
be descriptive, use proper grammar, and contain enough instructions so anyone else
could clone the repository and figure out what the scripts do and how to run them.
General guidelines to follow for the README for homework 02 are:

* Descriptive title
* High-level description of the folder contents / project objective. I.e. why
  does this exist and why is it important? (2-3 sentences)
* Specific description of the individual Python3 scripts (1-2 sentences each)
* Instructions to obtain any necessary data and where to copy it to
* Instructions to run the code from start to finish, plus how to interpret the
  results (2-3 sentences)
* Use markdown styles to your advantage, give the sections headers, use code
  blocks where appropriate, etc.

Remember, the README is your chance to document for yourself and explain to others
why the project is important, what the code is, and how to use it / interpret
the outputs / etc. This is a *software engineering and design* class, so we are
not just checking to see if your code works. We are also evaluating the design of
the overall submission, including how well the project is described in the README.





What to Turn In
---------------

A sample Git repository may contain the following new files after completing
homework 02:

.. code-block:: text
   :emphasize-lines: 6-11

   my-coe332-hws/
   ├── homework01
   │   ├── README.md
   │   ├── ml_json_converter.py
   │   └── ml_json_reader.py
   ├── homework02
   │   ├── README.md           
   │   ├── gcd_algorithm.py        # your file names may vary
   │   ├── ml_data_analysis.py
   │   ├── test_gcd_algorithm.py
   │   └── test_ml_data_analysis.py
   └── README.md

There is no need to email the link to your homework repo again, as we should have
it on file from the first homework. We will re-clone the same repo as before at the
due date / time for evaluation.




Additional Resources
--------------------

* `Meteorite Landings Data <https://data.nasa.gov/Space-Science/Meteorite-Landings/gh4g-9sfh/about_data>`_
* `JSON guide <https://coe-332-sp25.readthedocs.io/en/latest/unit02/json.html>`_
* `Latitude and Longitude as decimals <https://en.wikipedia.org/wiki/Decimal_degrees>`_
* `Great-circle distance formula <https://en.wikipedia.org/wiki/Great-circle_distance>`_
* `Markdown syntax <https://www.markdownguide.org/basic-syntax/>`_
* `Tips on writing a good README <https://www.makeareadme.com/>`_
* Please find us in the class Slack channel if you have any questions!
