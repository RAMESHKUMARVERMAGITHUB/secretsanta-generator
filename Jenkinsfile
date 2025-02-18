pipeline {
    agent any
    tools{
        jdk 'jdk17'
        maven 'maven'
    }
    environment{
        SCANNER_HOME= tool 'sonar-scanner'
    }

    stages {
        stage('git-checkout') {
            steps {
                git 'https://github.com/rameshkumarvermagithub/secretsanta-generator.git'
            }
        }

        stage('Code-Compile') {
            steps {
               sh "mvn clean compile"
            }
        }
        
        stage('Unit Tests') {
            steps {
               sh "mvn test"
            }
        }
        
		stage('OWASP Dependency Check') {
            steps {
               dependencyCheck additionalArguments: '--scan ./ --disableYarnAudit --disableNodeAudit', odcInstallation: 'DP-Check'
                dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
            }
        }


        stage('Sonar Analysis') {
            steps {
               withSonarQubeEnv('sonar-server'){
                   sh ''' $SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=Santa \
                   -Dsonar.java.binaries=. \
                   -Dsonar.projectKey=Santa '''
               }
            }
        }

		 
        stage('Code-Build') {
            steps {
               sh "mvn clean package"
            }
        }

         stage('Docker Build') {
            steps {
               script{
                   withDockerRegistry(credentialsId: 'docker') {
                    sh "docker build -t  santa123 . "
                 }
               }
            }
        }

        stage('Docker Push') {
            steps {
               script{
                   withDockerRegistry(credentialsId: 'docker') {
                    sh "docker tag santa123 rameshkumarverma/santa123:latest"
                    sh "docker push rameshkumarverma/santa123:latest"
                 }
               }
            }
        }
        
        	 
        stage('Docker Image Scan') {
            steps {
               sh "trivy image rameshkumarverma/santa123:latest "
            }
        }
        stage('Docker Deploy') {
            steps {
               sh "docker run -d -p 8080:8080 rameshkumarverma/santa123:latest "
            }
        }
        //  post {
        //     always {
        //         emailext (
        //             subject: "Pipeline Status: ${BUILD_NUMBER}",
        //             body: '''<html>
        //                         <body>
        //                             <p>Build Status: ${BUILD_STATUS}</p>
        //                             <p>Build Number: ${BUILD_NUMBER}</p>
        //                             <p>Check the <a href="${BUILD_URL}">console output</a>.</p>
        //                         </body>
        //                     </html>''',
        //             to: 'jaiswaladi246@gmail.com',
        //             from: 'jenkins@example.com',
        //             replyTo: 'jenkins@example.com',
        //             mimeType: 'text/html'
        //         )
        //     }
        // }
		
		
    }
    
}
