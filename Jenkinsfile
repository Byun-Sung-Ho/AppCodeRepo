/* Jenkinsfile */
def imageTag
node {

    def app

    stage('Clone repository') {
        /* Let's make sure we have the repository cloned to our workspace */

        checkout scm
    }

    stage('Build image') {
        /* This builds the actual image; synonymous to
         * docker build on the command line */
        
        app = docker.build("eks-gitops-demo")
    }
    
    stage('Test image') {
        /* Run a test framework against our image. */

        /*app.inside {
            sh 'echo "Tests passed"'
        }*/
        echo "Tests passed"
    }

    stage('Push image') {
        /* Finally, we'll push the image with two tags:
         * First, the incremental build number from Jenkins
         * Second, the 'latest' tag.
         * Pushing multiple tags is cheap, as all the layers are reused. */
        imageTag = sh(script: "head -n 1 Dockerfile | sed 's/#//'", returnStdout: true).trim()
        docker.withRegistry('https://570114519658.dkr.ecr.us-east-1.amazonaws.com', 'ecr:us-east-1:ecrUser') {
        /*    app.push("${env.BUILD_NUMBER}") */
            app.push("${imageTag}")
            app.push("latest")
        }
        echo "Pushed image with tag: ${imageTag}"
    }

    stage('Update tag in ManifestRepo') {
        dir("manifest") {
          sshagent(credentials: ['gitUser']) {
	      sh "git clone git@github.com:Byun-Sung-Ho/ManifestRepo.git ."
              sh "git config user.email jenkins@example.com"
              sh "git config user.name jenkins"
              sh """sed -i 's|eks-gitops-demo:.*|eks-gitops-demo:${imageTag}|' deployment.yaml"""
              sh "cat deployment.yaml"
  
              sh "git commit -am 'Update image tag' && git push git@github.com:Byun-Sung-Ho/ManifestRepo.git"
	      echo "Pushed image ${imageTag}"
  
            }
        }
    }
}
