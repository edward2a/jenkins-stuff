providers = ['aws', 'ali']
regions = ['dublin', 'london', 'frankfurt']
envs = ['dev', 'test', 'qa', 'stage', 'prod']

pipeline {
    agent any

    parameters {
        choice(name: 'MODE', choices: ['plan', 'apply'], description: 'The terraform operation to perform')
        choice(name: 'PROVIDER', choices: providers, description: 'The cloud provider to deploy to')
        choice(name: 'REGION', choices: regions, description: 'The cloud provider targe region for deployment')
        choice(name: 'ENV', choices: envs, description: 'The target deployment environment')
    }

    environment {
        WORKSPACE = "${env.PROVIDER}_${env.REGION}_${env.ENV}"
        TF_PLAN = 'plan.log'
    }

    stages {
        stage('Prepare Workspace'){
            steps {
                git branch: 'release/2.0', credentialsId: 'git-<ORG_NAME>-ci', poll: false, url: 'git@github.com:<ORG_NAME>/<PROJ_NAME>-cloud-stack'
            }
        }
        stage('Plan'){
            steps {
                dir('<PROJ_NAME>-cloud-stack-tf') {
                    withDockerContainer(args: '--entrypoint ""', image: 'hashicorp/terraform:0.11.8') {
                        sh 'terraform init'
                        sh """
                            if ! terraform workspace select ${env.WORKSPACE}; then 
                                terraform workspace new ${env.WORKSPACE}
                            fi
                        """
                        sh "terraform plan -no-color -refresh=true -out=${env.TF_PLAN}"
                    }
                }
            }
        }
        stage('Deploy'){
            steps {
                input(
                    message: "Confirm plan?"
                )
                dir('<PROJ_NAME>-cloud-stack-tf') {
                    withDockerContainer(args: '--entrypoint ""', image: 'hashicorp/terraform:0.11.8') {
                        sh """
                            if [ "${env.MODE}" == "apply" ]; then
                                terraform apply -no-color "${env.TF_PLAN}"
                            fi
                        """
                    }
                }
            }
        }
    }
}
