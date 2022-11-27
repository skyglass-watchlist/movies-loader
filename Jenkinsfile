def imageName = 'skyglass/movies-loader'
def registry = 'https://registry.slowcoder.com'

node('workers'){
    stage('Checkout'){
        checkout scm
    }

    stage('Unit Tests'){
        def imageTest= docker.build("${imageName}-test", "-f Dockerfile.test .")
        sh "docker run --rm -v $PWD/reports7:/app/reports ${imageName}-test"
        sh "chown -R \$(id -u ${USER}):\$(id -g ${USER}) $PWD/reports7"
        sh "ls -la $PWD/reports6"
        junit allowEmptyResults: false, testResults: "$PWD/reports7/*.xml"
    }

    stage('Build'){
        docker.build(imageName)
    }

    stage('Push'){
        docker.withRegistry(registry, 'registry') {
            docker.image(imageName).push(commitID())

            if (env.BRANCH_NAME == 'develop') {
                docker.image(imageName).push('develop')
            }
        }
    }
}

def commitID() {
    sh 'git rev-parse HEAD > .git/commitID'
    def commitID = readFile('.git/commitID').trim()
    sh 'rm .git/commitID'
    commitID
}