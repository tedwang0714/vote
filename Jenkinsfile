pipeline {
    agent none
    stages {
        stage('Build with Kaniko') {
            // 这段 Demo4 才需要，Demo1 不演示，突出 Git 提交全部都会触发的特性
            when {
                changeset "**/vote/*.*"
            }
            // 这段 Demo4 才需要
            agent {
                kubernetes {
                    defaultContainer 'kaniko'
                    //workspaceVolume persistentVolumeClaimWorkspaceVolume(claimName: "jenkins-workspace-pvc", readOnly: false)
                    yaml """
kind: Pod
spec:
  containers:
  - name: kaniko
    image: gcr.io/kaniko-project/executor:v1.11.0-debug
    imagePullPolicy: Always
    command:
    - sleep
    args:
    - 99d
    volumeMounts:
      - name: jenkins-docker-cfg
        mountPath: /kaniko/.docker
  volumes:
  - name: jenkins-docker-cfg
    projected:
      sources:
      - secret:
          name: regcred
          items:
            - key: .dockerconfigjson
              path: config.json
"""
                }
            }

            environment {
                HARBOR_URL     = credentials('harbor-url')
                IMAGE_PUSH_DESTINATION="${HARBOR_URL}/vote/vote"
                GIT_COMMIT="${checkout (scm).GIT_COMMIT}"
                IMAGE_TAG = "${BRANCH_NAME}-${GIT_COMMIT}"
                BUILD_IMAGE="${IMAGE_PUSH_DESTINATION}:${IMAGE_TAG}"
                BUILD_IMAGE_LATEST="${IMAGE_PUSH_DESTINATION}:latest"
            }

            steps {
                container(name: 'kaniko', shell: '/busybox/sh') {
                    withEnv(['PATH+EXTRA=/busybox']) {
                        sh '''#!/busybox/sh
                            cd vote
                            /kaniko/executor --context `pwd` --destination $IMAGE_PUSH_DESTINATION --insecure
                        '''
                    }
                }
            }
        }
    }
}
