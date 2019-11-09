# Udacity Project 4: 
Project 4: Refactor Udagram App into Microservices and Deploy

Important:
For the project screenshots and rubrics, please refer to the file: project4_refactor_microservices_and_deploy.pdf in the parent directory of the source.

Prerequisites:
1. Install Docker in your local computer.
2. Register a Docker Hub account in https://hub.docker.com .
3. Ensure Docker service is started in your computer and login to Docker Hub in the Terminal.
4. Ensure all the Docker images are built successful and runn

Setup Instructions:

1. Instructions for installation of Kubernetes cluster ("udacitykubeone"):

   Follow the installation steps for Terraform and Kubectl in https://github.com/kubermatic/kubeone/blob/master/docs/quickstart-aws.md .
    - Recommended to create an AWS IAM account with appropriate privileges (should NOT use the AWS ROOT account ).
    - With the AWS IAM account, export the AWS_ACCESS_KEY_ID and AWS_SECRET_ACCESS_KEY values ( can be found in the file ~/.aws/credentials )
      in .bash_profile.  The AWS key id and secret are required for Terraform to create the infrastructure and KubeOne to install Kubernetes and create worker nodes.
    - Ensure a new SSH key is generated and add it to the ssh-agent before running terraform plan .
      - execute: ssh-keygen -t rsa -b 4096 -C "your_email@example.com"
      - execute: ssh-add ~/.ssh/id_rsa
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
    
