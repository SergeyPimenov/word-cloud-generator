pipeline {
    agent {
        dockerfile {
            additionalBuildArgs  '-t svp-jenkins'
            args '--name svp-jenkins -u 0:0 --network=SVP_network -v /var/run/docker.sock:/var/run/docker.sock'
        }    
    }
    
    stages {
        stage('build') {
            steps {
                sh """
                    export PATH="\$PATH:${WORKSPACE}/bin"
                    sed -i 's/1.DEVELOPMENT/1.${BUILD_NUMBER}/g' ./rice-box.go
                    make
                    md5sum artifacts/*/word-cloud-generator* >artifacts/word-cloud-generator.md5
                    gzip artifacts/*/word-cloud-generator*
                """
                nexusArtifactUploader artifacts: [[artifactId: 'word-cloud-generator', classifier: '', file: 'artifacts/linux/word-cloud-generator.gz', type: 'gz']], credentialsId: 'nexus-creds', groupId: '1', nexusUrl: 'nexus:8081', nexusVersion: 'nexus3', protocol: 'http', repository: 'word-cloud-generator', version: '1.$BUILD_NUMBER'
            }
        }
        
        stage('tests') {
            environment {
                NEXUS_CREDS = credentials('nexus-creds')
            }
            
            steps {
                sh "docker build -t middle_image --build-arg NEXUS_CREDS=${NEXUS_CREDS} --build-arg BUILD_NUMBER=${BUILD_NUMBER} --network=SVP_network -f ./alpine_linux/Dockerfile ."
                
                sh "docker run -d --name build_image --network=SVP_network middle_image"
                
                sh """
                    res=`curl -s -H "Content-Type: application/json" -d '{"text":"ths is a really really really important thing this is"}' http://build_image:8888/version | jq '. | length'`
                    if [ "1" != "\$res" ]; then
                      exit 99
                    fi

                    res=`curl -s -H "Content-Type: application/json" -d '{"text":"ths is a really really really important thing this is"}' http://build_image:8888/api | jq '. | length'`
                    if [ "7" != "\$res" ]; then
                      exit 99
                    fi
                """
                
                sh "docker rm -f build_image"
                
                sh "docker rmi middle_image"
            }
        }
    }
}
