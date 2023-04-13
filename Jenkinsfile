pipeline {
    agent any

    stages {
        stage('Checkout') {
            steps {
                git url: 
'https://github.com/aekanun2020/2023-Automate-Model-Training-on-dark-axe-383408.git'
            }
        }

        stage('Run Google Cloud Composer') {
            steps {
                script {
                    // Replace with your Google Cloud project ID and location.
                    def projectId = 'dark-axe-383408'
                    def location = 'us-central1'
                    def composerEnvironment = 'airflow-from-jenkins'

                    // Authenticate with Google Cloud.
                    withCredentials([file(credentialsId: 
'project-owner-google-cloud-key-file', variable: 'keyFile')]) {
                        sh 'gcloud auth activate-service-account 
--key-file=${keyFile}'
                    }

                    // Configure gcloud project.
                    sh "gcloud config set project ${projectId}"
                    sh "gcloud config set composer/location ${location}"

                    // Trigger the DAG.
                    sh "gcloud composer environments run ${composerEnvironment} 
trigger_dag -- DAG-automateML_Notification"
                }
            }
        }
    }
}
