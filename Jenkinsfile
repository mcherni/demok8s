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
                 def registryIp = "mycluster.icp:8500"
                 repository = "${registryIp}/demo"

                 sh "ls && docker build -t ${repository}:${commitId} ."
                 sh "docker login ${repository}:${commitId} -u mcherni -p P@ssw0rd && docker push ${repository}:${commitId}"
             }
         }
        // stage ('Deploy Application Release') {
        //     container ('helm') {
        //         sh "/helm init --client-only --skip-refresh"
        //         sh "/helm upgrade --install --wait --set image.repository=${repository},image.tag=${commitId} demo chart/demo"
        //     }
        // }
    }
}