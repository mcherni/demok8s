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
        def registryIp = "mycluster.icp:8500"
        def appName = "default/demo"
        def repository = "${registryIp}/${appName}"
        environment {
            creds = credentials('docker')
        }
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
                 docker.withRegistry('https://mycluster.icp:8500/', 'docker') {
                    def pcImg = docker.build("${registryIp}/${appName}:${commitId}")
                    pcImg.push()
                    }
            }
         }
        
        stage ('Deploy Application Release') {
            container ('helm') {
                sh 'echo "149.81.85.219 mycluster.icp" >> /etc/hosts'
                sh "echo ${creds_usr}:${creds_psw}"
                sh "cloudctl login -a https://mycluster.icp:8443 --skip-ssl-validation -u mcherni -p P@ssw0rd -n default"
                sh 'ls ~/.kube'
                sh 'ls ~/.helm'

                // sh "helm init --client-only --skip-refresh"
                // sh "helm version --tls"
                sh "helm upgrade --install --wait --tls --set image.repository=${repository},image.tag=${commitId} demo chart/demo"
            }
        }
    }
}