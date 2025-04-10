pipeline {
  agent any
 
  stages {
                stage('Build Artifact') {
                        steps {
                          sh "mvn clean package -DskipTests=true"
                          archive 'target/*.jar'
                        }
                    }  
              //--------------------------
                stage('test unitaire ') {
                        steps {
                          sh "mvn test"
                        
                        }
                    }
              //--------------------------
                stage('Mutation Tests - PIT') {
                        steps {
                         sh "mvn org.pitest:pitest-maven:mutationCoverage"
                  }
            
                }
              //--------------------------
                stage('Docker Build and Push') {
                        steps {
                          withCredentials([string(credentialsId: 'password_DockHub', variable: 'DOCKER_HUB_PASSWORD')]) {
                            sh 'sudo docker login -u mehdimgm -p $DOCKER_HUB_PASSWORD'
                            sh 'printenv'
                            sh 'sudo docker build -t mehdimgm/formation-app:""$GIT_COMMIT"" .'
                            sh 'sudo docker push mehdimgm/formation-app:""$GIT_COMMIT""'
                }
        
              }
            }
               //--------------------------
                   stage('Deployment Kubernetes  ') {
                            steps {
                              withKubeConfig([credentialsId: 'kubeconfig']) {
                                    sh "sed -i 's#replace#mehdimgm/formation-app:${GIT_COMMIT}#g' k8s_deployment_service.yaml"
                                    sh "sudo kubectl apply -f k8s_deployment_service.yaml"
                                  }
                            }
                      
    }
                 //--------------------------
              
 
    }
    

}
