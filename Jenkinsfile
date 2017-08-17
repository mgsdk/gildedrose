node {
    stage ('Preparation') {
        deleteDir()
        checkout scm
        stash includes: '**', name: 'source'
    }
    def builders = [
        "build": {
            node {
                unstash name: 'source'
                sh('docker run -i --rm --name my-maven-project -v "$PWD":/usr/src/mymaven -w /usr/src/mymaven maven:3-jdk-8 mvn clean package')
                stash includes: 'target/gildedrose-*.jar', name: 'jar'
                stash includes: 'target/surefire-reports/TEST-*.xml', name: 'testResults'
            }
        },
        "javadoc": {
            node {
                unstash name: 'source'
                sh('docker run -i --rm --name my-maven-project -v "$PWD":/usr/src/mymaven -w /usr/src/mymaven maven:3-jdk-8 mvn site')
                stash includes: 'target/site/*', name: 'javadoc'
            }
        }
    ]
    stage ('Build') {
        parallel builders
    }
    stage ('Test') {
        unstash name: 'testResults'
        junit '**/target/surefire-reports/TEST-*.xml'
    }
    stage ('Publish') {
        unstash name: 'jar'
        archiveArtifacts artifacts: 'target/gildedrose-*.jar', onlyIfSuccessful: true
        unstash name: 'javadoc'
        archiveArtifacts artifacts: 'target/site/*', onlyIfSuccessful: true
    }
}
