pipeline {
    agent any
        environment {
            MAIL_AUTOR = sh(script: 'git --no-pager show -s --format=\'%ae\'',,returnStdout: true).trim()
            SSH_PASS = credentials('sionsshpass')
        }
    stages{    
        stage('Build') {
            steps {
                echo '==== BUILDING STAGE ===='
                
                    //sh 'npm install'
                    configFileProvider([configFile(fileId: '7c2f8046-80f9-46d9-96fa-c56ccfdefd69', variable: 'ENV_FILE')]) {
                    sh "cp $ENV_FILE .env"
                }
                
            }
        }
        
        /*
        stage('Test') {
            steps {
                echo '==== TESTING STAGE ===='
                dir('src') {
                catchError(buildResult: 'UNSTABLE', stageResult: 'FAILURE') {
                    sh './vendor/bin/phpunit --coverage-html reports-html/coverage --log-junit ./reports/results.xml ./tests/ '
                    
                }
                }
            }
             post{
                always{
                    archiveArtifacts artifacts: 'src/reports/', fingerprint: true
                    junit healthScaleFactor: 12.0, keepLongStdio: true, testResults: 'src/reports/'
                }
            }

        }
        
        stage('Cobertura'){
            steps{
            dir('src') {
                echo '==== ANALIZANDO COBERTURA ===='
                catchError(buildResult: 'UNSTABLE', stageResult: 'FAILURE') {
                    sh './vendor/bin/phpunit --configuration phpunit.xml --coverage-clover phpunit.coverage.xml --log-junit phpunit.report.xml'    
                }
            }
            }
            post{
                always{
                     publishHTML([allowMissing: false, alwaysLinkToLastBuild: true, keepAll: true, reportDir: 'src/reports-html/coverage', reportFiles: 'index.html', reportName: 'HTML Report', reportTitles: ''])
                }
            }

        }*/
         stage('SonarQube analysis') {
            steps{
                echo '==== Sonar STAGE ===='
            }
        }

        stage('Deploy') {
            steps { 
                echo '==== DEPLOYING STAGE ===='
                
                sh 'sudo docker build . -t password-front:${BUILD_ID}'
                sh 'docker rm -f password-front||true'
                sh 'docker run -dit --restart always -p 8089:80 --name password-front password-front:${BUILD_ID}'
               
            }
        }
        
    }
     
    post {
    
        always {
            gitlabCommitStatus('jenkins') {
                updateGitlabCommitStatus name: 'build', state: 'pending'
             updateGitlabCommitStatus name: 'build', state: 'success'
            }
        }
        success{
            updateGitlabCommitStatus name: 'jenkins', state:'success'
            slackSend channel: '#password-builds', color: 'good', message: "${JOB_NAME} build  #${BUILD_NUMBER} succeeded (<${RUN_DISPLAY_URL}|Open>)", tokenCredentialId: 'slack'
        }
        unstable {
            slackSend channel: '#password-builds', color: 'good', message: "${JOB_NAME} build  #${BUILD_NUMBER} succeeded (<${RUN_DISPLAY_URL}|Open>) pero hay error en los tests :weirdchamp:", tokenCredentialId: 'slack'
        }
        failure{
            cleanWs()
            updateGitlabCommitStatus name: 'jenkins', state:'failed'
            slackSend channel: '#password-builds', color: 'danger', message: "${JOB_NAME} build  #${BUILD_NUMBER} failed (<${RUN_DISPLAY_URL}|Open>)", tokenCredentialId: 'slack'
        }
    }

    
}
