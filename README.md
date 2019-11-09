# Udacity Project 4: 
Project 4: Refactor Udagram App into Microservices and Deploy

## Important:
Please refer to the pdf file: **project4_rubrics.pdf** in the parent directory for the project screenshots and meeting the rubrics.

## Prerequisites:
1. Install Docker in your local computer.
2. Register a Docker Hub account in https://hub.docker.com .
3. Ensure Docker service is started in your computer and login to Docker Hub in the Terminal.
4. Recommended to create an AWS IAM account with installation privileges (DO NOT use the AWS ROOT account ).
5. With the AWS IAM account, ensure the AWS_ACCESS_KEY_ID and AWS_SECRET_ACCESS_KEY values can be found in the file ~/.aws/credentials, 
   which is needed for the docker image jsleung1/udacity-restapi-feed to run properly when issue AWS request to getSignedUrl.
   Also export the AWS_ACCESS_KEY_ID and AWS_SECRET_ACCESS_KEY values in .bash_profile, which is needed by the terminal
   when execute Terraform commands to create the infrastructure and KubeOne to install Kubernetes cluster 
   and create worker nodes.
6. Ensure a new SSH key is generated and add it to the ssh-agent before running *terraform plan* .
   - *execute:* ssh-keygen -t rsa -b 4096 -C "your_email@example.com"
   - *execute:* ssh-add ~/.ssh/id_rsa

## Setup Instructions:

#### 1. Build the docker images and ensure the images are running properly in the local system before installing Kubernetes.

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
  2. Build the individual docker images:

    - cd udacity-c3-restapi-user/
    - *execute:* docker build -t jsleung1/udacity-restapi-user .

    - *cd* udacity-c3-restapi-feed/
    - *execute:* docker build -t jsleung1/udacity-restapi-feed .

    - *cd* udacity-c3-frontend/
    - *execute:* docker build -t jsleung1/udacity-frontend .

    - cd udacity-c3-deployment/docker
    - *execute:* docker build -t jsleung1/reverseproxy .
  
  3. Run the docker containers and verify the application behavior:

    - cd udacity-c3-deployment/docker
    - *execute:* docker-compose up

    This will run the above four docker images as defined in udacity-c3-deployment/docker/docker-compose.yaml .
    From the terminal, ensure the four docker images are started without any errors.
    Verify the behavior of the Udagram:
    
    - User is able to login to Udagram.
    - User is able to view the images of the feed.
    - User is able to upload images in Udagram.


  
  



2. Instructions for installation of Kubernetes cluster ("udacitykubeone"):

  Follow the installation steps in https://github.com/kubermatic/kubeone/blob/master/docs/quickstart-aws.md .

    -
    - Created terraform.tfvars (refer to source: terraform/aws/terraform.tfvars)
      - Specify the correct AWS region ("us-east-1")
      - Specify the name of my Kubernetes cluster ("udacitykubeone")
      - ssh_public_key_file: point to the new ssh key ("~/.ssh/id_rsa.pub")
    - execute: kubeone install config.yaml -t tf.json
    - After the Kubernetes cluster is created, in .bash_profile, export KUBECONFIG=/home/vagrant/udacity/udacity_project4/terraform/aws/udacitykubeone-kubeconfig . 
      This is required so kubectl will automatically refer to this config when execute any commands related to kubectl. 
    - The source related to creation of Kubernetes cluster can be found in the source folder terraform/aws :
      - terraform.tfvars: for Terraform to create the infrastructure.
      - config.yaml: Install Kubernetes and create the Kubernetes cluster.
      - udacitykubeone-kubeconfig: the full config including the sensitive data such as certificate authority data of my Kubernetes cluster ("udacitykubeone").
      - udacitykubeone-kubeconfig-bare-travis: the config of my Kubernetes cluster excluding the sensitive data 
        for Travis CI to update the Kubernetes pods after the image is successfully built.  The sensitive data are replaced by Travis environment variables whose values are defined in Travis Settings.

2. Instructions for creation of Kubernetes pods:
    
