---
layout: post
title:  "Importing Spark Dataframes from MySQL on Jupyter notebooks"
date:   2015-10-30
categories: spark python jupyter integration mysql
---

Lots of discussion and demand around [Jupyter][jupyter] notebooks these days, and no
wonder. Since the big [the big split][bigsplit] the project started to focus
more and more how to provide generic interactive notebook-like interface on top
of different interactive kernels. Now there's already over [50 kernels available][kernellist],
with support for the most used programming languages, not just only Python anymore.

Jupyter offers nice interactive sessions, as execution history can be diveded
into cells, that can be later re-evaluated. With libraries like
[matplotlib][matplotlib] or [Bokeh][bokeh] return values can be visualized
inline in the notebook. This makes Jupyter noteooks also nice candidate for
explorative data analysis. This post is about how to take a step further and
integrate Jupyter notebooks with existing data analysis stack, namely
[Spark][spark] and its dataframe environment that can also integrate with your
existing databases.

Installation of plain Jupyter is straightforward. The easiest way is just to use
your package manager and install Jupyter system-wide. For the sake of
environment isolation I'll use [virtualenv] & [virtualenvwrapper] here. So the first step is to create
the virtual environment for this Jupyter installation with virtualenvwrapper:

{% highlight bash %}
mkvirtualenv jupyter
{% endhighlight %}

As mentioned in the [Jupyter installation documentation][jupyter-install],
Jupyter installation with pip requires some dependencies to be compiled at least
on Ubuntu. Install these dependencies first, and then install Jupyter itself
with pip.

{% highlight bash %}
apt-get install build-essential python-dev
pip install jupyter
{% endhighlight %}

Notebook server can be password protected, note that such server should also has proper SSL setup, especially so if used in an open environment.
To generate password hash with Python shell.

{% highlight python %}
from notebook.auth import passwd
passwd()
{% endhighlight %}


Next create startup script to launch Jupyter notebook server. Replace IP with
the ip from the server interface if external access is required, and insert the
hashed password computed above.

{% highlight bash %}
#!/usr/bin/env bash
source $WORKON_HOME/ipy/bin/activate
jupyter-notebook --no-browser --ip 127.0.0.1 --NotebookApp.password='you-hashed-password'
{% endhighlight %}

Next target is to create SparkContext and submit jobs to Spark cluster. For this
to succeed the startup script script above must be modified to include
SPARK_HOME environment variable pointing to Spark installatino directory before the notebook is started.

{% highlight bash %}
#!/usr/bin/env bash
source $WORKON_HOME/ipy/bin/activate
export SPARK_HOME=/opt/spark
jupyter-notebook --no-browser --ip 127.0.0.1 --NotebookApp.password='you-hashed-password'
{% endhighlight %}

Launch new notebook on Jupyter notebook server and create Spark contexts (change
correct master argument pointing to master node of current Spark installation)

{% highlight python %}
import os
os.environ['PYSPARK_PYTHON'] = 'python2'
from pyspark import SparkContext
from pyspark.sql import SQLContext
sc = SparkContext(master='spark://spark-frontend-host:7077')
sqlContext = SQLContext(sc)
{% endhighlight %}

That's pretty straightforward. To connect to external database to retrieve data
into Spark dataframes additional jar file is required. E.g. with MySQL the [JDBC
driver][mysqlc] is required. Download the driver package and extract
mysql-connector-java-x.yy.zz-bin.jar in a path that's accessible from every node
in the cluster. Preferably this is a path on shared file system. E.g. with
[Pouta Virtual Cluster][pvc] such path would be under /shared_data, here I use
`/shared_data/thirdparty_jars/`.

With direct Spark job submissions from terminal one can specify --driver-class-path argument 
pointing to extra jars that should be provided to workers with the job. However
this does not work with this approach, so we must configure these paths for
front end and worker nodes in the spark-defaults.conf file, usually in 
`/opt/spark/conf` directory.

{% highlight bash %}
spark.driver.extraClassPath /shared_data/thirdparty_jars/mysql-connector-java-5.1.35-bin.jar
spark.executor.extraClassPath /shared_data/thirdparty_jars/mysql-connector-java-5.1.35-bin.jar
{% endhighlight %}

Now workers in the cluster should be able to load the MySQL connector and
connect to external database. To load that data with Python load method of the SQLContext object is used. The dbtable argument
can take either a table name or any subquery that is valid in a FROM clause.
This can be handy if only a partial data, matching some
condition, is to be loaded.

{% highlight python %}
df = sqlContext.read.format('jdbc').options(url='jdbc:mysql://mysqlserver:3306', dbtable='(select * from someTable limit 100) as myTable').load()
df.head()
{% endhighlight %}

Once the Spark context is created the cluster resources are allocated for that
context until it is stopped. To do this from the notebook use

{% highlight python %}
sc.stop()
{% endhighlight %}



[jupyter]: http://jupyter.org/
[bigsplit]: https://blog.jupyter.org/2015/04/15/the-big-split/
[kernellist]: https://github.com/ipython/ipython/wiki/IPython-kernels-for-other-languages
[spark]: https://spark.apache.org/
[virtualenv]: http://virtualenv.readthedocs.org/en/latest/
[virtualenvwrapper]: https://virtualenvwrapper.readthedocs.org/en/latest/
[jupyter-install]: https://jupyter.readthedocs.org/en/latest/install.html
[mysqlc]: http://dev.mysql.com/downloads/connector/j/
[pvc]: https://github.com/CSC-IT-Center-for-Science/pouta-virtualcluster
[matplotlib]: http://matplotlib.org/
[bokeh]: http://bokeh.pydata.org/en/latest/

