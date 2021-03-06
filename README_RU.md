[English version](/README.md)

# iondv-app
Установщик приложений IONDV. Framework из репозитория кода.

Устанавливает, собирает и запускает приложение из репозитория кода git, из архива zip (в т.ч. скачанного с визуального конструктора 
[IONDV. Studio](https://studio.iondv.com)) или с папки с файлами приложения (папка копируется, файлы в папке не меняются).

### Требования к окружению
По умолчанию используется метод сборки в docker-контейнере. Описание подготовки системы приведено ниже в разделе 
[Подготовка окружения](#Подготовка-окружения).

Для работы системы должна быть запущенна СУБД Монго ДБ. В docker-контейнере её можно запустить командой:
```
docker run --name mongodb -v mongodb_data:/data/db -p 27017:27017 -d mongo
```

### Установка, сбора и запуск приложения одной командой в докер-контейнере:
Примеры сборок ниже, подразумевают что СУБД запущена в контейнере с именем mongodb.

На примере простейшего приложения [nutrition-tickets](https://github.com/iondv/nutrition-tickets), команда скачивает установщик
с репозитория github, который собирает и запускает приложение в docker-контейнере в текущей директории.

* из репозтория git
```
bash <(curl -sL https://raw.githubusercontent.com/iondv/iondv-app/master/iondv-app) \
      -t docker -q -i -l mongodb develop-and-test
```
* из архива zip, например полученного с гитхаб 
`curl -L https://github.com/iondv/nutrition-tickets/archive/master.zip > ./nutrition-tickets.zip` или созданного в 
[IONDV. Studio](https://studio.iondv.com). Обратите внимание, что в `package.json` созданного приложения в атрибуте
`"ionModulesDependencies"` нужно указать модуль для отображения данным - обычно это `"registry"`.
```
bash <(curl -sL https://raw.githubusercontent.com/iondv/iondv-app/master/iondv-app) \
      -t docker -q -i -l mongodb ./nutrition-tickets.zip
```
* из папки, при этом оригинальная папка приложения не модифицируется. Обратите внимание, что название папки должно соответствовать неймспейсу приложения (если папка распакована с архива github - то в названии обычно добавляется код ветки - нужно переименовать)
```
bash <(curl -sL https://raw.githubusercontent.com/iondv/iondv-app/master/iondv-app) \
      -t docker -q -i -l mongodb ./nutrition-tickets
```

Адрес собранного приложения по умолчанию http://localhost:8888, пользователь `demo`, пароль `ion-demo`.

### Установка, сборка и запуск приложения в локальной файловой системе
Примеры сборок ниже, подразумевают что СУБД запущена локально и доступна по адресу localhost:27017.

Установка в локальной файловой системе позволяет получить приложение готовое к модификации и доработкам средствами разработчика, 
например в IDE - достаточно открыть папку приложения. Папка приложения создается в папке запуска, либо в папке заданной 
параметром `-p`

* из репозитория git в папке `/workspace`
```
bash <(curl -sL https://raw.githubusercontent.com/iondv/iondv-app/master/iondv-app) \
  -t git -p /workspace -m localhost:27017 https://github.com/iondv/nutrition-tickets.git
```
* из архива zip в текущей папке, например полученного с гитхаб 
`curl -L https://github.com/iondv/nutrition-tickets/archive/master.zip > ./nutrition-tickets.zip` или созданного в 
[IONDV. Studio](https://studio.iondv.com). Обратите внимание, что в `package.json` созданного приложения в атрибуте
`"ionModulesDependencies"` нужно указать модуль для отображения данным - обычно это `"registry"`

```
bash <(curl -sL https://raw.githubusercontent.com/iondv/iondv-app/master/iondv-app) \
  -t git -p /workspace -m localhost:27017 ./nutrition-tickets.zip
```

* из папки, при этом оригинальная папка приложения не модифицируется. Обратите внимание, что название папки должно соответствовать неймспейсу приложения (если папка распакована с архива github - то в названии обычно добавляется код ветки - нужно переименовать)
```
bash <(curl -sL https://raw.githubusercontent.com/iondv/iondv-app/master/iondv-app) \
      -t git -q -i -m localhost:27017 ./nutrition-tickets
```

## Параметры запуска
```
iondv-app [OPTION]... IONDV_APP_NAME|IONDV_APP_NAME@VERSION|GIT_URL|IONDV_APP_ZIP|IONDV_APP_PATH
   
Параметры:
Тип сборки приложения:
  -t [value]                   git: клонирование репозиториев в файловую систему (требуется установленный git)
                               docker: сборка в докер-контейнерах, не требует установки окружения на хост-машину
Универсальные параметры для разных типов:
  -c [value]                  запуск приложения как кластера с кол-вом инсталляций [value]
  -m [value]                  uri для монгодб, примеры: mongodb:27017. localhost:27017 - по умолчанию (при сборке
                              в докере выдаст ошибку подключения к БД(!). Для докера используйте параметр -l,
                              либо укажите внешний адрес СУБД
  -r                          проверка и удаление папки с именем приложения в директории сборки 
  -i                          импорт  данных при инициализации приложения
  -a                          импорт ролей и учетных записей пользователей при инициализации приложения
  -y                          применение всех значений по умолчанию (yes to all)
  -q                          тихий режим. Показывается только основная информация, предупреждения и ошибки
  -l [value]                  имя контейнера MongoDB для линковки к собранному контейнеру (тип сборки docker 
                              или параметр -d при типе сборки git), также формирует конфигурацию с указанием
                              значения mongo uri как [value]:27017
  -p [value]                  путь к директории в которой будет создавать папка с именем приложения и 
                              осуществляться сборка
  -s [value]                  полный путь к скрипту, запускаемому в папке приложения после сборки, но до деплоя 
                              приложения. Может использоваться для дополнительной обработки файлов приложения
  -n [value]                  параметри определяющий запуск изменение неймспейса приложения на новое, до деплоя
  -h                          пропуск переключения на версии зависимостей приложения, установка последних версий
  -x                          выход без запуска приложения
Параметры для метода git:
  -d                          на основе собранной версии подготовить также docker-контейнер. Также остановить и 
                              удалить контейнер, образ с таким именем
  -k                          пропустить проверку окружения
Параметры для метода сборки docker:
  -v                          сохранять временные версии контейнеров - позволяет ускорить последующие сборки. Но 
                              кэширование пропускается, если установлен флаг игнорировать версии зависимостей
Переменные окружения:
  IONDVUrlGitFramework        URL репозитория фреймворка, по умолчанию https://github.com/iondv/framework.git
                              Вы можете задать логин и пароль к своей версии в приватном репозитории. Напрмер:
                              https://login:password@git.company-name.com/iondv/framework.git
  IONDVUrlGitModules          URL к модулям, по умолчанию by default https://github.com/iondv
  IONDVUrlGitApp              URL к приложениям - используется если для сборки указано только имя приложения,
                              по умолчанию https://github.com/iondv
  IONDVUrlGitExtApp           URL к приложениям-расширениям, по умолчанию https://github.com/iondv
```

## Подготовка окружения
### Установка docker
Рекомендуется делать не под root

* Установка последней версии docker для CentOS:
```
# Обновляем систему
sudo yum update

# Устанавливаем необходимые библиотеки 
yum install -y yum-utils device-mapper-persistent-data lvm2
#Регистрируем  репозиторий 
yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
# Установка последней версии 
yum -y install docker-ce docker-ce-cli containerd.io
#Запускаем докер
systemctl start docker
#Для автоматического запуска докера 
systemctl enable docker
```

* Установка последней версии docker для Ubuntu:
```
# Добавляем ключ GDP
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
# Проверяем ключ
apt-key fingerprint 0EBFCD88
# Добавляем репозиторий
sudo add-apt-repository \
   "deb [arch=amd64] https://download.docker.com/linux/ubuntu \
   $(lsb_release -cs) \
   stable"
# Обновляем репозитории
sudo apt-get update
# Ставим последнюю версию
sudo apt-get install docker-ce docker-ce-cli containerd.io
```

Добавляем текущего пользователя в группу docker:
```
sudo groupadd docker
sudo usermod -aG docker $USER
```

Проверить можно `docker run hello-world`

### Запуск Mongo в докере

Запускаем с маппингом на локальный порт:
```
docker run --name mongodb -v mongodb_data:/data/db -p 27017:27017 -d mongo
```

### Установка node
Для ускорения сборки, рекомендуется предварительно скачать локально docker-образ node:10, т.к. он занимает 900Мб.
```
docker pull node:10
```

Проверить можно командой `docker images | grep node` - будет отображён спискок локальных образов node
