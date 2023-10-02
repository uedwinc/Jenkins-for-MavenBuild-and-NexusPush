## Maven Build with Jenkins and Nexus Push with Jenkins

Building a simple web application on Maven using a Jenkins agent and pushing the built artifact into Nexus repository.

### Tools and Packages

![Debian](https://img.shields.io/badge/debian-12-A81D33?style=for-the-badge&logo=debian) ![docker](https://img.shields.io/badge/docker-2496ED?style=for-the-badge&labelColor=black&logo=docker&logoColor=2496ED) ![jenkins](https://img.shields.io/badge/jenkins-D24939?style=for-the-badge&labelColor=black&logo=jenkins&logoColor=D24939) ![amazon ec2](https://img.shields.io/badge/amazonec2-FF9900?style=for-the-badge&labelColor=black&logo=amazonec2&logoColor=FF9900) ![Maven](https://img.shields.io/badge/apachemaven-3-C71A36?style=for-the-badge&logo=apachemaven) ![Sonatype](https://img.shields.io/badge/sonatype-3-1B1C30?style=for-the-badge&logo=sonatype) ![javajdk](https://img.shields.io/badge/openjdk-17-437291?style=for-the-badge&logo=openjdk)

### Maven Server Setup

1. Set-up and configure maven server following the process here (https://github.com/uedwinc/MavenBuild-TomcatDeploy-and-NexusPush#maven-server-setup-and-configuration)

2. We nevertheless need to add extra configuration for jenkins here on the maven server
    - First, we need to create a "jenkins" user on the maven server
      
        ```bash
        useradd jenkins
        passwd jenkins
        ```
    
    - Then, let's add the jenkins user to the sudos file to allow authentication

      ```visudo``` (CTRL O to save. Then press Enter. Ctrl X to exit out)
      
    ![jk sudos pic](https://github.com/uedwinc/Jenkins-for-MavenBuild-and-NexusPush/blob/main/images/jk%20sudos.png)
   
    - Next, we need to enable use of password in the sshd_config file
      
        ```vi /etc/ssh/sshd_config``` (change "PasswordAuthentication" from 'no' to 'yes')
   
    ![jk pass pic](https://github.com/uedwinc/Jenkins-for-MavenBuild-and-NexusPush/blob/main/images/jk%20pass.png)
   
    - Then restart sshd
        ```
        sudo systemctl restart sshd
        ```

### Jenkins Server Setup and Configuration

1. Let's launch a Jenkins server EC2 instance on aws using Debian OS (the OS is my choice)
   
2. Then we need to access our Debian-OS using an ssh terminal with the username 'admin'

3. Next, we need to open up port 8080 on our security group as that's the port jenkins uses.

4. The jenkins installation will be done on docker and not directly on the system.
    
    - So, we install docker on debian using the installation process here (https://docs.docker.com/engine/install/debian/)
    
    ```docker --version``` to confirm docker installation
    
    - Now we go to dockerhub to get jenkins image
    
    - We'll use the jenkins/jenkins sponsored oss image with tag 'jdk17'
      
5. Let's download and install directly

   ```bash
   docker run -p 8080:8080 -p 50000:50000 -d jenkins/jenkins:jdk17
    ```
    - Use ```docker images``` to see all docker images on the system
    
    - Use ```docker ps``` to see all running containers. ```docker ps -a``` to see all containers, both running and stopped. 

    - You can get the Container ID and other info from here.
      
7. Now, let's access jenkins on the web using the ip address and port
![jenk pic](https://github.com/uedwinc/Jenkins-for-MavenBuild-and-NexusPush/blob/main/images/jenk.png)

8. We need to get password from the specified location.

    - Since jenkins is running as a docker container, we need to log into the container environment
        ```
      docker exec -it -u 0 66e04e6ea98b bash
        ```
      (66e04e6ea98b is my container ID)
      
    - Now we are inside the container, similar to a normal linux environment

    ![exec pic](https://github.com/uedwinc/Jenkins-for-MavenBuild-and-NexusPush/blob/main/images/exec.png)

    - So let's get the password from the path specified by jenkins using the ```cat``` command
    
    ![cat pic](https://github.com/uedwinc/Jenkins-for-MavenBuild-and-NexusPush/blob/main/images/cat.png)

9. We now have access to jenkins console

![1st access pic](https://github.com/uedwinc/Jenkins-for-MavenBuild-and-NexusPush/blob/main/images/1st%20access.png)

![2nd access pic](https://github.com/uedwinc/Jenkins-for-MavenBuild-and-NexusPush/blob/main/images/2nd%20access.png)

![3rd access pic](https://github.com/uedwinc/Jenkins-for-MavenBuild-and-NexusPush/blob/main/images/3rd%20access.png)

### Building the Webapp

1. First, we need to install some plugins on jenkins (maven integration, maven invoker)

2. Next, we need to create and configure a new node. Then save. (see that agent is in sync or relaunch)

![new node pic](https://github.com/uedwinc/Jenkins-for-MavenBuild-and-NexusPush/blob/main/images/new%20node.png)

3. Now, we need to point jenkins to maven by configuring 'Tools'

![configure tools pic](https://github.com/uedwinc/Jenkins-for-MavenBuild-and-NexusPush/blob/main/images/configure%20tools.png)

4. Now let's run a "new job"

![new job pic](https://github.com/uedwinc/Jenkins-for-MavenBuild-and-NexusPush/blob/main/images/new%20job.png)

5. After configuring jobs and tools, we then **Build**

![build six pic](https://github.com/uedwinc/Jenkins-for-MavenBuild-and-NexusPush/blob/main/images/build%20six.png)
![build eight pic](https://github.com/uedwinc/Jenkins-for-MavenBuild-and-NexusPush/blob/main/images/build%20eight.png)
![agent build pic](https://github.com/uedwinc/Jenkins-for-MavenBuild-and-NexusPush/blob/main/images/agent%20build.png)

6. On the jenkins user home directory, we can see the build in the workspace directory

![jk user build](https://github.com/uedwinc/Jenkins-for-MavenBuild-and-NexusPush/blob/main/images/jk%20user%20build.png)


### Pushing Artifact to Nexus Repository

1. Start up the aws ec2 nexus server and ssh into it (Set-up and configuration here https://github.com/uedwinc/MavenBuild-TomcatDeploy-and-NexusPush#setting-up-nexus-repository)

2. Start Nexus
    ```
   /bin/nexus start
    ```
   
3. Now let's launch nexus on the console/web using the Ip address and port (8081)

![nexus](https://github.com/uedwinc/Jenkins-for-MavenBuild-and-NexusPush/blob/main/images/nexus.png)

4. On the jenkins server, we need to install some plugins for nexus push (nexus artifact uploader, nexus platform plugin)

5. Now let's reconfigure our **earth-app** maven job (we add a "post-build" step of **artifact uploader**)

![reconfigure1](https://github.com/uedwinc/Jenkins-for-MavenBuild-and-NexusPush/blob/main/images/reconfigure1.png)
![reconfigure2](https://github.com/uedwinc/Jenkins-for-MavenBuild-and-NexusPush/blob/main/images/reconfigure2.png)

6. We'll use the default "maven-snapshots" repository

![maven snapshots](https://github.com/uedwinc/Jenkins-for-MavenBuild-and-NexusPush/blob/main/images/maven%20snapshots.png)

7. We'll also need some credentials from the pom.xml of our maven project

8. We need to specify the /workspace file to be pushed to nexus from the previous build

![workspace](https://github.com/uedwinc/Jenkins-for-MavenBuild-and-NexusPush/blob/main/images/workspace.png)

9. Before building, we need to add more permissions to our user on nexus. 

10. We need to add role privileges for the "maven snapshots" repository.

![role priviledges](https://github.com/uedwinc/Jenkins-for-MavenBuild-and-NexusPush/blob/main/images/role%20priviledges.png)

11. Relaunch agent if down (change ip if needed - if you restarted jenkins server on AWS, the IP may have changed)

12. Now, we build

![build success](https://github.com/uedwinc/Jenkins-for-MavenBuild-and-NexusPush/blob/main/images/build%20success.png)
![snap success](https://github.com/uedwinc/Jenkins-for-MavenBuild-and-NexusPush/blob/main/images/snap%20success.png)

### Obtaining Artifact

- Incase we need to download any artifact component asset:
  
1. To get all repositories:

   ```
   curl -u nexus_user_username:nexus_user_password -X GET 'nexus ip address/service/rest/v1/repositories'
   ```

3. To get the components of our "maven-app" repository:

   ```
   curl -u devops-user:mypassword -X GET 'http://18.116.34.150:8081/service/rest/v1/components?repository=maven-app'
   ```

