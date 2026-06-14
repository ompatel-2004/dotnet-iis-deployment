def REPO_URL     = "https://gitlab.example.com/company/license-management-service.git"
def BRANCH       = "develop"
def GROUP_ID     = "license-management-service"
def ARTIFACT_ID  = "dev-license-management-service"
def BUILD_TIMESTAMP = new Date().format('yyyy-MM-dd-HH-mm-ss')
def NEXUS_REPO   = "dotnet-applications"
def NEXUS_URL    = "nexus.example.com:8081"
def PORT         = "5004"
def SITE_NAME    = "dev_license-management-service"
def VERSION = "1.0.0"

node('windows-agent') {
    stage("Checkout") {
        cleanWs()
        git branch: "${BRANCH}",
            changelog: false,
            credentialsId: 'git-credentials',
            poll: false,
            url: "${REPO_URL}"
    }

    stage("branch-check") {
        bat 'git branch'
    }

    stage('Install and Build') {
        bat 'dotnet restore LicenseManagement.sln'
        bat 'dotnet publish -c Release LicenseManagement.sln'
    }

    stage("Sonar Analysis") {
        bat """
            dotnet sonarscanner begin -k:"license-management-service" ^
            -n:"license-management-service" ^
            -v:"${VERSION}" ^
            -d:sonar.host.url="http://sonarqube.example.com:9000" ^
            -d:sonar.token="%SONAR_TOKEN%" ^
            -d:sonar.cs.vscoveragexml.reportsPaths="coverage.xml"
        """
        bat 'dotnet build --no-incremental -c Release LicenseManagement.sln'
        bat 'dotnet-coverage collect "dotnet test LicenseManagement.sln" -f xml -o "coverage.xml"'
        bat 'dotnet sonarscanner end -d:sonar.token="%SONAR_TOKEN%"'
    }

    stage('Zip Artifact') {
        zip archive: true,
            defaultExcludes: false,
            dir: './src/LicenseManagement.Api/bin/Release/net8.0/publish/',
            exclude: '',
            glob: '',
            zipFile: "./src/LicenseManagement.Api/bin/Release/${ARTIFACT_ID}-${VERSION}-${BUILD_TIMESTAMP}.zip"
    }

    stage('Upload to Nexus') {
        nexusArtifactUploader artifacts: [
            [
                artifactId: "${ARTIFACT_ID}",
                classifier: '',
                file: "./src/LicenseManagement.Api/bin/Release/${ARTIFACT_ID}-${VERSION}-${BUILD_TIMESTAMP}.zip",
                type: 'zip'
            ]
        ],

        credentialsId: 'nexus-credentials',
        groupId: "${GROUP_ID}",
        nexusUrl: "${NEXUS_URL}",
        nexusVersion: 'nexus3',
        protocol: 'http',
        repository: "${NEXUS_REPO}",
        version: "${VERSION}-${BUILD_TIMESTAMP}"
    }
}

node('ubuntu-agent') {
    stage('Ansible Playbook') {
        sh """
            ansible-playbook /opt/ansible/deploy-iis.yml \\
            -i /opt/ansible/inventory.ini \\
            -e "site_name=${SITE_NAME}" \\
            -e "project_name=${ARTIFACT_ID}" \\
            -e "host_port=${PORT}" \\
            -e "nexus_url=http://${NEXUS_URL}/repository/${NEXUS_REPO}/${GROUP_ID}/${ARTIFACT_ID}/${VERSION}-${BUILD_TIMESTAMP}/${ARTIFACT_ID}-${VERSION}-${BUILD_TIMESTAMP}.zip" \\
            -e "site_url=http://192.168.x.x:${PORT}" \\
            -e "expected_status_code=404"
        """
    }
}
