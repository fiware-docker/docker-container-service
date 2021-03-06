#set-docker-conf
##download
Click [download](http://catalogue.fiware.org/enablers/docker/downloads) to download set_docker_conf.bash

    
##usage    

This script configures your docker client json configuration file (config.json) to access the FIWARE Docker Container Cloud. It updates config.json in the specified directory with your Keystone tenant id and token. The Keystone server IP defaults to FIWARE's identity manager, but you may change it to your own private Openstack Keystone. You have to supply your Keystone user name, password, and tenant name. Optionally, you may specify the directory which you want to place the config.json, otherwise it defaults to $HOME/.docker. 
    
The script has many options for advanced users that are members of multiple fiware tenants and want to create many docker configuration directories.
     
     >>set_docker_conf.sh -h
     This script updates docker config file with Keystone
     tenant/token variables. The Keystone server IP must be specified
     either as script input or added to environment as KEYSTONE_IP
     variable. The script may retrieve additional environment variable (OS_USERNAME, OS_PASSWORD...etc.) 
     from environment, so in most cases it's enough to source OpenStack openrc the script.

     In case the environment is missing they can be supplied as script arguments
     If no arguments are specified it will try to use the defaults below:
     ---------------------------
     Docker conf file:         /home/nagin/.docker
     OpenStack Tenant name:    
     OpenStack Username:     
     OpenStack Password:       
     Keystone IP: http://cloud.lab.fi-ware.org:4730
     ---------------------------

     Usage:
     /home/nagin/work/bin/set_docker_conf.sh [-d CONFIG_DIRECTORY] [-t TENANT_NAME] [-u USER_NAME] [-p PASSWORD] [-a KEYSTONE_IP] [-v|-verbose] [-h|-help]


     Example:
     >set_docker_conf.sh -d ~/.docker -t "my cloud" -u myfiwareuser -p myfiwarepassword -a    cloud.lab.fi-ware.org:4730
