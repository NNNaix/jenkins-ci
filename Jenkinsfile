pipeline {
    agent {
        node {
            label 'deploy'
        }
    }

    options{
        timeout(time: 40, unit: 'MINUTES')
    }

    // builder with parameters
    parameters {
        string(name: 'BRANCH', defaultValue: 'master', description: 'git project branch')
    }

    // 环境变量
    environment {
        PROJECT_NAME = 'cbim-dataservice-frontend'

        PROJECT_REPO_DIR = '/data/repos'
        MOUDLE_REPO_DIR = "${PROJECT_REPO_DIR}/${PROJECT_NAME}"

        BRANCH = "${params.BRANCH}"
        BRANCH_BUILD_DIR = "${PROJECT_NAME}/${BRANCH}/${BUILD_NUMBER}"

        RELEASE_DIR = "/data/releases/${BRANCH_BUILD_DIR}"
        VERSION_FILE = "${RELEASE_DIR}/version.txt"
        REPORTS_DIR = "/data/reports/${BRANCH_BUILD_DIR}"

        JENKINS_HOME = '/data/jenkins'
        JAVA_HOME = '/home/centos/opt/jdk'
        M2_HOME = '/home/centos/opt/maven'
    }

    // 构建过程
    stages {

        // 拉取项目源码
        stage('Preparing SCM') {
            steps {
                echo "Pull source code with branch[${BRANCH}] ${BUILD_NUMBER}"
                sh '''
                cd ${MOUDLE_REPO_DIR}/
                git checkout master && git pull
                git checkout $BRANCH || (git fetch && git checkout -b $BRANCH "origin/"$BRANCH)
                git pull origin $BRANCH
                '''
            }
        }
        // 编译与测试
        stage('Build') {
            when {
                expression {
                    currentBuild.result == null || currentBuild.result == 'SUCCESS'
                }

            }
            steps {
                echo "Starting Building and Testing ${PROJECT_NAME} ......"
                //记录版本信息
                sh '''mkdir -p $RELEASE_DIR
                current_build_time=`date +%y-%m-%d_%H:%M:%S` && \
                current_branch=$BRANCH && \
                current_commit_id=`git log | head -3 | grep commit | awk '{print $2}'` && \
                echo -e "current_build_time:" $current_build_time "\ncurrent_version:" $BUILD_NUMBER "\ncurrent_branch:" $current_branch "\ncurrent_commit_id: " $current_commit_id "\n" > $VERSION_FILE ;
                '''

                // node 编译
                sh '''
                cd ${MOUDLE_REPO_DIR}/
                yarn install --verbose
                yarn run build
                '''
            }
        }
        // 整理包:如版本命名等
        stage('Release package') {
            when {
              expression {
                currentBuild.result == null || currentBuild.result == 'SUCCESS'
              }
            }
            steps {
                echo "Release package"
                sh '''
                JAR_VERSION=`cd ${MOUDLE_REPO_DIR} && head -15 pom.xml | grep -r "^\\ \\ \\ \\ <version>" | tail -1| awk -F ">" '{print $2}' | awk -F "<" '{print $1}'`
                ARTIFACTID=`cd ${MOUDLE_REPO_DIR} && head -15 pom.xml | grep -r "^\\ \\ \\ \\ <artifactId>" | tail -1 | awk -F ">" '{print $2}' | awk -F "<" '{print $1}'`
                mkdir -p ${RELEASE_DIR}
                mv ${MOUDLE_REPO_DIR}/dist/ ${RELEASE_DIR}/
                '''
            }
        }
    }

    // 构建后处理
    post {
        failure {
            // mail to: team@example.com, subject: 'The Pipeline failed :('
            echo "failed"
        }
        success {
            // mail to: team@example.com, subject: 'The Pipeline failed :('
            echo "success"
        }
    }
}