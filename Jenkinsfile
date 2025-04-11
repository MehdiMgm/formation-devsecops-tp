pipeline {
    agent any

    stages {
        
        // --------------- Stage de construction de l'artefact Maven ---------------
        stage('Build Artifact') {
            steps {
                // Nettoyage, compilation et packaging sans tests
                sh "mvn clean package -DskipTests=true"
                // Archivage des artefacts générés
                archive 'target/*.jar'
            }
        }

        // --------------- Stage des tests unitaires ---------------
        stage('Test Unitaire') {
            steps {
                catchError(buildResult: 'SUCCESS', stageResult: 'FAILURE') {
                    // Exécution des tests unitaires avec Maven
                    sh "mvn test"
                }
            }
            post {
                always {
                    // Publication des rapports de tests dans Jenkins
                    junit 'target/surefire-reports/*.xml'
                    // Génération du rapport de couverture de code avec JaCoCo
                    jacoco(execPattern: 'target/jacoco.exec')
                }
            }
        }

        // --------------- Stage des tests de mutation avec PIT ---------------
        stage('Mutation Tests - PIT') {
            steps {
                catchError(buildResult: 'SUCCESS', stageResult: 'FAILURE') {
                    // Exécution des tests de mutation avec PIT (Mutation Testing)
                    sh "mvn org.pitest:pitest-maven:mutationCoverage"
                }
            }
            post {
                always {
                    // Publication du rapport de tests de mutation généré
                    pitmutation mutationStatsFile: '**/target/pit-reports/**/mutations.xml'
                }
            }
        }

// --------------- Stage d'analyse SonarQube ---------------
stage('SonarQube Analysis') {
    steps {
        // Récupération du token SonarQube depuis Jenkins
        withCredentials([string(credentialsId: 'sonarqube', variable: 'SONAR'),string(credentialsId: 'projectK', variable: 'projectK')]) {                            
                // Capture les erreurs sans échouer le build
                catchError(buildResult: 'SUCCESS', stageResult: 'FAILURE') {
                    
                    // Exécution de l'analyse SonarQube avec Maven
                          sh "sudo mvn sonar:sonar \
                          -Dsonar.$projectK=H-ref_formation \
                          -Dsonar.host.url=http://formation.eastus.cloudapp.azure.com:9000 \
                          -Dsonar.login=$SONAR"
                }
            }
        }
    }
        // --------------- Stage de scan des vulnérabilités avec OWASP Dependency-Check ---------------
        stage('Vulnerability Scan - Docker') {
            steps {
                catchError(buildResult: 'SUCCESS', stageResult: 'FAILURE') {
                    // Exécution du scan OWASP Dependency-Check pour les vulnérabilités de dépendances
                    sh "sudo mvn dependency-check:check"
                }
            }
            post {
                always {
                    // Publication du rapport de vulnérabilités
                    dependencyCheckPublisher pattern: 'target/dependency-check-report.xml'
                    // Publication du rapport JaCoCo
                    jacoco(execPattern: 'target/jacoco.exec')
                }
            }
        }

        // --------------- Stage de build et push Docker ---------------
        stage('Docker Build and Push') {
            steps {
                withCredentials([string(credentialsId: 'DockHub', variable: 'DOCKER_HUB_P')]) {
                    // Connexion à Docker Hub
                    sh 'sudo docker login -u mehdimgm -p $DOCKER_HUB_P'
                    // Affichage des variables d'environnement pour debug
                    sh 'printenv'
                    // Construction de l'image Docker avec le commit Git
                    sh "sudo docker build -t mehdimgm/formation-app:${GIT_COMMIT} ."
                    // Push de l'image vers Docker Hub
                    sh "sudo docker push mehdimgm/formation-app:${GIT_COMMIT}"
                }
            }
        }

        // --------------- Stage de déploiement Kubernetes ---------------
        stage('Deployment Kubernetes') {
            steps {
                withKubeConfig([credentialsId: 'kubeconfig']) {
                    // Modification du fichier de déploiement Kubernetes avec l'image taguée avec le commit Git
                    sh "sed -i 's#replace#mehdimgm/formation-app:${GIT_COMMIT}#g' k8s_deployment_service.yaml"
                    // Application de la configuration Kubernetes (déploiement)
                    sh "sudo kubectl apply -f k8s_deployment_service.yaml"
                }
            }
        }
    }
}
