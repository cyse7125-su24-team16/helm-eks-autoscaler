pipeline {
    agent any
    environment {
        GITHUB_CREDENTIALS_ID = 'github_token'
        HELM_VERSION = '3.5.4'
        DOCKER_HUB_USERNAME = 'anu398'
        DOCKER_HUB_PASSWORD = 'dckr_pat_1XEm0AqyPtAIfAaW-BdQ7TK8fg8'
        PATH = "${env.WORKSPACE}/bin:${env.PATH}"
    }
    options {
        skipDefaultCheckout(true)
    }
    triggers {
        githubPush()
    }
    stages {
        stage('Checkout') {
            steps {
                script {
                    // Checkout the code
                    git credentialsId: GITHUB_CREDENTIALS_ID, url: 'https://github.com/cyse7125-su24-team16/helm-eks-autoscaler.git', branch: 'main'
                }
            }
        }
        stage('Fetch and Checkout PR Branch') {
            when {
                expression {
                    return env.CHANGE_ID != null
                }
            }
            steps {
                script {
                    // Fetch the latest changes from the origin using credentials
                    withCredentials([usernamePassword(credentialsId: GITHUB_CREDENTIALS_ID, usernameVariable: 'GITHUB_USER', passwordVariable: 'GITHUB_TOKEN')]) {
                        sh 'git config --global credential.helper store'
                        sh 'echo "https://${GITHUB_USER}:${GITHUB_TOKEN}@github.com" > ~/.git-credentials'
                        // Fetch all branches including PR branches
                        sh 'git fetch origin +refs/pull/*/head:refs/remotes/origin/pr/*'
                        // Dynamically fetch the current PR branch name using environment variables
                        def prBranch = env.CHANGE_BRANCH
                        echo "PR Branch: ${prBranch}"
                        // Checkout the PR branch
                        sh "git checkout -B ${prBranch} origin/pr/${env.CHANGE_ID}"
                    }
                }
            }
        }
        stage('Check Commit Messages') {
            when {
                expression {
                    return env.CHANGE_ID != null
                }
            }
            steps {
                script {
                    // Fetch the latest commit message in the PR branch
                    def latestCommitMessage = sh(script: "git log -1 --pretty=format:%s", returnStdout: true).trim()
                    echo "Latest commit message: ${latestCommitMessage}"
                    // Regex for Conventional Commits
                    def pattern = ~/^\s*(feat|fix|docs|style|refactor|perf|test|chore)(\(.+\))?: .+\s*$/
                    // Check the latest commit message
                    if (!pattern.matcher(latestCommitMessage).matches()) {
                        error "Commit message does not follow Conventional Commits: ${latestCommitMessage}"
                    }
                }
            }
        }
        stage('Run Helm Lint and Template') {
            when {
                expression {
                    return env.CHANGE_ID != null
                }
            }
            steps {
                script {
                    // Run helm lint
                    def lintResult = sh(script: 'helm lint .', returnStatus: true)
                    if (lintResult != 0) {
                        error 'Helm lint failed'
                    }
                    // Run helm template
                    def templateResult = sh(script: 'helm template .', returnStatus: true)
                    if (templateResult != 0) {
                        error 'Helm template failed'
                    }
                }
            }
        }
        stage('Install Crane') {
            steps {
                script {
                    sh '''
                    #!/bin/bash
                    set -e
                    mkdir -p ${WORKSPACE}/bin
                    if ! command -v crane &> /dev/null; then
                        echo "crane could not be found, installing..."
                        curl -LO https://github.com/google/go-containerregistry/releases/download/v0.10.0/crane-linux-amd64
                        # Verify the download
                        FILE_HASH=$(sha256sum crane-linux-amd64 | awk '{ print $1 }')
                        EXPECTED_HASH="d404d7e54a3e8a5d5e4d7c6dd6a21db86e1b5d8164e08a5a5148c6b6e8e663db"
                        if [ "$FILE_HASH" != "$EXPECTED_HASH" ]; then
                            echo "Downloaded file hash does not match expected hash. Exiting."
                            exit 1
                        fi
                        chmod +x crane-linux-amd64
                        mv crane-linux-amd64 ${WORKSPACE}/bin/crane
                    fi
                    '''
                }
            }
        }
        stage('Mirror Docker Image') {
            steps {
                script {
                    // Mirroring the Docker image
                    sh '''
                    #!/bin/bash
                    set -e
                    SOURCE_IMAGE="registry.k8s.io/autoscaling/cluster-autoscaler:v1.29.3"
                    DEST_IMAGE="anu398/cluster-autoscaler:v1.29.3"
                    echo "$DOCKER_HUB_PASSWORD" | docker login --username "$DOCKER_HUB_USERNAME" --password-stdin
                    echo "Mirroring image from $SOURCE_IMAGE to $DEST_IMAGE..."
                    crane copy "$SOURCE_IMAGE" "$DEST_IMAGE"
                    echo "Image mirrored successfully."
                    '''
                }
            }
        }
        stage('Semantic-Release') {
            when {
                allOf {
                    branch 'main'
                    not { changeRequest() }
                }
            }
            steps {
                script {
                    withCredentials([usernamePassword(credentialsId: 'github_token', usernameVariable: 'GH_USERNAME', passwordVariable: 'GH_TOKEN')]) {
                        env.GIT_LOCAL_BRANCH = 'main'
                        def releaseOutput = sh(script: 'npx semantic-release --dry-run --json', returnStdout: true).trim()
                        def versionLine = releaseOutput.find(/Published release (\d+\.\d+\.\d+) on default channel/)
                        if (versionLine) {
                            // Extract the new version
                            def newVersion = (versionLine =~ /(\d+\.\d+\.\d+)/)[0][0]
                            echo "New version: v${newVersion}"
                            // Package and release Helm chart
                            sh """
                                helm package --version ${newVersion} .
                                gh release create 'v${newVersion}' *${newVersion}.tgz
                                rm *.tgz
                            """
                        } else {
                            error "Failed to capture the new version from semantic-release."
                        }
                    }
                }
            }
        }
    }
    post {
        failure {
            script {
                echo "Pipeline failed."
            }
        }
        success {
            script {
                echo "Pipeline succeeded."
            }
        }
    }
}
