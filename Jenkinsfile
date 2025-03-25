pipeline {
  agent any
  tools {
    maven 'MAVEN'
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
        sh 'ls -l /var/lib/jenkins/workspace/Healthcare/target/surefire-reports'  // Check if report exists
        
        echo 'Generating test report using TestNG'
        publishHTML([
          allowMissing: false, 
          alwaysLinkToLastBuild: false, 
          keepAll: false, 
          reportDir: '/var/lib/jenkins/workspace/Healthcare/target/surefire-reports', 
          reportFiles: 'index.html', 
          reportName: 'HTML Report', 
          reportTitles: ''
        ])
      }
    }
      
  stage('Create Docker Image') {
  steps {
    echo 'Creating a Docker image'
       sh 'docker build -t shivareddy24/healthcare .'
     }
}
    
  stage('Login to DockerHub') {
     steps {
       echo 'Logging into DockerHub'
       withCredentials([usernamePassword(credentialsId: 'dockerpwd', passwordVariable: 'dockerpass', usernameVariable: 'dockeruser')]) {
               sh "docker login -u ${dockeruser} -p ${dockerpass}"
}
     }
  }
      // withCredentials([usernamePassword(credentialsId: 'docker-pwd', passwordVariable: 'dockerpass', usernameVariable: 'dockeruser')]) {
       //  sh "docker login -u ${dockeruser} -p ${dockerpass}"
       // }
    // }
  //  }
    
   stage('Docker Push Image') {
      steps {
        echo 'Pushing the Docker image to DockerHub'
       sh 'docker push shivareddy24/healthcare'
      }
   }
  stage('Deploying to Kubernetes with Ansible') {
  steps {
    echo 'Deploying application to Kubernetes cluster using Ansible'
    sh """
      ansible-playbook -i /etc/ansible/hosts ansible-playbook.yml -e 'ansible_ssh_common_args=-o StrictHostKeyChecking=no'
    """
    sh 'kubectl apply -f deploy.yml'
  }
}
    
  //  stage('Deploying to Kubernetes with Ansible') {
   //   steps {
   //     echo 'Deploying application to Kubernetes cluster using Ansible'
   //     sh 'ansible-playbook -i /etc/ansible/hosts ansible-playbook.yml'
   //     sh 'kubectl apply -f deploy.yml'  // Apply the Kubernetes manifest after Ansible deployment
   //   }
   // }
  }
}
