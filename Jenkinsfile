def imageName = 'skyglass/movies-loader'
def registry = 'https://registry.hub.docker.com'

node('workers'){
    stage('Checkout'){
        checkout scm
    }

    stage('Unit Tests'){
        def imageTest= docker.build("${imageName}-test", "-f Dockerfile.test .")
        sh "docker run --rm -v $PWD/reports:/app/reports ${imageName}-test"
        junit allowEmptyResults: true, testResults: "$PWD/reports/*.xml"
    }

    stage('Build'){
        docker.build(imageName)
    }

    stage('Push'){
        docker.withRegistry(registry, 'dockerHubCredentials') {
            docker.image(imageName).push(commitID())

            if (env.BRANCH_NAME == 'develop') {
                docker.image(imageName).push('develop')
            }
        }
    }

    stage('Deploy'){
        if(env.BRANCH_NAME == 'develop' || env.BRANCH_NAME == 'preprod'){
            build job: "watchlist-deployment/${env.BRANCH_NAME}"
        }

        if(env.BRANCH_NAME == 'master'){
            timeout(time: 2, unit: "HOURS") {
                input message: "Approve Deploy?", ok: "Yes"
            }
            build job: "watchlist-deployment/master"
        }
    }    
}

def commitID() {
    sh 'git rev-parse HEAD > .git/commitID'
    def commitID = readFile('.git/commitID').trim()
    sh 'rm .git/commitID'
    commitID
}