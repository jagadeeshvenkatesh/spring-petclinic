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
        stage('Selenium') {
            agent {
                docker {
                    image 'liatrio/selenium-firefox'
                    args '--network=demodeploymentpipeline_default'
                }
            }
            steps {
                sh 'ruby petclinic_spec.rb'
            }
        }
        stage('Build Container') {
            agent any
            steps {
                sh 'docker build -t petclinic-tomcat .'
            }
        }
       stage('Run container') {
           agent {
               docker {
                   image 'alpine'
               }
           }
           steps {
               sh 'docker rm -f petclinic-tomcat || true'
               sh 'docker run -p 80:8080 -d --network=demodeploymentpipeline_default --name petclinic-tomcat petclinic-tomcat'
               input 'Should be accessible at http://localhost/petclinic/'
               sh 'docker stop petclinic-tomcat'
           }
       }
       stage('Deploy to bluemix') {
            agent {
                    docker {
                        image 'liatrio/cf-cli'
                    }
            }
            steps {
                withCredentials([usernamePassword(credentialsId: 'bluemix', passwordVariable: 'PASSWORD', usernameVariable: 'USERNAME')]) {
                    sh 'cf api https://api.ng.bluemix.net'
                    sh "cf login -u ${env.USERNAME} -p ${env.PASSWORD}"
                    sh 'cf push'
                }
            }
        }
    }
}
