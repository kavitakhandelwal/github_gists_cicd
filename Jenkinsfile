pipeline {
    agent {
        kubernetes {
            cloud 'kubernetes'
            yaml '''
apiVersion: v1
kind: Pod
spec:
  serviceAccountName: jenkins-sa
  containers:
  - name: build-tools
    image: alpine/k8s:1.29.2
    command: ['cat']
    tty: true
  - name: jnlp # This represents your main Jenkins agent container with Buildah installed
    securityContext:
      privileged: false
      capabilities:
        add: ["SETFCAP"] 
    env:
      - name: STORAGE_DRIVER
        value: vfs
    volumeMounts:
      - name: docker-config
        mountPath: /root/.docker
  volumes:
    - name: docker-config
      secret:
        secretName: dockerhub-secret
        items:
          - key: .dockerconfigjson
            path: config.json
'''
        }
    }

    environment {
        DOCKERHUB_REPO = "kavitakhandelwal/github_gists"
    }

    stages {
        stage('Build & Push') {
            steps {
                // Since Buildah is in your main agent, you use 'jnlp' or no container block at all
                container('jnlp') {
                    sh "buildah bud --storage-driver=vfs -t ${DOCKERHUB_REPO}:${BUILD_NUMBER}-NEW ."
                    sh "buildah push --storage-driver=vfs --authfile /root/.docker/config.json ${DOCKERHUB_REPO}:${BUILD_NUMBER}-NEW"
                }
            }
        }

        stage('Deploy') {
            steps {
                container('build-tools') {
                    sh "sed -i 's|image: kavitakhandelwal/github_gists:latest|image: ${DOCKERHUB_REPO}:${BUILD_NUMBER}-NEW|g' deploy/deployment.yaml"
                    sh 'kubectl apply -f deploy/deployment.yaml'
                }
            }
        }
    }
}