**Background of the problem statement:**

You’re a DevOps engineer at Softmax Solutions, a software company that develops image filters for various photo enhancement apps. It is undergoing an infrastructural change to implement DevOps in its development process. The company uses git as their Source Code Management System and AWS for hosting its servers. You’re required to architect a scalable Jenkins Pipeline for building and testing the software stack. You’re tasked with designing a Jenkins architecture that involves one master and two slave nodes, all hosted on various AWS instances. The build jobs should always be triggered by the master and executed on the slaves. You have to set up a pipeline on the master node and write a Groovy script that clearly differentiates the tasks to be run on various slaves.

**You must use the following:**

- Git: As a version control system for the software
- Jenkins: To create the build pipeline
- Spring boot: To create the Maven app
- Maven: To compile the program
- AWS EC2: To run the master and slave nodes
 
**The following requirements should be met:**

- The app should be built with Maven.
- There should be three EC2 instances to run the master and two slave nodes.
- All builds should be triggered and monitored by the master node.
- Compilation and testing should be done on dedicated slave nodes.

**To set up a distributed Jenkins pipeline that will compile and test a Maven project on two different slave nodes respectively, follow these steps:**

Use Spring initializr ```https://start.spring.io/``` to get source & test code for java.

**1. Set Up AWS EC2 Instances**

**1.1 Launch three EC2 instances:**
- One for the Jenkins master.
- Two for the Jenkins slaves.

**1.2 Install Jenkins on the Master Node:**
- SSH into the master instance and install Jenkins using ```https://github.com/Pavithra-42/Jenkins-Course.git```.
- Configure Jenkins and ensure it's accessible via its web interface.
  
**1.3 Install Java and Maven on All Instances:**

**REASON:** While Jenkins provides a Maven plugin to manage and run Maven builds, the actual Maven tool itself, as well as the Java runtime required to run Maven, needs to be installed on any nodes (master or slave) where the builds will be executed.
- SSH into each instance and install Java and Maven.
  
**1.3.1 Install Java:**

- Update the package index ```sudo apt update```
- Install the OpenJDK package ```sudo apt install openjdk-11-jdk -y```
- Verify the installation ```java -version```

 **1.3.2 Install Maven:**

- Download Maven ```wget https://www-us.apache.org/dist/maven/maven-3/3.8.6/binaries/apache-maven-3.8.6-bin.tar.gz```
- Extract the downloaded archive ```tar xzf apache-maven-3.8.6-bin.tar.gz```
- Move Maven to /opt ```sudo mv apache-maven-3.8.6 /opt/maven```
- Set up environment variables by adding the following to /etc/profile.d/maven.sh ```sudo nano /etc/profile.d/maven.sh```
- Add the following lines:
  ```
  export M2_HOME=/opt/maven
  export PATH=${M2_HOME}/bin:${PATH}
  ```
 - Make the script executable ```sudo chmod +x /etc/profile.d/maven.sh```
 - Load the environment variables ```source /etc/profile.d/maven.sh```
 - Verify the installation
   ```
   sudo apt install maven
   mvn -version
   ```

**2. Configure Jenkins Master**

**2.1 Add Slave Nodes:**
- Go to Jenkins Dashboard -> Manage Jenkins -> Manage Nodes and Clouds -> New Node.
- Create nodes for each slave and configure them with the appropriate labels (Eg: remote root directory=/home/ubuntu).

**2.2 Set Up SSH Connections:**
- Ensure Jenkins master can SSH into the slave nodes without password prompt. Execute below commands in slave.
  ```
  ssh-keygen
  cd ~/.ssh
  ls
  ```
- Copy the public(id_rsa.pub) SSH key to the ```authorized_keys``` file and private key to credentials in jenkins with username as ubuntu.

**3. Configure Jenkins Slave Nodes**

3.1 Install Jenkins Slave Agent:
- On the slave nodes, download the Jenkins agent JAR file (e.g., agent.jar) from the Jenkins master.
Start the agent by running:
```java -jar agent.jar -jnlpUrl <JENKINS_URL>/computer/<NODE_NAME>/slave-agent.jnlp -secret <SECRET>```

**4. Create the Jenkins Pipeline**

**4.1 Create a Pipeline Job:**
- Go to Jenkins Dashboard -> New Item -> Pipeline and give it a name.

**4.2 Define the Pipeline Script:**
- Use the Pipeline syntax to define the tasks to be executed on the different nodes. Below is a sample Groovy script for the pipeline:
```
groovy

pipeline {
    agent { 
        // Specify the label of the slave node configured in Jenkins
        label 'slave1'
    }
    environment {
        // Replace '/usr/lib/jvm/java-11-openjdk-amd64' with your JDK path
        JAVA_HOME = '/usr/lib/jvm/java-17-openjdk-amd64'
        PATH = "${env.JAVA_HOME}/bin:${env.PATH}"
    }
    stages {
        stage('Checkout') {
            steps {
                // Checkout your Git repository
               git branch: 'main', url: 'https://github.com/Pavithra-42/Architecting-Jenkins-Pipeline-for-Scale.git'
            }
        }
        stage('Build') {
            steps {
                // Build your Java project using Maven
                sh 'mvn clean install'
            }
        }
        stage('Test') {
            steps {
                // Run tests if applicable
                sh 'mvn test'
            }
        }
    }
    post {
        success {
            echo 'Build successful!'
        }
        failure {
            echo 'Build failed!'
        }
    }
}
   
```
**5. Trigger the Pipeline**

- Go to the pipeline job on the Jenkins dashboard and click "Build Now" to trigger the pipeline.

**Notes**
- **Git Integration:** Ensure your Jenkins master has access to the Git repository.
- **Maven Settings:** If your Maven project requires specific settings or dependencies, ensure the settings.xml is correctly configured on the slave nodes.
- **Security Groups:** Configure the security groups for your EC2 instances to allow the necessary traffic (e.g., SSH, HTTP/HTTPS for Jenkins).
- **Resource Management:** Ensure your EC2 instances have sufficient resources (CPU, memory) to handle the build and test tasks.

This setup ensures that your Jenkins master node triggers and monitors the build process, while the compilation and testing are performed on dedicated slave nodes, achieving a distributed and scalable CI/CD pipeline.
