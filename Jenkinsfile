pipeline {
    agent {
    label 'jenslave'
        }
    tools {
          maven "maven"
    }
    environment {
        COMMITTER = "Liubomir"
        MAIL = "lubomiro@gmail.com"
    }
    stages {
        stage('Build') {
            steps {
                checkout([$class: 'GitSCM', branches: [[name: '*/main']],
                 doGenerateSubmoduleConfigurations: false, 
                 extensions: [], 
                 submoduleCfg: [], 
                 userRemoteConfigs: [[url: 'https://github.com/lubomiro/petclinic.git']]])
                sh "mvn -Dmaven.test.failure.ignore=true clean package"
             }
            post{
                always{
                    echo "Name of the committer ${env.COMMITTER}"
                    echo "email ${env.MAIL}"
                }
             } 
        }
        stage('Test') {
            steps {
                junit '**/target/surefire-reports/TEST-*.xml'
                archiveArtifacts 'target/*.jar'
                }
            }
        stage('Deploy') {
            steps {
                s3Upload consoleLogLevel: 'INFO', 
                        dontSetBuildResultOnFailure: false, 
                        dontWaitForConcurrentBuildCompletion: false, 
                        entries: [[bucket: 'mybuilds/${JOB_NAME}-${BUILD_NUMBER}', 
                        excludedFile: '', 
                        flatten: false, 
                        gzipFiles: false, 
                        keepForever: false, 
                        managedArtifacts: false, 
                        noUploadOnFailure: true, 
                        selectedRegion: 'eu-central-1', 
                        showDirectlyInBrowser: false, 
                        sourceFile: 'target/*.jar', 
                        storageClass: 'STANDARD', 
                        uploadFromSlave: true, 
                        useServerSideEncryption: false]], 
                        pluginFailureResultConstraint: 'FAILURE', 
                        profileName: 'petclinic-bucket', 
                        userMetadata: []
            
            }
        }
    }
}
