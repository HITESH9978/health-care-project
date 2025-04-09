pipeline {
  agent any

  tools {
    maven 'MAVEN'
  }

  environment {
    KUBECONFIG = "${WORKSPACE}/kubeconfig"  // We'll use a kubeconfig file
  }

  stages {
    stage('Git Checkout') {
      steps {
        echo 'Cloning the repository from GitHub'
        git branch: 'master', url: 'https://github.com/ShivashankarareddyBA/health-care-project.git'
      }
    }

    stage('Create Package') {
      steps {
        echo 'Compiling, testing, and packaging the application'
        sh 'mvn package'
      }
    }

    stage('Generate Test Report') {
      steps {
        echo 'Checking TestNG report generation'
        sh 'ls -l target/surefire-reports'

        echo 'Generating test report using TestNG'
        publishHTML([
          allowMissing: false,
          alwaysLinkToLastBuild: false,
          keepAll: false,
          reportDir: 'target/surefire-reports',
          reportFiles: 'index.html',
          reportName: 'HTML Report',
          reportTitles: ''
        ])
      }
    }

    stage('Create Docker Image') {
      steps {
        echo 'Creating a Docker image'
        sh 'docker build -t shivareddy24/healthcare:latest .'
      }
    }

    stage('Login to DockerHub') {
      steps {
        echo 'Logging into DockerHub'
        withCredentials([usernamePassword(credentialsId: 'dockerpwd', passwordVariable: 'dockerpass', usernameVariable: 'dockeruser')]) {
          sh 'echo $dockerpass | docker login -u $dockeruser --password-stdin'
        }
      }
    }

    stage('Docker Push Image') {
      steps {
        echo 'Pushing the Docker image to DockerHub'
        sh 'docker push shivareddy24/healthcare:latest'
      }
    }

    stage('Deploying with Ansible') {
      steps {
        echo 'Deploying application to Kubernetes cluster using Ansible'

        withCredentials([
          sshUserPrivateKey(credentialsId: 'devops-key', keyFileVariable: 'SSH_KEY'),
          file(credentialsId: 'kubeconfig-cred', variable: 'KUBECONFIG_FILE') // kubeconfig as a Jenkins file credential
        ]) {
          sh '''
            echo "Using Ansible to deploy..."
            ansible-playbook -i /etc/ansible/hosts ansible-playbook.yml \
              --private-key $SSH_KEY \
              -e "ansible_ssh_common_args='-o StrictHostKeyChecking=no'"

            echo "Copying kubeconfig for kubectl auth"
            cp $KUBECONFIG_FILE $KUBECONFIG

            echo "Deploying to Kubernetes cluster"
            kubectl apply -f deploy.yml
          '''
        }
      }
    }
  }
}
