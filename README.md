# Django site

Докеризированный сайт на Django для экспериментов с Kubernetes.

Внутри конейнера Django запускается с помощью Nginx Unit, не путать с Nginx. Сервер Nginx Unit выполняет сразу две функции: как веб-сервер он раздаёт файлы статики и медиа, а в роли сервера-приложений он запускает Python и Django. Таким образом Nginx Unit заменяет собой связку из двух сервисов Nginx и Gunicorn/uWSGI. [Подробнее про Nginx Unit](https://unit.nginx.org/).

## Как запустить dev-версию

Запустите базу данных и сайт:

```shell-session
$ docker-compose up
```

В новом терминале не выключая сайт запустите команды для настройки базы данных:

```shell-session
$ docker-compose run web ./manage.py migrate  # создаём/обновляем таблицы в БД
$ docker-compose run web ./manage.py createsuperuser
```

Для тонкой настройки Docker Compose используйте переменные окружения. Их названия отличаются от тех, что задаёт docker-образа, сделано это чтобы избежать конфликта имён. Внутри docker-compose.yaml настраиваются сразу несколько образов, у каждого свои переменные окружения, и поэтому их названия могут случайно пересечься. Чтобы не было конфликтов к названиям переменных окружения добавлены префиксы по названию сервиса. Список доступных переменных можно найти внутри файла [`docker-compose.yml`](./docker-compose.yml).

## Переменные окружения

Образ с Django считывает настройки из переменных окружения:

`SECRET_KEY` -- обязательная секретная настройка Django. Это соль для генерации хэшей. Значение может быть любым, важно лишь, чтобы оно никому не было известно. [Документация Django](https://docs.djangoproject.com/en/3.2/ref/settings/#secret-key).

`DEBUG` -- настройка Django для включения отладочного режима. Принимает значения `TRUE` или `FALSE`. [Документация Django](https://docs.djangoproject.com/en/3.2/ref/settings/#std:setting-DEBUG).

`ALLOWED_HOSTS` -- настройка Django со списком разрешённых адресов. Если запрос прилетит на другой адрес, то сайт ответит ошибкой 400. Можно перечислить несколько адресов через запятую, например `127.0.0.1,192.168.0.1,site.test`. [Документация Django](https://docs.djangoproject.com/en/3.2/ref/settings/#allowed-hosts).

`DATABASE_URL` -- адрес для подключения к базе данных PostgreSQL. Другие СУБД сайт не поддерживает. [Формат записи](https://github.com/jacobian/dj-database-url#url-schema).

# Kubernetes
## Поднимаем кластер
Чтобы развернуть приложение в кластере, необходимо выполнить следующие команды:

```
minikube start --driver=virtualbox --no-vtx-check
```

## Разворачиваем приложение
- Загрузите образ django в minikube (PowerShell Windows)
```
& minikube -p minikube docker-env --shell powershell | Invoke-Expression
docker build -t django_app E:\DevMan\k8s-test-django-main\backend_main_django
```
Присвойте тег
```
& minikube -p minikube docker-env --shell powershell | Invoke-Expression
docker tag django_app:latest django_app:devman
```
```
kubectl apply -f configmap.yaml
```
```
kubectl apply -f django_deploy.yaml
```
```
minikube service list
```

Если данные в configmap.yaml изменились необходимо выполнить следующие команды:
```
kubectl apply -f configmap.yaml
```
```
kubectl rollout restart deployment django-deployment
```

## Запуск ingress
```
kubectl apply -f ingress.yaml
```
```
inikube addons enable ingress
```
```
kubectl get ingress
```
- полученный из предыдущего пункта ADDRESS добавить в /etc/hosts указав соответствие IP (ADDRESS) и host
- проверить доступность приложения в браузере
- если на втором шаге возникает ошибка, можно удалить кластер и установить все по новой

```
minikube delete --all
```

## Удаление сессий
```
kubectl apply -f django_clearsessions_job.yaml
```

## Миграции

```
kubectl apply -f django_migrate_job.yaml
```

For creation one time job from existing cronjob

## Запуск postgres с помощью helm chart

- Установить [helm](https://github.com/helm/helm/releases)
- Не забудьте прописать путь до helm в системную переменную PATH
- Запустить release of postgresql chart
```
helm install my-release oci://registry-1.docker.io/bitnamicharts/postgresql
```

- При запуске вы увидите подробные инструкции, как подключиться к БД


