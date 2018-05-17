timestamps {
    podTemplate(
        label: 'ubuntubuid',
        containers: [
            containerTemplate(name: 'ubuntu', image: 'ubuntu', ttyEnabled: true, command: 'cat', args: '-v'),
            containerTemplate(name: 'docker', image: 'docker', ttyEnabled: true, command: 'cat', args: '-v')
        ],
        volumes: [
            // this is necessary for docker build to work
            hostPathVolume(hostPath: '/run/docker.sock', mountPath: '/run/docker.sock')
        ]) {
        echo 'Hello'
        node ('ubuntubuid') {
            echo 'In Node'
            container ('ubuntu') {
                echo 'In Container ubuntu'
            }
            container ('docker') {
                echo 'In Container docker'

                sh """
                echo
                docker version
                echo
                docker images
                echo
                docker ps
                echo
                """
            }
        }
    }
}
