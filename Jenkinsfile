pipeline {
    agent any

    environment {
        google_cloud_composer_environment = 'airflow-from-jenkins'
        google_cloud_composer_location = 'us-central1'
        google_cloud_project_id = 'iot-class-feb2017'
        git_repo_url = 'https://github.com/aekanun2020/2023-Automate-Model-Training-using-DevOps.git'
        git_branch = 'main'
        pyspark_bucket_name = 'production-mlmodeling'
    }

    stages {
        stage('Checkout Git repository') {
            steps {
                checkout([
                    $class: 'GitSCM',
                    branches: [[name: "${git_branch}"]],
                    extensions: [],
                    userRemoteConfigs: [[url: "${git_repo_url}"]]
                ])
            }
        }

        stage('Trigger Google Cloud Composer') {
            steps {
                echo 'Activating Google Cloud Service Account and triggering Cloud Composer DAG...'
                withCredentials([file(credentialsId: 'iot-class-feb2017', variable: 'google_cloud_key_file')]) {
                    sh '''gcloud auth activate-service-account --key-file="${google_cloud_key_file}"'''
                    sh '''gcloud config set project "${google_cloud_project_id}"'''

                    script {
                        env.composer_environment_exists = sh(script: 'gcloud composer environments list --project="${google_cloud_project_id}" --locations="${google_cloud_composer_location}" --filter="name:${google_cloud_composer_environment}" --format="value(name)"', returnStdout: true).trim()
                        if (env.composer_environment_exists == '') {
                            echo 'Creating Cloud Composer environment...'
                            sh '''gcloud composer environments create "${google_cloud_composer_environment}" --location="${google_cloud_composer_location}" --image-version=composer-1.20.10-airflow-2.4.3 --machine-type=n1-standard-1 --node-count=3 --python-version=3 --project="${google_cloud_project_id}" --zone=us-central1-a --disk-size=30GB'''
                        } else {
                            echo 'Environment already exists...'
                        }
                    }

                    script {
                        env.gcs_bucket_name = sh(script: 'gcloud composer environments describe "${google_cloud_composer_environment}" --location="${google_cloud_composer_location}" --format="value(config.dagGcsPrefix)"', returnStdout: true).trim().split('/')[2]
                    }

                    echo 'Uploading Python files to Google Cloud Storage...'
                    sh '''gsutil cp G-Student.py gs://${pyspark_bucket_name}/'''
                    sh '''gsutil cp refinedzone_H-Student.py gs://${pyspark_bucket_name}/'''

                    echo 'Uploading DAG file to the Cloud Composer environment...'
                    sh '''gsutil cp DAG-automateML_Notification.py gs://${gcs_bucket_name}/dags/'''

                    
                }
            }
        }
    }
}
