pipeline {
    agent {
        dockerfile {
            dir 'docker'
        }
    }
    options {
        buildDiscarder(logRotator(numToKeepStr: '10'))
    }
    environment {
        GOOGLE_APPLICATION_CREDENTIALS = credentials('google')
        TF_IN_AUTOMATION = 1
        TF_CLI_ARGS_apply = "-auto-approve -input=false"
        TF_CLI_ARGS_destroy = "-auto-approve -input=false"

    }
    parameters {
        string( name: 'projectID',
                defaultValue: 'NOT USED',
                description: ''
        )
        string( name: 'activator_params',
                description: 'json string'
        )
        choice( name: 'bootstrap_action',
                choices: ['APPLY', 'PLAN', 'DESTROY', 'IGNORE'],
                description: 'Terraform action'
        )
        )
    }
    stages {
        stage('Initialise') {
            steps {
                sh '''
                    echo Setup activator_params
                    echo $activator_params | jq > ./bootstrap/bootstrap.auto.tfvars.json TODO: need to validate dir for this file


                    echo Activate service account
                    gcloud auth activate-service-account --key-file $GOOGLE_APPLICATION_CREDENTIALS
                    export GOOGLE_OAUTH_ACCESS_TOKEN=$(gcloud auth print-access-token --impersonate-service-account ${_TF_SA_EMAIL})
                '''
           }
        }
        stage('Bootstrap') {
            when {
                      expression { params.bootstrap_action == 'PLAN' || params.bootstrap_action == 'APPLY' || params.bootstrap_action == 'DESTROY' }
            }
            steps {
                sh """

                    make bootstrap-${params.bootstrap_action.toLowerCase()}
                """
            }
        }
    post {
        always {
            script {
                deleteDir()
            }
        }
    }

}
