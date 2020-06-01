pipeline {
    agent any

    options{
        timeout(time: 50, unit: 'MINUTES')
    }

    environment {
        SVC_NAME = "cbim-dataservice-frontend"
        REPO_NAME = "$SVC_NAME"
        BRANCH = "${params.BRANCH}"
        REPO_DIR = "/Users/naix/work/cbim-dataservice-frontend"
        RELEASE_DIR = "/data/autodeploy/releases/$SVC_NAME/$BRANCH"
        OLD_VERSION_FILE = "$RELEASE_DIR/latest/version.txt"
        NEW_VERSION_FILE = "$RELEASE_DIR/version.txt"
    }
    stages {
        stage('build cbim-dataservice-frontend') {
            steps {
                sh '''
                    cd $REPO_DIR
                    pwd
                    npm -v
                    npm run build
                '''
//                 sh '''source ~/.bash_profile
//                       cd $REPO_DIR
//                       git checkout master && git pull
//                       git checkout $BRANCH || (git fetch && git checkout -b $BRANCH "origin/"$BRANCH)
//                       git reset --hard
//                       git pull origin $BRANCH
//                       if [ -d "$RELEASE_DIR/latest" ] ;then \
//                          current_build_time=`date +%y-%m-%d_%H:%M:%S` && \
//                          current_branch=$BRANCH && \
//                          current_commit_id=`git log | head -3 | grep commit | awk '{print $2}'` && \
//                          last_build_time=`cat $OLD_VERSION_FILE | grep current_build_time | awk '{print $2}'` && \
//                          last_commit_id=`cat $OLD_VERSION_FILE | grep current_commit_id | awk '{print $2}'` && \
//                          last_version=`cat $OLD_VERSION_FILE | grep current_version | awk '{print $2}'` && \
//                          last_branch=`cat $OLD_VERSION_FILE | grep current_branch | awk '{print $2}'` && \
//                          num=`git log | grep -n "$last_commit_id" | awk -F ":" '{print $1}'` && \
//                          echo -e "current_build_time:" $current_build_time "\ncurrent_version:" $BUILD_NUMBER "\ncurrent_branch:" $current_branch "\ncurrent_commit_id: " $current_commit_id "\n\nlast_build_time:" $last_build_time "\nlast_version:" $last_version "\nlast_branch:" $last_branch "\nlast_commit_id:" $last_commit_id "\n\ngit-log between these two versions:\n------------------------------------" > $NEW_VERSION_FILE && \
//                          git log | head -$(($num+5)) >> $NEW_VERSION_FILE ; \
//                       else \
//                          mkdir -p $RELEASE_DIR
//                          current_build_time=`date +%y-%m-%d_%H:%M:%S` && \
//                          current_branch=$BRANCH && \
//                          current_commit_id=`git log | head -3 | grep commit | awk '{print $2}'` && \
//                          echo -e "current_build_time:" $current_build_time "\ncurrent_version:" $BUILD_NUMBER "\ncurrent_branch:" $current_branch "\ncurrent_commit_id: " $current_commit_id "\n" > $NEW_VERSION_FILE ; \
//                       fi
//                       cd $REPO_DIR
//                       git checkout $BRANCH
//                       git reset --hard
//                       git pull origin $BRANCH
//                       npm install
//                    '''
            }
        }
//         stage('release file') {
//             steps {
//                 sh '''TAR_VERSION=`cd $REPO_DIR && head -12 package.json | grep "version" | awk -F "\\"" '{print $4}'`
//                       RELEASE_FILE=$REPO_NAME-$TAR_VERSION.tar.gz
//                       mkdir -p $RELEASE_DIR/$BUILD_NUMBER
//                       mv $NEW_VERSION_FILE $RELEASE_DIR/$BUILD_NUMBER
//                       mv $REPO_DIR/dist/$RELEASE_FILE $RELEASE_DIR/$BUILD_NUMBER ; \
//                       if [ -L $RELEASE_DIR/latest ] ;then \
//                          unlink $RELEASE_DIR/latest && \
//                          ln -s $RELEASE_DIR/$BUILD_NUMBER $RELEASE_DIR/latest ; \
//                       else \
//                          ln -s $RELEASE_DIR/$BUILD_NUMBER $RELEASE_DIR/latest ; \
//                       fi
//                   '''
//             }
//         }
        stage('note to html') {
            steps {
                script{
                    try{
                        wrap([$class: 'BuildUser']) {
                            sh '''source ~/.bash_profile
                                  cd $REPO_DIR
                                  git checkout $BRANCH
                                  branch=$BRANCH
                                  commit_id_long=$(git show |sed -n "1p" | awk \'{print $2}\')
                                  commit_id=$(echo ${commit_id_long:0:6})
                                  commit_description=$(git log -1 --pretty=%B | sed \'/^$/d\'|awk \'{if(NR%2==0){printf $0 "\\n"}else{printf "%s：",$0}}\')
                                  build_description=$build_description
                                  build_version=$BUILD_NUMBER
                                  build_time=$(date -d today +"%Y-%m-%d-%H:%M")
                                  sed -i "78i<p class=\'oneList\'><span class=\'width100\'>$build_version</span><span class=\'width150\'>$build_time</span><span class=\'width150\'>$commit_id</span><span class=\'width100\'>$branch</span><span class=\'width400\'>$commit_description</span><span class=\'width400\'>$build_description</span><span class=\'width150\'>$BUILD_USER</span></p>" /usr/share/nginx/html/jenkins/$SVC_NAME.html
                                  cp -a /usr/share/nginx/html/jenkins/$SVC_NAME.html /usr/share/nginx/html/jenkins/html-history/$SVC_NAME-$build_time.html
                               '''
                        }
                    }
                    catch(Exception e){
                        echo 'Note the build html error!!!'
                    }
                }
            }
        }
    }

    //Send Email
    post{
        //SUCCESS
        success{
            script {
                try{
                    wrap([$class: 'BuildUser']) {
                        sh '''
                            echo "success"
                            python /data/autodeploy/send_email/send_email.py ${BUILD_USER_EMAIL} SUCCESS ${JOB_NAME} ${BUILD_NUMBER} ${BUILD_URL} ${BUILD_USER}
                         '''
                    }
                }
                catch(Exception e){
                    echo 'Skipped send email'
                }
            }
        }
        //FAILURE
        failure{
            script {
                try{
                    wrap([$class: 'BuildUser']) {
                        sh '''
                            echo "failure"
                            python /data/autodeploy/send_email/send_email.py ${BUILD_USER_EMAIL} FAILURE ${JOB_NAME} ${BUILD_NUMBER} ${BUILD_URL} ${BUILD_USER}
                          '''
                        }
                }
                catch(Exception e){
                    echo 'Skipped send email'
                }
            }
        }
    }
} 
