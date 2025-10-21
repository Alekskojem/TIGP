# TIGP
### Telegraf InfluxDB Grafana Prometheus NGINX for monitoring VMWare ESXi 
### Telegraf
### InfluxDB v2
### Grafana
### Prometheus
### NGINX
### Docker-compose
docker-cmpose.yml
## Docker Compose
Docker Compose — это «инструмент для определения многоконтейнерных Docker-приложений». Он позволяет создавать несколько контейнеров и автоматически объединять их, а также упрощает развертывание и управление всем стеком TICK. Начнём с определения различных сервисов, входящих в состав стека TICK, в файле docker-compose.yml, а затем развернём эти сервисы с помощью инструмента командной строки Compose.
__ Ниже приведён пример файла docker-compose.yml (с использованием формата файлов Compose версии 3), в котором определены три сервиса: influxdb, influxdb_cli и telegraf:__
'''
version: '3'
services:
  influxdb:
    image: influxdb:2.6-alpine
    env_file:
      - influxv2.env
    volumes:
      # Mount for influxdb data directory and configuration
      - influxdbv2:/var/lib/influxdb2:rw
    ports:
      - "8086:8086"
  telegraf:
    image: telegraf:1.25-alpine
    depends_on:
      - influxdb
    volumes:
      # Mount for telegraf config
      - ./telegraf/mytelegraf.conf:/etc/telegraf/telegraf.conf:ro
    env_file:
      - influxv2.env
volumes:
  influxdbv2:
'''
Чтобы развернуть эти службы, выполните команду docker-compose up -d (как и docker run, аргумент -d запускает контейнеры в режиме headless (отсоединённом)). docker-compose использует каталог influxv2, в котором он выполняется, для наименования различных компонентов, которыми управляет, поэтому рекомендуется разместить файл docker-compose.yml в каталоге с подходящим именем, influxv2. 
В этом примере мы поместим файл docker-compose.yml в каталог influxv2.
Запуск docker-compose up -d выполнит несколько действий:
сначала создаст новую сеть Docker с именем influxv2_default, 
а затем откроет контейнер для каждой из определенных нами служб, назвав их influxv2_influxdb_1, influxv2_influxdb_cli_1 и influxv2_telegraf_1 соответственно.

$ docker-compose up -d
[+] Running 3/3
 ⠿ Network influxdbv2_telegraf_docker_default       Created                0.0s
 ⠿ Container influxdbv2_telegraf_docker-influxdb-1  Started                0.4s
 ⠿ Container influxdbv2_telegraf_docker-telegraf-1  Started                0.7s

$ docker ps -a
CONTAINER ID   IMAGE                  COMMAND                  CREATED          STATUS                    PORTS                          NAMES
e6d60ddc155e   telegraf:1.25-alpine   "/entrypoint.sh tele…"   39 seconds ago   Up 37 seconds             8092/udp, 8125/udp, 8094/tcp   influxdbv2_telegraf_docker-telegraf-1
086e06ca4416   influxdb:2.6-alpine    "/entrypoint.sh infl…"   39 seconds ago   Up 38 seconds             0.0.0.0:8086->8086/tcp         influxdbv2_telegraf_docker-influxdb-1
         Influxdb
У вас должна быть возможность перейти к пользовательскому интерфейсу InfluxDB, посетив http://localhost:8086 , и войти в систему, передав свое имя пользователя и пароль (myu, как определено командой настройки influx в определении точки входа в сервисе influxdb_cli.
В частности, вы будете использовать имя пользователя и парольпароль в качестве имени пользователя и пароля соответственно. Помните, что ваш пароль должен состоять минимум из 8 символов.
Наконец, убедитесь, что параметры настройки InfluxDB соответствуют параметрам конфигурации Telegraf. 
Например, часть вывода mytelegraf.conf будет выглядеть так:
# Output Configuration for telegraf agent
[[outputs.influxdb_v2]]	
  ## The URLs of the InfluxDB cluster nodes.
  ##
  ## Multiple URLs can be specified for a single cluster, only ONE of the
  ## urls will be written to each interval.
  ## urls exp: http://127.0.0.1:8086
  urls = ["http://influxdb:8086"]
  ## Token for authentication.
  token = "$DOCKER_INFLUXDB_INIT_ADMIN_TOKEN"
  ## Organization is the name of the organization you wish to write to; must exist.
  organization = "$DOCKER_INFLUXDB_INIT_ORG"
  ## Destination bucket to write into.
  bucket = "$DOCKER_INFLUXDB_INIT_BUCKET"
  insecure_skip_verify = true

Подведение итогов
Не забудьте очистить все контейнеры, которые вы создали, читая этот пост в блоге! Для этого используйте команды docker stop и docker rm. Еще одна вещь, на которую следует обратить внимание при использовании Docker, — это сами образы контейнеров.
Образ Docker не удаляется автоматически, поэтому со временем, когда вы обновляете версии используемого вами программного обеспечения, старые контейнеры могут накапливаться и начинать занимать дисковое пространство.
Существует несколько сценариев, которые справятся с этим за вас, но по большей части достаточно просто знать о потенциальной проблеме и время от времени выполнять очистку вручную.
Код, связанный с этим блогом, можно найти в этом репозитории.


Я надеюсь, что вы найдете этот блог полезным. Если у вас есть какие-либо вопросы или отзывы о продукте, опубликуйте их на сайте сообщества, канале Slack или напишите нам в Твиттере @InfluxDB. Спасибо!
