@Library('Bluemix') _
pipeline {
    agent none
    stages {
       stage('Build') {
           agent {
               docker {
                   image 'maven:3.5.0'
                   args '--network=demodeploymentpipeline_default'
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
                   args '--network=demodeploymentpipeline_default'
               }
           }
           steps {
               sh '/opt/sonar-runner-2.4/bin/sonar-runner'
           }
       }
       stage('Build and Run container') {
              agent any
              steps {
                  sh 'docker build -t petclinic-tomcat .'
                  sh 'docker rm -f petclinic-tomcat || true'
                  sh 'docker run -p 18887:8080 -d --network=demodeploymentpipeline_default --name petclinic-tomcat petclinic-tomcat'
              }
          }
          stage('Selenium') {
              agent {
                  docker {
                      image 'liatrio/selenium-firefox'
                      args '--network=demodeploymentpipeline_default'
                  }
              }
              steps {
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
        //   stage('Deploy to bluemix') {
        //        agent {
        //                docker {
        //                    image 'liatrio/cf-cli'
        //                }
        //        }
        //     steps {
        //         DeployToBluemix()
        //     }
        // }
        stage('Add Version to manifest') {
            agent any
            steps {
                build job:"sample-manifest-pipeline"
            }
        }
    }
}
