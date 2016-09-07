node ('infrastructure'){
  checkout scm

  try {
    stage 'build'
    sh 'docker-compose build base-application'

    stage 'database setup'
    sh 'docker-compose up -d --remove-orphans build-database'
    sleep 3
    sh 'docker-compose run build-application gulp seed'

    stage 'test integration'
    sh 'docker-compose run build-application gulp test_integration'

    stage 'test http interface'
    sh 'docker-compose run build-application gulp test_http_interface'

    stage 'build'
    sh 'docker build --tag ${REGISTRY_HOST}:${REGISTRY_PORT}/microservice-lab --tag ${REGISTRY_HOST}:${REGISTRY_PORT}/microservice-lab:latest --tag ${REGISTRY_HOST}:${REGISTRY_PORT}/microservice-lab:$(date "+%d-%m-%Y_%H-%M-%S") .'

    stage 'login to registry'
    sh 'docker login --username ${REGISTRY_USERNAME} --password ${REGISTRY_PASSWORD} ${REGISTRY_HOST}:${REGISTRY_PORT}'

    stage 'push'
    sh 'docker push ${REGISTRY_HOST}:${REGISTRY_PORT}/microservice-lab'

    stage 'deploy'
    sh 'ansible-playbook /infrastructure/ansible/role.yml -i /infrastructure/ansible/hosts/$ENV_ENVIRONMENT -e "HOST=infrastructure00" -e "ROLE=$(pwd)/ansible/roles/deploy"'
  } finally {
    stage 'teardown'
    sh 'docker-compose stop'
    sh 'docker-compose rm --all --force -v'
  }
}