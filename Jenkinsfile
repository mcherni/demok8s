podTemplate(
    label: 'jenkins-agt', 
    inheritFrom: 'default',
    containers: [
        containerTemplate(
            name: 'node', 
            image: 'mycluster.icp:8500/default/node:8-stretch-slim',
            ttyEnabled: true,
            command: 'cat'
        ),
        containerTemplate(
            name: 'docker', 
            image: 'mycluster.icp:8500/default/docker:18.03.1',
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
    node('jenkins-agt') {
        withCredentials([usernamePassword(credentialsId: 'docker', passwordVariable: 'creds_psw', usernameVariable: 'creds_usr'),usernamePassword(credentialsId: 'host', passwordVariable: 'hostip', usernameVariable: 'hostdns'),\
                usernamePassword(credentialsId: 'repository', passwordVariable: 'repo_port', usernameVariable: 'repo_url'), usernamePassword(credentialsId: 'appname', passwordVariable: 'imagename', usernameVariable: 'namespace'),\
                usernamePassword(credentialsId: 'kubernetes', passwordVariable: 'k8sport', usernameVariable: 'k8sip')]) {
        def commitId
        def endpoint
        def registryIp
        def appName
        def repository
        
        stage ('Extract') {
            checkout scm
            commitId = sh(script: 'git rev-parse --short HEAD', returnStdout: true).trim().toString()
        }
        stage ('Build Appliction') {
            container ('node') {
                sh 'npm install --production --silent'
            }
        }
        stage ('Build Applicaion Docker Image & Publish to Registry') {
             container ('docker') {
                
                    endpoint = "https://${repo_url}:${repo_port}"
                    registryIp = "${repo_url}:${repo_port}"
                    appName="${namespace}/${imagename}"
                    docker.withRegistry("${endpoint}", 'docker') {
                        def pcImg = docker.build("${registryIp}/${appName}:${commitId}")
                        pcImg.push()
                    }   
            }
         }
        
        stage ('Deploy Application Release') {
            container ('helm') {
              
                    sh 'echo "${hostip}" "${hostdns}" >> /etc/hosts'        
                    sh "cloudctl login -a https://${k8sip}:${k8sport} --skip-ssl-validation -u ${creds_usr} -p ${creds_psw} -n ${namespace}"
                    repository = "${registryIp}/${appName}"
                    sh "helm upgrade --install --wait --tls --set image.repository=${repository},image.tag=${commitId} demo chart/demo"
              
            }
        }
    }
    }
}