def scenarios = [
    "ubuntu2204": [
        "install-linux",
        "install-tcp-linux",
        "uninstall-linux"
    ],
    "ubuntu2004": [
        "install-linux",
        "install-tcp-linux",
        "uninstall-linux"
    ],
    "ubuntu1804": [
        "install-linux",
        "install-tcp-linux",
        "uninstall-linux"
    ],
    "centos8": [
        "install-linux",
        "install-tcp-linux",
        "uninstall-linux"
    ],
    "centos7": [
        "install-linux",
        "install-tcp-linux",
        "uninstall-linux"
    ]
]

def role = "docker"

parallel_stages = [:]

for (kv in mapToList(scenarios)) {
    def platform = kv[0]
    def scenarioList = kv[1]

    parallel_stages[platform] = {

        docker.image('fabos4ai/molecule:4.0.1').inside('-u root') {

            stage("Install dependencies") {
                sh "ansible-galaxy install -f -r requirements.yml"
            }

            stage("${platform} - Create") {
                sh "cd ./${role} && molecule create -s install-${platform}"
            }

            for(int i = 0; i < scenarioList.size(); i++) {
                def scenario = scenarioList[i]
                stage("${platform} - ${scenario}") {
                    sh "cd ./${role} && molecule test -s ${scenario} -p ${platform} --destroy never"
                }
            }
        }
    }
}

node {
    checkout scm

    withCredentials([usernamePassword(
            credentialsId: 'jenkins_infra_account',
            usernameVariable: 'VSPHERE_USER',
            passwordVariable: 'VSPHERE_PASSWORD'
    )]) {

        try {
            parallel(parallel_stages)
        } finally {
            stage("Destroy") {
                sh "cd ./${role} && molecule destroy -s install-linux"
            }
        }
    }
}

@NonCPS
List<List<?>> mapToList(Map map) {
    return map.collect { it ->
        [it.key, it.value]
    }
}