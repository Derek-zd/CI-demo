
node('go-jnlp') {
    stage('Git CLone') {
        checkout scm
        script {
            build_tag = sh(returnStdout: true, script: 'git rev-parse --short HEAD').trim()
        }
    }
    
    stage('Build Binary') {
        sh "cd ${WORKSPACE} && /usr/local/go/bin/go build -o cicd-demo ."
    }

    stage('Build Docker Image') {
        sh "docker build -t motecshine/cicd-demo-${env.BRANCH_NAME}:${build_tag} ."
    }

    stage('Commit Docker Image') {
        withCredentials([usernamePassword(credentialsId: 'DockerHub', passwordVariable: 'dockerHubPassword', usernameVariable: 'dockerHubUser')]) {
            sh "docker login -u ${dockerHubUser} -p ${dockerHubPassword}"
            sh "docker push motecshine/cicd-demo-${env.BRANCH_NAME}:${build_tag}"
        }
    }

    stage('Migrate Kubernetes Yaml Config') {
        sh "sed -i 's/<build_tag>/${build_tag}/' ./k8s/deployement.yaml"
    }

    stage('Deploy To K8S') {     
       def name = sh "kubectl  get pod -l app=cicd-demo jsonpath='{.items[0].metadata.name}'"
       print(name)
    }
}