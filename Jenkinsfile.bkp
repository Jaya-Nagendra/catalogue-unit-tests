pipeline {
    agent {
        node{
            label 'ROBOSHOP'
        }
    }
    environment {
        appVersion =""
        ACC_ID = "050734936364"
    }
    options {
        //disableConcurrentBuilds()
        timeout(time: 5, unit: 'MINUTES')
    }
    // parameters {
    //     string(name: 'PERSON', defaultValue: 'Mr Jenkins', description: 'Who should I say hello to?')
    //     text(name: 'BIOGRAPHY', defaultValue: '', description: 'Enter some information about the person')
    //     booleanParam(name: 'DEPLOY', defaultValue: false, description: 'Toggle this value')
    //     choice(name: 'CHOICE', choices: ['One', 'Two', 'Three'], description: 'Pick something')
    //     password(name: 'PASSWORD', defaultValue: 'SECRET', description: 'Enter a password')
    // }
        stages {
            stage('Read Version') {
                steps {
                    script {
                        def packageJson = readJSON file: 'package.json'
                        appVersion = packageJson.version
                        echo "Building version - ${appVersion}"
                    }
                }
            }
            stage('Install Dependencies') {
                steps {
                    script{
                        sh """
                            npm install
                        """    
                    }
                }
            }
            stage('Unit Tests') {
                steps {
                    script{
                        sh """
                            npm test
                        """    
                    }
                }
            }

            stage('sonarqube Analysis') {
                steps {
                    script {
                        def scannerHome = tool name: 'sonar-8' // agent configuration
                        withSonarQubeEnv('sonar-server') { // analysing and uploading to server
                            sh "${scannerHome}/bin/sonar-scanner"
                        }
                    }
                }
            }
            stage('Quality Gate'){
                steps {
                    timeout(time: 1, unit: 'HOURS') {
                    waitForQualityGate abortPipeline: true
                    }
                } 
            }         
            stage('Build Image') {
                steps {
                    script{
                        withAWS(credentials: 'aws-cred', region: 'us-east-1') {
                        sh """
                            aws ecr get-login-password --region us-east-1 | docker login --username AWS --password-stdin ${ACC_ID}.dkr.ecr.us-east-1.amazonaws.com
                            docker build -t ${ACC_ID}.dkr.ecr.us-east-1.amazonaws.com/roboshop/catalogue:${appVersion} .
                            docker push ${ACC_ID}.dkr.ecr.us-east-1.amazonaws.com/roboshop/catalogue:${appVersion}
                        """  
                        }  
                    }
                }
            }
       
    }    
    post {
        always {
            echo 'I will always say hello again!'
        }
    }
    
}