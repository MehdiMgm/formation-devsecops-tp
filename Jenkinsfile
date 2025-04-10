pipeline {
  agent any
 
  stages {
      stage('Build Artifact') {
            steps {
              sh "mvn clean package -DskipTests=true"
              archive 'target/*.jar'
            }
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

}
///// Je comrends mieux maintenant