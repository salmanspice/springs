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
             NEXUS_REPOSITORY = "repository-example"
             // Jenkins credential id to authenticate to Nexus OSS
             NEXUS_CREDENTIAL_ID = "nexus-credentials"
    }

    stages {
        stage("Build") {
            steps {
                sh "mvn -version"
                sh "mvn clean install"
            }
        }
        stage("publish to nexus") {
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
    }

    post {
        always {
            cleanWs()
        }
    }
}