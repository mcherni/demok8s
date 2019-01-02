podTemplate(
    label: 'mypod', 
    inheritFrom: 'default',
    containers: [
        containerTemplate(
            name: 'node', 
            image: 'node:8-stretch-slim',
            ttyEnabled: true,
            command: 'cat'
        ),
        containerTemplate(
            name: 'docker', 
            image: 'docker:18.03',
            ttyEnabled: true,
            command: 'cat'
        ),
        containerTemplate(
            name: 'helm', 
            image: 'mycluster.icp:8500/default/icp-tools:3.1.1',
            ttyEnabled: true,
            command: 'cat'
        )
    ],
    volumes: [
        hostPathVolume(
            hostPath: '/var/run/docker.sock',
            mountPath: '/var/run/docker.sock'
        )
    ]
) {
    node('mypod') {
        
        def commitId
        def endpoint
        //def registryIp = "mycluster.icp:8500"
        def registryIp
        //def appName = "default/demo"
        def appName
        def repository
        
        stage ('Extract') {
            checkout scm
            commitId = sh(script: 'git rev-parse --short HEAD', returnStdout: true).trim()
        }
        stage ('Build Appliction') {
            container ('node') {
                sh 'npm install --production --silent && ls'
            }
        }
        stage ('Build Applicaion Docker Image & Publish to Registry') {
             container ('docker') {
                 withCredentials([usernamePassword(credentialsId: 'repository', passwordVariable: 'repo_port', usernameVariable: 'repo_url'), usernamePassword(credentialsId: 'appname', passwordVariable: 'imagename', usernameVariable: 'namespace')]) {
                    endpoint = "https://${repo_url}:${repo_port}"
                    registryIp = "${repo_url}:${repo_port}"
                    appName="${namespace}/${imagename}"

                    docker.withRegistry("${endpoint}", 'docker') {
                        def pcImg = docker.build("${registryIp}/${appName}:${commitId}")
                        pcImg.push()
                    }
                 }
                 
            }
         }
        
        stage ('Deploy Application Release') {
            container ('helm') {
                withCredentials([usernamePassword(credentialsId: 'docker', passwordVariable: 'creds_psw', usernameVariable: 'creds_usr'),usernamePassword(credentialsId: 'host', passwordVariable: 'hostip', usernameVariable: 'hostdns'),\
                usernamePassword(credentialsId: 'repository', passwordVariable: 'repo_port', usernameVariable: 'repo_url'), usernamePassword(credentialsId: 'appname', passwordVariable: 'imagename', usernameVariable: 'namespace')]) {
                    //sh "echo ${creds_usr}:${creds_psw}"
                    sh "echo ${hostip} ${hostdns}" >> "/etc/hosts"
                    sh "echo ${creds_usr}:${creds_psw}"
                    sh "cloudctl login -a https://${repo_url}:8443 --skip-ssl-validation -u ${creds_usr} -p ${creds_psw} -n ${namespace}"
                    //sh 'ls ~/.kube'
                    //sh 'ls ~/.helm'
                    repository = "${registryIp}/${appName}"
                    sh "helm upgrade --install --wait --tls --set image.repository=${repository},image.tag=${commitId} demo chart/demo"
                }
                // 
            }
        }
    }
}