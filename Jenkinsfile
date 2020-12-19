pipeline {
    agent {
        label 'jenslave'
    }
    triggers {
         GenericTrigger(
             genericVariables: [[key: 'ref', value: '$.ref'], 
             [key: 'changed_files', value: '$.commits[*].[\'modified\',\'added\',\'removed\'][*]']], 
             causeString: 'Triggered on $ref',
             regexpFilterExpression: 'refs/heads/stage .*readme.md.*', 
             regexpFilterText: '$ref $changed_files', 
             token: 'lubomir', 
             tokenCredentialId: ''
             )
    }
    tools {
        maven "maven"
    }
    stages {
        stage('Build') {
            steps {
                checkout(
                    [$class: 'GitSCM', branches: [[name: '*/stage']],
                    doGenerateSubmoduleConfigurations: false, 
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
        stage('Deploy to AWS S3') {
            steps {
                s3Upload consoleLogLevel: 'INFO', 
                         dontSetBuildResultOnFailure: false, 
                         dontWaitForConcurrentBuildCompletion: false, 
                         entries: [[bucket: 'mybuilds/${JOB_NAME}-${BUILD_NUMBER}', 
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
