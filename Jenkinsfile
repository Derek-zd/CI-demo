properties([parameters([string(defaultValue: '0.0.0', description: '服务版本号', name: 'serverversion')])])
node('go111-npm') {
    stage('Git CLone') {
        checkout scm
        script {
            version="${params.serverversion}"
            build_tag = sh(returnStdout: true, script: 'git rev-parse --short HEAD').trim()
            sh "echo 用户指定version：${version}"  
            if("${version}"=='0.0.0' || "${version}"=='null' ){
                version="${build_tag}"
            }

        }
    }
    
    stage('Build Binary') {
        sh "cd ${WORKSPACE} && /usr/local/go/bin/go build -o cicd-demo ."
    }

    stage('Build Docker Image') {
        sh "docker build -t repository.parkingwang.com/default/cicd-demo-${env.BRANCH_NAME}:${build_tag} ."
    }
    
    stage('Notify K8S Deployagent') {
        TimeZone.getTimeZone('PRC')
        def date = new Date(); 
        deployed_time = date.getTime()
        // 存储历史上线日志
        def git_comments = sh(returnStdout: true, script: "git log -n 1 --pretty=format:\"%cd  %h  %s\" --date=short --since=\"last day\"")
        def remove_blank_comments = java.net.URLEncoder.encode(git_comments, "UTF-8")
        def git_commit_date = sh(returnStdout: true, script: "git log -n 1 --pretty=format:\"%cd\" --date=short --since=\"last day\"")
        def url = ""
        if (env.BRANCH_NAME == 'master') {
            url = "http://k8s-porter.frm:8443"
        } else {
            url = "http://117.39.63.66:9502"
        }
        def response = httpRequest "/v1/save-deployed?commit_date=${git_commit_date}&branch_name=${env.BRANCH_NAME}&resource_type=Deployment&commit_id=${build_tag}&project_name=cicd-demo&commit_msg=${remove_blank_comments}&ns=default&deployed_time=${deployed_time}&version=${version}"
        println(response)       
    }
}