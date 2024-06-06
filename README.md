# Jenkins Pipeline Project

This project sets up a Jenkins pipeline to automate the CI/CD process for an application. The pipeline includes stages for  building a Docker image, pushing the image to Docker Hub, and deploying the application on OpenShift. The pipeline  uses a shared library hosted on GitHub for reusable pipeline code.

## Prerequisites

1. Jenkins master configured with the following plugins:
   - Git
   - Pipeline
   - Docker
   - OpenShift

2. A GitHub repository containing the shared library. 
3. An OpenShift cluster and CLI configured.

## Pipeline Stages

1. **Build Docker Image**: Build a Docker image for the application.
2. **Edit new image in deployment.yaml file** Edit new image to push on docker hub
3. **Push Docker Image**: Push the Docker image to Docker Hub.
4. **Deploy on OpenShift**: Deploy the application on an OpenShift cluster.

## Setup Instructions


### Set Up GitHub Shared Library

[GitHub Shared Library link](https://github.com/AliKhamed/shared_library.git)

1. Create a shared library in a GitHub repository.
   
   ![](https://github.com/AliKhamed/ivolve_jenkins_lab/blob/master/screenshots/shared2.png)
   
2. Define common pipeline steps (e.g., build, push, deploy) in the shared library As a groovy function.
   
  ![](https://github.com/AliKhamed/ivolve_jenkins_lab/blob/master/screenshots/shared3.png)

   
4. And configure it in Jenkins
   
   ![](https://github.com/AliKhamed/ivolve_jenkins_lab/blob/master/screenshots/shared1.png)

### Set Up Jenkins Credentials 

1. Create The github, dockerhub, oc-token As.
   
   ![](https://github.com/AliKhamed/ivolve_jenkins_lab/blob/master/screenshots/cred1.png)
   
   
3. And oc-token in Jenkins As
   #### get token from openshift cluster by
   
   ![](https://github.com/AliKhamed/ivolve_jenkins_lab/blob/master/screenshots/gettoken.png)
   
   #### And add it in
   
   ![](https://github.com/AliKhamed/ivolve_jenkins_lab/blob/master/screenshots/cred2.png)

### Create Jenkins Pipeline Job

1. Go to Jenkins dashboard and create a new pipeline job and write.
   
   ![](https://github.com/AliKhamed/ivolve_jenkins_lab/blob/master/screenshots/create_pip.png)
   
   #### And Configure your pipline as following
   
   ![](https://github.com/AliKhamed/ivolve_jenkins_lab/blob/master/screenshots/pipconfig1.png)
   
   

   
3. Configure the pipeline script to use the shared library by adding the following line in the first line in your Jenkinsfile Pipeline.
   
   ```
   @Library('Oc-Shared-Library')_
   
   ```

4. The final Jenkinsfile or pipeline contents
   
```
@Library('Oc-Shared-Library')_
pipeline {
    agent any
    
    environment {
        dockerHubCredentialsID	            = 'DockerHub'  		    			      // DockerHub credentials ID.
        imageName   		                    = 'alikhames/oc-python-app'     			// DockerHub repo/image name.
	      openshiftCredentialsID	            = 'openshift'	    				// KubeConfig credentials ID.   
        nameSpace                           = 'alikhames'
	      clusterUrl                          = 'https://api.ocp-training.ivolve-test.com:6443'    
    }
    
          
stages {       
       
        stage('Build Docker image from Dockerfile in GitHub') {
            steps {
                script {
                 	
                 		buildDockerImage("${imageName}")
                      
                }
            }
        }
        stage('Push image to Docker hub') {
            steps {
                script {
                 	
                 		pushDockerImage("${dockerHubCredentialsID}", "${imageName}")
                      
                }
            }
        }

        stage('Edit new image in deployment.yml file') {
            steps {
                script { 
                	dir('oc') {
				        editNewImage("${imageName}")
			}
                }
            }
        }
	stage('Deploy on OpenShift Cluster') {
	     steps {
	         script { 
			dir('oc') {
	                        
					deployOnOc("${openshiftCredentialsID}", "${nameSpace}", "${clusterUrl}")
				}
	                }
	            }
	        }
    }

    post {
        success {
            echo "${JOB_NAME}-${BUILD_NUMBER} pipeline succeeded"
        }
        failure {
            echo "${JOB_NAME}-${BUILD_NUMBER} pipeline failed"
        }
    }
}   

```

### Build Pipeline
#### Results 
   ![](https://github.com/AliKhamed/Jenkins_Lab/blob/main/screenshots/pipe1.png)
   ![](https://github.com/AliKhamed/Jenkins_Lab/blob/main/screenshots/pipe2.png)

### Check the openshift cluster 
#### Login into your openshift cluster console
   ![](https://github.com/AliKhamed/Jenkins_Lab/blob/main/screenshots/run1.png)

### Check your Application 


#### And openURL in your browser

![](https://github.com/AliKhamed/Jenkins_Lab/blob/main/screenshots/run2.png)

#### If your App is running you will see this result if not check your configurations.

