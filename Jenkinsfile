#!/usr/bin/env groovy
/*
Jenkinsfile for deploying Terraform
*/
properties([
    parameters([
        string(name: 'bucket', description: 'Bucket Name for State File', defaultValue: 'sandbox', trim: false),
        string(name: 'project', description: 'Project', defaultValue: 'sandbox', trim: false),
        string(name: 'git_creds', description: 'GithHub Credentials', defaultValue: 'sandbox', trim: false),
        string(name: 'aws_credentials', description: 'AWS Credentials', defaultValue: 'aws-terraform-iacl', trim: false),
        choice(choices: ['Plan', 'Apply', 'Destroy'], name: 'plan_apply_or_destroy', description: 'Apply or Destroy Terraform')
    ])
])

node {

    /*
    withCredentials([
        [
            $class: 'SSHUserPrivateKeyBinding',
            credentialsId: "${params.git_creds}",
            keyFileVariable: 'ssh_key_file'
        ],
        [
            $class: 'AmazonWebServicesCredentialsBinding',
            credentialsId: "${params.aws_creds}",
            accessKeyVariable: 'AWS_ACCESS_KEY_ID',
            secretKeyVariable: 'AWS_SECRET_ACCESS_KEY'
        ]
    ])
    */
    
    try {
        withCredentials([ 
            usernamePassword(credentialsId: "${params.aws_creds}", usernameVariable: 'username', passwordVariable: 'password'),
        ])
        {
            withEnv(["AWS_ACCESS_KEY_ID=${username}","AWS_SECRET_ACCESS_KEY=${password}"])
            {
                lock(resource: 'iac_sandbox_infrastructure_lock')
                {
                    deleteDir()

                    stage('Checkout') {
                        checkout scm
                    }

                    stage('Initialize the Backend') {
                        dir("base-vpc") {
                            sh "terraform init -backend-config=\"bucket=${params.bucket}\" -backend-config=\"key=terraform/${params.project}.tfstate\" -backend-config=\"dynamodb_table=terraform=${params.project}-lock\" -backend-config=\"region=us-east-1\""
                        }
                    }

                    stage('Terraform Plan') {
                        dir("base-vpc") {
                            if (params. plan_apply_or_destroy == 'Destroy') {
                                sh "terraform plan -destroy -input=false -refresh=true -module-depth=-1 -var-file=environments/globals/inputs.tfvars -var-file=environments/${params.project}/inputs.tfvars"
                            } else { // if plan or apply is selected
                                sh "terraform plan -input=false -refresh=true -module-depth=-1 -var-file=environments/globals/inputs.tfvars -var-file=environments/${params.project}/inputs.tfvars"
                            }
                        }
                    }

                    stage('Terraform Apply/Destroy') {
                        dir("base-vpc") {
                            if (params. plan_apply_or_destroy == 'Destroy') {
                                sh "terraform destroy -auto-approve -var-file=environments/globals/inputs.tfvars -var-file=environments/${params.project}/inputs.tfvars"
                            } else if (params. plan_apply_or_destroy == 'Apply') {
                                sh "terraform apply -input=true -auto-approve -refresh=true -var-file=environments/globals/inputs.tfvars -var-file=environments/${params.project}/inputs.tfvars"
                            }
                        }
                    }
                }
            }
        }
    }
    catch (err)
    {
        throw (err)
    }
}