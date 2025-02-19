pipeline {
    agent {
        label 'AGENT-1'
    }
    options {
        timeout(time:30, unit:'MINUTES')
        disableConcurrentBuilds()
        ansiColor('xterm')
    }
    environment {
        def appVersion = '' //variable declaration
        nexusUrl = 'nexus.avinexpense.online:8081'
    }
    stages {
         stage('read the version') {
            steps {
                script {
                    def packageJson = readJSON file: 'package.json'
                    appVersion = packageJson.version
                    echo "application version: $appVersion"
                }            
            }
        }
       
        stage('Build') {
            steps {
                sh """
                    zip -q -r frontend-${appVersion}.zip * -x Jenkinsfile -x frontend-${appVersion}.zip
                    ls -ltr
                """
            }
        }
        stage('Nexus artifact upload') {
            steps {
                nexusArtifactUploader(
                    nexusVersion: 'nexus3',
                    protocol: 'http',
                    nexusUrl: "${nexusUrl}",
                    groupId: 'com.expense',
                    version: "${appVersion}",
                    repository: 'frontend',
                    credentialsId: 'nexus_auth',
                    artifacts: [
                        [artifactId: "frontend",
                        classifier: '',
                        file: 'frontend-' + "${appVersion}" + '.zip',
                        type: 'zip']
                    ]
                )
            }
        }
        stage ('Starting downstream job ') {
            steps {
                script {
                    def params = [
                        string(name: 'appVersion', value: "${appVersion}")
                    ]
                     build job: 'frontend-deploy', parameters: params, wait: false
                }
               
            }
}
       
        
    }
    post {
        always {
            echo 'I will always run.'
            deleteDir()
        }
        success {
            echo 'I will run only when pipeline is success'
        }
        failure {
            echo 'I will run only when pipeline is failure'
        }
    }
}