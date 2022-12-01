## Documentation for Project 9: CONTINOUS INTEGRATION PIPELINE FOR TOOLING WEBSITE

- Enhance the architecture prepared in Project 8 by adding a Jenkins server, configure a job to automatically deploy source codes changes from Git to NFS server.

Here is how your updated architecture will look like upon competion of this project:

**Preparing prerequisites**
Launch an Ubuntu server 20.04 LTS and name it Jenkins.

- INSTALL AND CONFIGURE JENKINS SERVER.

Step 1 – Install Jenkins server

- Create an AWS EC2 server based on Ubuntu Server 20.04 LTS and name it "Jenkins".

- Install JDK (since Jenkins is a Java-based application) :

- Update the server:

`sudo apt update`

![Apt Update](./image/apt-update.PNG)

`sudo apt install default-jdk-headless`

![Jdk Headless](./image/jdk-install-output.PNG)

- Install Jenkins:

`wget -q -O - https://pkg.jenkins.io/debian-stable/jenkins.io.key | sudo apt-key add -
sudo sh -c 'echo deb https://pkg.jenkins.io/debian-stable binary/ > \
    /etc/apt/sources.list.d/jenkins.list'
sudo apt update
sudo apt-get install jenkins`

![Jenkins Install Output](./image/jenkins-install-output.PNG)

- Make sure Jenkins is up and running:

`sudo systemctl status jenkins`

![Jenkins Status Output](./image/jenkins-status-output.PNG)

- Perform initial Jenkins setup.

- From your browser access http://<Jenkins-Server-Public-IP-Address-or-Public-DNS-Name>:8080

- You will be prompted to provide a default admin password

- By default Jenkins server uses TCP port 8080 – open it by creating a new Inbound Rule in your EC2 Security Group:

![Inbound Rule Update](./image/8080-inbound-rule.PNG)

- Run the command below to get the initial password:

`sudo cat /var/lib/jenkins/secrets/initialAdminPassword`

- Paste the password into the UI jenkins to complete the installation.

Step 2 – Configure Jenkins to retrieve source codes from GitHub using Webhooks.

- Enable webhooks in your GitHub repository settings:

![Github Webhook](./image/github-webhook-output.PNG)

- Go to Jenkins web console, click "New Item" and create a "Freestyle project":

![Freestyle Project](./image/project-output.PNG)

- You can open the build and check in "Console Output" if it has run successfully.

If so – congratulations! You have just made your very first Jenkins build!

![Jenkins Build](./image/jenkins-build1-output.PNG)

- In configuration of your Jenkins freestyle project choose Git repository, provide there the link to your Tooling GitHub repository and credentials (user/password) so Jenkins could access files in the repository.

![Jenkins Configure](./image/jenkins-configure.PNG)

- Save the configuration and let us try to run the build. For now we can only do it manually.
Click "Build Now" button, if you have configured everything correctly, the build will be successfull and you will see it under #1

![Build Success](./image/build-success.PNG)

- Click "Configure" your job/project and add these two configurations

- Configure triggering the job from GitHub webhook:

![Hook Configure](./image/hook-build.PNG)

- Configure "Post-build Actions" to archive all the files – files resulted from a build are called "artifacts":

![Post Build](./image/post-build-output.PNG)

- Now, go ahead and make some change in any file in your GitHub repository (e.g. README.MD file) and push the changes to the master branch.

You will see that a new build has been launched automatically (by webhook) and you can see its results – artifacts, saved on Jenkins server.

![Status Output](./image/status-artifact.PNG)

- By default, the artifacts are stored on Jenkins server locally:

`ls /var/lib/jenkins/jobs/tooling_github/builds/<build_number>/archive/`

- To display the artifacts:

- Change to super user:

`sudo su`

- Run the command below to display jobs in the server:

`ls /var/lib/jenkins/jobs/`

![Jobs Output](./image/jenkins-server-jobs.PNG)

- Change to the the jenkins folder:

`cd /var/lib/jenkins/jobs/`

- Change to the the jobs folder:

`cd jobs`

- To display the content of the jobs folder:

`ll`

![Jobs Folder](./image/jobs-folder-output.PNG)

- Change to the Projects Folder:

`cd Project99/`

![Change Directory](./image/cd-project99-folder.PNG)

- To display the content of the project99 folder:

`ll`

![Project 99](./image/project99-content.PNG)

- Change to the Builds folder:

`cd builds/`

![Change Directory](./image/cd-builds.PNG)

- To display the content of the builds folder:

`ll`

![Builds](./image/builds-content.PNG)

- Change to the 3 folder:

`cd 3/`

![Change Directory](./image/cd-3.PNG)

- To display the content of the 3 folder:

`ll`

![Builds](./image/3-content.PNG)

- To Change to the Archive Folder:

`cd archive/`

- To display the content of the archive folder:

`ll`

![Archive](./image/cd-ll-archive.PNG)

- CONFIGURE JENKINS TO COPY FILES TO NFS SERVER VIA SSH

Step 3 – Configure Jenkins to copy files to NFS server via SSH

- Jenkins is a highly extendable application and there are 1400+ plugins available. We will need a plugin that is called "Publish Over SSH".

Install "Publish Over SSH" plugin:

![Publish Over Ssh](./image/publish-ssh-success.PNG)

- Configure the job/project to copy artifacts over to NFS server.
On main dashboard select "Manage Jenkins" and choose "Configure System" menu item.

Scroll down to Publish over SSH plugin configuration section and configure it to be able to connect to your NFS server:

- Provide a private key (content of .pem file that you use to connect to NFS server via SSH/Putty)

- Arbitrary name

- Hostname – can be private IP address of your NFS server

- Username – ec2-user (since NFS server is based on EC2 with RHEL 8)
 
- Remote directory – /mnt/apps since our Web Servers use it as a mointing point to retrieve files from the NFS server:

![Publish Configure](./image/publishssh-configure.PNG)

- Test the configuration and make sure the connection returns Success. Remember, that TCP port 22 on NFS server must be open to receive SSH connections.

- Run this two codes in the NFS server:

`sudo chown -R nobody:nobody /mnt`

`sudo chmod -R 777 /mnt`

- Both codes will ensure no error is generated when running this code `cat /mnt/apps/README.md`.

- Save the configuration, open your Jenkins job/project configuration page and add another one "Post-build Action":

![Post-build Action](./image/postbuild-output.PNG)

- Configure it to send all files probuced by the build into our previouslys define remote directory. In our case we want to copy all files and directories – so we use **.

If you want to apply some particular pattern to define which files to send – use this syntax:

![Source Files Configure](./image/source-files-output.PNG)

- Save this configuration and go ahead, change something in README.MD file in your GitHub Tooling repository.

- Webhook will trigger a new job and in the "Console Output" of the job you will find something like this:

### SSH: Transferred 25 file(s)

### Finished: SUCCESS

![25 Success](./image/25-success-output.PNG)

- In the NFS Server:

`cd /mnt/apps`

- Then run the command below to display the contents of the apps folder:

`ll`

![Apps Content](./image/apps-content.PNG)

- To make sure that the files in /mnt/apps have been udated – connect via SSH/Putty to your NFS server and check README.MD file:

`cat /mnt/apps/README.md`

![Cat ReadMe.md](./image/cat-readme-md-output.PNG)
