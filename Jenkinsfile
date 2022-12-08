node {

    def address = "github.com/mo40070369/SamplePOC.git"
      
    stage("Checkout code: configuration"){
        git credentialsId: 'Massil@123', url: "https://${address}", branch: "main"
    }
    
    echo 'Remove old temp file ... '
    sh 'cd repository/deployment/server/temp_deploy && rm -rf ./*'
    
    stage("Checkout code: current project"){
        dir('repository/deployment/server/temp_deploy'){
            checkout scm    
        }
    }
    
    def props = readYaml file: "repository/deployment/server/temp_deploy/jenkinsfile.pipeline.yml"

    stage('Load properties from Jenkins params') {
        dir ('repository/deployment/server/temp_deploy') {
            props.environments."${env.BRANCH_NAME}".each {
                updateProperty("{${it.key}}", "${it.value}" ,'*/src/main/synapse-config/*/*.xml')
            }
        }
    }
    
    def slave_instance = docker.image('jenkins_slave_jdk8')
 
    slave_instance.inside('-v /data/m2:/root/.m2') {
    
        //stage("Test in Maven"){
        //    sh "cd repository/deployment/server/temp_deploy/*_esb && /usr/bin/mvn test -DtestServerType=remote -DtestServerHost=10.122.12.139 -DtestServerPort=9008"
        //}
     
        stage("Build in Maven"){
            sh "cd repository/deployment/server/temp_deploy && /usr/bin/mvn -Dmaven.test.skip=true clean install -Pdev"
        }
        
        sh "cd repository/deployment/server/temp_deploy/*_ESBCAR/ && ls"
        
        stage("Cope car"){
            sh 'cp repository/deployment/server/temp_deploy/*_ESBCAR/target/*.car repository/deployment/server/carbonapps'
        }
        
        echo 'Remove temp file ... '
        sh 'cd repository/deployment/server/temp_deploy && rm -rf ./*'
    }
    
    stage("commit & push code"){
             withCredentials([[$class: 'UsernamePasswordMultiBinding', credentialsId: 'devops_svc',usernameVariable: 'USERNAME', passwordVariable: 'PASSWORD']]){
                sh 'git config --global user.email "yaomj1@lenovo.com"'
                sh 'git config --global user.name "yaomj1"'
                sh 'git add .'
                sh 'git commit -m "add new car file"'
                sh "git push http://$USERNAME:$PASSWORD@${address} ${env.BRANCH_NAME}"
        }
    }
}

def updateProperty(property, value, file) { 
        escapedProperty = property.replace('[', '\\[').replace(']', '\\]').replace('.', '\\.')

        sh "sed -i 's|$escapedProperty|$value|g' $file"
}
