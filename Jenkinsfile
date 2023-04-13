pipeline {
  agent any
  triggers {
    cron('H 0 * * *')
  }

  stages {

    stage('Scan used images with clair') {
      steps {
        node(label: 'clair') {
         script {
           catchError(buildResult: 'SUCCESS', stageResult: 'UNSTABLE') {
             sh '''/scan_catalog_entry.sh  templates/plonesaas'''
           } 
         }
       }
     }
   }

    stage('Build & Tests - PLONE') {
      steps {
        build job: '../eea.docker.plone/master', parameters: [[$class: 'StringParameterValue', name: 'TARGET_BRANCH', value: 'master']]
      }
    }

    stage('Release - PLONE') {
      steps {
        node(label: 'docker') {
          withCredentials([string(credentialsId: 'eea-jenkins-token', variable: 'GITHUB_TOKEN'),  string(credentialsId: 'plone-trigger', variable: 'TRIGGER_MAIN_URL'), usernamePassword(credentialsId: 'jekinsdockerhub', usernameVariable: 'DOCKERHUB_USER', passwordVariable: 'DOCKERHUB_PASS')]) {
           sh '''docker pull eeacms/gitflow; docker run -i --rm --name="$BUILD_TAG-nightly-plone" -e GIT_BRANCH="master" -e GIT_NAME="eea.docker.plone" -e DOCKERHUB_REPO="eeacms/plone" -e GIT_TOKEN="$GITHUB_TOKEN" -e DOCKERHUB_USER="$DOCKERHUB_USER" -e DOCKERHUB_PASS="$DOCKERHUB_PASS"  -e TRIGGER_MAIN_URL="$TRIGGER_MAIN_URL" -e DEPENDENT_DOCKERFILE_URL="eea/eea.docker.plonesaas/blob/master/Dockerfile" -e GITFLOW_BEHAVIOR="TAG_ONLY" eeacms/gitflow'''
         }
       }
     }
   }

    stage('Build & Tests - PLONESAAS - new PLONE release') {
      steps {
        build job: '../eea.docker.plonesaas/master', parameters: [[$class: 'StringParameterValue', name: 'TARGET_BRANCH', value: 'master']]
      }
    }

    stage('Release - PLONESAAS') {
      steps {
        node(label: 'docker') {
          withCredentials([string(credentialsId: 'eea-jenkins-token', variable: 'GITHUB_TOKEN'), string(credentialsId: 'plonesaas-devel-trigger', variable: 'TRIGGER_URL'), string(credentialsId: 'plonesaas-trigger', variable: 'TRIGGER_MAIN_URL'),usernamePassword(credentialsId: 'jekinsdockerhub', usernameVariable: 'DOCKERHUB_USER', passwordVariable: 'DOCKERHUB_PASS')]) {
           sh '''docker pull eeacms/gitflow; docker run -i --rm --name="$BUILD_TAG-nightly-plonesaas" -e GIT_BRANCH="master" -e GIT_NAME="eea.docker.plonesaas" -e GIT_TOKEN="$GITHUB_TOKEN" -e TRIGGER_MAIN_URL="$TRIGGER_MAIN_URL" -e DOCKERHUB_USER="$DOCKERHUB_USER" -e DOCKERHUB_PASS="$DOCKERHUB_PASS" -e DOCKERHUB_REPO="eeacms/plonesaas" -e DEPENDENT_DOCKERFILE_URL="devel/Dockerfile"  -e TRIGGER_RELEASE="eeacms/plonesaas-devel;$TRIGGER_URL" -e GITFLOW_BEHAVIOR="TAG_ONLY" eeacms/gitflow'''
         }
       }
     }
   }

  }

  post {
    always {
      cleanWs(cleanWhenAborted: true, cleanWhenFailure: true, cleanWhenNotBuilt: true, cleanWhenSuccess: true, cleanWhenUnstable: true, deleteDirs: true)
      script {
        def details = """<h1>${env.JOB_NAME} - Build #${env.BUILD_NUMBER} - ${currentBuild.currentResult}</h1>
                         <p>Check console output at <a href="${env.BUILD_URL}/display/redirect">${env.JOB_BASE_NAME} - #${env.BUILD_NUMBER}</a></p>
                      """
        emailext(
        subject: '$DEFAULT_SUBJECT',
        body: details,
        attachLog: true,
        compressLog: true,
        to: '$DEFAULT_RECIPIENTS'
        )
      }
    }
  }
}
