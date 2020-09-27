// install  nexus-artifact-uploader and pipeline-utility-steps

pipeline {
    agent any

    tools {
            maven "3.6.0" // You need to add a maven with name "3.6.0" in the Global Tools Configuration page
        }
    environment {
             // This can be nexus3 or nexus2
            NEXUS_VERSION = "nexus3"
             // This can be http or https
            NEXUS_PROTOCOL = "http"
             // Where your Nexus is running
            NEXUS_URL = "192.168.8.107:8081"
             // Repository where we will upload the artifact
            NEXUS_REPOSITORY = "nbk-artifact"
             // Jenkins credential id to authenticate to Nexus OSS
            NEXUS_CREDENTIAL_ID = "nexus-credentials"


    }
    stages {

        stage("clone code") {
                    steps {
                        script {
                            // Let's clone the source
                            git 'https://github.com/kadirsahan/springs.git';
                        }
                    }
         }

        stage("Build") {
            steps {
                sh "mvn -version"
                sh "mvn clean package -DskipTests -Dimage=192.168.8.107:9060/repository/nbknexus/spring/nbkhello:${BUILD_NUMBER}"
            }
        }

        stage("publish artifact to nexus") {
                    steps {
                        script {
                            // Read POM xml file using 'readMavenPom' step , this step 'readMavenPom' is included in: https://plugins.jenkins.io/pipeline-utility-steps
                            pom = readMavenPom file: "pom.xml";
                            // Find built artifact under target folder
                            filesByGlob = findFiles(glob: "target/*.${pom.packaging}");
                            // Print some info from the artifact found
                            echo "${filesByGlob[0].name} ${filesByGlob[0].path} ${filesByGlob[0].directory} ${filesByGlob[0].length} ${filesByGlob[0].lastModified}"
                            // Extract the path from the File found
                            artifactPath = filesByGlob[0].path;
                            // Assign to a boolean response verifying If the artifact name exists
                            artifactExists = fileExists artifactPath;
                            if(artifactExists) {
                                echo "*** File: ${artifactPath}, group: ${pom.groupId}, packaging: ${pom.packaging}, version ${pom.version}";
                                nexusArtifactUploader(
                                    nexusVersion: NEXUS_VERSION,
                                    protocol: NEXUS_PROTOCOL,
                                    nexusUrl: NEXUS_URL,
                                    groupId: pom.groupId,
                                    version: pom.version,
                                    repository: NEXUS_REPOSITORY,
                                    credentialsId: NEXUS_CREDENTIAL_ID,
                                    artifacts: [
                                        // Artifact generated such as .jar, .ear and .war files.
                                        [artifactId: pom.artifactId,
                                        classifier: '',
                                        file: artifactPath,
                                        type: pom.packaging],
                                        // Lets upload the pom.xml file for additional information for Transitive dependencies
                                        [artifactId: pom.artifactId,
                                        classifier: '',
                                        file: "pom.xml",
                                        type: "pom"]
                                    ]
                                );
                            } else {
                                error "*** File: ${artifactPath}, could not be found";
                            }
                        }
                    }
                }

        stage('Push Image To Nexus') {
            environment {
                NEXUS_ID = credentials("nexus_user_id")

                NEXUS_PWD = credentials("nexus_user_pwd")
            }

            steps {

                    sh 'docker login 192.168.8.107:9060 -u $NEXUS_ID -p $NEXUS_PWD'
                    sh 'docker push 192.168.8.107:9060/repository/nbknexus/spring/nbkhello:${BUILD_NUMBER}'

            }
        }

    }

}