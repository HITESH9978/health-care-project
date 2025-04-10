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
          sh "docker login -u ${dockeruser} -p ${dockerpass}"
        }
      }
    }

    stage('Docker Push Image') {
      steps {
        echo 'Pushing the Docker image to DockerHub'
        sh 'docker push shivareddy24/healthcare:latest'
      }
    }

    stage('Deploying to Kubernetes with Ansible') {
      steps {
        echo 'Deploying application to Kubernetes cluster using Ansible'

        withCredentials([sshUserPrivateKey(credentialsId: 'devops-key', keyFileVariable: 'SSH_KEY')]) {
          sh '''
            ansible-playbook -i /etc/ansible/hosts ansible-playbook.yml \
            -e ansible_ssh_private_key_file=$SSH_KEY \
            -e "ansible_ssh_common_args='-o StrictHostKeyChecking=no'"
          '''
        }

        // Apply Kubernetes deployment using the kubeconfig file from Jenkins credentials
        withCredentials([file(credentialsId: 'kubeconfig-id', variable: 'KUBECONFIG')]) {
          sh 'kubectl apply -f deploy.yml --validate=false'
        }
      }
    }
  }
}



// pipeline {
//   agent any
//  environment {
//     KUBECONFIG = '/path/to/your/kubeconfig'
//   }
//   tools {
//     maven 'MAVEN'
//   }

//   stages {
//     stage('Git Checkout') {
//       steps {
//         echo 'Cloning the repository from GitHub'
//         git branch: 'master', url: 'https://github.com/ShivashankarareddyBA/health-care-project.git'
//       }
//     }

//     stage('Create Package') {
//       steps {
//         echo 'Compiling, testing, and packaging the application'
//         sh 'mvn package'
//       }
//     }

//     stage('Generate Test Report') {
//       steps {
//         echo 'Checking TestNG report generation'
//         sh 'ls -l target/surefire-reports'

//         echo 'Generating test report using TestNG'
//         publishHTML([
//           allowMissing: false,
//           alwaysLinkToLastBuild: false,
//           keepAll: false,
//           reportDir: 'target/surefire-reports',
//           reportFiles: 'index.html',
//           reportName: 'HTML Report',
//           reportTitles: ''
//         ])
//       }
//     }

//     stage('Create Docker Image') {
//       steps {
//         echo 'Creating a Docker image'
//         sh 'docker build -t shivareddy24/healthcare:latest .'
//       }
//     }

//     stage('Login to DockerHub') {
//       steps {
//         echo 'Logging into DockerHub'
//         withCredentials([usernamePassword(credentialsId: 'dockerpwd', passwordVariable: 'dockerpass', usernameVariable: 'dockeruser')]) {
//           sh "docker login -u ${dockeruser} -p ${dockerpass}"
//         }
//       }
//     }

//     stage('Docker Push Image') {
//       steps {
//         echo 'Pushing the Docker image to DockerHub'
//         sh 'docker push shivareddy24/healthcare:latest'
//       }
//     }

//     stage('Deploying to Kubernetes with Ansible') {
//       steps {
//         echo 'Deploying application to Kubernetes cluster using Ansible'

//         // Use SSH private key stored in Jenkins Credentials
//         withCredentials([sshUserPrivateKey(credentialsId: 'devops-key', keyFileVariable: 'SSH_KEY')]) {
//   sh '''
//     ansible-playbook -i /etc/ansible/hosts ansible-playbook.yml \
//     -e ansible_ssh_private_key_file=$SSH_KEY \
//     -e "ansible_ssh_common_args='-o StrictHostKeyChecking=no'"
//   '''
// }
//       //  withCredentials([sshUserPrivateKey(credentialsId: 'devops-key', keyFileVariable: 'SSH_KEY')]) {
//         //  sh """
//          //   ansible-playbook -i /etc/ansible/hosts ansible-playbook.yml \
//         //    -e ansible_ssh_private_key_file=$SSH_KEY \
//         //    -e ansible_ssh_common_args='-o StrictHostKeyChecking=no'
//         //  """
//        // }
//        withCredentials([file(credentialsId: 'kubeconfig-id', variable: 'KUBECONFIG')]) {
//   sh 'kubectl apply -f deploy.yml'
// }
//         // Deploy to Kubernetes
//         sh 'kubectl apply -f deploy.yml --validate=false'
//         // sh 'kubectl apply -f deploy.yml'
//       }
//     }
//   }
// }
