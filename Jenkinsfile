pipeline {
  agent any

tools {
    nodejs "Nodejs"
}

 environment {
    DOCKER_REGISTRY = "docker.io"
    DOCKERHUB_CREDENTIALS = credentials('DOCKER_HUB_CREDENTIAL')
    VERSION = "${env.BUILD_ID}"
  }

  stages {

 stage('Install Dependencies') {
      steps {
//NPM clean install what it will do, it will check all the dependencies 
//that are available in package.json file and install all those for you.
        sh 'npm ci' 
      }
    }

    stage('Build Project') {
      steps {
 // run the NPM run build to make sure that you're able to build the project successfully.
        sh 'npm run build'
      }
    }
//So taking the Docker hub credentials, you log in to Docker Hub, you build an image with the new version
//and push that image to Docker Hub with that version.

    stage('Docker Build and Push') {
      steps {
          sh 'echo $DOCKERHUB_CREDENTIALS_PSW | docker login -u $DOCKERHUB_CREDENTIALS_USR --password-stdin'
          sh 'docker build -t jiheneayoub1/food-delivery-app-fe:${VERSION} .'
          sh 'docker push jiheneayoub1/food-delivery-app-fe:${VERSION}'
      }
    }
//delete the directory

     stage('Cleanup Workspace') {
      steps {
        deleteDir()
      }
    }


     stage('Update Image Tag in GitOps') {
  //you have to do task fot CD : from the deployment-folder/AWS, this is the URL, use git SSH for pushing this image.
  //With the new version, you have to push it to master 
      steps {
         checkout scmGit(branches: [[name: '*/master']], extensions: [], userRemoteConfigs: [[credentialsId: 'git-ssh', url: 'git@github.com:AyoubNina/deployment-folder.git']])
        script {
          // then you have Set the new image tag with the Jenkins build number
       sh '''
          sed -i "s/image:.*/image: jiheneayoub1\\/food-delivery-app-fe:${VERSION}/" aws/angular-manifest.yml
        '''
//When our pipeline is successfull and we changer the version of our image, the deploymentfolder is updated with that build number as the version number with the tag 'update immage tag' 
//and of course the version of our image in the yaml file.And wherever there's a change in the version number,ArgoCD comes and pull this folder ....
          //
          //
          sh 'git checkout master'
          sh 'git add .'
          sh 'git commit -m "Update image tag"'
        sshagent(['git-ssh'])
            {
                  sh('git push')
            }
        }
      }
    }
  }

}


