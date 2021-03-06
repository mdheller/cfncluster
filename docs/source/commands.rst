.. _commands:

.. toctree::
   :maxdepth: 2

###################
CfnCluster Commands
###################

Most commands provided are just wrappers around CloudFormation functions.

.. note:: When a command is called and it starts polling for status of that call it is safe to :code:`Ctrl-C` out. you can always return to that status by calling :code:`cfncluster status mycluster`

create
======

Creates a CloudFormation stack with the name :code:`cfncluster-[stack_name]`. To read more about CloudFormation see `AWS CloudFormation <https://cfncluster.readthedocs.io/en/latest/aws_services.html#aws-cloudformation>`_.

positional arguments:
  cluster_name          create a cfncluster with the provided name.

optional arguments:
  -h, --help            show this help message and exit
  --config CONFIG_FILE, -c CONFIG_FILE
                        specify a alternative config file
  --region REGION, -r REGION
                        specify a specific region to connect to
  --nowait, -nw         do not wait for stack events, after executing stack command
  --norollback, -nr     disable stack rollback on error
  --template-url TEMPLATE_URL, -u TEMPLATE_URL
                        specify a URL for a custom cloudformation template
  --cluster-template CLUSTER_TEMPLATE, -t CLUSTER_TEMPLATE
                        specify a specific cluster template to use
  --extra-parameters EXTRA_PARAMETERS, -p EXTRA_PARAMETERS
                        add extra parameters to stack create
  --tags TAGS, -g TAGS  tags to be added to the stack, TAGS is a JSON formatted string encapsulated by single quotes

::

	$ cfncluster create mycluster

create cluster with tags:

::

        $ cfncluster create mycluster --tags '{ "Key1" : "Value1" , "Key2" : "Value2" }'

update
======

Updates the CloudFormation stack using the values in the :code:`config` file or a :code:`TEMPLATE_URL` provided. For more information see `AWS CloudFormation Stacks Updates <https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/using-cfn-updating-stacks.html>`_.

positional arguments:
  cluster_name          update a cfncluster with the provided name.

optional arguments:
  -h, --help            show this help message and exit
  --config CONFIG_FILE, -c CONFIG_FILE
                        specify a alternative config file
  --region REGION, -r REGION
                        specify a specific region to connect to
  --nowait, -nw         do not wait for stack events, after executing stack command
  --norollback, -nr     disable stack rollback on error
  --template-url TEMPLATE_URL, -u TEMPLATE_URL
                        specify a URL for a custom cloudformation template
  --cluster-template CLUSTER_TEMPLATE, -t CLUSTER_TEMPLATE
                        specify a specific cluster template to use
  --extra-parameters EXTRA_PARAMETERS, -p EXTRA_PARAMETERS
                        add extra parameters to stack update
  --reset-desired, -rd  reset the current ASG desired capacity to initial
                        config values

::

    $ cfncluster update mycluster

stop
====

Sets the Auto Scaling Group parameters to :code:`min/max/desired = 0/0/0`

.. note:: A stopped cluster will only terminate the compute-fleet.

Previous versions of CfnCluster stopped the master node after terminating
the compute fleet. Due to a number of challenges with the implementation
of that feature, the current version only terminates the compute fleet.
The master will remain running. To terminate all EC2 resources and avoid EC2 charges,
consider deleting the cluster.

positional arguments:
  cluster_name  stops the compute-fleet of the provided cluster name.

optional arguments:
  -h, --help    show this help message and exit
  --config CONFIG_FILE, -c CONFIG_FILE
                        specify a alternative config file
  --region REGION, -r REGION
                        specify a specific region to connect to

::

    $ cfncluster stop mycluster


start
=====

Starts a cluster. This sets the Auto Scaling Group parameters to either the
initial configuration values (`max_queue_size
<https://cfncluster.readthedocs.io/en/latest/configuration.html#max-queue-size>`_
and `initial_queue_size
<https://cfncluster.readthedocs.io/en/latest/configuration.html#initial-queue-size>`_)
from the template that was used to create the cluster or to the configuration
values that were used to update the cluster since creation.

positional arguments:
  cluster_name          starts the compute-fleet of the provided cluster name.

optional arguments:
  -h, --help            show this help message and exit
  --config CONFIG_FILE, -c CONFIG_FILE
                        specify a alternative config file
  --region REGION, -r REGION
                        specify a specific region to connect to

::

    $ cfncluster start mycluster

delete
======

Delete a cluster. This causes a CloudFormation delete call which deletes all the resources associated with that stack.

positional arguments:
  cluster_name  delete a cfncluster with the provided name.

optional arguments:
  -h, --help    show this help message and exit
  --config CONFIG_FILE, -c CONFIG_FILE
                        specify a alternative config file
  --region REGION, -r REGION
                        specify a specific region to connect to
  --nowait, -nw         do not wait for stack events, after executing stack command

::

    $ cfncluster delete mycluster

ssh
====

Runs ssh to the master node, with username and ip filled in based on the provided cluster.

For example:
    cfncluster ssh mycluster -i ~/.ssh/id_rsa

Results in an ssh command with username and ip address pre-filled.

    ssh ec2-user@1.1.1.1 -i ~/.ssh/id_rsa

SSH command is defined in the global config file, under the aliases section and can be customized:

    [aliases]
    ssh = ssh {CFN_USER}@{MASTER_IP} {ARGS}

Variables substituted:
    {CFN_USER}
    {MASTER_IP}
    {ARGS} (only if specified on the cli)

positional arguments:
  cluster_name  name of the cluster to set variables for.

optional arguments:
  -h, --help    show this help message and exit
  --dryrun, -d  print command and exit.

::

    $ cfncluster ssh mycluster -i ~/.ssh/id_rsa -v

stepfunctions
=============

Creates a step function that automatically creates a cluster, runs user specified jobs, and tears the cluster down.

For example:
    cfncluster stepfunctions -b s3bucket -j path/to/jobs.config

This uses the bucket name s3bucket, or creates it if it doesn't exist, to store the lambda source code and user specified jobs. To see how jobs in the ``jobs.config`` file should be specified, see `Job Config <stepfunctions.html#jobs-config>`_.

arguments:
  -h, --help            show this help message and exit
  --jobs JOBS_CONFIG, -j JOBS_CONFIG
                        specify jobs config file to use (REQUIRED)
  --bucket BUCKET_NAME, -b BUCKET_NAME
                        specify s3 bucket to use/create (REQUIRED)
  --config CONFIG_FILE, -c CONFIG_FILE
                        specify an alternative config file (default: ~/.cfncluster/config)
  --region REGION, -r REGION
                        specify a region to deploy in (default: us-east-1)
  --stack-name STACK_NAME, -s STACK_NAME
                        specify the stack name to use (default: CfnClusterStepFunction)
  --key-name KEY_NAME, -k KEY_NAME
                        specify the ec2 key pair (default: cfncluster-stepfunctions)

::

    $ cfncluster stepfunctions -b s3bucket -j path/to/jobs.config

status
======

Pull the current status of the cluster. Polls if the status is not CREATE_COMPLETE or UPDATE_COMPLETE.
For more info on possible statuses see the `Stack Status Codes <https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/using-cfn-describing-stacks.html#d0e9320>`_ page.

positional arguments:
  cluster_name  show the status of cfncluster with the provided name.

optional arguments:
  -h, --help    show this help message and exit
  --config CONFIG_FILE, -c CONFIG_FILE
                        specify a alternative config file
  --region REGION, -r REGION
                        specify a specific region to connect to
  --nowait, -nw         do not wait for stack events, after executing stack command

::

    $ cfncluster status mycluster

list
====

Lists clusters currently running or stopped. Lists the :code:`stack_name` of the CloudFormation stacks with the name :code:`cfncluster-[stack_name]`.

optional arguments:
  -h, --help  show this help message and exit
  --config CONFIG_FILE, -c CONFIG_FILE
                        specify a alternative config file
  --region REGION, -r REGION
                        specify a specific region to connect to

::

    $ cfncluster list

instances
=========

Shows EC2 instances currently running on the given cluster.

positional arguments:
  cluster_name  show the status of cfncluster with the provided name.

optional arguments:
  -h, --help    show this help message and exit
  --config CONFIG_FILE, -c CONFIG_FILE
                        specify a alternative config file
  --region REGION, -r REGION
                        specify a specific region to connect to

::

    $ cfncluster instances mycluster

configure
=========

Configures the cluster. See `Configuring CfnCluster <https://cfncluster.readthedocs.io/en/latest/getting_started.html#configuring-cfncluster>`_.

optional arguments:
  -h, --help  show this help message and exit
  --config CONFIG_FILE, -c CONFIG_FILE
                        specify a alternative config file

::

    $ cfncluster configure mycluster

version
=======

Displays CfnCluster version.

optional arguments:
  -h, --help  show this help message and exit
  --region REGION, -r REGION
                        specify a specific region to connect

::

    $ cfncluster version
