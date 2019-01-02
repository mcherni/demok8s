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
            image: 'ibmcom/ibm-cloud-developer-tools-amd64',
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
                sh 'kubectl config set-cluster mycluster --server=https://mycluster.icp:8001 --insecure-skip-tls-verify=true'
                sh 'kubectl config set-context mycluster-context --cluster=mycluster'
                sh 'kubectl config set-credentials mcherni --token=eyJ0eXAiOiJKV1QiLCJhbGciOiJSUzI1NiJ9.eyJhdF9oYXNoIjoianVxcjVkMHl6bnNibjljaHp5ejUiLCJyZWFsbU5hbWUiOiJjdXN0b21SZWFsbSIsInVuaXF1ZVNlY3VyaXR5TmFtZSI6Im1jaGVybmkiLCJpc3MiOiJodHRwczovL215Y2x1c3Rlci5pY3A6OTQ0My9vaWRjL2VuZHBvaW50L09QIiwiYXVkIjoiZDZmOWY1MTI4NzMxNjMzYjUwNzE3MmNjODFmMjc5ZWIiLCJleHAiOjE1NDY0NTI0NTcsImlhdCI6MTU0NjQyMzY1Nywic3ViIjoibWNoZXJuaSIsInRlYW1Sb2xlTWFwcGluZ3MiOltdfQ.qbCvQZaIgyXt1dC2jkaEnuOx6qyQUJquZHa7YDQ9EM6nc5z7lWOOUmSTB5f65VfJjikrKUNBUuzjSf9-shrhjxGe5u08xeP8c2e4ykRrVkgzSzMYHD9LahUekMg4-9oXp5Vs0L6mi-PxVO_I7t3E-OLxTeQPLeiDl3vwHb3uou6FeA8umYoUMMQqtP-DWOJPbz2YY8h9Fdf8gWvJAguCRFoGOw_lUIrKZx4ooS6pWi-nOnNXgnDt3dWiUVYZW2fTroYg5YK-Cwv_Ik1-X84umLdpx-qMfFXMecmurcvEqNBImUixmbjG0G3xcdQeprGkCJgb9KBzRtUsk6mMUBMFEQ'
                sh 'kubectl config set-context mycluster-context --user=mcherni --namespace=cert-manager'
                sh 'kubectl config use-context mycluster-context'
                sh 'ls ~/.kube'
                //sh 'cp ~/.kube/mycluster/*.pem ~/.helm/'
                // sh 'chmod 755 ~/.helm/*.pem'
                // //sh "cloudctl login -a https://mycluster.icp:8443 --skip-ssl-validation -u mcherni -p P@ssw0rd -n default"
                // sh "helm init --client-only --skip-refresh"
                // sh "helm version --tls"
                // sh "helm upgrade --install --wait --tls --set image.repository=${repository},image.tag=${commitId} demo chart/demo"
            }
        }
    }
}