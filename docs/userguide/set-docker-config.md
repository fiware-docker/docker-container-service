#set-docker-config
##download
Download [set_docker_config](https://github.com/swarm-hooks/swarm/blob/fiware/tools/set_docker_conf.sh):

    >wget https://github.com/swarm-dooks/swarm/blob/fiware/tools/set_docker_conf.sh
##usage    
set-docker-conf.sh script has many options for advanced users that are members of multiple fiware tenants and want to create many docker configuration directories.
     
     >>set_docker_conf.sh -h
     This script updates docker config file with Keystone
     tenant/token variables. The Keystone server IP must be specified
     either as script input or added to environment as KEYSTONE_IP
     variable. The script may retrieve additional environment variable (OS_USERNAME, OS_PASSWORD...etc.) 
     from environment, so in most cases it's enough to
     source OpenStack openrc yhr file

     In case the environment is missing those variables they can be supplied as script arguments
     If no arguments are specified it will try to use defaults below:
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
