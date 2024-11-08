// properties([
//     parameters: [
//         string(
//             name: 'namespace',
//             description: 'Enter your target namespace:',
//             defaultValue: 'devlake',
//             trim: true
//     )]
// ])

podTemplate(
        inheritFrom: 'default',
        containers: [
                containerTemplate(name: 'docker', image: 'docker', command: 'cat', ttyEnabled: true),
                containerTemplate(name: 'gradle', image: 'gradle:latest', command: 'cat', ttyEnabled: true)
        ],
        volumes: [
                hostPathVolume(mountPath: '/var/run/docker.sock', hostPath: '/var/run/docker.sock')
        ]
) {
    node(POD_LABEL) {
        checkout scm
        // stage('Deployment Trigger') {
        //     input "Trigger deployment?"
        // }
        stage('Build') {
            container('gradle') {
                sh 'gradle build -x test -x processTestAot -x processAot'
            }
        }
        stage('Install Build Tools') {
            withKubeConfig([credentialsId: 'minikube-jenkins-robot-secret']) {
                // See https://github.com/helm/helm/releases for latest release
                sh 'curl -sLO "https://get.helm.sh/helm-v3.16.1-linux-amd64.tar.gz"'
                sh 'curl -sLO "https://dl.k8s.io/release/$(curl -sL https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"'
                sh 'tar -xvzf helm-v3.16.1-linux-amd64.tar.gz'
                sh 'chmod 700 linux-amd64/helm kubectl'
                sh './kubectl config set-context --current --namespace=devlake'
                // echo params.namespace
            }
        }
        stage('Create Docker Image') {
            container('docker') {
                sh 'docker build -t dora-poc/spring-petclinic .'
            }
        }
        stage('Deploy') {
            withKubeConfig([credentialsId: 'minikube-jenkins-robot-secret']) {
                sh './linux-amd64/helm --namespace devlake upgrade --debug --install --force --create-namespace dora-poc-app dora-poc-app'
                sh './kubectl rollout status deployment dora-poc-app --timeout=300s'
            }
        }
    }
}
