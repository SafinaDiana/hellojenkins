## Автоматизация сборки и деплоя Python Docker-приложений через Jenkins с интеграцией GitHub
### Ход работы
#### Шаг 1. Клонирование репозитория
Выгрузила репозиторий с исходным кодом в свой репозиторий 
<img width="1133" height="707" alt="image" src="https://github.com/user-attachments/assets/c538df56-c31b-4a7c-aeef-5ca5342aef84" />

#### Шаг 2. Создание и конфигурирование Jenkins Job
Авторизовалась на Jenkins-сервере по указанному IP и порту.

В верхнем меню выбрала кнопку New Item для создания нового проекта.

Задала уникальное имя проекта — student-safuna-diana.

В списке типов проектов выбрала Pipeline и тыкнула OK.

В открывшемся окне перешела к разделу Pipeline:

В поле Definition выбрала Pipeline script.
В поле ниже вставила заготовленный Pipeline скрипт (Jenkinsfile), предоставленный преподавателем.
В Jenkinsfile отдельно отредактировала параметры:

В поле STUDENT_NAME указал своё имя — Safina_Diana
В поле PORT указал уникальный порт - 8038
Сохранилa конфигурацию проекта.

#### Шаг 3. Структура Jenkinsfile

```
pipeline {
    agent any
    
    parameters {
        string(name: 'STUDENT_NAME', defaultValue: 'Safina_Diana', description: 'Имя студента')
        string(name: 'PORT', defaultValue: '8038', description: 'Порт')
    }
    
    stages {
        stage('Удаляем старые контейнеры и образы') {
            steps {
                script {
                    // Останавливаем и удаляем контейнер с нужным именем, если он есть
                    sh "docker ps -a -q --filter name=hello-safina-container | xargs -r docker rm -f"
                    // Удаляем образ с нужным именем, если он есть
                    sh "docker images -q student-safina-app | xargs -r docker rmi -f"
                }
            }
        }
        stage('Выгружаем код из репозитория') {
            steps {
               git branch: 'master', url: 'https://github.com/SafinaDiana/hellojenkins.git'
                // либо git branch:'main', url: 'https://github.com/xDeshka/hellojenkins.git'
            }
        }
        stage('Собираем docker image') {
            steps {
                script {
                    dockerImage = docker.build("student-safina-app")
                }
            }
        }
        stage('Запускаем тесты в докере') {
            steps {
                script {
                    dockerImage.inside {
                        sh 'python -m unittest test_app.py'
                    }
                }
            }
        }
        stage('Запускаем докер контейнер') {
            steps {
                script {
                    sh "docker run -d --name hello-safina-container -p ${params.PORT}:${params.PORT} -e STUDENT_NAME='${params.STUDENT_NAME}' -e PORT=${params.PORT} student-safina-app"
                }
            }
        }
    }
}
```

#### Шаг 4. Настройка GitHub webhook для автоматического запуска сборки
Перешела в репозиторий на GitHub.

Во вкладке Settings выбрала пункт Webhooks.

Нажала кнопку Add webhook.

В поле Payload URL ввела адрес Jenkins вебхука:
http://158.160.194.244:8080/github-webhook/

В поле Content type выбрала application/json.

В разделе Which events would you like to trigger this webhook? выбрала Just the push event.

Подтвердила добавление вебхука.

#### Шаг 5. Работа с Jenkins и проверка
В интерфейсе Jenkins открыла созданную job.

Нажала кнопку Build Now, чтобы инициировать первую сборку.

Проверила, что приложение собрано

Открыла приложение в браузере по адресу: http://158.160.194.244:8038
<img width="1854" height="972" alt="image" src="https://github.com/user-attachments/assets/d8718dfa-e5b6-4cb7-8ef4-1da50c2abf55" />


Изменила в файле index.html цвет текста на красный. Снова инициировала сборку, появилось окошко.

Вывод на http://158.160.194.244:8038/ изменился.

<img width="1858" height="893" alt="image" src="https://github.com/user-attachments/assets/a6acd007-a743-4453-a9d1-544943778585" />




