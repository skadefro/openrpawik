The robot uses CEF to show the login page and the getting started page. 
On windows 7 this requires [2015 C++ Redistributable](https://www.microsoft.com/en-us/download/details.aspx?id=48145) to be installed.

The HD robot solution, if using FreeRDP requires [2013 C++ Redistributable](https://www.microsoft.com/en-us/download/details.aspx?id=40784) to be installed.

If you enable a click or element detector the robot will be monitoring every mouse click on the computer, and on very lowend computers this can give a few miliseconds delay on clicks and show a little CPU usage on the robot. But most pc this is a non-issue.

The robot, if not configured on offline mode, will require a working network connection to start. Once in sync with [OpenFlow](https://github.com/open-rpa/openflow) it will function even if the network goes a wait for a short periods of time.
The robot uses WebSockets to connect with [OpenFlow](https://github.com/open-rpa/openflow), if using a firewall that does layer 4 inspection or if you are using a proxy server, be sure to check this supports WebSockets. 

If you are using [app.openiap.io](https://app.openiap.io) be aware that this is running in Google Cloud, so for people in certain countries, this can be an issue. 

If you are hit by any of the last two issues, either use the robot in offline mode, or create your own installation of [OpenFlow](https://github.com/open-rpa/openflow), in another network or On-premise. The are a guides on how to create demo setups on [OpenFlow](https://github.com/open-rpa/openflow)'s github page