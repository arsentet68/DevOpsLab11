# ФИТ-2024 НМ. Полынский Арсений. DevOps. ЛР11

## Клонируем этот репозиторий с гитхаба

<img width="727" height="332" alt="image" src="https://github.com/user-attachments/assets/33217853-2869-4537-95f5-d86d6a0a2234" />

## Будущая структура каталогов

<img width="359" height="195" alt="image" src="https://github.com/user-attachments/assets/a43643ab-cae2-47f4-a58b-f4e16582baad" />

## Приложение

server/application.py

```
import http.server
import socketserver

PORT = 8000

class TestMe():
    def take_five(self):
        return 4
    def port(self):
        return PORT

if __name__ == '__main__':
    Handler = http.server.SimpleHTTPRequestHandler

    with socketserver.TCPServer(("", PORT), Handler) as http:
        print ("serving at port", PORT)
        http.serve_forever()
```

## Юнит-тесты

server/test_application.py

```
import pytest

from application import TestMe

def test_server():
    assert TestMe().take_five() == 5

def test_port():
    assert TestMe().port() == 8000
```

## Файл с зависимостями для самих тестов

requirements.txt

```
pylint
pytest
```

## Файл для сборки образа

dockerfile

```
FROM python:3.7-slim

RUN mkdir -p /usr/local/http-server

RUN useradd runner -d /home/runner -m -s /bin/bash

WORKDIR /usr/local/http-server

ADD ./application.py /usr/local/http-server/application.py

RUN chown -R runner:runner /usr/local/http-server/

EXPOSE 8000
USER runner
CMD ["python3", "-u", "usr/local/http-server/application.py"]
```

## Проверяем, что всё пушится в гитхаб

<img width="707" height="640" alt="image" src="https://github.com/user-attachments/assets/a85fe4a4-8cda-4524-a4fa-fad0ee51a231" />

## Переводим разработку в отдельную ветку dev

<img width="629" height="266" alt="image" src="https://github.com/user-attachments/assets/18e1660f-ccc3-48f2-990c-7793be899d14" />

## Защищаем ветку main от прямых изменений, теперь только merge-requests

<img width="1154" height="440" alt="image" src="https://github.com/user-attachments/assets/6ccca1ac-890f-4de7-81e8-cbf45f0a4bd2" />

<img width="983" height="427" alt="image" src="https://github.com/user-attachments/assets/53cd5b69-7eec-4026-a0d7-5df7ea7acee6" />

<img width="980" height="224" alt="image" src="https://github.com/user-attachments/assets/9e613d11-e62c-43d9-8991-542c47352c42" />

<img width="1005" height="652" alt="image" src="https://github.com/user-attachments/assets/5217f806-d175-46e4-b468-1de0d0989a20" />

## Пример: Теперь новая работа вносится только в dev

```
git checkout dev

... Работаем...

git status
git add <файлы>
git commit -m <сообщение>
git push
```

Для переноса кода в main идём в гитхаб и производим merge/pull request>approve

## Пример: добавляем индексный файл, пушим, открываем pull/merge-request

<img width="592" height="351" alt="image" src="https://github.com/user-attachments/assets/a58915f7-c4cf-4894-a88f-f0a4f359ee27" />

<img width="1131" height="233" alt="image" src="https://github.com/user-attachments/assets/e9114347-8be3-427b-b779-1b4526c32cf1" />

<img width="1147" height="819" alt="image" src="https://github.com/user-attachments/assets/2385e959-2e25-4480-bf72-0ba2e124a5f9" />

<img width="1161" height="685" alt="image" src="https://github.com/user-attachments/assets/740aa97a-6ccd-49ed-9cb7-3889e9573612" />

# CI = Continuous Integration

## Добавляем CI-пайплайны (сценарии) - Github Actions

<img width="1542" height="636" alt="image" src="https://github.com/user-attachments/assets/78608e96-53d5-43c9-ac55-6be7c437a66c" />

<img width="357" height="298" alt="image" src="https://github.com/user-attachments/assets/bba93cbb-b871-41e9-a398-89fe7d9bc36a" />

## Тестовый сценарий - посмотреть как работает runner на стороне GitHub

.github/workflows/devops_course_pipeline.yml

```
name: Github Actions DevOps course
on: push
jobs:
  DevOps-Course-first-job:
    runs-on: ubuntu-latest
    steps:
      - run: echo "I run my first Github Actions"
      - name: Show uptime
        run: uptime
      - name: Where am I?
        run: pwd
      - name: Who am I?
        run: whoami
```

## Добавьте сценарии CI в гит

```
git add .github/
git commit -m ci
git push
```

<img width="1087" height="784" alt="image" src="https://github.com/user-attachments/assets/447a04af-1439-473a-953b-eee9f24af56f" />

## Боевой сценарий - линтер > юнит-тесты > интеграционный тест

.github/workflows/cicd.yml

```
name: LINT-TEST-BUILD-CHECK
on:
  push:
    branches: [dev]

jobs:

  lint:
    runs-on: ubuntu-24.04
    steps:
    - uses: actions/checkout@v4
    - uses: actions/setup-python@v5
      with:
        python-version: '3.10'
    - run: pip install -r requirements.txt; pylint server/application.py

  unit-tests:
    runs-on: ubuntu-24.04
    steps:
    - uses: actions/checkout@v4
    - uses: actions/setup-python@v5
      with:
        python-version: '3.10'
    - run: pip install -r requirements.txt; pytest --junitxml=junit/test-results.xml

  build-test:
    needs: [ lint, unit-tests ]
    runs-on: ubuntu-24.04
    steps:
    - uses: actions/checkout@v4
    - run: docker build -t test-image ./server --file ./server/dockerfile
    - run: docker run -d --name app-server8000 -p 8000:8000 --restart unless-stopped test-image
    - run: sleep 11
    - run: docker ps -a
    - run: sleep 11
    - run: 'curl 127.0.0.1:8000'
```

## Добавляем новый сценарий в гит

<img width="519" height="213" alt="image" src="https://github.com/user-attachments/assets/7bc549d1-6b47-4641-81ae-a2b4c61691e2" />

<img width="641" height="311" alt="image" src="https://github.com/user-attachments/assets/60d1d058-34b9-4faf-b6b2-a828ed3e73fc" />

## Проверяем как отрабатывает новый сценарий

<img width="1894" height="716" alt="image" src="https://github.com/user-attachments/assets/8ccf1f11-bd1e-49a3-8c7f-b75a6d46fee2" />

## Отлаживаем приложение, чтобы все тесты прошли успешно

<img width="1252" height="829" alt="image" src="https://github.com/user-attachments/assets/6724d9da-bd53-4e9f-b176-329f7eaff972" />

Открываем последний прогон пайплайна и смотрим крайние ошибки, на которые ругается pylint и pytest

Например, если в организации не требуются комментарии для классов, то эти сигнатуры линтера можно отключить.
Добавляем в пайплайне .github/workflows/cicd.yml на стадии линтера ключ -d и перечисляем коды сигнатур для отключения:

<img width="975" height="31" alt="image" src="https://github.com/user-attachments/assets/9eaf8b23-7327-4b8f-abe3-20fb5a9fc7cd" />

Добавляем файл в новый коммит и пушим

<img width="859" height="309" alt="image" src="https://github.com/user-attachments/assets/beae7647-7b8b-48b1-b0c2-2388e54a0162" />

## Куберизуем наше приложение
## Создаём для приложения k8s-манифесты

<img width="615" height="45" alt="image" src="https://github.com/user-attachments/assets/540c2d71-4db6-44ad-a8ba-11cad51cceff" />

