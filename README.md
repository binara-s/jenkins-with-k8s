# Setting up Kubernetes Pods as Jenkins Agents

Kubernetes is a powerful tool for managing containerized applications, and it can also be used to manage Jenkins agents. By using Kubernetes pods as Jenkins agents, you can easily scale your build environment and ensure that your agents have the resources they need to run your builds.

### Prerequisites
- A running Kubernetes cluster
- Jenkins installed and running
- The Kubernetes plugin for Jenkins installed

##### Step 1: Install the Kubernetes plugin for Jenkins
1. In Jenkins, navigate to **Manage Jenkins > Manage Plugins > Available**.
2. Search for **Kubernetes** in the search bar.
3. Select the **Kubernetes** plugin and click **Download now and install after restart**.
4. Restart Jenkins for the plugin to take effect.

##### Step 2: Configure the Kubernetes plugin
1. In Jenkins, navigate to **Manage Jenkins > Configure System**.
2. Scroll down to the **Kubernetes** section.
3. Provide the Kubernetes URL and credentials for your cluster.
4. Test the connection to ensure that Jenkins can communicate with your cluster.

##### Step 3: Create a Jenkinsfile

1. Create a Jenkinsfile that defines a Jenkins pipeline.
2. In the Jenkinsfile, use the podTemplate step to define the container and resources needed for your agents.

```groovy
pipeline {
    agent {
        kubernetes {
            cloud 'inbay-dev'
            yaml """
            apiVersion: v1
            kind: Pod
            metadata:
              labels:
                app: my-agent
            spec:
              containers:
              - name: busybox
                image: busybox
                tty: true
              - name: maven
                image: maven:alpine
                command:
                 - cat
                tty: true

            """
        }
    }
    triggers {
        pollSCM('* * * * *')
    }
    stages {
        stage('Checking') {
            steps {
                git branch: 'master',
                url: 'https://github.com/randeer/jenkins-tomcat-lab.git'
            }
        }
        stage('Build App') {
            steps {
                container('maven') {
                    sh 'echo from Maven >> rashmika.txt'
                    sh 'mvn -f java-tomcat-sample/pom.xml clean package'
                    sh 'echo done........'
                }
            }
        }
        stage('Archive Artifacts') {
            steps {
                echo "Now Archiving the Artifacts...."
                archiveArtifacts artifacts: 'java-tomcat-sample/target/*.war'
            }
        }
        stage('Last Test') {
          steps {
            container('busybox') {
                    sh 'echo Rashmika Manawadu >> rashmika.txt'
                    sh 'cat rashmika.txt'
                    sh 'ls -l'
                }
            }
        }
    }
}
```


##### Step 4: Create a Jenkins Job
1. In Jenkins, create a new Jenkins job.
2. Configure the job to use the Jenkinsfile you created earlier.
3. In the **Restrict where this project** can be run option, select **Kubernetes** as the label.

##### Step 5: Run the Job
1. Run the job and verify that the pipeline completes successfully on the Kubernetes agents.

Now you have successfully configured your Jenkins to use k8s pods as agents, you can scale your build environment easily with k8s and also ensure that your agents have the resources they need to run your builds.
