properties([
    parameters([
        string(
            defaultValue: 'dev',
            name: 'Environment'
        ),
        choice(
            choices: ['plan', 'apply', 'destroy'],
            name: 'Terraform_Action'
        )
    ])
])

pipeline {
    agent any

    stages {

        stage('Preparing') {
            steps {
                sh 'echo Preparing'
            }
        }

        stage('Git Pulling') {
            steps {
                git branch: 'main',
                    url: 'https://github.com/darshant15/EKS-Terraform-Project.git'
            }
        }

        stage('AWS Test') {
            steps {
                withAWS(credentials: 'aws-creds', region: 'us-east-1') {
                    sh '''
                        echo "===== AWS Identity ====="
                        aws sts get-caller-identity

                        echo "===== S3 Bucket Test ====="
                        aws s3 ls s3://dev-aman-tf-bucket
                    '''
                }
            }
        }

        stage('Init') {
            steps {
                withAWS(credentials: 'aws-creds', region: 'us-east-1') {
                    sh 'terraform -chdir=eks/ init'
                }
            }
        }

        stage('Validate') {
            steps {
                withAWS(credentials: 'aws-creds', region: 'us-east-1') {
                    sh 'terraform -chdir=eks/ validate'
                }
            }
        }

        stage('Action') {
            steps {
                withAWS(credentials: 'aws-creds', region: 'us-east-1') {
                    script {

                        if (params.Terraform_Action == 'plan') {

                            sh """
                                terraform -chdir=eks/ plan \
                                -var-file=${params.Environment}.tfvars
                            """

                        } else if (params.Terraform_Action == 'apply') {

                            sh """
                                terraform -chdir=eks/ apply \
                                -var-file=${params.Environment}.tfvars \
                                -auto-approve
                            """

                        } else if (params.Terraform_Action == 'destroy') {

                            sh """
                                terraform -chdir=eks/ destroy \
                                -var-file=${params.Environment}.tfvars \
                                -auto-approve
                            """

                        } else {

                            error "Invalid Terraform_Action: ${params.Terraform_Action}"
                        }
                    }
                }
            }
        }
    }
}
