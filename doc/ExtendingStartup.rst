Extending PLATFORM Startup
==============================

Now that's you've had PLATFORM for a while, you may be interested in how to extend it— perhaps add some more payloads, or augment the URDF.

Startup Launch Context
----------------------

When ROS packages are grouped together in a directory and then built as one, the result is referred to as a workspace. Each workspace generates a ``setup.bash`` file which the user may source in order to correctly set up important environment variables such as ``PATH``, ``PYTHONPATH``, and ``CMAKE_PREFIX_PATH``.

The standard system-wide setup file is in ``opt``:

.. code-block:: bash

    source /opt/ros/noetic/setup.bash

When you run this command, you'll have access to `rosrun`, `roslaunch`, and all the other tools and packages installed on your system from debian packages.

However, sometimes you want to add additional system-specific environment variables, or perhaps packages built from source. For this reason, Clearpath platforms use a wrapper setup file, located in ``/etc/ros``:

.. code-block:: bash

    source /etc/ros/setup.bash

This is the setup file which gets sourced by PLATFORM's background launch job, and in the default configuration, it is also sourced on your login session. For this reason it can be considered the "global" setup file for PLATFORM's ROS installation.

This file sets some environment variables and then sources a chosen ROS workspace, so it is one of your primary modification points for altering how PLATFORM launches.

Launch Files
------------

The second major modification point is the ``/etc/ros/noetic/ros.d`` directory. This location contains the launch files associated with the ``ros`` background job. If you add launch files here, they will be launched with PLATFORM's startup.

However, it's important to note that in the default configuration, any launch files you add may only reference ROS software installed in ``/opt/ros/noetic``. If you want to launch something from workspace in the home directory, you must change ``/etc/ros/setup.bash`` to source that workspace's setup file rather than the one from ``opt``.

Adding URDF
-----------

There are two possible approaches to augmenting PLATFORM's URDF. The first is that you may simply set the ``PLATFORM_URDF_EXTRAS`` environment variable in ``/etc/ros/setup.bash``. By default, it points to an empty dummy file, but you can point it to a file of additional links and joints which you would like mixed into PLATFORM's URDF (via xacro) at runtime.

The second, more sophisticated way to modify the URDF is to create a *new* package for your own robot, and build your own URDF which wraps the one provided by :roswiki:`PLATFORM_description`.