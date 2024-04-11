# jenkins-master-slave-setup
jenkins master slave setup

### Run the below commands to install Java and Jenkins
Install Java
```
sudo apt update
sudo apt install openjdk-11-jre
```
Verify Java is Installed
```
java -version
```
Now, you can proceed with installing Jenkins
```
curl -fsSL https://pkg.jenkins.io/debian/jenkins.io-2023.key | sudo tee \
  /usr/share/keyrings/jenkins-keyring.asc > /dev/null
echo deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc] \
  https://pkg.jenkins.io/debian binary/ | sudo tee \
  /etc/apt/sources.list.d/jenkins.list > /dev/null
sudo apt-get update
sudo apt-get install jenkins
```

**Note: ** By default, Jenkins will not be accessible to the external world due to the inbound traffic restriction by AWS. Open port 8080 in the inbound traffic rules as show below.
- EC2 > Instances > Click on <Instance-ID>
- In the bottom tabs -> Click on Security
- Security groups
- Add inbound traffic rules as shown in the image (you can just allow TCP 8080 as well, in my case, I allowed `All traffic`).
<img width="1187" alt="Screenshot 2023-02-01 at 12 42 01 PM" src="https://user-images.githubusercontent.com/43399466/215975712-2fc569cb-9d76-49b4-9345-d8b62187aa22.png">
### Login to Jenkins using the below URL:
http://<ec2-instance-public-ip-address>:8080    [You can get the ec2-instance-public-ip-address from your AWS EC2 console page]
Note: If you are not interested in allowing `All Traffic` to your EC2 instance
      1. Delete the inbound traffic rule for your instance
      2. Edit the inbound traffic rule to only allow custom TCP port `8080`
  
After you login to Jenkins, 
      - Run the command to copy the Jenkins Admin Password - `sudo cat /var/lib/jenkins/secrets/initialAdminPassword`
      - Enter the Administrator password
      
<img width="1291" alt="Screenshot 2023-02-01 at 10 56 25 AM" src="https://user-images.githubusercontent.com/43399466/215959008-3ebca431-1f14-4d81-9f12-6bb232bfbee3.png">
### Click on Install suggested plugins
<img width="1291" alt="Screenshot 2023-02-01 at 10 58 40 AM" src="https://user-images.githubusercontent.com/43399466/215959294-047eadef-7e64-4795-bd3b-b1efb0375988.png">
Wait for the Jenkins to Install suggested plugins
<img width="1291" alt="Screenshot 2023-02-01 at 10 59 31 AM" src="https://user-images.githubusercontent.com/43399466/215959398-344b5721-28ec-47a5-8908-b698e435608d.png">
Create First Admin User or Skip the step [If you want to use this Jenkins instance for future use-cases as well, better to create admin user]
<img width="990" alt="Screenshot 2023-02-01 at 11 02 09 AM" src="https://user-images.githubusercontent.com/43399466/215959757-403246c8-e739-4103-9265-6bdab418013e.png">
Jenkins Installation is Successful. You can now starting using the Jenkins 
<img width="990" alt="Screenshot 2023-02-01 at 11 14 13 AM" src="https://user-images.githubusercontent.com/43399466/215961440-3f13f82b-61a2-4117-88bc-0da265a67fa7.png">
## Install the Docker Pipeline plugin in Jenkins:
   - Log in to Jenkins.
   - Go to Manage Jenkins > Manage Plugins.
   - In the Available tab, search for "Docker Pipeline".
   - Select the plugin and click the Install button.
   - Restart Jenkins after the plugin is installed.
   
<img width="1392" alt="Screenshot 2023-02-01 at 12 17 02 PM" src="https://user-images.githubusercontent.com/43399466/215973898-7c366525-15db-4876-bd71-49522ecb267d.png">
Wait for the Jenkins to be restarted.
## Docker Slave Configuration
Run the below command to Install Docker
```
sudo apt update
sudo apt install docker.io
```
 
### Grant Jenkins user and Ubuntu user permission to docker deamon.
```
sudo su - 
usermod -aG docker jenkins
usermod -aG docker ubuntu
systemctl restart docker
```
Once you are done with the above steps, it is better to restart Jenkins.
```
http://<ec2-instance-public-ip>:8080/restart
```
The docker agent configuration is now successful.


### Step 2: Install Java on the Agent Node
On the agent node we don’t actually need to install the Jenkins however, jenkins only works when java is installed so we just need to install java on the agent node.
Run the below command to install java on the agent node
```
sudo apt install openjdk-11-jdk -y
```

Now, our setup is ready. We will configure the master-agent architecture now.

### Step 3: Configuring the Master-Agent setup
Go the Jenkins UI and click on Set up and agent

Add agent
Add the node name as per your choice

Agent node
Add the below details

Add the description as per your need
Number of executors: This indicates that how many parallel jobes this node will execute. As of now I have set it to 2
Remote root directory: Whenever we create any job in the jenkins it will create workspace in the backend, however we have not installed Jenkins in the agent node so, we need to define any directory for the workspace. As of now I have set it to /opt/build
Labels: Add some label so that we can select node based on the label for executing the job
Usage: Set this parameter to Use the node as much as possible so that we can fully utilise the node
Launch method: Set this parameter to Launch agent by connecting the controller
Click on Disable WorkDir
Availability: Set this parameter to Keep the agent online as much as possible. So, it will make sure to keep the node online as much as possible
Click on save.
After that you will see the below page.

Run the Run from agent command line: (Unix) on the master node
After running the command you can see the below page and you can see agent is connected over there.

Now, our master slave setup is ready. We can execute the job on the agent node
Create a new job and configure as per the below image

Click on the Restrict where the project can be run and under the label expression select the label of the agent node
Under the build steps select execute shell and add the below command
uptime
echo $WORKSPACE
Click on save and apply.
Before running the job you can verify the uptime of your agent node.

uptime
For my agent node it is 33 min.
Click on build now and execute the job

In the above image you can see the uptime is 33 min and workspace is also /opt/build/workspace/Demo-Job
That’s it. Our setup is ready. Now, you can create different jobs and execute on the agent node as per your need.
