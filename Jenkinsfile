pipeline {
    agent any
    environment {
        GITHUB_CREDENTIALS_ID = 'github_token'
        HELM_VERSION = '3.5.4'
        DOCKER_USERNAME = 'anu398'
        DOCKER_PASSWORD = 'dckr_pat_1XEm0AqyPtAIfAaW-BdQ7TK8fg8'
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
                    withCredentials([usernamePassword(credentialsId: GITHUB_CREDENTIALS_ID, usernameVariable: 'GITHUB_USER', passwordVariable: 'GITHUB_TOKEN')]) {
                        sh 'git config --global credential.helper store'
                        sh 'echo "https://${GITHUB_USER}:${GITHUB_TOKEN}@github.com" > ~/.git-credentials'
                        sh 'git fetch origin +refs/pull/*/head:refs/remotes/origin/pr/*'
                        def prBranch = env.CHANGE_BRANCH
                        echo "PR Branch: ${prBranch}"
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
                    def latestCommitMessage = sh(script: "git log -1 --pretty=format:%s", returnStdout: true).trim()
                    echo "Latest commit message: ${latestCommitMessage}"
                    def pattern = ~/^\s*(feat|fix|docs|style|refactor|perf|test|chore)(\(.+\))?: .+\s*$/
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
                    def lintResult = sh(script: 'helm lint .', returnStatus: true)
                    if (lintResult != 0) {
                        error 'Helm lint failed'
                    }
                    def templateResult = sh(script: 'helm template .', returnStatus: true)
                    if (templateResult != 0) {
                        error 'Helm template failed'
                    }
                }
            }
        }
        stage('Build and Push Docker Image') {
            steps {
                script {
                    withCredentials([usernamePassword(credentialsId: 'dockerhub_credentials', usernameVariable: 'DOCKER_USERNAME', passwordVariable: 'DOCKER_PASSWORD')]) {
                        sh '''
                        set -ex

                        # Check Docker Buildx installation
                        if ! docker buildx version; then
                            echo "Docker Buildx not found, installing..."
                            mkdir -p ~/.docker/cli-plugins/
                            curl -sSL https://github.com/docker/buildx/releases/download/v0.8.2/buildx-v0.8.2.linux-amd64 > ~/.docker/cli-plugins/docker-buildx
                            chmod +x ~/.docker/cli-plugins/docker-buildx
                        fi

                        # Login to Docker Hub
                        echo $DOCKER_PASSWORD | docker login -u $DOCKER_USERNAME --password-stdin

                        # Define the source and destination images
                        SOURCE_IMAGE=registry.k8s.io/autoscaling/cluster-autoscaler:v1.29.3
                        DEST_IMAGE=anu398/cluster-autoscaler:v1.29.3

                        # Create a new builder instance if not already created
                        docker buildx create --name mybuilder --use || true
                        docker buildx inspect --bootstrap

                        # Build and push the Docker image using Buildx
                        docker buildx build --platform linux/amd64,linux/arm64 -t $DEST_IMAGE --push -f ./Dockerfile .
                        '''
                    }
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
                            def newVersion = (versionLine =~ /(\d+\.\d+\.\d+)/)[0][0]
                            echo "New version: v${newVersion}"
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
