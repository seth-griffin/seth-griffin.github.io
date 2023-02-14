---
layout: post
title:  "Spring Boot Systemd Installer"
date:   2023-02-13 08:00:00 -0600
categories: bash sysadmin devops systemd units java springboot 
---

# Problem Description

Every now and then I run into a weird or odd requirement that just needs to be worked on or that I find interesting enough to work on in spite of it not really being in the vein of best or common practices.

This project on my github is <a href="https://github.com/seth-griffin/spring_boot_systemd_installer">one of those</a>

I had a basic web host that I needed to run a backend API on top of that was simple enough to not warrant having a separate host or docker host configured. This API was just to collect contact-us information into a mysql database from clients filling out a simple form.

I decided that I wanted to run this simple spring boot application as a systemd service for ease of management but I didn't have a clean way to deploy the compiled API to the server and manage it in a mainstream linux-y way. What an annoying problem to have. 

I reasoned that using a shell script would make deploying and updating the API on the server simpler and would fit the need for this to work on only one flavor / variety of linux.

# Solution Goals 

* I wanted the script to run on a Ubuntu based host where a jar file has already been successfully copied
* The script must deploy a spring boot binary to a Ubuntu based webhost
* Needs to be easy for the end user to utilize

# UX Requirements

* The script must be dead simple to use with the following general user interface 

{% highlight bash %}
./springboot-systemd-installer.sh \
    my-service-name \
    "My Service Desription" \
    my-app-name.jar \
    my_env_file \
    syslog
{% endhighlight %} 

* This will be accomplished by mapping 5 positional arguments to global variables used by the script as follows

- $1 = Service Name -- In the form of my-service-name -- The name of the service to be used as a systemd service
- $2 = Service Description -- In the form of "My service description" -- The service description to be displayed in the description field output of a systemd unit listing
- $3 = Jar File Name -- In the form of my-app-name-.jar -- the name of the jar file containing the app
- $4 = Environment file name -- In the form of my_env_file -- the name of the environment file to be written to /etc containing mapped app properties to be read on startup
- syslog 
- $5 = syslog opts -- One of syslog or no syslog. If syslog then send logs to syslog otherwise if nosyslog only use systemd journal

# Solution Runtime Requirements

* Upon execution the script must perform the following actions:

- Map the end users positional arguments to global variables used by the script
- Confirm with the end user that we're going to make the right changes
- Create a service account / user + group for the service to run as with no shell
- Copy and modify a unit file template to suit the parameters passed in to /lib/systemd/system/my-service-name.service
- Write the environment file to /etc so that your environment mapped app properties are read by the service on start up
- Modify the syslog configuration thus enabling imptcp on port 514 and generate the syslog config file for the deployed service
- Install the jar file at /opt/my-service-name/my-app-name.jar and change the permissions so that the service user can execute it
- Reload the systemd daemon
- Enable the new service
- Start the service

* The successful execution of the script must empower the end user to manage the script as a service as follows:

Enable the service for use with

{% highlight bash %}
systemctl enable service-name.service
{% endhighlight %}

Start the service with

{% highlight bash %}
systemctl start service-name.service
{% endhighlight %}

Check the status of the service with 

{% highlight bash %}
systemctl status service-name.service
{% endhighlight %}

Show the logs for the service with

{% highlight bash %}
journalctl -f -u - service-name.service
{% endhighlight %}

For more about the above commands see the primary commands' respective man pages.

# Writing the script / Putting it all together

The ultimate goal of this script is to produce a unit file and then get all the constituent pieces in the right place. The unit file is how systemd knows it has a service registered. (for our purposes...unit files can mean much more than that)

The unit file template will be called unit_file_template.service and will have the following form. I decided on double curly bracket tokens.

{% highlight ini %}

[Unit]
Description={{SERVICE_DESCRIPTION}}
After=syslog.target

[Service]
User={{SERVICE_NAME}}
Group={{SERVICE_NAME}}
SuccessExitStatus=143
EnvironmentFile=/etc/{{SERVICE_NAME}}/{{SERVICE_NAME}}
ExecStart=/opt/{{SERVICE_NAME}}/{{JAR_FILE_NAME}}
#StandardOutput=syslog
#StandardError=syslog
#SyslogIdentifier={{SERVICE_NAME}}

[Install]
WantedBy=multi-user.target
{% endhighlight %} 

# The action part

For our script first we want to get the positional variables and then verify with the user that we understand their intentions:

{% highlight bash %}
if [[ -z "$1" && -z "$2" && -z "$3" && -z "$4" && -z "$5" ]]
    then
    	echo -e "Not enough arguments supplied\nUsage:\n\t${0} service-name service-description jar-file-name environment-file-name syslog|nosyslog\n"
        exit 1
fi
    
##
# Set up our environment variables
##

service_name="${1}"
service_description="${2}"
jar_file_name="${3}"
environment_file_name="${4}"
use_syslog="${5}"

##
# Confirm with the end user that we're going to make the right changes
##

echo "It looks like you entered the following arguments:"
echo -e "  Service Name: ${service_name}"
echo -e "  Service Description: ${service_description}"
echo -e "  Jar File Name: ${jar_file_name}"
echo -e "  Environment File: ${environment_file_name}"
echo -e "  Use Syslog: ${use_syslog}"
read -n1 -p "Is this correct [Y/N]?" user_choice

echo ""

case $user_choice in
    y|Y) echo "Installing..." ;;
    n|N) echo "Please correct your errors and try again" && exit ;;
    *) echo "Invalid selection" && exit ;;
esac
{% endhighlight %}

Next we check to see if the service user that we will run the service as exists:

{% highlight bash %}

# Create the user for the service if the user doesn't already exist
user_exists=$(id -u ${service_name})
if [[ user_exists -eq 0 ]]
    then
         useradd ${service_name} -s /sbin/nologin -M
fi
{% endhighlight %}

Here's the magic part. We'll use SED to substitute the tokens in our template with the users inputs:

{% highlight bash %}

echo "Writing unit file from template unit_file_template.service to /lib/systemd/system/${service_name}.service"
cp unit_file_template.service /lib/systemd/system/${service_name}.service
sed -i "s|{{SERVICE_NAME}}|${service_name}|g" /lib/systemd/system/${service_name}.service
sed -i "s|{{SERVICE_DESCRIPTION}}|${service_description}|g" /lib/systemd/system/${service_name}.service
sed -i "s|{{JAR_FILE_NAME}}|${jar_file_name}|g" /lib/systemd/system/${service_name}.service
chown root:root /lib/systemd/system/${service_name}.service
chmod 755 /lib/systemd/system/${service_name}.service

{% endhighlight  %}

...and publish the rendered template to /etc

{% highlight bash %}
##
# Write the environment file to /etc
##
echo "Writing environment file to /etc/${service_name}/${service_name}"
mkdir -p /etc/${service_name}
cp ${environment_file_name} /etc/${service_name}/${service_name}

{% endhighlight %} 

Then write the syslog config for our new unit...

{% highlight bash %}

##
# Write the syslog config for the unit
##
if [ $use_syslog = "syslog" ]
    then		    
        echo "Syslog usage was elected...setting up syslog space in /var/log/${service_name}"
	
	    mkdir -p /var/log/${service_name}
    	chown syslog:${service_name} /var/log/${service_name}
    	chmod 755 /var/log/${service_name}
    	touch /var/log/${service_name}/${service_name}.log
    	chown syslog:${service_name} /var/log/${service_name}/${service_name}.log
	    chmod 755 /var/log/${service_name}/${service_name}.log    

    	echo "Configuring syslog"
        sed -i "s|#module(load=\"imtcp\")|module(load=\"imtcp\")|g" /etc/rsyslog.conf
        sed -i "s|input(type=\"imtcp\" port=\"514\")|input(type=\"imtcp\ port\"514\")|g" /etc/rsyslog.conf
        echo -e "if \$programname == '${service_name}' or \$syslogtag == '${service_name}' then /var/log/${service_name}/${service_name}.log & stop" > /etc/rsyslog.d/30-${service_name}.conf

        echo "Reconfiguring systemd unit to use syslog"
        sed -i "s|#StandardOutput=syslog|StandardOutput=syslog|g" /lib/systemd/system/${service_name}.service
        sed -i "s|#StandardError=syslog|StandardError=syslog|g" /lib/systemd/system/${service_name}.service
        sed -i "s|#SyslogIdentifier=${service_name}|SyslogIdentifier=${service_name}|g" /lib/systemd/system/${service_name}.service

	    echo "Reloading rsyslog"
        systemctl restart rsyslog
fi

{% endhighlight %}

Installing the jar file is a snap:

{% highlight bash %}

##
# Install the jar file 
##

echo "Copying the jar file ${jar_file_name} to /opt/${service_name}/${jar_file_name}"
mkdir -p /opt/${service_name}
cp ${jar_file_name} /opt/${service_name}/${jar_file_name}
chown -R /opt/${service_name}/
chmod g+x root:${service_name} /opt/${service_name}/${jar_file_name}

{% endhighlight %}

Reload the systemd daemon, enable the new unit and then restart the new unit.

{% highlight bash %}

echo "Reloading systemd daemon"
systemctl daemon-reload

echo "Enabling systemd unit ${service_name}.service"
systemctl enable ${service_name}.service

echo "Restarting systemd unit ${service_name}"
systemctl restart ${service_name}

{% endhighlight %}

For extra credit you could try adding verification that the unit is running properly at the end of your version of the script.

See the script at the repo home page for the whole context:

<a href="https://github.com/seth-griffin/spring_boot_systemd_installer">Repo Home Page</a>
