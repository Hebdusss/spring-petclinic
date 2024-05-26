pipeline 
{
    agent any

    stages {
        // Etap zbierania kodu ze zdalnego repozytorium
        stage('Collect') 
        {
            steps 
            {
                git branch: "main", url: "https://github.com/Hebdusss/spring-petclinic"
                sh 'git config user.email "Hebdusss11@gmail.com"'
                sh 'git config user.name "Hebdusss"'
            }
        }

        // Etap budowania obrazów Docker
        stage('Build') 
        {
            steps 
            {
                script 
                {
                    try 
                    {
                        docker.build("spring-builder", "-f Dockerfile-builder .")
                    } 
                    catch (Exception e) 
                    {
                        currentBuild.result = 'FAILURE'
                        error "Błąd podczas budowania obrazu Docker: ${e.message}"
                    }
                }
            }
        }

        // Etap testowania obrazów Docker
        stage('Test') 
        {
            steps 
            {
                script 
                {
                    try 
                    {
                        docker.build("spring-tester", "-f Dockerfile-tester .")
                    } 
                    catch (Exception e) 
                    {
                        currentBuild.result = 'FAILURE'
                        error "Błąd podczas testowania obrazu Docker: ${e.message}"
                    }
                }
            }
        }

        // Etap deployowania kontenerów Docker
        stage('Deploy & Publish') 
        {
            agent any
            steps 
            {
                script 
                {
                    try 
                    {
                        def TIMESTAMP = sh(script: 'date +%Y%m%d%H%M%S', returnStdout: true).trim()
                        env.TIMESTAMP = TIMESTAMP
                        def builderContainer = docker.image("spring-builder").run("-d --name builder${TIMESTAMP}")
                        def testerContainer = docker.image("spring-tester").run("-d --name tester${TIMESTAMP}")
                        sh "docker logs ${builderContainer.id} > builder_log.txt"
                        sh "docker logs ${testerContainer.id} > tester_log.txt"
                        builderContainer.stop()
                        testerContainer.stop()
                        sh "tar -czf Artifact_${TIMESTAMP}.tar.gz builder_log.txt tester_log.txt"
                        // Archiwizacja artefaktu
                        archiveArtifacts artifacts: "Artifact_${TIMESTAMP}.tar.gz", onlyIfSuccessful: true
                    } 
                    catch (Exception e) 
                    {
                        currentBuild.result = 'FAILURE'
                        error "Błąd podczas deployu: ${e.message}"
                    }
                }
            }
        }
    }
}