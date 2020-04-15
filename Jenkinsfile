def targetMail = "jenkinsautomationuser@gmail.com"
def emailDevOps(targetMail, msg) {
    emailext body: "${msg}\nplease look into the build: ${BUILD_URL}", subject: "${currentBuild.result}: ${BUILD_TAG}", to: "${targetMail}"
}
pipeline {
  agent {label "slave"}
    options {
		timestamps()
	}
        parameters {
                string(name: 'Docker_image_base_version',defaultValue: '1',description:"docker base image version for pet-clinic application.\nex: 1,2,3 as in 1.0,2.1,3.5")       
        }
     
stages {
  stage("build") {
    steps {
      sh "pwd"
      sh "ls -lrtha"
      dir("spring-petclinic") {
      sh "./mvnw clean package -Dcheckstyle.skip"
      }
    }
  }
  stage ("sonar cloud code analysis") {
          steps {
                  dir("spring-petclinic") {
                sh "./mvnw verify sonar:sonar -Dcheckstyle.skip"
                  }
          }
  }
  stage("deploy") {
     when {
  branch 'master'
}
          steps {
           
      dir("spring-petclinic") {
    sh """
    docker build -t nagarajub123/pet-clinic:${params.Docker_image_base_version}.${BUILD_NUMBER} .
    docker ps -q --filter name=pet-clinic_container|grep -q . && (docker stop pet-clinic_container && docker rm pet-clinic_container) ||echo pet-clinic_container doesn\\'t exists
    docker run --name pet-clinic_container -d -p 8080:8080 nagarajub123/pet-clinic:${params.Docker_image_base_version}.${BUILD_NUMBER}
    """
      }
    }
  }
  stage('publish to docker registry') {
      when {
  branch 'master'
}
          steps {
          withCredentials([usernamePassword(credentialsId: 'hub.docker',passwordVariable: 'docker_PSW', usernameVariable: 'docker_USR')]) {
          sh """
          docker login -u ${docker_USR} -p ${docker_PSW}
          docker images
          docker push nagarajub123/pet-clinic:${params.Docker_image_base_version}.${BUILD_NUMBER}
          """
      }
      }
  }
        stage('publish to azure container registry') {
      when {
  branch 'master'
}
          steps {
          withCredentials([usernamePassword(credentialsId: 'acr_creds',passwordVariable: 'acr_PSW', usernameVariable: 'acr_USR')]) {
          sh """
          docker login myfirstprivateregistry.azurecr.io -u ${acr_USR} -p ${acr_PSW}
          docker tag nagarajub123/pet-clinic:${params.Docker_image_base_version}.${BUILD_NUMBER} myfirstprivateregistry.azurecr.io/pet-clinic:${params.Docker_image_base_version}.${BUILD_NUMBER}
          docker push myfirstprivateregistry.azurecr.io/pet-clinic:${params.Docker_image_base_version}.${BUILD_NUMBER}
          """
      }
      }
  }
}
post {
    always{
         deleteDir()
         emailDevOps(targetMail, "This is the mail notification.")
         
    }
}
}
