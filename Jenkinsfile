pipeline {
  agent {
    node {
      label 'master'
    }
  }

  parameters {
    string(name: 'JENKINSDIR', defaultValue: '/Users/abhijithvg/Desktop/WorkFolder/Schogini-Training/Demos/CICD-Pipeline/forjenkins/jenkins_home', description: 'The project path in the host machine')
    string(name: 'MVNCACHE', defaultValue: '/Users/abhijithvg/Training/Collabera/DevOpsPlus1/.m2', description: 'Maven repository cache in the host machine')
    string(name: 'DOCKER_U', defaultValue: 'abhijithvg', description: 'Docker Hib Username')
  }

  stages {

      stage('Clone') {
        steps {
          git url: 'https://github.com/schogini/java-war-junit-acceptance-add.git'
          sh '''sed -i \"s/BUILD_ID/${BUILD_ID}/\" src/main/webapp/index.jsp'''
          sh '''sed -i \"s/BUILD_ID/${BUILD_ID}/\" kubernetes/deploy-svc.yml'''
          sh '''sed -i \"s/BUILD_ID/${BUILD_ID}/\" ansible/docker-image-creation.yml'''
          sh '''sed -i \"s/BUILD_ID/${BUILD_ID}/\" ansible/sample.yml'''

        }
      }

      stage('Unit Tests') {
        steps {
         sh "sudo docker run --rm -i -v ${params.MVNCACHE}:/root/.m2 -v ${params.JENKINSDIR}/workspace/${JOB_BASE_NAME}:/project -w /project maven:3.6.3-jdk-8-openj9 mvn clean test"
        }
      }
      stage('Build Java App') {
        steps {
          sh "sudo docker run --rm -i -v ${params.MVNCACHE}:/root/.m2 -v ${params.JENKINSDIR}/workspace/${JOB_BASE_NAME}:/project -w /project maven:3.6.3-jdk-8-openj9 mvn package"
       }
      }

      stage('Docker Image Context') {
        steps {
          sh 'cp -f target/SampleJava.war tomcat/webapp.war'
        }
      }

      stage('Build Image') {
          agent {
              docker {
                  image "abhijithvg/ansible-with-docker-ws"
                  args "-v /var/run/docker.sock:/var/run/docker.sock -w /etc/ansible -e 'BUILD_ID=${env.BUILD_ID}' -e 'DOCKER_USER=${params.DOCKER_U}'"
              }
          }
          steps {
              sh('ansible-playbook ansible/build-image.yml')
          }
      }

      stage('Push Docker Image') {
          steps {
              withCredentials([usernamePassword(credentialsId: 'docker-credentials', passwordVariable: 'DOCKER_PASSWORD', usernameVariable: 'DOCKER_USERNAME')]) {
                  sh("docker login -u=${DOCKER_USERNAME} -p=${DOCKER_PASSWORD}")
                  sh("docker push ${DOCKER_USERNAME}/tomcat:pipeline-${env.BUILD_ID}")
              }

              //To remove the image locally
              //sh('docker rmi ${params.DOCKER_U}/tomcat:pipeline-${env.BUILD_ID}')
          }
      }
      stage('Deploy to Docker Tomcat') {
        steps {
      sh "sudo docker inspect my-tcc2 >/dev/null 2>&1 && sudo docker rm -f my-tcc2 || echo No container to remove. Proceed."
      sh "sudo docker run -id -p 7081:8080 --name my-tcc2 docker.io/${params.DOCKER_U}/tomcat:pipeline-${BUILD_ID}"
        }
      }

      stage('Deploy to Kubernetes') {
        steps {
      #sh "sudo kubectl apply -f kubernetes/deploy-svc.yml"
      #sh "sudo kubectl set image deploy/webapp-demo-deploy webapp=docker.io/${params.DOCKER_U}/tomcat:pipeline-${BUILD_ID}"
          sh("kubectl set image deploy/ab-deploy-tomcat tomcat=${params.DOCKER_U}/tomcat:pipeline-${env.BUILD_ID}")
        }
      }

    }
}