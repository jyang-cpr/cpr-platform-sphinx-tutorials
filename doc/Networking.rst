Setting Up Networking
======================

First Connection
-----------------

In order to set up PLATFORM to connect to your own wireless network, you will first need to ``ssh`` access PLATFORM's computer from you computer over a wired connection:

1. Configure your computer to have a static IP address on the ``192.168.131.x`` subnet, e.g. ``192.168.131.100``.

2. Connect an ethernet cable between PLATFORM's computer and your computer.

3. ``ssh`` into PLATFORM's computer from your computer. In your computer's terminal, run:

.. code-block:: bash

    ssh administrator@192.168.131.1

4. Enter the default password ``clearpath``. You should now be logged into PLATFORM as the ``administrator`` user.

Changing the Default Password
------------------------------

.. Note::

  All Clearpath robots ship from the factory with their login password set to ``clearpath``. Upon receipt of your robot we recommend changing the password.

1. To change the password to log into your PLATFORM, you can use the ``passwd`` command. In PLATFORM's computer's terminal, run:

.. code-block:: bash

  passwd

2. You will be prompted to enter the current password, followed by the new password twice. While typing the passwords in the ``passwd`` prompt there will be no visual feedback (e.g. "*" characters).

To further restrict access to your PLATFORM, you can reconfigure the PLATFORM's ``ssh`` service to disallow logging in with a password and require ``ssh`` certificates to log in.  This_ tutorial covers how to configure ``ssh`` to disable password-based login.

.. _This: https://linuxize.com/post/how-to-setup-passwordless-ssh-login/

Connecting to Wifi Access Point
--------------------------------

PLATFORM uses ``netplan`` for configuring its wired and wireless interfaces. You can configure ``netplan`` so that PLATFORM's computer can connect to your own wireless network:

1. In PLATFORM's computer's terminal, create the file ``/etc/netplan/60-wireless.yaml``.

2. Populate the file ``/etc/netplan/60-wireless.yaml`` with the following:

.. code-block:: yaml

    network:
      wifis:
        # Replace WIRELESS_INTERFACE with the name of the wireless network device, e.g. wlan0 or wlp3s0
        # Fill in the SSID and PASSWORD fields as appropriate.  The password may be included as plain-text
        # or as a password hash.  To generate the hashed password, run
        #   echo -n 'WIFI_PASSWORD' | iconv -t UTF-16LE | openssl md4 -binary | xxd -p
        # If you have multiple wireless cards you may include a block for each device.
        # For more options, see https://netplan.io/reference/
        WIRELESS_INTERFACE:
          optional: true
          access-points:
            SSID_GOES_HERE:
              password: PASSWORD_GOES_HERE
          dhcp4: true
          dhcp4-overrides:
            send-hostname: true

3. Modify the variables in the file ``/etc/netplan/60-wireless.yaml`` with the details of your wireless network.

4. Save the file ``/etc/netplan/60-wireless.yaml``. 

5. Apply your new ``netplan`` configuration and bring up your wireless connection. In PLATFORM's computer's terminal, run:

.. code-block:: bash

    sudo netplan apply

6. Verify that PLATFORM successfully connected to your wireless network. In PLATFORM's computer's terminal:

.. code-block:: bash

    ip a

This will show all active connections and their IP addresses, including the connection to your wireless network, and the IP address assigned to PLATFORM's computer.

Remote ROS Connection
---------------------

It is useful to connect your computer to the PLATFORM's ROS master, particularly if you want to use ROS desktop tools to interface with the PLATFORM:

1. Ensure both your computer and PLATFORM's computer are connected to the same wireless network. This process will also work for a wired connection, but for the purposes of establishing a remote ROS connection, it makes sense to use a wireless connection.

2. On your computer, set the ``ROS_MASTER_URI`` and ``ROS_IP`` environment variables. The ``ROS_MASTER_URI`` environment variable tells your computer how to find the ROS master on the PLATFORM's computer. The ``ROS_IP`` environment variable tells processes on the PLATFORM's computer how to find your computer. In your computer's terminal, create a script in your computer's home directory called ``remote-PLATFORM.sh`` with the following contents:

.. code-block:: bash

    export ROS_MASTER_URI=http://<PLATFORM_HOSTNAME>:11311  # PLATFORM's computer's hostname
    export ROS_IP=<COMPUTER_IP>                             # Your computer's wireless IP address

3. If your network doesn't already resolve PLATFORM's computer's hostname to its wireless IP address, you may need to add a corresponding line to your computer's ``/etc/hosts`` file:

.. code-block:: bash

    <PLATFORM_IP> <PLATFORM_HOSTNAME>

4. When ready to communicate remotely with PLATFORM's computer from your computer, you can source the ``remote-PLATFORM.sh`` script; thus, defining those two key environment variables in the present context. In your computer's terminal, run:

.. code-block:: bash

    source remote-PLATFORM.sh

5. You should be able to now be able to access PLATFORMS's ROS data from your computer, such as the list of ROS nodes, the list of ROS topics, the ROS messages being published on ROS topics, and the frequencies/rates at which the ROS messages are being published at. In terminal on your computer, run:

.. code-block:: bash

    rosnode list
    rostopic list
    rostopic hz <ROS_TOPIC>
    rostopic echo <ROS_TOPIC>

6. Once you've verified the basics from the prompt, try launching some of the standard visual ROS tools. In terminal on your computer, run:

.. code-block:: bash

    roslaunch PLATFORM_viz view_robot.launch
    rosrun rqt_robot_monitor rqt_robot_monitor
    rosrun rqt_console rqt_console

If there are particular :roswiki:`rqt` widgets you find yourself using a lot, you may find it an advantage to dock them together and then export this configuration as the default RQT perspective. Then, to bring up your standard GUI, in terminal on your computer, run:

.. code-block:: bash

    rqt

Configuring Network Bridge
---------------------------

PLATFORM is configured to bridge its physical ethernet ports together. This allows any ethernet port to be used as a connection to the internal ``192.168.131.1/24`` network for connecting sensors, diagnostic equipment, or manipulators, or for connecting the PLATFORM to the internet for the purposes of installing updates.

In the unlikely event you must modify PLATFORM's ethernet bridge, you can do so by editing the configuration file found at ``/etc/netplan/50-clearpath-bridge.yaml``:

.. code-block:: yaml

    # Configure the wired ports to form a single bridge
    # We assume wired ports are en* or eth*
    # This host will have address 192.168.131.1
    network:
    version: 2
    renderer: networkd
    ethernets:
    bridge_eth:
      dhcp4: no
      dhcp6: no
      match:
        name: eth*
    bridge_en:
      dhcp4: no
      dhcp6: no
      match:
        name: en*
    bridges:
    br0:
      dhcp4: yes
      dhcp6: no
      interfaces: [bridge_en, bridge_eth]
      addresses:
        - 192.168.131.1/24

This file will create a bridged interface called ``br0`` that will have a static address of 192.168.131.1, but will also be able to accept a DHCP lease when connected to a wired router. By default, all network ports named ``en*`` and ``eth*`` are added to the bridge. This includes all common wired port names, such as: ``eth0``, ``eno1``, ``enx0123456789ab``, ``enp3s0``, etc.

To include/exclude additional ports from the bridge, edit the ``match`` fields, or add additional ``bridge_*`` sections with their own ``match`` fields, and add those interfaces to the ``interfaces: [bridge_en, bridge_eth]`` line near the bottom of the file.

We do not recommend changing the static address of the bridge to be anything other than ``192.168.131.1``; changing this may cause sensors that communicate over ethernet (e.g. lidars, cameras, GPS arrays) from working properly.
