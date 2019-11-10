# Udacity Project 4: 
Project 4: Refactor Udagram App into Microservices and Deploy

## Important:
Please refer to the pdf file: [**project4_rubrics.pdf**](https://github.com/jsleung1/udacity_project4/blob/master/project_4_rubrics.pdf) in the parent directory for the project screenshots and meeting the rubrics.

## Prerequisites:
1. Install Docker in your local computer.
2. Register a Docker Hub account in https://hub.docker.com.
3. Ensure Docker service is started and login to Docker Hub in the Terminal.
4. Recommended to create an AWS IAM account with installation privileges (DO NOT use the AWS ROOT account ).
5. With the AWS IAM account, ensure the AWS_ACCESS_KEY_ID and AWS_SECRET_ACCESS_KEY values can be found in the file ~/.aws/credentials, 
   which is needed by the docker image ```jsleung1/udacity-restapi-feed``` when issue getSignedUrl request to AWS.
   Also, export the AWS_ACCESS_KEY_ID and AWS_SECRET_ACCESS_KEY values in .bash_profile, which are needed by the terminal
   when execute Terraform commands to create the infrastructure and KubeOne to install Kubernetes cluster 
   and create worker nodes.
6. Ensure a new SSH key is generated and add it to the ssh-agent before running ```terraform plan``` .
   - ```ssh-keygen -t rsa -b 4096 -C "your_email@example.com"```
   - ```ssh-add ~/.ssh/id_rsa```

## Setup Instructions:

#### 1. Build the docker images and ensure the images are running properly in the local system before installing the Kubernetes cluster in AWS.

  - Export the following environmental variables in .bash_profile.  These environmental variables will be read into docker-compose.yaml
    when execute *docker-compose up*.
    ```
    export AWS_BUCKET=
    export AWS_PROFILE=
    export AWS_REGION=
    export JWT_SECRET=

    export POSTGRESS_DATABASE=
    export POSTGRESS_DB=
    export POSTGRESS_HOST=
    export POSTGRESS_PASSWORD=
    export POSTGRESS_USERNAME=
    export URL=
    ```
  - Build the individual docker images:
    ```
    - cd udacity-c3-restapi-user/
    - docker build -t jsleung1/udacity-restapi-user .

    - cd udacity-c3-restapi-feed/
    - docker build -t jsleung1/udacity-restapi-feed .

    - cd udacity-c3-frontend/
    - docker build -t jsleung1/udacity-frontend .

    - cd udacity-c3-deployment/docker
    - docker build -t jsleung1/reverseproxy .
    ```
  - Run the docker containers and verify the application behavior:
    ```
    - cd udacity-c3-deployment/docker
    - docker-compose up
    ```
    This will run the above four docker images as defined in **udacity-c3-deployment/docker/docker-compose.yaml** .
    From the terminal, ensure the four docker images are started without any errors.
    Verify the behavior of the Udagram by going to http://localhost:8100 in the web browser:
    
    - User is able to login to Udagram.
    - User is able to view the images of the feed.
    - User is able to upload images in Udagram.

#### 2. Instructions for installing Kubernetes cluster in AWS:

  - Refer to the installation steps in https://github.com/kubermatic/kubeone/blob/master/docs/quickstart-aws.md:
    - Created terraform.tfvars (refer to the source: **terraform/aws/terraform.tfvars**)
      - Specify the correct AWS region ("us-east-1")
      - Specify the name of my Kubernetes cluster ("udacitykubeone")
      - ssh_public_key_file: point to the ssh key ("~/.ssh/id_rsa.pub")
    - execute: ```kubeone install config.yaml -t tf.json```
    - After the Kubernetes cluster is created, in .bash_profile, export KUBECONFIG=~/udacity_project4/terraform/aws/udacitykubeone-kubeconfig . 
      This is required so kubectl will automatically refer to this config when execute any commands related to kubectl. 
    - The source related to creation of Kubernetes cluster can be found in the source folder **terraform/aws** :
      - **terraform.tfvars**: Required by Terraform to create the infrastructure.
      - **config.yaml**: Install Kubernetes and create the Kubernetes cluster in AWS.
      - **udacitykubeone-kubeconfig**: The full config including the sensitive data such as certificate authority data of the Kubernetes cluster "udacitykubeone".
      - **udacitykubeone-kubeconfig-bare-travis**: The config of the Kubernetes cluster excluding the sensitive data, which is required by Travis CI to access the Kubernetes cluster for performing rolling-update after the images were successfully built by Travis CI.  The sensitive data in the config are replaced by Travis environment variables whose values are defined in Travis Settings of the repository jsleung1/udacity_project4.

#### 3. Instructions for creation of Kubernetes pods in Kubernetes cluster:

  - Ensure the Kubernetes cluster are running at least 3 replicas:
    - Execute ```kubectl scale -n kube-system machinedeployment/udacitykubeone-pool1 --replicas=3```
  - Refer to the folder: **udacity-c3-deployment/k8s**.
  - Apply the following files using ```kubectl apply -f <file_name.yaml>```  for the configuration of the Kubernetes cluster:
    - **aws-secret.yaml**: Contains the credentials to access AWS.  In the file, under credentials, fill in the value by convert the .aws/credentials file to base64 string.
    - **env-secret.yaml**: Contains authenticaion details to the DB:  The Db login user and password are encoded in base64 string. 
    - **env-configmap.yaml**: Contains non-senstivie configuration values, such as AWS_BUCKET, AWS_REGION, and DB information such as POSTGRESS_DB, POSTGRESS_HOST.
  - Apply the following files using ```kubectl apply -f <file_name.yaml>``` for the creation of the Kubernetes pods:
    - **backend-feed-deployment.yaml** - for the deployment of docker image jsleung1/udacity-restapi-feed in the Kubernetes pods.
    - **backend-user-deployment.yaml** - for the deployment of docker image jsleung1/udacity-restapi-user in the Kubernetes pods.
    - **reverseproxy-deployment.yaml** - for the deployment of docker image jsleung1/reverseproxy in the Kubernetes pods.
    - **frontend-deployment.yaml** - for the deployment of docker image jsleung1/udacity-frontend in the Kubernetes pods.
  - Apply the following files using ```kubectl apply -f <file_name.yaml>``` for the creation of services in the Kubernetes cluster:
    - **backend-user-service.yaml** - for the creation of backend-user service in the Kubernetes cluster.
    - **backend-feed-service.yaml** - for the creation of backend-feed service in the Kubernetes cluster.
    - **reverseproxy-service.yaml** - for the creation of reverseproxy service in the Kubernetes cluster.
    - **frontend-service.yaml** - for the creation of frontend service in the Kubernetes cluster.
  - Execute ```kubectl get pods``` and ensure all the pods have **STATUS = running**.
  - At this point, we want to verify the behavior of the Udagram application after succcessfully created the pods in the Kubernetes cluster:
    - Execute ```kubectl port-forward service/reverseproxy 8080:8080```
    - Execute ```kubectl port-forward service/frontend 8100:8100```
    - Verify the behavior of the Udagram by going to http://localhost:8100 in the web browser:
      - User is able to login to Udagram.
      - User is able to view the images of the feed.
      - User is able to upload images in Udagram.

#### 4. Instructions for integrating Travis CI with Github repository

  - Go to https://github.com/jsleung1/udacity_project4 , go to Settings, and under Apps and integrations, select Travis CI and grant Travis CI access to the repository jsleung1/udacity_project4.
  - Go to https://travis-ci.com.  Since you were already logged in to GitHub, Travis CI will display a list of repository from Github including udacity_project4.  Ensure udacity_project4 is activated in Travis CI.
  - In Travis CI, select the repository jsleung1/udacity_project4. Go to more options, settings.
    - In the settings, under Environment Variables, create the following:
    ```
      - DOCKER_USERNAME - Docker hub login username.
      - DOCKER_PASSWORD - Docker hub password.
      - KUBE_CA_CERT - copy the certificate-authority-data defined in the file terraform/aws/udacitykubeone-kubeconfig.
      - KUBE_ENDPOINT - copy the server URL path defined in the file terraform/aws/udacitykubeone-kubeconfig.
      - KUBE_ADMIN_CERT - copy the client-certificate-data defined in the file terraform/aws/udacitykubeone-kubeconfig.
      - KUBE_ADMIN_KEY - copy the client-key-data defined in the file terraform/aws/udacitykubeone-kubeconfig.
    ``` 
  - Create a "bare" kubeconfig file that strips out the above sensitive information related to udacitykubeone-kubeconfig:
    - This file is placed in **terraform/aws/udacitykubeone-kubeconfig-bare-travis** under the main project folder.
  - To make Travis CI automatically build the images from Github, we create the .travis.yml file in the main project folder with the following instructions:
    - Before the build process start, it install docker-compose and kubectl.  Also, it will start Docker and login to my Docker hub account.
    - For the build process, it run docker-compose with the latest source code changes.
    - After the docker images were successfully built, it will push the images to my Docker hub, by TRAVIS_BUILD_ID, and also tag/push the Docker images as latest.
    - Using the environmental variables in Travis settings, it will fill in the stripped udacitykubeone-kubeconfig-bare-travis such that Travis can access our Kubernetes cluster.
    - Travis CI will update the four images in the Kubernetes pod using rolling-update by running ```kubectl set image```. (For details, please refer to the .travis.yml in the project main folder).
  - Run the following command to ensure Kubernetes successfully rollout the new deployment from Travis CI:
    ```
    kubectl rollout status deployment reverseproxy
    kubectl rollout status deployment frontend
    kubectl rollout status deployment backend-feed
    kubectl rollout status deployment backend-user
    ```
  - Also, we can verify a new version is deployed by Kubernetes from Travis CI by looking at the scale up/down of the replica sets using the following command:
    ```
    kubectl describe deployment reverseproxy
    kubectl describe deployment frontend
    kubectl describe deployment backend-feed
    kubectl describe deployment backend-user
    ```
  - At this point, we also want to verify the behavior of the updated Udagram application after succeesfully deployed in Kubernetes cluster using Travis CI:
    - Execute ```kubectl port-forward service/reverseproxy 8080:8080```
    - Execute ```kubectl port-forward service/frontend 8100:8100```
    - Verify the software changes of the Udagram by going to http://localhost:8100 in the web browser:
      - Udagram will appear with different colors (For demostration purpose, I have updated the scss of the ionic client).
      - User is able to login to Udagram.
      - User is able to view the images of the feed.
      - User is able to upload images in Udagram.

#### 5. Instructions for installing FluentD to redirect Kubernetes logs to AWS Cloudwatch
  - The FluentD project is stored in the directory **kube-fluentd-cloudwatch**.  It is based on the Github project: https://github.com/zerda/kube-fluentd-cloudwatch
    - In AWS, ensure our IAM user has the following permissions to access AWS Cloudwatch:
    ```
    {
      "Version": "2012-10-17",
      "Statement": [
          {
              "Effect": "Allow",
              "Action": [
                  "logs:CreateLogGroup",
                  "logs:CreateLogStream",
                  "logs:PutLogEvents",
                  "logs:DescribeLogStreams"
              ],
              "Resource": [
                  "arn:aws:logs:*:*:*"
              ]
          }
      ]
    }
    ```
    - Execute ```kubectl apply -f aws-secret.yaml```: This file is required by fluentd to access our AWS Cloudwatch.  Note the namespace must be set as 'default' (which is the namespace of our Kubernetes cluster).  The aws_access_key_id and aws_secret_access_key are base64 strings.
    - Execute ```kubectl apply -f fluentd-configmap.yaml```:  Ensure the namespace is set to 'default'.  The log_group_name_key is set to "udacity_project4_log_group".  The log_stream_name is set to the actual full name of the pods running in our Kubernetes cluster.  FluentD will automatically create the log group and log stream under our AWS Cloudwatch.
    - Execute ```kubectl apply -f fluentd-ds.yaml```:  This file will create the FluentD pods in our Kubernetes cluster.  The FluentD pods will redirect the logs of our pods (reverseproxy, frontend, backend-feed, backend-user) to the AWS Cloudwatch.
    - Create a Metric filter for log group "udacity_project4_log_group".
    - Create an alarm in AWS CloudWatch which will trigger when the log count of our Kubernetes cluster exceeded 100. 
    - Select the SNS topic group for the alarm such that appropriate users subscribed to SNS will be notified by the alarm.
