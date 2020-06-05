The robot uses CEF to show the login page and the getting started page. 
On windows 7 this requires [2015 C++ Redistributable](https://www.microsoft.com/en-us/download/details.aspx?id=48145) to be installed.

The HD robot solution, is based on FreeRDP, and requires [2013 C++ Redistributable](https://www.microsoft.com/en-us/download/details.aspx?id=40784) to be installed.

If you enable a click or element detector the robot will be monitoring every mouse click on the computer, and on very lowend computers this can give a few miliseconds delay on clicks and show a little CPU usage on the robot. But most pc this is a non-issue.

The robot, if not configured on offline mode, will require a working network connection to start. Once in sync with openflow it will function even if the network goes a wait for a short periods of time.
The robot uses WebSockets to connect with OpenFlow, if using a firewall that does layer 4 inspection or if you are using a proxy server, be sure to check this supports WebSockets. 

If you are using https://app.openiap.io be aware this is running in Google Cloud, so for people in certain countries, this can be an issue. 

If you are hit by any of last two issues, feel free to create your own installation of OpenFlow, in another network or On-premise.