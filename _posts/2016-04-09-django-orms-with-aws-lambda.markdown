---
layout: post
title:  "Asynchronous processing with Django ORMs on AWS Lambda"
date:   2016-04-09
categories: django python aws lambda
---

I have been running a Django Rest Framework service on AWS for some time, and ended up in the point when there
was a need for asynchronous processing outside the simple request-response processing flow.

Common approach to have capability to do asynchronous processing is to have a message queue or similar
communication channel from the request processing entity to some worker entity that carries out the
asynchronous task.

On AWS platform there are multiple choices to work as the channel: AWS SNS (Simple Notification Service), AWS
SQS (Simple Queue Service), and AWS SWF (Simple Workflow Service). With each of these services the developer
can implement code that runs on the worker node upon as a consequence of some action initiated by the request
processing entity. All these services are viable alternatives for the communication channel and we are left
with the question about where to run the worker code. AWS EC2 (Elastic Compute Cloud) virtual machine instances or ECS
(Elastic Container Service) are viable options. Of course one can also run own messaging solution on either of
these services omitting the messaging services offered by the AWS.

I've been running [Celery](http://www.celeryproject.org/) in few project to fulfill the high level task
distribution needs while inside it I've been using both RabbitMQ, Redis, and AWS SQS in different projects.
These all work pretty nicely (even the Celery with AWS SQS with its experimental status). But
having your own worker infrastructure has some issues that the system architect must keep in mind.
Firstly, having your own worker infra requires you to setup and manage the worker infrastructure. Quite
trivial? Not necessarily. Concepts to about are scaling issues, availability, task monitoring, scheduling, and
duplication just to mention the most trivial ones. In addition some overhead is usually added due to 
workers running on idle load when there are no requests to serve.

As an alternative worker platform one can utilize [AWS Lambda](https://aws.amazon.com/lambda/) (btw.
[released](https://aws.amazon.com/releasenotes/AWS-Lambda/3857079333029488) generally available exactly one
year ago). With AWS Lambda the worker code is bundled and uploaded to AWS Lambda, which executes the code 
triggered by on some external event. This event can be e.g. new file uploaded to AWS S3 (Simple Storage
Service) or a notification from AWS SNS. Billing is based on the execution time and the allocated memory.
Memory allocation and some maximum execution time limits can be configured for each Lambda task allowing you 
to limit the resource consumption of your tasks. Currently the AWS Lambda has a free tier service level for
everybody including 1 million free requests per month and 400 000 gigabyte seconds of compute time per month,
so you're free to go and play & prototype with it.

So recently I ended up converting the Django stack from running my own Celery task queue to AWS Lambda.
Based on the user request the application must do heavy processing with the data from Django models. Also there are
periodically executed tasks which used to be initiated by the periodic task scheduler included in Celery.

The main reason for this conversion is that I don't want to run any long running instances (virtual machines should not be
treated as pets after all) and I need to minimize the consumed resources and the required maintenance. AWS Lambda
tackles all these: the code is ready to execute in a few dozen milliseconds after the request hits AWS Lambda
service and resources are consumed only for the duration of the task. No servers included.

One could also go even further with a very interesting project [Django Zappa](https://github.com/Miserlou/django-zappa)
which allows the whole Django installation to run on AWS Lambda. But before that, the encountered limitations
follows soon.

AWS Lambda can currently execute your Python 2.7, Node.js v4.3.2 or Java 8 applications. The execution
environment is of course managed by AWS and the details are available [here](http://docs.aws.amazon.com/lambda/latest/dg/current-supported-versions.html).
Any extra libraries not included in the standard library of your platform or pre-installed in the listed AWS
EC2 instances must be included in the code bundle uploaded to AWS Lambda. In case of my Django application, I
really want to share the my models with the code running on AWS Lambda, so I need mostly the same libraries
for the models which are also used on the application server serving the user requests. In case of Python this
the usage of Virtualenvs is encouraged by AWS and instructions for doing so are [here](http://docs.aws.amazon.com/lambda/latest/dg/lambda-python-how-to-create-deployment-package.html).

Next issue was some C extensions, namely [Pillow](https://github.com/python-pillow/Pillow/) and the
[Psycopg2](http://initd.org/psycopg/) used by my Django application to connect to database running on AWS RDS
(Relational Database Service). These extensions must be binary compatible with the AWS execution environment. The
only easy way of doing this is to launch a new EC2 instance from Amazon Machine Image used by AWS Lambda which
are listed in the execution environment [description](http://docs.aws.amazon.com/lambda/latest/dg/current-supported-versions.html).

The Psycopg2 dependency is a bit more tricky as it is requiring libpq.so shared object file not available on
the EC2 Lambda optimized images. Instructions how to build Lambda capable Psycopg2 library are provided e.g.
in [https://github.com/jkehler/awslambda-psycopg2](https://github.com/jkehler/awslambda-psycopg2).

In order to connect to your RDS database the AWS Lambda task requires a role with AWS predefined policy "AWSLambdaVPCAccessExecutionRole". 
This way the Lambda task can connect to RDS instance running on your AWS VPC (Virtual Private Cloud).

Once dependencies are installed and you have built the bundle as instructed by Amazon the next thing is to
create a new Lambda function, upload the bundle, and create event source that triggers the Lambda function.

Your Lambda function log outputs are available in CloudWatch as well as the execution times and the real
amount of memory consumed by your Lambda function.
