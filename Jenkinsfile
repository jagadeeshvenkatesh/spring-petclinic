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
       stage('Build container') {
             agent any
             steps {
                 sh 'docker build -t petclinic-tomcat .'
             }
         }
         stage('Run local container') {
             agent any
             steps {
                 sh 'docker rm -f petclinic-tomcat-temp || true'
                 sh 'docker run -p 18887:8080 -d --network=${LDOP_NETWORK_NAME} --name petclinic-tomcat-temp petclinic-tomcat'
             }
         }
         stage('Smoke-Test & OWASP Security Scan') {
             agent {
                 docker {
                     image 'maven:3.5.0'
                     args '--network=${LDOP_NETWORK_NAME}'
                 }
             }
             steps {
                 sh "cd regression-suite"
                 sh "mvn clean -B test -DPETCLINIC_URL=http://petclinic-tomcat:8080/petclinic/"
             }
         }
         stage('Stop local container') {
             agent any
             steps {
                 sh 'docker rm -f petclinic-tomcat-temp || true'
             }
         }
         stage('Deploy to dev') {
             agent any
             steps {
                 sh 'docker rm -f dev-petclinic || true'
                 sh 'docker run -p 18888:8080 -d --network=${LDOP_NETWORK_NAME} --name dev-petclinic petclinic-tomcat'
             }
         }
         stage('Smoke test dev') {
             agent {
                 docker {
                     image 'maven:3.5.0'
                     args '--network=${LDOP_NETWORK_NAME}'
                 }
             }
             steps {
                 sh "cd regression-suite"
                 sh "mvn clean -B test -DPETCLINIC_URL=http://dev-petclinic:8080/petclinic/"
                 echo "Should be accessible at http://localhost:18888/petclinic"
             }
         }
         stage('Deploy to qa') {
             agent any
             steps {
                 sh 'docker rm -f qa-petclinic || true'
                 sh 'docker run -p 18889:8080 -d --network=${LDOP_NETWORK_NAME} --name qa-petclinic petclinic-tomcat'
             }
         }
         stage('Smoke test qa') {
             agent {
                 docker {
                     image 'maven:3.5.0'
                     args '--network=${LDOP_NETWORK_NAME}'
                 }
             }
             steps {
                 sh "cd regression-suite"
                 sh "mvn clean -B test -DPETCLINIC_URL=http://qa-petclinic:8080/petclinic/"
                 echo "Should be accessible at http://localhost:18889/petclinic"
                 input 'Deploy to Prod?'
             }
         }
         stage('Deploy to prod') {
             agent any
             steps {
                 sh 'docker rm -f prod-petclinic || true'
                 sh 'docker run -p 18890:8080 -d --network=${LDOP_NETWORK_NAME} --name prod-petclinic petclinic-tomcat'
             }
         }
         stage('Smoke Test prod') {
             agent {
                 docker {
                     image 'maven:3.5.0'
                     args '--network=${LDOP_NETWORK_NAME}'
                 }
             }
             steps {
                 sh "cd regression-suite"
                 sh "mvn clean -B test -DPETCLINIC_URL=http://prod-petclinic:8080/petclinic/"
                 echo "Should be accessible at http://localhost:18890/petclinic"
             }
         }
    }
}
