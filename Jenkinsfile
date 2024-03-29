node ('infrastructure'){
  checkout scm

  try {
  	stage 'build'
    sh 'make build'

    stage 'database setup'
    sh 'docker-compose up -d --remove-orphans build-database'
    sleep 10
    sh 'make seed'

    stage 'test integration'
    sh 'make test_integration'
    stage 'test http interface'
    sh 'make test_http_interface'

		stage 'publish sdk'
    sh 'make publish_sdk'

    stage 'build'
    sh 'docker build --tag ${REGISTRY_HOST}:${REGISTRY_PORT}/$(cat package.json | jq --raw-output ".name") --tag ${REGISTRY_HOST}:${REGISTRY_PORT}/$(cat package.json | jq --raw-output ".name"):latest --tag ${REGISTRY_HOST}:${REGISTRY_PORT}/$(cat package.json | jq --raw-output ".name"):$(date "+%d-%m-%Y_%H-%M-%S") .'

    stage 'login to registry'
    sh 'docker login --username ${REGISTRY_USERNAME} --password ${REGISTRY_PASSWORD} ${REGISTRY_HOST}:${REGISTRY_PORT}'

    stage 'push'
    sh 'docker push ${REGISTRY_HOST}:${REGISTRY_PORT}/$(cat package.json | jq --raw-output ".name")'

    stage 'deploy'
    sh 'ansible-playbook /infrastructure/ansible/role.yml -i /infrastructure/ansible/hosts/${ENV_ENVIRONMENT} -e "HOST=${DEPLOYMENT_HOST}" -e "ROLE=$(pwd)/ansible/roles/deploy"'

    stage 'verify deployment'
    sh 'ansible-playbook /infrastructure/ansible/role.yml -i /infrastructure/ansible/hosts/${ENV_ENVIRONMENT} -e "HOST=${DEPLOYMENT_HOST}" -e "ROLE=$(pwd)/ansible/roles/deploy/tests"'
  } finally {
    stage 'teardown'
    sh 'docker-compose stop'
    sh 'docker-compose rm --all --force -v'
  }
}