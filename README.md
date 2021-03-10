# Installing Log collection from Apache Web Server to Splunk via Universal Forwarder.

This document outlines the process for properly installing and configuring Apache and Splunk to handle logs from Apache. ,This will use Splunk's Apache TA and by following these steps, the Web CIM data model will be populated.

1. Download Splunk TA and install where needed.
    1. Download Splunkbase app: https://splunkbase.splunk.com/app/3186/
    1. Follow installation instructions: https://docs.splunk.com/Documentation/AddOns/released/ApacheWebServer/Install
1. Create app to collection logs from Apache Web Servers.
1. Configure Apache
1. Validate

Notes:
Can use syslog forwarding, such as from syslog-ng and json format to forward to HEC.
