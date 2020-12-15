pipeline {
    agent {
    label 'jenslave'
    }
    tools {
          maven "maven"
    }
    stages {
        stage('Build') {
            steps {
                checkout([$class: 'GitSCM', branches: [[name: '*/qa']],
                 doGenerateSubmoduleConfigurations: false, 
                 extensions: [], 
                 submoduleCfg: [], 
                 userRemoteConfigs: [[url: 'https://github.com/lubomiro/petclinic.git']]])
                script {
                env.COMMITTER = sh(script:'git log -n 1 --pretty=format:"%an"', returnStdout: true).trim()
                env.EMAIL = sh(script:'git log -n 1 --pretty=format:"%ae"', returnStdout: true).trim()
                }
                sh "mvn -Dmaven.test.failure.ignore=true clean package"
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
            post{
                always{
                    echo "Name of the committer ${env.COMMITTER}"
                    echo "Email of the committer ${env.EMAIL}"
                }
            }
        }
    }
}
