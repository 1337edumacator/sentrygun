# sentrygun

#Key Features

 - Capable of detecting karma attacks based on probe request/response patterns
 - Capable of detecting evil twin attacks through the use of whitelist crossreferencing
 - Capable of detecting evil twin attacks based on fluctuations in signal strength
 - Can automatically respond to evil twin attacks by actively disabling attacker's rogue access points
 - Sends email alerts when attacks are detected

# Project Overview

#Required Hardware - Sentrygun

dual band wireless adapter

#Required Hardware - Sentrygun-Server

#Full Setup

##Step 1 - Setup Infrastructure

##Step 2 - Establish Connections

Sensor devices equipped with SentryGun communicate with the central SentryGun-Server over HTTP and WebSockets. Commands issued by the web frontend are sent via HTTP POST request to the SentryGun-Server instance, whicb broadcasts the command to all sensor devices. 

##Step 3 - Calibrate SentryGun sensors

SentryGun sensor devices must be calibrated against your wireless network if evil twin detection is to be enabled. To calibrate the clients, first populate the whitelist.txt file on each of your sensor devices with the bssid and essid of each access point on your network. The access points should be listed in whitelist.txt using the following format.

	# essid bssid
	gdslabs ff:ff:ff:aa:aa:aa
	gdslabs 00:11:22:33:44:00
	gdslabs 00:11:22:33:44:55

	# and so on and so forth

Next, run sentrygun's calibration routine using the provided sg-calibrator.py script. The syntax for using sg-calibrator.py is as follows:

	python sg-calibrator.py -i IFACE

Substitute IFACE with the name of the device's external wireless interface. The specified wireless interface should be in monitor mode.

The sg-calibrator.py script will collect packets from your wireless access points over a period of time. It will then calculate the mean tx value across all packets collected. Finally, sg-calibrator.py will set a low and high bound equal to plus or minus N times the maximum deviation from the mean tx, where N is the value specified in configs.py.

It is imperative that sg-calibrator.py is run in a physically secure environment to prevent statistical poisoning attacks. Sentrygun's evil twin detection relies on detecting tx values that fall outside of an expected threshold. An attacker could render this functionality useless by broadcasting their own tx values during the calibration phase.

Once all devices have been calibrated, we can proceed to step 4 to initialize the system.

##Step 4 - Run System

To run the SentryGun system, first start the SentryGun-Server instance by issuing the following command on the CnC machine:

	python run.py <options go here> 

SentryGun-Server accepts the following command line arguments.

 - --port   - specifies the port on which sentrygun-server should listen (defaults to 80)
 - --host   - specifies the address at which sentrygun-server should listen (defaults to 0.0.0.0 if --tunnels flag is not used. Defaults to 127.0.0.1 if --tunnels flag is used).
 - --debug  - Run in debug mode (not recommended for production environments)
 - --expire - Sets the number of seconds that alerts should remain active before they are automatically dismissed. To disable alert expiration, set this to 0 (default).
 - --tunnels - Creates ssh tunnels from localhost:PORT on sentrygun-server to localhost:PORT on a list of sentrygun clients, where PORT is the port at which sentrygun-server listens on. When this flag is used, sentrygun-server will always listen on localhost regardless of whether the --host is used. Use this option when running sentrygun on a hostle network (you should assume that you are). SentryGun clients should be specified with the format user@host:port.

Once the server is running, start up each of the sentrygun sensor instances using the following syntax:

	python sentrygun.py <options go here>

sentrygun.py accepts the following command line arguments:

 - -a SERVER_ADDRESS - connect to sentrygun-server CnC at address SERVER_ADDRESS
 - -p SERVER_PORT - connect to sentrygun-server CnC on port SERVER_PORT
 - --evil-twin - enable evil-twin detection
 - --karma - enable karma attack detection
 - -i IFACE - substitute IFACE with name of external wireless interface (should be in monitor mode)

For example, to enable evil-twin and karma detection with server located at example.com:4444 use the following command:

	python sentrygun.py -i wlan1 -p 4444 -a example.com --evil-twin --karma

In the above example, the wireless interface is named wlan1.
 
#Usage Instructions

Once the system is up and running, navigate to the address and port at which the sentrygun-server instance is running. For example, if we started sentrygun-server at 192.168.1.39:4444 in step 4 of the setup instructions, we would navigate to the following address in our browser:

	http://192.168.1.39:4444

You should be presented with a blank screen labeled "SentryGun Dashboard". There will also be an expandable toolbar on the left with links to features that have not been implemented yet.

	<screenshot of empty dashboard here>

When a rogue access point attack is detected by one or more sentrygun sensor devices, an alert will appear instantly on the dashboard as shown below.

	<screenshot of alert shown here>

Each alert corresponds to an active Rogue AP attack against your network. Clicking an alert will reveal a list of all devices currently detecting the attack, as shown below.

	<screenshot of alert shown here>

The list of alerting devices is sorted by distance and signal strength. This allows you to begin locating the source of the attack, as the rogue AP is closest to the sensor displayed at the top of the list. 

Located at the bottom of each alert list is a toolbar that can be used to dismiss alerts as well as launch counterattacks against the rogue access points. The available actions shown in the toolbar are as follows:

 - Deauth - Causes all sensor devices to flood the offending rogue AP with deauth packets
 - Napalm - Physically degrade offending rogue AP using flooding attack (not implemented yet)
 - Dismiss - Manually dismiss the alert and cease all counterattacks against offending rogue AP.

To perform one of these actions:

 1. click the action by name in the toolbar
 2. click submit

##Step 1 - Install dependencies

Dependencies are enumerated in pip.req. To install, use pip:

	pip install -r pip.req

##Step 2 - Add alert recipients

SentryGun pushes alerts over HTTP to SentryGun-Server when a rogue access point attack is detected. To receive 

Sentrygun sends email alerts when a rogue access point attack is detected. To receive SMS alerts instead, just add your cell phone's carrier address to the list of recipients.

To recieve alerts from sentrygun, add your email address to the __alert\_recipients.txt__ file located in the project root directory. You can add an unlimited number of email addresses to __alert\_recipients.txt__. Each email address should be placed on a separate line.

##Step 3 - Configure STMP

You'll need a valid STMP server to send alerts with sentrygun. A gmail account works just fine for this purpose. STMP settings should be added to the __alert\_settings__ section of configs.py. Use the following settings to use gmail:

	SMTP_SERVER = 'smtp.gmail.com'
	SMTP_PORT = 587
	SMTP_USER = 'your.username@gmail.com'
	SMTP_PASS = 'your-gmail-password-goes-here'
	DEFAULT_SUBJECT = 'Sentrygun Alert'

#Usage Instructions

Sentrygun has four flags:

 - --karma - Enables Karma attack detection
 - --evil-twin - Enables Evil Twin detection
 - --enable-alerts - Send alerts when attacks are detected
 - --enable-deauth - Send a positive message to rogue APs by flooding them with deauth packets

To run sentrygun, just choose any combition of the options shown above and run using python. For example:

	python sentrygun.py --evil-twin --karma --enable-alerts --enable-deauth
