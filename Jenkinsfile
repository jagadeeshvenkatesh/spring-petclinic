pipeline {
    agent none
    stages {
       stage('Build') {
           agent {
               docker {
                   image 'maven:3.5.0'
                   args '-e INITIAL_ADMIN_USER -e INITIAL_ADMIN_PASSWORD --network=${LDOP_NETWORK_NAME}'
               }
           }
           steps {
               configFileProvider(
                       [configFile(fileId: 'nexus', variable: 'MAVEN_SETTINGS')]) {
                   sh 'mvn -s $MAVEN_SETTINGS clean deploy -DskipTests=true -B'
               }
           }
       }
       stage('Sonar') {
           agent  {
               docker {
                   image 'sebp/sonar-runner'
                   args '-e SONAR_ACCOUNT_LOGIN -e SONAR_ACCOUNT_PASSWORD -e SONAR_DB_URL -e SONAR_DB_LOGIN -e SONAR_DB_PASSWORD --network=${LDOP_NETWORK_NAME}'
               }
           }
           steps {
               sh '/opt/sonar-runner-2.4/bin/sonar-runner -e -D sonar.login=${SONAR_ACCOUNT_LOGIN} -D sonar.password=${SONAR_ACCOUNT_PASSWORD} -D sonar.jdbc.url=${SONAR_DB_URL} -D sonar.jdbc.username=${SONAR_DB_LOGIN} -D sonar.jdbc.password=${SONAR_DB_PASSWORD}'
           }
       }
       stage('Build and Run container') {
             agent any
             steps {
                 sh 'docker build -t petclinic-tomcat .'
                 sh 'docker rm -f petclinic-tomcat || true'
                 sh 'docker run -p 18887:8080 -d --network=${LDOP_NETWORK_NAME} --name petclinic-tomcat petclinic-tomcat'
             }
         }
         stage('Selenium') {
             agent {
                 docker {
                     image 'liatrio/selenium-firefox'
                     args '--network=${LDOP_NETWORK_NAME}'
                 }
             }
             steps {
                 sh "curl http://petclinic-tomcat:8080/petclinic/"
                 sh 'ruby petclinic_spec.rb'
                 input 'Should be accessible at http://localhost:18887/petclinic/'
             }
         }
         stage('Stop container') {
             agent any
             steps {
                 sh 'docker stop petclinic-tomcat'
             }
         }
    }
}
