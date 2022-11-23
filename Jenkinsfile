pipeline {
  agent {
    docker { image 'liquibase/liquibase:4.17' }
  }
  environment {
    TEST_URL='jdbc:h2:tcp://localhost:9090/mem:dev'
    QA_URL='jdbc:h2:tcp://localhost:9090/mem:integration'
    changeLogFile='changelog_version-3.2.h2.sql'
    driver='org.postgresql.Driver'
  }
  stages {
    stage('Status') {
      steps {
        sh 'liquibase status --url="$env:TEST_URL" --changeLogFile=$env:changeLogFile --driver=$env:driver --username=postgres --password=fellaini'
        sh 'liquibase status --url="$env:QA_URL" --changeLogFile=$env:changeLogFile --driver=$env:driver --username=postgres --password=fellaini'
      }
    }
    stage('test') {
       steps {
        sh 'mvn clean package -Dmaven.test.skip=true'
        sh 'mvn liquibase:update --url=$env:TEST_URL --changeLogFile=$env:changeLogFile --username=postgres --password=fellaini'
        sh 'mvn liquibase:status -PTEST'
        sh 'mvn spring-boot:run'
      }
    }
    stage('QA') {
      steps {
        sh 'mvn clean package'
        sh 'mvn liquibase:update --url=$env:QA_URL --changeLogFile=wrapper.xml --username=postgres --password=fellaini'
        sh 'mvn liquibase:status -PQA'
        sh 'mvn Spring-boot:run'
       }
      }
    stage('Update') {
      steps {
        sh 'liquibase update --url="jdbc:postgresql://database-1.cpuc6bgspxr2.eu-central-1.rds.amazonaws.com:5432/postgres" --changeLogFile=wrapper.xml --username=postgres --password=fellaini'
      }
    }
  }
  post {
    always {
      cleanWs()
    }
  }
}
