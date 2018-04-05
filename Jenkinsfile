stage('CI') {
    node {
        
        checkout scm
    
        // pull dependencies from npm
        // on windows use: bat 'npm install'
        sh 'npm install'
    
        // stash code & dependencies to expedite subsequent testing
        // and ensure same code & dependencies are used throughout the pipeline
        // stash is a temporary archive
        stash name: 'everything', 
              excludes: 'test-results/**', 
              includes: '**'
    }
}

//Parallel Integration Testing
stage ('Browser Testing') {
    parallel chrome: {
        runTests("ChromeHeadless")
    }, PhantomJS: {
        runTests("PhantomJS")
    }
}

node {
   notify('Deploy to staging?')
}

stage ('Deploy') {
    node {
        input 'Deploy to Staging?'

        sh "echo '<h1>${env.BUILD_DISPLAY_NAME}</h1>' >> app/index.html"

        sh 'docker-compose up -d --build'

        notify 'Solitare Deployed!'
    }
}

def runTests(browser) {
    node("Agent") {
        sh 'rm -rf *'
        unstash 'everything'
        sh "npm run test-single-run -- --browsers ${browser}"
        step([$class: 'JUnitResultArchiver',
            testResults: 'test-results/**/test-results.xml'])
    }
}

def notify(status){
    emailext (
      to: "davidmgrantham@gmail.com",
      subject: "${status}: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]'",
      body: """<p>${status}: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]':</p>
        <p>Check console output at <a href='${env.BUILD_URL}'>${env.JOB_NAME} [${env.BUILD_NUMBER}]</a></p>""",
    )
}