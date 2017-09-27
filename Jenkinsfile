#!/usr/bin/env groovy
@Library('t-jenkins-shared-library') _

node {
  try {
    project = "burrow"
    f_compose = "docker-compose.yml"
    img_compose = "burrow_burrow"  // defined in ${f_compose}
    // remove potential '/' in branch name (Ex: feature/bla_bla)
    img_tag = env.BRANCH_NAME.replaceAll(/\//, '_')
    img_dockerhub = "harbor.infra.tiki.services/ci/${project}:${img_tag}"
    channel = '#system-tmp'

    notifyBuild('STARTED', channel)

    stage('pull code') {
      checkout scm
			sh "git checkout ${env.BRANCH_NAME} && git reset --hard origin/${env.BRANCH_NAME}"
    }

    stage('build') {
      sh "docker-compose -p ${project} build burrow"
    }

    switch(env.BRANCH_NAME) {
      case 'master':
        stage('img -> dockerhub') {
          sh "docker tag ${img_compose} ${img_dockerhub}"
          sh "docker push ${img_dockerhub}"
        }

        stage('deploy') {
          build job: '../deploy-production'
        }
        break
    }

  } catch (e) {
    // exception thrown, build failed
    currentBuild.result = "FAILED"
    throw e
  } finally {
    notifyBuild(currentBuild.result, channel)
  }
}
