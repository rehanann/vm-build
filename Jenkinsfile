pipeline {
  agent any
  environment {
    SVC_ACCOUNT_KEY = credentials('terraform-auth')
    INVENTORY = credentials('INVENTORY_INI')
    VARIABLES = credentials('VARIABLES')
    PROVIDER = credentials('PROVIDER')
  }
  stages {
            stage('Checkout') {
                steps {
                    checkout scm
                    sh 'echo $SVC_ACCOUNT_KEY | base64 -d > serviceaccount.json'
                    sh 'echo $INVENTORY  | base64 -d > inventory.ini'
                    sh 'echo $PROVIDER | base64 -d > provider.tf'
                    sh 'echo $VARIABLES | base64 -d > variable.tf'
                    }
             }
            stage('TF Plan') {
                steps {
                sh 'terraform init'
                sh 'terraform plan -out myplan'
            }
        }
        stage('Approval') {
                steps {
                    script {
                     def userInput = input(id: 'confirm', message: 'Apply Terraform?', parameters: [ [$class: 'BooleanParameterDefinition', defaultValue: false, description: 'Apply terraform', name: 'confirm'] ])
                }
            }
        }
        stage('TF Apply') {
                steps {
                    sh 'terraform apply myplan'
                    sh 'cp myplan ../gluster-destroy/'
                    sh 'cp terraform.tfstate* ../gluster-destroy/'
                    sh 'sleep 60'
            }
        }
        stage('Pre-req Installation') {
                steps {
                    sh 'export ANSIBLE_HOST_KEY_CHECKING=False'
                    sh 'ansible-playbook -i host.ini ansible-pb/config.yml'
                    sh 'sleep 120'
                }
        }
        // stage('Docker storage') {
        //     steps {
        //             sh 'ansible-playbook -i inventory.ini ansible-pb/docker-storage-setup-ofs.yml'
        //     }
        // }
        // stage('OpenShift Installation') {
        //     steps {
        //             sh 'ansible-playbook -i inventory.ini openshift-ansible/playbooks/prerequisites.yml'
        //             sh 'ansible-playbook -i inventory.ini openshift-ansible/playbooks/deploy_cluster.yml'
        //     }
        // }
    }
}