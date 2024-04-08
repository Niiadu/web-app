// pipeline{
//   agent any
//   tools{
//     maven "maven3.8.8"
//   }

//     stages{
//       stage("1. Git clone from repo"){
//         steps{
//          sh "echo start of git clone"
//          git branch: 'main', url: 'https://github.com/JOMACS-IT/web-app.git'
//          sh "echo end of git clone"
//         }
//       }
      
//       stage("2. Build from Maven"){
//         steps{
//           sh "echo start building from Maven"
//           sh "mvn clean package"
//           sh "echo end of build"
//         }  
//       }
      
//       stage("3. Code Scan"){
//         steps{
//           sh "echo start of code scan"
//           sh "mvn sonar:sonar"
//           sh "echo end of code scan"
//         }  
//       }
      
//       stage("4. Store Artifacts"){
//         steps{
//            sh "echo Deploy Artifact"
//            sh "mvn deploy"
//         }  
//       }
      
//       stage("5. Deploying to Tomcat in UAT"){
//         steps{
//            sh "echo start deploying to server in UAT Env"
//            deploy adapters: [tomcat9(credentialsId: 'tomcat_cred', path: '', url: 'http://18.117.162.68:9090')], contextPath: null, war: 'target/*.war'
//         }  
//       }
      
//       stage("6. Email Notification"){
//         steps{
//             sh "echo Email Notification to DevOps Team"
//             emailext body: 'The Deployment is Successful', subject: 'Deployment Success', to: 'info@jomacsit.com'
//         }  
//       }
//     }
// }



/*
node{
    def MHD = tool name: "maven3.8.4"
    stage('code'){
        git branch: 'development', url: 'https://github.com/team16flight/web-app.git'
    }
    stage('BUILD'){
       sh "${MHD}/bin/mvn clean package"
 
    }
        
    
    stage('deploy'){
  sshagent(['tomcat']) {
  sh "scp -o StrictHostKeyChecking=no target/*war ec2-user@172.31.15.31:/opt/tomcat9/webapps/"
}
}
stage('email'){
emailext body: '''Build is over

JOMACS 
437212483''', recipientProviders: [developers(), requestor()], subject: 'Build', to: 'tdapp@gmail.com'
}
}
 */

pipeline {
	agent any

	tools {
		maven 'maven3'
	}

	stages {
		stage ('Git Clone') {
			steps {
				git branch: 'main', url: 'https://github.com/Niiadu/web-app.git'
			}	
		}

		stage ('Testing with Maven') {
			steps {
				sh 'mvn test'
			}
		}

		stage ('Package with Maven') {
			steps {
				sh 'mvn package'
			}
		}

		stage ('Sonar Analysis') {
			environment {
				scannerHome = tool 'sonarqube'
			}
			steps {
				withSonarQubeEnv('sonar') {
					sh  '${scannerHome}/bin/sonar-scanner -Dsonar.projectKey=jomac-application'
				}
			}
		}

		stage("Quality Gate") {
            steps {
              	timeout(time: 1, unit: 'HOURS') {
                waitForQualityGate abortPipeline: true
               	}
            }
        }

		stage ('Uploading to Nexus') {
			steps {
				nexusArtifactUploader artifacts: [[artifactId: 'maven-web-application', classifier: '', file: '/var/lib/jenkins/workspace/first-build/target/web-app.war', type: 'war']], credentialsId: 'nexus-credentials', groupId: 'com.mt', nexusUrl: '54.160.113.61:8081/', nexusVersion: 'nexus3', protocol: 'http', repository: 'webapp-release', version: '3.0.6-RELEASE'
			}
		}

		stage ('Deploy to Tomcat') {
			steps {
				deploy adapters: [tomcat9(credentialsId: 'tomcat-credentials', path: '', url: 'http://54.91.43.5:8080/')], contextPath: null, war: 'target/*.war'
			}
		}
	}

	post {
        success {
            slackSend channel: 'team7-africa', color: 'good', message: "Build successful: ${currentBuild.fullDisplayName}"
        }
        failure {
            slackSend channel: 'team7-africa', color: 'danger', message: "Build failed: ${currentBuild.fullDisplayName}"
        }
    }
}
