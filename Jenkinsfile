pipeline {
  agent any
   stages {
    stage ('Build') {
      steps {
        keepRunning {
        sh '''#!/bin/bash
        python3 -m venv test3
        source test3/bin/activate
        pip install pip --upgrade
        pip install -r requirements.txt
        export FLASK_APP=application
        flask run &
        '''
        }
     }
      
     post{
        success { 
          slackSend channel: 'jenkinsnotifications',
                    color: 'good',
                    message: "Successful Build Stage!"
        }
        failure  {
          slackSend channel: 'jenkinsnotifications',
                    color: 'warning',
                    message: "Build Stage Failed!"
        }
      }
   }
    stage ('test') {
      steps {
        keepRunning {
        sh '''#!/bin/bash
        source test3/bin/activate
        py.test --verbose --junit-xml test-reports/results.xml
        ''' 
        }
      }
    
      post{
        always {
          junit 'test-reports/results.xml'
        }
        success { 
          slackSend channel: 'jenkinsnotifications',
                    color: 'good',
                    message: "Successful Test Stage!"
        }
        failure  {
          slackSend channel: 'jenkinsnotifications',
                    color: 'warning',
                    message: "Test Stage Failed!"
        }
      }
    } 
    stage ('Clean'){
      agent{label 'awsDeploy'}
      steps {
        keepRunning {
        sh '''#!/bin/bash
        if [[ $(ps aux | grep -i "gunicorn" | tr -s " "| head -n 1 | cut -d " " -f 2) != 0 ]]
        then
          ps aux | grep -i "gunicorn" | tr -s " " | head -n 1 | cut -d " " -f 2 > pid.txt
          kill $(cat pid.txt)
          exit 0
        fi
        '''
        }
      }
      post{
       success { 
         slackSend channel: 'jenkinsnotifications',
                    color: 'good',
                    message: "Successful Clean Stage!"
        }
        failure  {
          slackSend channel: 'jenkinsnotifications',
                    color: 'warning',
                    message: "Clean Stage Failed!"
        }
      }
    }
    stage ('Deploy'){
      agent{label 'awsDeploy'}
      steps {
        keepRunning {
          sh '''#!/bin/bash
          pip install -r requirements.txt
          pip install gunicorn
          python3 -m 
          JENKINS_NODE_COOKIE=stayAlive gunicorn -w 4 application:app -b 0.0.0.0 --daemon
          '''
        }
        post{
       success { 
         slackSend channel: 'jenkinsnotifications',
                    color: 'good',
                    message: "Successful Deployment Stage!"
        }
        failure  {
          slackSend channel: 'jenkinsnotifications',
                    color: 'warning',
                    message: "Deployment Stage Failed!"
        }
      }
    }
  }
 }
}
