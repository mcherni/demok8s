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
            image: 'ibmcom/k8s-helm:v2.6.0',
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
        stage ('Extract') {
            checkout scm
            commitId = sh(script: 'git rev-parse --short HEAD', returnStdout: true).trim()
        }
        stage ('Build Appliction') {
            container ('node') {
                sh 'npm install --production --silent && ls'
            }
        }
        // docker.withRegistry('https://mycluster.icp:8500/', 'docker'){
        //     stage "Build IMAGE"
        //     def pcImg = docker.build("mycluster.icp:8500/default/demo:${commitId}", "-f Dockerfile.ppc64le .")
        //     sh "cp /root/.dockercfg ${HOME}/.dockercfg"
        //     pcImg.push()
        // }
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
                sh "/helm init --client-only --skip-refresh"
                
                sh "/helm upgrade --install --wait --set image.repository=${repository},image.tag=${commitId} demo chart/demo"
            }
        }
    }
}