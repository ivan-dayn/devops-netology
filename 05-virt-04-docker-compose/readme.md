# Домашнее задание к занятию "5.4. Оркестрация группой Docker контейнеров на примере Docker Compose"


---

## Задача 1

Создать собственный образ операционной системы с помощью Packer.

Для получения зачета, вам необходимо предоставить:
- Скриншот страницы, как на слайде из презентации (слайд 37).

![image](https://user-images.githubusercontent.com/93118042/152298753-155ad5b6-ab20-4533-9054-76c3e0c53d0d.png)

![image](https://user-images.githubusercontent.com/93118042/152298969-a88c1adf-c098-4ae9-9faa-bc42204edd33.png)


## Задача 2

Создать вашу первую виртуальную машину в Яндекс.Облаке.
На память как генерить key.json https://cloud.yandex.ru/docs/iam/quickstart-sa
```bash
yc iam key create --service-account-name my-robot --output key.json
```

![image](https://user-images.githubusercontent.com/93118042/152302495-a6eb8771-94b6-4ee5-b072-d7afa0c4acba.png)



## Задача 3

Создать ваш первый готовый к боевой эксплуатации компонент мониторинга, состоящий из стека микросервисов.

![image](https://user-images.githubusercontent.com/93118042/152304947-737ef08b-d40d-4c7d-a461-0dcbb5c2300f.png)


## Задача 4 (*)

Создать вторую ВМ и подключить её к мониторингу развёрнутому на первом сервере.

С помощью Terraform создал вторую ВМ node02 в том же каталоге и сети, что и node01.
Закрепил внешине адреса созданных ВМ, т.к. в яндекс-облаке они по умолчанию динамические и менялись после рестарта ВМ.
С помощью ansible развернул на node02 nodexporter и cAdvisor, поправив docker-compose.yml (добавил проброс портов)
```yaml
version: '2.1'

networks:
  monitoring:
    driver: bridge

services:

  nodeexporter:
    image: prom/node-exporter:v0.18.1
    container_name: nodeexporter
    volumes:
      - /proc:/host/proc:ro
      - /sys:/host/sys:ro
      - /:/rootfs:ro
    command:
      - '--path.procfs=/host/proc'
      - '--path.rootfs=/rootfs'
      - '--path.sysfs=/host/sys'
      - '--collector.filesystem.ignored-mount-points=^/(sys|proc|dev|host|etc)($$|/)'
    restart: always
    networks:
      - monitoring
    ports:
      - 9100:9100
    labels:
      org.label-schema.group: "monitoring"

  cadvisor:
    image: gcr.io/google-containers/cadvisor:v0.34.0
    container_name: cadvisor
    volumes:
      - /:/rootfs:ro
      - /var/run:/var/run:rw
      - /sys:/sys:ro
      - /var/lib/docker:/var/lib/docker:ro
      - /cgroup:/cgroup:ro
    restart: always
    networks:
      - monitoring
    ports:
      - 8080:8080
    labels:
      org.label-schema.group: "monitoring"
```
Далее поправил конфиг прометея на node1 добавил

```yaml
  - job_name: 'nodeexporter2'
    scrape_interval: 5s
    static_configs:
      - targets: ['node02:9100']

  - job_name: 'cadvisor2'
    scrape_interval: 5s
    static_configs:
      - targets: ['node02:8080']
```
и сделал релоад прометея
```bash
[root@node01 stack]# docker exec -it prometheus kill -HUP 1
```
С графаной также имел дело первый раз) добавил панельку с источником данных с node02
![image](https://user-images.githubusercontent.com/93118042/152520208-ce9828d5-744b-4dbe-a421-fdca4dcd6d4c.png)



