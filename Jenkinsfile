pipeline {
    agent any
    parameters {
      choice(name: 'action', choices: 'create\ndestroy', description: 'Create/update or destroy the eks cluster.')
      string(name: 'cluster', defaultValue : 'hexa-cluster', description: "EKS cluster name.")
      choice(name: 'k8s_version', choices: '1.21\n1.20\n1.19\n1.18\n1.17\n1.16', description: 'K8s version to install.')
      string(name: 'NodeG', defaultValue : 'ng', description: "EKS cluster name.")
      string(name: 'instance_type', defaultValue : 't3.medium', description: "k8s worker node instance type.")
      string(name: 'num_workers', defaultValue : '1', description: "k8s number of worker instances.")
      string(name: 'region', defaultValue : 'ap-south-1', description: "AWS region.")
      string(name: 'key_pair', defaultValue : 'Jenkinskey', description: "EC2 instance ssh keypair.")

   }
  environment {
    PATH = "${env.WORKSPACE}/bin:${env.PATH}"
    //KUBECONFIG = "${env.WORKSPACE}/.kube/config"
  }
  stages {

    stage('Install eksctl') {
        when {
        expression { params.action == 'create' }
      }
      steps {
             sh '''
             #mkdir bin
             cd bin
             curl --silent --location "https://github.com/weaveworks/eksctl/releases/latest/download/eksctl_$(uname -s)_amd64.tar.gz" | tar xzf -
             sudo chmod u+x eksctl
             eksctl version
             '''
            }
        }
    stage('Create Cluster') {
        when {
        expression { params.action == 'create' }
      }
        steps {
            sh """
            eksctl create cluster \
            --name ${params.cluster} \
            --version ${params.k8s_version} \
            --region ${params.region} \
            --nodegroup-name ${params.NodeG}-0 \
            --nodes ${params.num_workers} \
            --node-type ${params.instance_type} \
            --ssh-access \
            --ssh-public-key "${params.key_pair}" \
            """
        }
    }
    stage('Setup Cluster') {
        when {
        expression { params.action == 'create' }
      }
      steps {
        sh "aws eks update-kubeconfig --name ${params.cluster} --region ${params.region}"
      }
    }
    stage('Delete Cluster') {
        when {
        expression { params.action == 'destroy' }
      }
      steps {
         sh "eksctl delete cluster --name=${params.cluster} --region=${params.region}"
      }
    }     
   }
}
