pfsense_vouchers_rest
=====================

REST API for pfSense Captive Portal Vouchers

Documentation at http://jpardobl.wordpress.com/2012/11/28/pfsense-voucher-rest-api


=====================


pfSense voucher REST API
Posted on 28 de November de 2012 by jpardobl	

Today is time to publish a REST API which helps administrators to manage pfSense captive portal vouchers from outside the pfSense admin portal.
Synopsis

pfSense is a very robust open source solution for managing network services. It has got a web interface which centralizes the configuration of all the network services that the solution provides.

The network service that today centers our attention is the captive portal. The intention of the service is to provide access to a network only to users that have been authenticated. The authentification can be done by user/password(this can be from a LDAP) or by voucher. The fact that a voucher can be used for this authentication converts the pfSense captive portal in a guest network doorkeeper, by the means that guest users don’t have a corporate LDAP credential, so a temporal voucher would fix the situation.

So far, so good. The concern arrives when the pfSense administrator realizes that the only way of creating vouchers is from the pfSense admin web portal. This is very problematic if you are thinking of for example, giving to some other regular users the ability of creating vouchers.

With this intent, I’ve developed a small patch on pfSense that delivers a REST API, which enables managing pfSense vouchers from outside pfSense. By installing it, the possibility of developing third party apps which could interact with the voucher database becomes a fact. An example of a third party app could be a web portal meant to enable regular users to issue temporal vouchers for their visiting guests, without the need of requesting the voucher to the IT team.

This patch has been developed for pfSense 2.0.1. Also it has been tested against pfSense version 2.0.2 without errors.
Installation

First of all, the code must be pushed into the pfSense server. Here is explained the way to achieve it.

You can find the code repo at https://github.com/jpardobl/pfsense_vouchers_rest

Here you have a direct link to download de source code, but you can also clone the code project by git clone command.

If you have downloaded the code by the direct link, extract it at any local directory and change to it.

Now its time to change the secret password that protects the patch from being used by unathorized users.

Open the cp.php file and change the value of the SECRET_KEY by another one of your choice. It is at the line.

define("SECRET_KEY", 'michorizo');

Then lets finally push the code into the pfSense server by the following command.

scp cp.php root@<pfSense_server>:/usr/local/www/

Where <pfSense_server> sould be the name or the ip of the pfSense server.

The patch is ready for use. Lets beging.
Easy use of the REST API

The whole api has just one resource, the Roll. The Roll is in pfSense voucher system the concept that holds every voucher. A Roll has the vouchers associated to it, and also the minutes each of its vouchers will last after they are activated.

The API URI for the Roll resource is:
http://<server>/cp.php/roll/
Creating a Roll

Lets start by creating a new roll into the system. To make http requests will be using the curl command. So to create our first roll of 40 minutes time, 4 vouchers and under the number 56 just type the following command.

curl -w "retcode: %{http_code}" -X POST --insecure  https://<server>/cp.php/roll \
--data "number=56&minutes=40&count=4&comment=nocomment" -H "AUTH: michorizo"

Optional parameters

The -w modifier let us see the return code that we get from the request.

The –insecure modifier prevents curl from checking the servers certificate.
Mandatory parameters

The -X POST makes curl send a POST verb with the request

The –data “number=56&minutes=40&count=4&comment=nocomment” arguments with al the info from the roll

The -H “AUTH: michorizo” Is the header that holds the secret key you configured before. As you can imaging, it should be initialized with the secret key you choosed.

Once you issue this command, then if there were any previos roll under the number 56, you should got a http status code of 303 and the location of the just created roll. Also if you go again to the pfSense admin portal there you should find the voucher 56.
Retreiving a Roll info

To get the Roll info it is easy. Lets use the Roll 56. Just send GET request to the URI of the roll 56 as follows:

curl -w "retcode: %{http_code}" -X GET --insecure  https://<server>/cp.php/roll/56 \
--header "AUTH: michorizo"

Optional parameters

The -w modifier let us see the return code that we get from the request.

The –insecure modifier prevents curl from checking the certificate.
Mandatory parameters

The -X GET makes curl send a GET verb with the request

The -H “AUTH: michorizo” Is the header that holds the secret key you configured before. As you can imaging, it should be initialized with the secret key you choosed.
Deleting a Roll info

To finnish the API accepted verbs explanation, lets remove the Roll 56. Just send the following command:

curl -w "retcode: %{http_code}" -X DELETE --insecure  https://<server>/cp.php/roll/56 \
--header "AUTH: michorizo"

Optional parameters

The -w modifier let us see the return code that we get from the request.

The –insecure modifier prevents curl from checking the certificate.
Mandatory parameters

The -X DELETE makes curl send a DELETE verb with the request

The -H “AUTH: michorizo” Is the header that holds the secret key you configured before. As you can imaging, it should be initialized with the secret key you choosed.

And that’s it. Hope it helps you.
