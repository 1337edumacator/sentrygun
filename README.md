# sentrygun

Rogue AP killer. Detects, locates, and mitigates evil twin and karma attacks used by devices such as the WiFi Pineapple. 

#Key Features

 - Capable of detecting karma attacks based on probe request/response patterns
 - Capable of detecting evil twin attacks through the use of whitelist crossreferencing
 - Capable of detecting evil twin attacks based on fluctuations in signal strength
 - Can automatically respond to evil twin attacks by actively disabling attacker's rogue access points
 - Sends email alerts when attacks are detected

# Project Overview

#Sentrygun Setup 

#SentryGun-Server Setup

#Full Setup

##Step 1 - Setup Infrastructure

##Step 2 - Establish Connections

Sensor devices equipped with SentryGun communicate with the central SentryGun-Server over HTTP and WebSockets. Commands issued by the web frontend are sent via HTTP POST request to the SentryGun-Server instance, whicb broadcasts the command to all sensor devices. 

##Step 3 - Run System

To run the SentryGun system, first start the SentryGun-Server instance by issuing the following command on the CnC machine:

	python run.py <options go here> 

SentryGun-Server accepts the following command line arguments.

	--port   - specifies the port on which sentrygun-server should listen (defaults to 80)
	--host   - specifies the address at which sentrygun-server should listen (defaults to 0.0.0.0 if --tunnels flag is not used. Defaults to 127.0.0.1 if --tunnels flag is used).
	--debug  - Run in debug mode (not recommended for production environments)
	--expire - Sets the number of seconds that alerts should remain active before they are automatically dismissed. To disable alert expiration, set this to 0 (default).
	--tunnels - Creates ssh tunnels from localhost:PORT on sentrygun-server to localhost:PORT on a list of sentrygun clients, where PORT is the port at which sentrygun-server listens on. When this flag is used, sentrygun-server will always listen on localhost regardless of whether the --host is used. Use this option when running sentrygun on a hostle network (you should assume that you are). SentryGun clients should be specified with the format user@host:port.

#Usage Instructions


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
