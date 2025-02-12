def imageTag
node {
    def app

    stage('Clone repository') {
        /* Let's make sure we have the repository cloned to our workspace */
        checkout scm
    }

    stage('Build image') {
        /* This builds the actual image; synonymous to docker build on the command line */
        app = docker.build("eks-gitops-demo")
    }

    stage('Test image') {
        /* Run a test framework against our image. */
        app.inside {
            sh 'echo "Tests passed"'
        }
    }

    stage('Push image') {
        /* Finally, we'll push the image with two tags:
         * First, the incremental build number from Jenkins
         * Second, the 'latest' tag.
         * Pushing multiple tags is cheap, as all the layers are reused. */
        imageTag = sh(script: "head -n 1 Dockerfile | sed 's/#//'", returnStdout: true).trim()

        // ECR 로그인
        docker.withRegistry('https://495599735720.dkr.ecr.us-west-2.amazonaws.com/eks-gitops', 'ecr:us-west-2:ecrUser') {
            app.push("${imageTag}")   // 빌드된 이미지 푸시
            app.push("latest")        // 최신 태그 푸시
        }

        echo "Pushed image with tag: ${imageTag}"
    }
}

