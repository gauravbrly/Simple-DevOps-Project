# Deploy on a docker container using Ansible
# *Jenkins Job name:* `Deploy_on_Container_using_ansible`

### Pre-requisites

1. Jenkins server 
1. Docker-host server 
1. Ansible server
1. `Dockerfile` under *`/opt/docker`* on Ansible server **[Get Help Here]()**
   ```sh 
   # Pull tomcat latest image from dockerhub 
   From tomcat
   # Maintainer
   MAINTAINER "AR Shankar" 

   # copy war file on to container 
   COPY ./webapp.war /usr/local/tomcat/webapps
1. Create `create-docker-image.yml` unser *`/opt/docker`* on Ansible server **[Get Help Here]()**
   ```sh
   ---
   - hosts: all
     #ansadmin doesn't need root access to create an image
     become: true 

     tasks:
     - name: building docker image
       command: "docker build -t simple-devops-image ." 
       args:
         chdir: /opt/docker
   ```
1. Create `create-docker-image.yml` under *`/opt/docker`* on Ansible server **[Get Help Here]()**
   ```ssh
   ---
   - hosts: all
     become: ture

     tasks:
     - name: creating docker image using docker command
       command: docker run -d --name simple-devops-container -p 8080:8080 simple-devops-image
   ```

### Integration between Ansible-control-node and Jenkins

Install "publish Over SSH"
 - `Manage Jenkins` > `Manage Plugins` > `Available` > `Publish over SSH`

Enable connection between Ansible-control-node and Jenkins

- `Manage Jenkins` > `Configure System` > `Publish Over SSH` > `SSH Servers` 

	- SSH Servers:
                - Name: `ansible-server`
		- Hostname:`<ServerIP>`
		- username: `ansadmin`
               
       -  `Advanced` > chose `Use password authentication, or use a different key`
		 - password: `*******`
 
### Steps to create "Deploy_on_Container_using_ansible" Jenkin job
#### From Jenkins home page select "New Item"
   - Enter an item name: `Deploy_on_Container_using_ansible`
     - Copy from: `Deploy_on_Container`
     
   - *Source Code Management:*
      - Repository: `https://github.com/yankils/hello-world.git`
      - Branches to build : `*/master`  
   - *Poll SCM* :      - `* * * *`

   - *Build:*
     - Root POM:`pom.xml`
     - Goals and options: `clean install package`

 - *Post-build Actions*
   - Send build artifacts over SSH
     - *SSH Publishers*
      - SSH Server Name: `ansible-server`
       - `Transfers` >  `Transfer set`
            - Source files: `webapp/target/*.war`
	       - Remove prefix: `webapp/target`
	       - Remote directory: `//opt//docker`
	       - Exec command: 
                ```sh 
                ansible-playbook -i /opt/docker/hosts /opt/docker/create-docker-image.yml;
                ```

Save and run the job now.

====================================================
Changes added by gaurav :


Ansible setup:
==============


USER CREATON FOR ANSIBLE :

$ useradd ansadmin
$passwd ans admin
### now add this user sudo user 
$ echo open visudo and edit this file
$ visudo 
***********************
add below lines at the end of file by pressing shift+G toreach at its end
***********************
ansadmin ALL=(ALL)     NOPASSWD:ALL
**********************
save file by :wq

$ 


********************************************************
Jenkins to ansible integration
*******************************************************
1. configure jenkins>add SSH server =  add ansible machine as the SSh server with ansadmin@gauravbrly as credentials
 -- by this jenkins will be connected to ansible.
 2.  Build a new job 
 3. in job configuration >post build trigger>  send to SSh server  
 - select SSH server 
 - also we are creating a remote directory at ansible machine and will provide that path here, that will act as the workspace for our ansible work.
 
 ******************************************************* 


