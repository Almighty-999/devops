pipeline {
    agent any // Выбираем Jenkins агента, на котором будет происходить сборка: нам нужен любой

    triggers {
        pollSCM('H/5 * * * *') // Запускать будем автоматически по крону примерно раз в 5 минут
    }

    tools {
        maven 'maven-3.9.9' // Для сборки бэкенда нужен Maven
        jdk 'jdk' // И Java Developer Kit нужной версии
        nodejs 'NodeJS' // А NodeJS нужен для фронта
    }

    stages {
        stage('Build & Test backend') {
            steps {
                dir("backend") { // Переходим в папку backend
                    sh 'mvn package' // Собираем мавеном бэкенд
                }
            }

            post {
                success {
                    junit 'backend/target/surefire-reports/**/*.xml' // Передадим результаты тестов в Jenkins
                }
            }
        }

        stage('Build frontend') {
            steps {
                dir("frontend") {
                    sh 'npm install' // Для фронта сначала загрузим все сторонние зависимости
                    sh 'npm run build' // Запустим сборку
                }
            }
        }
        
        stage('Save artifacts') {
            steps {
                archiveArtifacts(artifacts: 'backend/target/sausage-store-0.0.1-SNAPSHOT.jar')
                archiveArtifacts(artifacts: 'frontend/dist/frontend/*')
            }
             post {
                success {
                    script {
                        def chatId = "398663910" // Укажите chat_id Telegram-канала
                        def botToken = "7642134915:AAFUUwcZ-GW7d0WBAJx7qxKqCjiRM6NrEpQ" // Укажите токен вашего Telegram-бота

                        sh """
                        curl -X POST \
                        https://api.telegram.org/bot${botToken}/sendMessage \
                        -d chat_id=${chatId} \
                        -d text="Сборка успешно завершена! 🟢\\nАртефакты сохранены в Jenkins.\\nСборщик: ${env.BUILD_USER ?: 'Jenkins'}."
                        """
                    }  
                }  
            }
        }
    }
} 