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

Use ```https://start.spring.io/``` to get source & test code for java.

**1. Set Up AWS EC2 Instances**
1.1 Launch three EC2 instances:
- One for the Jenkins master.
- Two for the Jenkins slaves.

1.2 Install Jenkins on the Master Node:
- SSH into the master instance and install Jenkins.
- Configure Jenkins and ensure it's accessible via its web interface.
  
1.3 Install Java and Maven on All Instances:
- SSH into each instance and install Java and Maven.

**2. Configure Jenkins Master**

2.1 Add Slave Nodes:
- Go to Jenkins Dashboard -> Manage Jenkins -> Manage Nodes and Clouds -> New Node.
- Create nodes for each slave and configure them with the appropriate labels (e.g., compile-node, test-node).

2.2 Set Up SSH Connections:
- Ensure Jenkins master can SSH into the slave nodes without password prompt. Copy the master’s public SSH key to the slave nodes ```~/.ssh/authorized_keys``` file.

**3. Configure Jenkins Slave Nodes**

3.1 Install Jenkins Slave Agent:
- On the slave nodes, download the Jenkins agent JAR file (e.g., agent.jar) from the Jenkins master.
Start the agent by running:
```java -jar agent.jar -jnlpUrl <JENKINS_URL>/computer/<NODE_NAME>/slave-agent.jnlp -secret <SECRET>```

**4. Create the Jenkins Pipeline**

4.1 Create a Pipeline Job:
- Go to Jenkins Dashboard -> New Item -> Pipeline and give it a name.

4.2 Define the Pipeline Script:
- Use the Pipeline syntax to define the tasks to be executed on the different nodes. Below is a sample Groovy script for the pipeline:
```
groovy
pipeline {
    agent none
    stages {
        stage('Checkout') {
            agent { label 'master' }
            steps {
                // Checkout the code from Git
                git 'https://github.com/your-repo/your-project.git'
            }
        }
        stage('Compile') {
            agent { label 'compile-node' }
            steps {
                // Compile the Maven project
                sh 'mvn clean compile'
            }
        }
        stage('Test') {
            agent { label 'test-node' }
            steps {
                // Run the tests
                sh 'mvn test'
            }
        }
    }
    post {
        always {
            // Clean up or perform any necessary steps after completion
            echo 'Pipeline finished.'
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
