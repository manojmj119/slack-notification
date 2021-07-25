pipeline {

     agent {
          node{
               label 'master'
          }
     }
     
    stages {
         
      stage('SonarCloud_static_code_analysis') {
  environment {
    SCANNER_HOME = tool 'sonarqube'
    ORGANIZATION = "slack-notification"
    PROJECT_NAME = "slack-notification"
  }
  steps {
    withSonarQubeEnv('sonarqube') {
        sh '''$SCANNER_HOME/bin/sonar-scanner -Dsonar.organization=$ORGANIZATION \
        -Dsonar.java.binaries=build/classes/java/ \
        -Dsonar.projectKey=$PROJECT_NAME \
        -Dsonar.sources=.'''
    }
  }
}
          stage("Quality Gate") {
        steps {
         timeout(time: 1, unit: 'MINUTES') {
            
             waitForQualityGate abortPipeline: true
      }
        }
      }
     
        
        stage('Code_testing') {
            steps {
                sh 'mvn test'
            }
            
        }
        
        stage('Build') {
            steps {
                sh 'mvn install'
            }
            
        }
        
        
        stage('Deploy') {
            steps {
                
                sh 'mvn deploy'
            }
            
        }           
    }
      post {
       success {
         notifySuccess()
       }
       unstable {
         notifyUnstable()
       }
       failure {
         notifyFailed()
       }
     }
         
}


def notifyBuild(String buildStatus = 'STARTED', String colorCode = '#5492f7', String notify = '') {

  def project = 'slack-notification'
  def channel = "prabhu.p"
  def base = "https://github.com/manojmj119/slack-notification.git/${project}/commits/"

  def commit = sh(returnStdout: true, script: 'git log -n 1 --format="%H"').trim()
  def link = "${base}${commit}"
  def shortCommit = commit.take(6)
  def title = sh(returnStdout: true, script: 'git log -n 1 --format="%s"').trim()
  def subject = "<${link}|${shortCommit}> ${title}"

  def summary = "${buildStatus}: Job <${env.RUN_DISPLAY_URL}|${env.JOB_NAME} [${env.BUILD_NUMBER}]>\n${subject} ${notify}"

  slackSend (channel: "${channel}", color: colorCode, message: summary)

}

def author() {
  return sh(returnStdout: true, script: 'git log -n 1 --format="%an" | awk \'{print tolower($1);}\'').trim()
}


def notifyStarted() {
  notifyBuild()
}

def notifySuccess() {
  notifyBuild('SUCCESS', 'good')
}

def notifyUnstable() {
  notifyBuild('UNSTABLE', 'warning', "\nAuthor: @${author()} <${RUN_CHANGES_DISPLAY_URL}|Changelog>")
}

def notifyFailed() {
  notifyBuild('FAILED', 'danger', "\nAuthor: @${author()} <${RUN_CHANGES_DISPLAY_URL}|Changelog>")
}

