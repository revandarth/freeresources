# Comprehensive DevOps Guide for Managing a Java API Service

This guide provides an in-depth, step-by-step process for setting up and managing a production-ready Java API service using modern DevOps practices and tools. It is based on the sample Java application from https://github.com/buddy-works/simple-java-project.

## Table of Contents

1. [Prerequisites](#prerequisites)
2. [Version Control with GitHub](#1.-version-control-with-github)
3. [Continuous Integration and Deployment with Jenkins](#2.-continuous-integration-and-deployment-with-jenkins)
4. [Configuration Management with Ansible](#configuration-management-with-ansible)
5. [Infrastructure as Code with Terraform](#infrastructure-as-code-with-terraform)
6. [AWS Cloud Setup](#aws-cloud-setup)
7. [Load Balancing with NLB and Nginx](#load-balancing-with-nlb-and-nginx)
8. [Monitoring with Prometheus and Grafana](#monitoring-with-prometheus-and-grafana)
9. [Logging with ELK Stack](#logging-with-elk-stack)
10. [Alerting and Communication with Slack](#alerting-and-communication-with-slack)
11. [Security Considerations](#security-considerations)
12. [Backup and Disaster Recovery](#backup-and-disaster-recovery)
13. [Maintenance and Upgrades](#maintenance-and-upgrades)

## Prerequisites

Before starting, ensure you have:

- Java Development Kit (JDK) 11 or later installed
- Maven 3.6 or later installed
- Git installed
- An AWS account with appropriate permissions
- A GitHub account
- A Slack workspace for your team
- Basic familiarity with Java, RESTful APIs, and the Linux command line

## 1. Version Control with GitHub

Version control is a crucial aspect of DevOps and software development. It allows teams to collaborate effectively, track changes, manage releases, and maintain code quality. GitHub, a popular platform for version control, offers powerful features to support these practices.

### 1.1. Setting Up a GitHub Repository

1. Create a new repository:
   - Go to GitHub and click the '+' icon, then 'New repository'
   - Name your repository (e.g., "java-api-service")
   - Choose 'Public' or 'Private' based on your needs
   - Initialize with a README, .gitignore (choose Java), and license (e.g., MIT)

2. Clone the repository locally:
   ```
   git clone https://github.com/your-username/java-api-service.git
   cd java-api-service
   ```

   Why this is important: Cloning creates a local copy of the repository, allowing you to work on your code offline and sync changes later.

### 1.2 Branching Strategy

Implement GitFlow or a similar branching strategy. GitFlow uses two main branches with infinite lifetime:

- `main` (or `master`): Always reflects production-ready state
- `develop`: Integration branch for features

And several supporting branches:

- Feature branches
- Release branches
- Hotfix branches

Here's how to implement this:

1. Create the develop branch:
   ```
   git checkout -b develop
   git push -u origin develop
   ```

2. For a new feature:
   ```
   git checkout -b feature/new-feature develop
   # Work on your feature
   git push -u origin feature/new-feature
   ```

3. For a release:
   ```
   git checkout -b release/1.0.0 develop
   # Make release-specific changes if any
   git checkout main
   git merge --no-ff release/1.0.0
   git tag -a 1.0.0 -m "Version 1.0.0"
   git checkout develop
   git merge --no-ff release/1.0.0
   git branch -d release/1.0.0
   ```

4. For a hotfix:
   ```
   git checkout -b hotfix/critical-bug main
   # Fix the bug
   git checkout main
   git merge --no-ff hotfix/critical-bug
   git tag -a 1.0.1 -m "Version 1.0.1"
   git checkout develop
   git merge --no-ff hotfix/critical-bug
   git branch -d hotfix/critical-bug
   ```

Why this is important: A well-defined branching strategy helps manage concurrent development, isolates new features, facilitates releases, and allows for quick fixes to production issues.

### 1.3 Pull Requests

Pull Requests (PRs) are a way to propose changes to a repository. They're crucial for code review and maintaining code quality.

1. Create a Pull Request:
   - Push your feature branch to GitHub
   - Go to your repository on GitHub
   - Click 'Pull requests' > 'New pull request'
   - Select your feature branch as 'compare' and 'develop' as 'base'
   - Fill in the PR template with a description of your changes
   - Click 'Create pull request'

2. Review process:
   - Assign reviewers to your PR
   - Reviewers comment on code, suggest changes
   - Make necessary changes and push to update the PR
   - Once approved, merge the PR

Why this is important: PRs provide a platform for code review, ensuring code quality, knowledge sharing, and catching potential issues before they reach the main codebase.

### 1.4 Branch Protection Rules

Branch protection rules in GitHub help enforce certain workflows and keep important branches stable.

1. Set up branch protection:
   - Go to your repository on GitHub
   - Click 'Settings' > 'Branches'
   - Under 'Branch protection rules', click 'Add rule'
   - Apply the rule to 'main' and 'develop' branches

2. Configure protection settings:
   - Require pull request reviews before merging
   - Require status checks to pass before merging
   - Require branches to be up to date before merging
   - Include administrators in these restrictions

Example configuration:
- Required approving reviews: 2
- Require status checks: 
  - Require branches to be up to date before merging
  - Status checks: CI build, unit tests
- Restrict who can push to matching branches: Selected users/teams

Why this is important: Branch protection ensures that changes to critical branches go through proper review and testing, maintaining code quality and stability.

### 1.5 Commit Best Practices

1. Make atomic commits (one logical change per commit)
2. Write meaningful commit messages:
   ```
   Short (50 chars or less) summary of changes

   More detailed explanatory text, if necessary. Wrap it to about 72
   characters or so. In some contexts, the first line is treated as the
   subject of an email and the rest of the text as the body. The blank
   line separating the summary from the body is critical (unless you omit
   the body entirely); tools like rebase can get confused if you run the
   two together.

   Further paragraphs come after blank lines.

   - Bullet points are okay, too
   - Typically a hyphen or asterisk is used for the bullet, preceded by a
     single space, with blank lines in between, but conventions vary here
   ```
3. Use present tense ("Add feature" not "Added feature")
4. Reference issue numbers in commit messages when applicable

Why this is important: Good commit practices make it easier to understand the history of a project, revert changes if necessary, and maintain a clean, understandable codebase.


### 1.6 GitHub Issues and Project Boards

1. Use GitHub Issues to track tasks, enhancements, and bugs
2. Create labels for easy categorization (e.g., "bug", "enhancement", "documentation")
3. Use Project Boards to visualize and manage work:
   - Create a new project (Kanban-style board)
   - Add columns: To Do, In Progress, Review, Done
   - Create issues and add them to the board

Why this is important: Proper issue tracking and project management help teams stay organized, prioritize work, and maintain transparency in the development process.

Effective version control is fundamental to DevOps practices. It facilitates collaboration, ensures code quality, and provides a stable foundation for continuous integration and deployment. By following these GitHub best practices, you'll be well-equipped to manage your Java API service's codebase effectively. Remember, these practices may need to be adapted based on your team's specific needs and workflows. Regular review and refinement of your version control processes is key to maintaining an efficient DevOps pipeline.

## 2. Continuous Integration and Deployment with Jenkins

Continuous Integration and Continuous Deployment (CI/CD) are crucial practices in DevOps that automate the process of integrating code changes, running tests, and deploying applications. Jenkins is a popular open-source automation server that helps implement these practices.

### 2.1. Understanding CI/CD

Before diving into Jenkins, let's understand what CI/CD means:

- Continuous Integration (CI): The practice of frequently merging code changes into a central repository, after which automated builds and tests are run.
- Continuous Deployment (CD): The practice of automatically deploying all code changes to a testing or production environment after the build stage.

Why it's important: CI/CD practices help detect errors quickly, reduce the time to validate and release new software updates, and improve overall software quality and developer productivity.

### 2.2. Setting Up Jenkins

1. Install Jenkins:
   - On Ubuntu:
     ```
     wget -q -O - https://pkg.jenkins.io/debian-stable/jenkins.io.key | sudo apt-key add -
     sudo sh -c 'echo deb https://pkg.jenkins.io/debian-stable binary/ > /etc/apt/sources.list.d/jenkins.list'
     sudo apt-get update
     sudo apt-get install jenkins
     ```
   - On other systems, follow the [official Jenkins installation guide](https://www.jenkins.io/doc/book/installing/)

2. Access Jenkins:
   - Open a web browser and navigate to `http://localhost:8080` (or your server's IP address)
   - Follow the initial setup wizard to unlock Jenkins and install suggested plugins

3. Install necessary plugins:
   - Go to "Manage Jenkins" > "Manage Plugins"
   - Install: Git plugin, Pipeline plugin, Blue Ocean (for a modern UI)

Why it's important: A properly set up Jenkins server is the foundation for implementing CI/CD in your project.

### 2.3. Creating a Basic Jenkins Pipeline

A Jenkins Pipeline is a suite of plugins that supports implementing and integrating continuous delivery pipelines into Jenkins.

1. Create a new Pipeline job:
   - Click "New Item" on the Jenkins dashboard
   - Enter a name for your job and select "Pipeline"
   - Click "OK"

2. Configure the Pipeline:
   - In the job configuration page, scroll down to the "Pipeline" section
   - Choose "Pipeline script" for now (we'll use "Pipeline script from SCM" later)

3. Write a basic Pipeline script:

```groovy
pipeline {
    agent any
    
    stages {
        stage('Build') {
            steps {
                echo 'Building the application...'
                sh 'mvn clean package'
            }
        }
        stage('Test') {
            steps {
                echo 'Running tests...'
                sh 'mvn test'
            }
        }
        stage('Deploy') {
            steps {
                echo 'Deploying the application...'
                // Add deployment steps here
            }
        }
    }
    post {
        always {
            echo 'This will always run'
        }
        success {
            echo 'This will run only if successful'
        }
        failure {
            echo 'This will run only if failed'
        }
    }
}
```

4. Save the configuration and run the Pipeline

Why it's important: This basic Pipeline defines the entire build/test/deploy process as code, making it version-controlled, reviewable, and iterative.

### 2.4. Jenkinsfile: Pipeline as Code

Instead of defining the Pipeline in the Jenkins UI, it's better to have it as part of your project's source code.

1. Create a `Jenkinsfile` in the root of your project repository:

```groovy
pipeline {
    agent any
    
    tools {
        maven 'Maven 3.6.3'
        jdk 'JDK 11'
    }
    
    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }
        
        stage('Build') {
            steps {
                sh 'mvn clean package'
            }
        }
        
        stage('Test') {
            steps {
                sh 'mvn test'
            }
            post {
                always {
                    junit '**/target/surefire-reports/TEST-*.xml'
                }
            }
        }
        
        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('SonarQube') {
                    sh 'mvn sonar:sonar'
                }
            }
        }
        
        stage('Deploy to Staging') {
            when {
                branch 'develop'
            }
            steps {
                // Add steps to deploy to staging
                sh 'echo "Deploying to Staging"'
            }
        }
        
        stage('Deploy to Production') {
            when {
                branch 'main'
            }
            steps {
                // Add steps to deploy to production
                sh 'echo "Deploying to Production"'
            }
        }
    }
    
    post {
        always {
            cleanWs()
        }
        success {
            slackSend channel: '#devops-notifications', color: 'good', message: "Success: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]' (${env.BUILD_URL})"
        }
        failure {
            slackSend channel: '#devops-notifications', color: 'danger', message: "Failed: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]' (${env.BUILD_URL})"
        }
    }
}
```

2. Update your Jenkins job:
   - In the job configuration, change "Pipeline" to "Pipeline script from SCM"
   - Set SCM to Git and provide your repository URL
   - Set the Script Path to "Jenkinsfile"

Why it's important: Storing the Jenkinsfile in the repository allows you to version control your Pipeline, review it with code reviews, and iterate on it along with your application code.

### 2.5. Multi-Branch Pipelines

Multi-branch Pipelines automatically create a Pipeline for each branch in your repository that contains a Jenkinsfile.

1. Create a new Multi-branch Pipeline:
   - Click "New Item" on the Jenkins dashboard
   - Enter a name and select "Multibranch Pipeline"
   - Click "OK"

2. Configure the Multi-branch Pipeline:
   - In "Branch Sources", add your Git repository
   - In "Build Configuration", set the Script Path to "Jenkinsfile"
   - Under "Scan Multibranch Pipeline Triggers", check "Periodically if not otherwise run" and set an interval (e.g., 1 hour)

3. Save the configuration and let Jenkins scan the repository

Why it's important: Multi-branch Pipelines automatically manage Pipelines for all your branches, making it easier to implement CI/CD for feature branches, release branches, and your main branch.

### 2.6. Best Practices for Jenkins Pipelines

1. Keep Pipelines short and simple
2. Use Shared Libraries for common functionality
3. Parallelize stages when possible to speed up the Pipeline
4. Use input steps for manual approvals when necessary
5. Implement proper error handling and notifications
6. Use environment variables for configuration
7. Regularly backup Jenkins configuration

Example of a more advanced Pipeline using some of these practices:

```groovy
@Library('my-shared-library') _

pipeline {
    agent any
    
    environment {
        APP_NAME = 'my-java-app'
        DEPLOY_TO = 'production'
    }
    
    stages {
        stage('Build and Test') {
            parallel {
                stage('Build') {
                    steps {
                        script {
                            buildApp()
                        }
                    }
                }
                stage('Static Code Analysis') {
                    steps {
                        script {
                            runSonarQube()
                        }
                    }
                }
            }
        }
        
        stage('Deploy') {
            when {
                expression { DEPLOY_TO ==~ /(staging|production)/ }
            }
            steps {
                script {
                    deployTo(env.DEPLOY_TO)
                }
            }
        }
    }
    
    post {
        always {
            cleanWs()
        }
        success {
            notifySlack("Success")
        }
        failure {
            notifySlack("Failure")
        }
    }
}
```

Why it's important: Following best practices ensures your Pipelines are maintainable, efficient, and robust.

Jenkins Pipelines and Multi-branch Pipelines are powerful tools for implementing CI/CD in your development workflow. They allow you to define your entire build/test/deploy process as code, making it version-controlled and easily repeatable. By mastering these concepts, you'll be well on your way to implementing effective DevOps practices in your projects.

Remember, CI/CD is not just about tools, but also about culture. 

## 3. Configuration Management with Ansible

Configuration Management is a crucial aspect of DevOps that involves maintaining systems in a desired state. Ansible is an open-source automation tool that provides a simple yet powerful framework for configuration management, application deployment, and task automation.

### 3.1. Understanding Configuration Management

Configuration Management involves:
- Maintaining consistent system configurations
- Automating repetitive tasks
- Ensuring systems are in a known, desired state
- Facilitating scalability and reproducibility

Why it's important: Configuration Management reduces manual errors, increases efficiency, and ensures consistency across your infrastructure.

### 3.2. Ansible Basics

Ansible is agentless, meaning it doesn't require any software to be installed on the managed nodes. It uses SSH for communication and can manage Unix-like systems as well as Windows.

Key Ansible concepts:
- Control Node: The machine where Ansible is installed and from which management tasks are run
- Managed Nodes: The systems being managed by Ansible
- Inventory: A list of managed nodes
- Modules: Units of code Ansible executes
- Tasks: Units of action in Ansible
- Playbooks: Ordered lists of tasks, written in YAML

### 3.3. Setting Up Ansible

1. Install Ansible on the control node:
   - On Ubuntu or Debian:
     ```
     sudo apt update
     sudo apt install ansible
     ```
   - On CentOS or RHEL:
     ```
     sudo yum install epel-release
     sudo yum install ansible
     ```
   - On macOS (using Homebrew):
     ```
     brew install ansible
     ```

2. Verify the installation:
   ```
   ansible --version
   ```

3. Configure SSH key-based authentication:
   ```
   ssh-keygen -t rsa -b 4096
   ssh-copy-id user@target-host
   ```

Why it's important: Proper setup ensures Ansible can communicate securely with managed nodes and execute tasks effectively.

### 3.4. Creating Your First Ansible Inventory

The inventory file defines the hosts and groups of hosts upon which commands, modules, and tasks in a playbook operate.

1. Create a new directory for your Ansible project:
   ```
   mkdir ansible-example
   cd ansible-example
   ```

2. Create an inventory file named `inventory.ini`:
   ```ini
   [webservers]
   web1 ansible_host=192.168.1.10
   web2 ansible_host=192.168.1.11

   [databases]
   db1 ansible_host=192.168.1.20

   [all:vars]
   ansible_user=your_ssh_user
   ```

3. Test your inventory:
   ```
   ansible all -i inventory.ini -m ping
   ```

Why it's important: The inventory file is crucial for organizing your infrastructure and defining which hosts Ansible should manage.

### 3.5. Writing Your First Ansible Playbook

Playbooks are Ansible's configuration, deployment, and orchestration language. They can describe a policy you want your remote systems to enforce, or a set of steps in a general IT process.

1. Create a playbook file named `webserver.yml`:

```yaml
---
- name: Configure webserver with nginx
  hosts: webservers
  become: yes
  tasks:
    - name: Install nginx
      apt:
        name: nginx
        state: present

    - name: Copy nginx config file
      copy:
        src: files/nginx.conf
        dest: /etc/nginx/sites-available/default

    - name: Enable configuration
      file:
        dest: /etc/nginx/sites-enabled/default
        src: /etc/nginx/sites-available/default
        state: link

    - name: Copy index.html
      template:
        src: templates/index.html.j2
        dest: /var/www/html/index.html

    - name: Restart nginx
      service:
        name: nginx
        state: restarted
```

2. Create the necessary files:
   ```
   mkdir -p files templates
   touch files/nginx.conf templates/index.html.j2
   ```

3. Add content to `files/nginx.conf`:
   ```nginx
   server {
       listen 80 default_server;
       listen [::]:80 default_server;
       root /var/www/html;
       index index.html index.htm index.nginx-debian.html;
       server_name _;
       location / {
           try_files $uri $uri/ =404;
       }
   }
   ```

4. Add content to `templates/index.html.j2`:
   ```html
   <html>
   <head>
       <title>Welcome to {{ ansible_hostname }}</title>
   </head>
   <body>
       <h1>Hello, World!</h1>
       <p>This is {{ ansible_hostname }}.</p>
   </body>
   </html>
   ```

5. Run the playbook:
   ```
   ansible-playbook -i inventory.ini webserver.yml
   ```

Why it's important: Playbooks allow you to define and execute complex configuration management tasks in a repeatable, scalable manner.

### 3.6. Ansible Roles

Roles in Ansible provide a way to organize playbooks and all their related files in a standardized directory structure.

1. Create a role structure:
   ```
   ansible-galaxy init webserver
   ```

2. This creates the following directory structure:
   ```
   webserver/
   ├── defaults
   │   └── main.yml
   ├── files
   ├── handlers
   │   └── main.yml
   ├── meta
   │   └── main.yml
   ├── tasks
   │   └── main.yml
   ├── templates
   ├── tests
   │   ├── inventory
   │   └── test.yml
   └── vars
       └── main.yml
   ```

3. Move your tasks into `webserver/tasks/main.yml`

4. Create a new playbook `site.yml` to use the role:

```yaml
---
- hosts: webservers
  become: yes
  roles:
    - webserver
```

5. Run the playbook:
   ```
   ansible-playbook -i inventory.ini site.yml
   ```

Why it's important: Roles promote code reuse and provide a way to share and organize complex playbooks.

### 3.7. Ansible Variables

Variables in Ansible can be defined at various levels and used to make your playbooks more flexible and reusable.

1. Define variables in your inventory file:
   ```ini
   [webservers]
   web1 ansible_host=192.168.1.10 http_port=8080
   web2 ansible_host=192.168.1.11 http_port=8081
   ```

2. Use variables in your playbook:

```yaml
---
- hosts: webservers
  become: yes
  tasks:
    - name: Set up nginx
      template:
        src: templates/nginx.conf.j2
        dest: /etc/nginx/sites-available/default
      vars:
        listen_port: "{{ http_port }}"
```

3. Create `templates/nginx.conf.j2`:
   ```nginx
   server {
       listen {{ listen_port }} default_server;
       # ... rest of your nginx config
   }
   ```

Why it's important: Variables make your Ansible code more dynamic and adaptable to different environments and requirements.

### 3.8. Ansible Vault

Ansible Vault is used to encrypt sensitive data like passwords or API keys.

1. Create an encrypted file:
   ```
   ansible-vault create secrets.yml
   ```

2. Edit an encrypted file:
   ```
   ansible-vault edit secrets.yml
   ```

3. Use encrypted file in a playbook:
   ```yaml
   ---
   - hosts: webservers
     vars_files:
       - secrets.yml
     tasks:
       - name: Use secret
         debug:
           msg: "The secret is {{ secret_variable }}"
   ```

4. Run playbook with vault:
   ```
   ansible-playbook site.yml --ask-vault-pass
   ```

Why it's important: Vault allows you to securely manage sensitive data within your Ansible projects.

### 3.9. Best Practices

1. Use version control for your Ansible code
2. Keep your playbooks simple and focused
3. Use roles to organize and reuse code
4. Use variables to make your playbooks flexible
5. Use Ansible Vault to secure sensitive data
6. Use tags to selectively run parts of your playbooks
7. Use `ansible-lint` to check your playbooks for best practices
8. Regularly update Ansible and your roles

### 3.10. Advanced Topics

1. Dynamic Inventories: Use scripts to generate inventories dynamically
2. Ansible Tower/AWX: Web-based solution for managing Ansible
3. Ansible Callbacks: Extend Ansible's output and logging capabilities
4. Custom Modules: Write your own Ansible modules when built-in ones aren't sufficient


Ansible is a powerful tool for configuration management that can significantly improve your DevOps workflows. By mastering Ansible, you can automate complex tasks, ensure consistency across your infrastructure, and collaborate more effectively with your team. Remember that while Ansible provides the tools, effective configuration management also requires good practices, clear communication, and a solid understanding of your infrastructure needs.


## 4. Infrastructure as Code with Terraform

Infrastructure as Code (IaC) is a key DevOps practice where infrastructure is provisioned and managed using code and software development techniques. Terraform is a popular open-source IaC tool that allows you to define and provide data center infrastructure using a declarative configuration language.

### 1. Understanding Infrastructure as Code

IaC treats infrastructure configuration files as software code. This approach brings several benefits:

- Version Control: Track changes to your infrastructure over time.
- Consistency: Ensure consistent environments across development, testing, and production.
- Automation: Easily replicate and scale infrastructure.
- Documentation: The code itself serves as documentation of your infrastructure.

Why it's important: IaC reduces manual errors, increases deployment speed, and allows for better collaboration among team members.

### 2. Terraform Basics

Terraform uses its own domain-specific language (DSL) called HashiCorp Configuration Language (HCL). Key concepts include:

- Providers: Plugins for interacting with cloud providers, SaaS providers, and other APIs.
- Resources: Elements of your infrastructure (e.g., virtual machines, DNS records).
- Data Sources: Query external data for use in your configuration.
- Variables: Parameterize your configurations.
- Outputs: Return values from your infrastructure.
- State: Terraform's representation of your real-world resources.

### 3. Setting Up Terraform

1. Install Terraform:
   - On macOS with Homebrew: `brew install terraform`
   - On Windows with Chocolatey: `choco install terraform`
   - On Linux, download the binary and add it to your PATH:
     ```
     wget https://releases.hashicorp.com/terraform/0.15.5/terraform_0.15.5_linux_amd64.zip
     unzip terraform_0.15.5_linux_amd64.zip
     sudo mv terraform /usr/local/bin/
     ```

2. Verify the installation:
   ```
   terraform version
   ```

3. Set up AWS credentials (assuming we're using AWS):
   - Install AWS CLI
   - Run `aws configure` and provide your AWS Access Key ID and Secret Access Key

Why it's important: Proper setup ensures Terraform can communicate with your cloud provider and manage resources effectively.

### 4. Creating Your First Terraform Configuration

Let's create a basic configuration to launch an EC2 instance:

1. Create a new directory for your Terraform project:
   ```
   mkdir terraform-ec2-example
   cd terraform-ec2-example
   ```

2. Create a file named `main.tf`:

```hcl
# Configure the AWS Provider
provider "aws" {
  region = "us-west-2"
}

# Create a VPC
resource "aws_vpc" "example" {
  cidr_block = "10.0.0.0/16"
  
  tags = {
    Name = "example-vpc"
  }
}

# Create a subnet
resource "aws_subnet" "example" {
  vpc_id     = aws_vpc.example.id
  cidr_block = "10.0.1.0/24"
  
  tags = {
    Name = "example-subnet"
  }
}

# Create a security group
resource "aws_security_group" "example" {
  name        = "example"
  description = "Allow inbound traffic"
  vpc_id      = aws_vpc.example.id

  ingress {
    description = "SSH from VPC"
    from_port   = 22
    to_port     = 22
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }

  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }

  tags = {
    Name = "example-sg"
  }
}

# Create an EC2 instance
resource "aws_instance" "example" {
  ami           = "ami-0c55b159cbfafe1f0"  # Amazon Linux 2 AMI (HVM), SSD Volume Type
  instance_type = "t2.micro"
  subnet_id     = aws_subnet.example.id
  
  vpc_security_group_ids = [aws_security_group.example.id]
  
  tags = {
    Name = "example-instance"
  }
}
```

3. Initialize Terraform:
   ```
   terraform init
   ```

4. Plan your changes:
   ```
   terraform plan
   ```

5. Apply your changes:
   ```
   terraform apply
   ```

Why it's important: This basic configuration demonstrates how to define infrastructure as code, showing the declarative nature of Terraform and how different resources relate to each other.

### 5. Terraform State

Terraform uses a state file to keep track of the current state of your infrastructure. By default, this is stored locally in a file named `terraform.tfstate`.

1. Never manually edit the state file.
2. For team environments, use remote state storage (e.g., S3 with DynamoDB for locking).

To set up remote state with S3 and DynamoDB:

1. Create an S3 bucket and DynamoDB table:

```hcl
resource "aws_s3_bucket" "terraform_state" {
  bucket = "my-terraform-state-bucket"
  
  versioning {
    enabled = true
  }
  
  server_side_encryption_configuration {
    rule {
      apply_server_side_encryption_by_default {
        sse_algorithm = "AES256"
      }
    }
  }
}

resource "aws_dynamodb_table" "terraform_locks" {
  name         = "terraform-up-and-running-locks"
  billing_mode = "PAY_PER_REQUEST"
  hash_key     = "LockID"
  
  attribute {
    name = "LockID"
    type = "S"
  }
}
```

2. Configure the backend in your Terraform configuration:

```hcl
terraform {
  backend "s3" {
    bucket         = "my-terraform-state-bucket"
    key            = "global/s3/terraform.tfstate"
    region         = "us-west-2"
    dynamodb_table = "terraform-up-and-running-locks"
    encrypt        = true
  }
}
```

Why it's important: Proper state management is crucial for team collaboration and maintaining an accurate representation of your infrastructure.

### 6. Terraform Modules

Modules are containers for multiple resources that are used together. They allow you to create reusable components and organize your code.

1. Create a directory structure for your module:

```
modules/
  webserver/
    main.tf
    variables.tf
    outputs.tf
```

2. Define your module (modules/webserver/main.tf):

```hcl
resource "aws_instance" "web" {
  ami           = var.ami_id
  instance_type = var.instance_type
  
  tags = {
    Name = "WebServer"
  }
}

resource "aws_eip" "web" {
  instance = aws_instance.web.id
  vpc      = true
}
```

3. Define variables (modules/webserver/variables.tf):

```hcl
variable "ami_id" {
  description = "The AMI to use for the web server"
  type        = string
}

variable "instance_type" {
  description = "The type of instance to start"
  type        = string
  default     = "t2.micro"
}
```

4. Define outputs (modules/webserver/outputs.tf):

```hcl
output "public_ip" {
  value       = aws_eip.web.public_ip
  description = "The public IP of the web server"
}
```

5. Use the module in your main configuration:

```hcl
module "webserver" {
  source        = "./modules/webserver"
  ami_id        = "ami-0c55b159cbfafe1f0"
  instance_type = "t2.small"
}

output "web_public_ip" {
  value = module.webserver.public_ip
}
```

Why it's important: Modules promote code reuse, simplify your main configuration, and allow for better organization of your infrastructure code.

### 7. Terraform Workspaces

Workspaces allow you to manage multiple distinct sets of infrastructure resources from the same working directory.

1. Create and switch to a new workspace:
   ```
   terraform workspace new development
   ```

2. List available workspaces:
   ```
   terraform workspace list
   ```

3. Switch between workspaces:
   ```
   terraform workspace select production
   ```

4. Use workspace name in your configuration:

```hcl
resource "aws_instance" "example" {
  ami           = "ami-0c55b159cbfafe1f0"
  instance_type = terraform.workspace == "production" ? "t2.medium" : "t2.micro"
  
  tags = {
    Environment = terraform.workspace
  }
}
```

Why it's important: Workspaces allow you to manage multiple environments (e.g., development, staging, production) using the same Terraform configuration.

### 8. Best Practices

1. Use consistent formatting (run `terraform fmt` before committing).
2. Use variables to make your configurations flexible and reusable.
3. Use data sources to fetch dynamic values.
4. Use remote state storage and state locking for team environments.
5. Use modules to organize and reuse your code.
6. Version control your Terraform configurations.
7. Use Terraform workspaces or file structure to manage multiple environments.
8. Implement a CI/CD pipeline for your infrastructure code.

### 9. Advanced Topics

1. Terraform Import: Import existing infrastructure into Terraform state.
2. Provisioners: Execute scripts on local or remote machines as part of resource creation or destruction.
3. Terraform Cloud: Terraform's managed service for state storage, version control integration, and more.
4. Terragrunt: A thin wrapper for Terraform that provides extra tools for working with multiple Terraform modules.



## 5. AWS Cloud Setup

1. Set up an AWS account and create an IAM user for Terraform:
   - Go to AWS IAM Console
   - Create a new user with programmatic access
   - Attach the `AdministratorAccess` policy (Note: In production, use more restrictive policies)

2. Configure AWS CLI with your credentials:
   ```
   aws configure
   ```
   Enter your AWS Access Key ID, Secret Access Key, default region, and output format.

3. Use Terraform to provision your infrastructure as described in the previous section.

4. Set up AWS Systems Manager for secure parameter storage:
   - Go to AWS Systems Manager Console
   - Navigate to Parameter Store
   - Create parameters for sensitive information (e.g., database passwords)
   ```
   aws ssm put-parameter --name "/myapp/staging/db_password" --value "your-secure-password" --type SecureString
   ```

For more information on AWS, refer to the [AWS documentation](https://docs.aws.amazon.com/).

## 6. Load Balancing with NLB and Nginx

1. Use Terraform to create a Network Load Balancer (NLB):

   Add the following to your Terraform configuration:
   ```hcl
   resource "aws_lb" "nlb" {
     name               = "my-nlb"
     internal           = false
     load_balancer_type = "network"
     subnets            = module.vpc.public_subnet_ids

     enable_deletion_protection = false

     tags = {
       Environment = "staging"
     }
   }

   resource "aws_lb_target_group" "http" {
     name     = "http-tg"
     port     = 80
     protocol = "TCP"
     vpc_id   = module.vpc.vpc_id
   }

   resource "aws_lb_listener" "http" {
     load_balancer_arn = aws_lb.nlb.arn
     port              = "80"
     protocol          = "TCP"

     default_action {
       type             = "forward"
       target_group_arn = aws_lb_target_group.http.arn
     }
   }
   ```

2. Configure Nginx as a reverse proxy on your EC2 instances:

   Create an Nginx configuration file (roles/nginx/templates/nginx.conf.j2):
   ```nginx
   server {
       listen 80;
       server_name _;

       location / {
           proxy_pass http://localhost:8080;
           proxy_set_header Host $host;
           proxy_set_header X-Real-IP $remote_addr;
           proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
           proxy_set_header X-Forwarded-Proto $scheme;
       }
   }
   ```

   Update your Ansible nginx role to use this template.

3. Implement health checks for your application:

   Add a health check endpoint to your Java application:
   ```java
   @RestController
   public class HealthController {
       @GetMapping("/health")
       public ResponseEntity<String> health() {
           return ResponseEntity.ok("OK");
       }
   }
   ```

   Update the Terraform configuration to use this health check:
   ```hcl
   resource "aws_lb_target_group" "http" {
     # ...
     health_check {
       path                = "/health"
       protocol            = "HTTP"
       interval            = 30
       timeout             = 10
       healthy_threshold   = 2
       unhealthy_threshold = 2
     }
   }
   ```

4. Set up SSL termination at the Nginx level:

   Update your Nginx configuration to include SSL:
   ```nginx
   server {
       listen 80;
       server_name yourdomain.com;
       return 301 https://$server_name$request_uri;
   }

   server {
       listen 443 ssl;
       server_name yourdomain.com;

       ssl_certificate /etc/letsencrypt/live/yourdomain.com/fullchain.pem;
       ssl_certificate_key /etc/letsencrypt/live/yourdomain.com/privkey.pem;

       location / {
           proxy_pass http://localhost:8080;
           proxy_set_header Host $host;
           proxy_set_header X-Real-IP $remote_addr;
           proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
           proxy_set_header X-Forwarded-Proto $scheme;
       }
   }
   ```

   Use Certbot to obtain and renew SSL certificates:
   ```
   sudo apt-get update
   sudo apt-get install certbot python3-certbot-nginx
   sudo certbot --nginx -d yourdomain.com
   ```

For more information on AWS NLB, refer to the [Network Load Balancer documentation](https://docs.aws.amazon.com/elasticloadbalancing/latest/network/introduction.html).

## 7. Monitoring with Prometheus and Grafana

1. Install Prometheus on a dedicated EC2 instance:

   Create a Terraform configuration for the Prometheus instance:
   ```hcl
   resource "aws_instance" "prometheus" {
     ami           = "ami-0c55b159cbfafe1f0"
     instance_type = "t2.micro"
     subnet_id     = module.vpc.private_subnet_ids[0]
     key_name      = "your-key-pair"

     tags = {
       Name = "Prometheus"
     }
   }
   ```

   Use Ansible to install and configure Prometheus:
   ```yaml
   - name: Install Prometheus
     apt:
       name: prometheus
       state: present
       update_cache: yes

   - name: Configure Prometheus
     template:
       src: prometheus.yml.j2
       dest: /etc/prometheus/prometheus.yml
     notify: Restart Prometheus

   handlers:
     - name: Restart Prometheus
       systemd:
         name: prometheus
         state: restarted
   ```

2. Configure Prometheus to scrape metrics from your Java application:

   Add the Micrometer dependency to your `pom.xml`:
   ```xml
   <dependency>
     <groupId>io.micrometer</groupId>
     <artifactId>micrometer-registry-prometheus</artifactId>
   </dependency>
   ```

   Enable Prometheus endpoint in your `application.properties`:
   ```
   management.endpoints.web.exposure.include=prometheus
   ```

   Update Prometheus configuration to scrape your application:
   ```yaml
   scrape_configs:
     - job_name: 'spring-actuator'
       metrics_path: '/actuator/prometheus'
       static_configs:
         - targets: ['your-app-ip:8080']
   ```

3. Set up Grafana and connect it to Prometheus:

   Install Grafana:
   ```yaml
   - name: Install Grafana
     apt:
       name: grafana
       state: present
       update_cache: yes

   - name: Start Grafana
     systemd:
       name: grafana-server
       state: started
       enabled: yes
   ```

   Configure Grafana to use Prometheus as a data source:
   - Log in to Grafana
   - Go to Configuration > Data Sources
   - Add a new Prometheus data source with the Prometheus server URL

4. Create dashboards for key metrics (CPU, memory, request rate, error rate):
   - In Grafana, create a new dashboard
   - Add panels for key metrics using PromQL queries, e.g.:
     - CPU Usage: `rate(process_cpu_usage[5m])`
     - Memory Usage: `jvm_memory_used_bytes`
     - Request Rate: `rate(http_server_requests_seconds_count[5m])`
     - Error Rate: `rate(http_server_requests_seconds_count{status="5xx"}[5m])`

5. Set up alerting rules in Prometheus:

   Add alert rules to your Prometheus configuration:
   ```yaml
   alerting:
     rules:
       - alert: HighCPUUsage
         expr: process_cpu_usage > 0.8
         for: 5m
         labels:
           severity: warning
         annotations:
           summary: "High CPU usage detected"
           description: "CPU usage is above 80% for 5 minutes"
   ```

For more information on Prometheus and Grafana, refer to their official documentation:
- [Prometheus Documentation](https://prometheus.io/docs/introduction/overview/)
- [Grafana Documentation](https://grafana.com/docs/)

## 8. Logging with ELK Stack

1. Set up Elasticsearch, Logstash, and Kibana on dedicated EC2 instances or use Elastic Cloud:

   For self-hosted ELK, create Terraform configurations for each component:
   ```hcl
   resource "aws_instance" "elasticsearch" {
     ami           = "ami-0c55b159cbfafe1f0"
     instance_type = "t2.medium"
     subnet_id     = module.vpc.private_subnet_ids[0]
     key_name      = "your-key-pair"

     tags = {
       Name = "Elasticsearch"
     }
   }

   # Similar configurations for Logstash and Kibana
   ```

   Use Ansible to install and configure the ELK stack:
   ```yaml
   - name: Install Elasticsearch
     apt:
       name: elasticsearch
       state: present
       update_cache: yes

   - name: Configure Elasticsearch
     template:
       src: elasticsearch.yml.j2
       dest: /etc/elasticsearch/elasticsearch.yml
     notify: Restart Elasticsearch

   # Similar tasks for Logstash and Kibana
   ```

2. Configure Filebeat on your application servers to ship logs:

   Install Filebeat:
   ```yaml
   - name: Install Filebeat
     apt:
       name: filebeat
       state: present
       update_cache: yes

   - name: Configure Filebeat
     template:
       src: filebeat.yml.j2
       dest: /etc/filebeat/filebeat.yml
     notify: Restart Filebeat
   ```

   Configure Filebeat to ship logs to Logstash:
   ```yaml
   filebeat.inputs:
   - type: log
     enabled: true
     paths:
       - /path/to/your/application/logs/*.log

   output.logstash:
     hosts: ["logstash-server:5044"]
   ```

3. Create Logstash filters to parse your application logs:

   Create a Logstash configuration file:
   ```
   input {
     beats {
       port => 5044
     }
   }

   filter {
     grok {
       match => { "message" => "%{TIMESTAMP_ISO8601:timestamp} %{LOGLEVEL:log_level} %{GREEDYDATA:log_message}" }
     }
   }

   output {
     elasticsearch {
       hosts => ["elasticsearch-server:9200"]
       index => "application-logs-%{+YYYY.MM.dd}"
     }
   }
   ```

4. Set up Kibana dashboards for log visualization and analysis:
   - Log in to Kibana
   - Create an index pattern for your application logs
   - Build visualizations based on log fields (e.g., log levels, timestamps)
   - Create a dashboard combining these visualizations

For more information on the ELK stack, refer to the [Elastic documentation](https://www.elastic.co/guide/index.html).


## 9. Alerting and Communication with Slack

1. Create a Slack app and get webhook URL for notifications:
   - Go to https://api.slack.com/apps
   - Click "Create New App" and choose "From scratch"
   - Select the workspace where you want to add the app
   - Go to "Incoming Webhooks" and activate it
   - Click "Add New Webhook to Workspace"
   - Choose the channel where you want to receive notifications
   - Copy the Webhook URL

2. Configure Jenkins to send build and deployment notifications to Slack:
   - Install the Slack Notification plugin in Jenkins
   - Go to Manage Jenkins > Configure System
   - Find the Slack section and add your team subdomain and integration token
   - In your Jenkinsfile, add Slack notifications:

   ```groovy
   post {
       success {
           slackSend (color: '#00FF00', message: "SUCCESSFUL: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]' (${env.BUILD_URL})")
       }
       failure {
           slackSend (color: '#FF0000', message: "FAILED: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]' (${env.BUILD_URL})")
       }
   }
   ```

3. Set up Prometheus Alertmanager to send alerts to Slack:
   - Install Alertmanager:
     ```yaml
     - name: Install Alertmanager
       apt:
         name: prometheus-alertmanager
         state: present
     ```
   - Configure Alertmanager to use Slack:
     ```yaml
     global:
       slack_api_url: 'https://hooks.slack.com/services/YOUR/SLACK/WEBHOOK'

     route:
       receiver: 'slack-notifications'

     receivers:
     - name: 'slack-notifications'
       slack_configs:
       - channel: '#alerts'
         send_resolved: true
     ```
   - Update Prometheus configuration to use Alertmanager:
     ```yaml
     alerting:
       alertmanagers:
       - static_configs:
         - targets:
           - 'localhost:9093'
     ```

4. Create a #devops channel in Slack for team communication:
   - Open Slack and click on the '+' next to 'Channels'
   - Name the channel 'devops'
   - Invite team members to the channel

For more information on Slack integrations, refer to the [Slack API documentation](https://api.slack.com/docs).

## 10. Security Considerations

1. Implement AWS WAF for web application firewall protection:
   - Use Terraform to set up AWS WAF:
     ```hcl
     resource "aws_wafv2_web_acl" "main" {
       name        = "managed-rule-set"
       description = "Managed rule set example."
       scope       = "REGIONAL"

       default_action {
         allow {}
       }

       rule {
         name     = "rule-1"
         priority = 1

         override_action {
           count {}
         }

         statement {
           managed_rule_group_statement {
             name        = "AWSManagedRulesCommonRuleSet"
             vendor_name = "AWS"
           }
         }

         visibility_config {
           cloudwatch_metrics_enabled = false
           metric_name                = "friendly-rule-metric-name"
           sampled_requests_enabled   = false
         }
       }

       visibility_config {
         cloudwatch_metrics_enabled = false
         metric_name                = "friendly-metric-name"
         sampled_requests_enabled   = false
       }
     }
     ```

2. Use AWS GuardDuty for threat detection:
   - Enable GuardDuty in the AWS Console or use Terraform:
     ```hcl
     resource "aws_guardduty_detector" "main" {
       enable = true
     }
     ```

3. Regularly update and patch all systems:
   - Use AWS Systems Manager Patch Manager to automate patching
   - Create a patch baseline and patch group:
     ```hcl
     resource "aws_ssm_patch_baseline" "production" {
       name             = "production-patch-baseline"
       operating_system = "AMAZON_LINUX_2"

       approval_rule {
         approve_after_days = 7
         compliance_level   = "CRITICAL"

         patch_filter {
           key    = "PRODUCT"
           values = ["AmazonLinux2"]
         }

         patch_filter {
           key    = "CLASSIFICATION"
           values = ["Security"]
         }

         patch_filter {
           key    = "SEVERITY"
           values = ["Critical"]
         }
       }
     }
     ```

4. Implement proper IAM policies and roles:
   - Use the principle of least privilege
   - Create specific roles for EC2 instances, Lambda functions, etc.
   - Example IAM role for EC2:
     ```hcl
     resource "aws_iam_role" "ec2_role" {
       name = "ec2_role"

       assume_role_policy = jsonencode({
         Version = "2012-10-17"
         Statement = [
           {
             Action = "sts:AssumeRole"
             Effect = "Allow"
             Principal = {
               Service = "ec2.amazonaws.com"
             }
           }
         ]
       })
     }

     resource "aws_iam_role_policy_attachment" "ssm_policy" {
       policy_arn = "arn:aws:iam::aws:policy/AmazonSSMManagedInstanceCore"
       role       = aws_iam_role.ec2_role.name
     }
     ```

5. Use AWS KMS for encryption of sensitive data:
   - Create a KMS key:
     ```hcl
     resource "aws_kms_key" "main" {
       description             = "KMS key for application secrets"
       deletion_window_in_days = 10
     }
     ```
   - Use the key to encrypt sensitive data in your application

For more information on AWS security best practices, refer to the [AWS Security Documentation](https://docs.aws.amazon.com/security/).

## 11. Backup and Disaster Recovery

1. Set up regular EBS snapshots for EC2 instances:
   - Use Amazon Data Lifecycle Manager to automate EBS snapshots:
     ```hcl
     resource "aws_dlm_lifecycle_policy" "example" {
       description        = "Example DLM lifecycle policy"
       execution_role_arn = aws_iam_role.dlm_lifecycle_role.arn
       state              = "ENABLED"

       policy_details {
         resource_types = ["VOLUME"]

         schedule {
           name = "2 weeks of daily snapshots"

           create_rule {
             interval      = 24
             interval_unit = "HOURS"
             times         = ["23:45"]
           }

           retain_rule {
             count = 14
           }

           tags_to_add = {
             SnapshotCreator = "DLM"
           }

           copy_tags = false
         }

         target_tags = {
           Snapshot = "true"
         }
       }
     }
     ```

2. Configure RDS automated backups and consider read replicas:
   - Enable automated backups in your RDS instance configuration:
     ```hcl
     resource "aws_db_instance" "default" {
       # ... other configuration ...
       backup_retention_period = 7
       backup_window           = "03:00-04:00"
     }
     ```
   - Create a read replica for improved performance and disaster recovery:
     ```hcl
     resource "aws_db_instance" "replica" {
       identifier     = "myapp-replica"
       instance_class = "db.t3.micro"
       replicate_source_db = aws_db_instance.default.identifier
     }
     ```

3. Implement a disaster recovery plan:
   - Consider using AWS Elastic Disaster Recovery (DRS) for critical systems
   - Set up cross-region replication for S3 buckets:
     ```hcl
     resource "aws_s3_bucket" "primary" {
       bucket = "my-primary-bucket"
     }

     resource "aws_s3_bucket" "replica" {
       bucket = "my-replica-bucket"
       provider = aws.replica_region
     }

     resource "aws_s3_bucket_replication_configuration" "replication" {
       role   = aws_iam_role.replication.arn
       bucket = aws_s3_bucket.primary.id

       rule {
         id     = "foobar"
         status = "Enabled"

         destination {
           bucket        = aws_s3_bucket.replica.arn
           storage_class = "STANDARD"
         }
       }
     }
     ```

4. Regularly test your recovery procedures:
   - Schedule quarterly disaster recovery drills
   - Document and update your disaster recovery plan based on test results

For more information on AWS backup and disaster recovery, refer to the [AWS Backup Documentation](https://docs.aws.amazon.com/aws-backup/latest/devguide/whatisbackup.html).

## 12. Maintenance and Upgrades

1. Implement a strategy for zero-downtime deployments:
   - Use blue-green deployments with your load balancer
   - Update your Terraform configuration to support blue-green deployments:
     ```hcl
     resource "aws_lb_target_group" "blue" {
       name     = "blue-tg"
       port     = 80
       protocol = "HTTP"
       vpc_id   = aws_vpc.main.id
     }

     resource "aws_lb_target_group" "green" {
       name     = "green-tg"
       port     = 80
       protocol = "HTTP"
       vpc_id   = aws_vpc.main.id
     }

     resource "aws_lb_listener" "front_end" {
       load_balancer_arn = aws_lb.front_end.arn
       port              = "80"
       protocol          = "HTTP"

       default_action {
         type             = "forward"
         target_group_arn = var.deploy_green ? aws_lb_target_group.green.arn : aws_lb_target_group.blue.arn
       }
     }
     ```

2. Regularly review and optimize AWS costs:
   - Use AWS Cost Explorer to identify cost-saving opportunities
   - Implement AWS Budgets to set cost thresholds and receive alerts
   - Consider using Savings Plans or Reserved Instances for predictable workloads

3. Keep all tools and dependencies up to date:
   - Regularly update your application dependencies
   - Keep your CI/CD tools (Jenkins, Ansible, etc.) up to date
   - Use tools like Dependabot to automate dependency updates

4. Conduct regular security audits:
   - Perform quarterly security assessments
   - Use tools like AWS Inspector for automated security assessments
   - Conduct penetration testing at least annually

5. Document all processes and keep documentation up to date:
   - Maintain a wiki or documentation repository
   - Update documentation after each significant change
   - Conduct regular reviews of documentation to ensure accuracy

Remember to adapt these practices to your specific needs and always test thoroughly before implementing changes in production.

## Conclusion

This comprehensive guide provides a solid foundation for managing a Java API service using DevOps practices. By following these steps and continuously improving your processes, you can build a robust, scalable, and maintainable application infrastructure.

Key takeaways:
1. Automate everything possible
2. Implement robust monitoring and alerting
3. Prioritize security at every stage
4. Plan for disasters and test your recovery procedures
5. Continuously optimize and improve your infrastructure and processes

As you implement these practices, remember that DevOps is not just about tools, but also about culture and continuous improvement. Encourage collaboration between development and operations teams, and always be open to learning and adopting new technologies and practices.

For any questions or further assistance, please refer to the official documentation of the tools and services mentioned in this guide, or consult with your team and AWS support.
