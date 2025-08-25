# Настройка n8n на базе облака Yandex
### Установка и настройка n8n в Yandex Cloud. Развертывание и тестирование автоматических процессов.
---
* Время на чтение: 5 мин.
* Время затраченное на реализацию:
---
Проект призван реализовать развертывание серверной платформы для малого и среднего предприятия, с высокой надёжностью, с целью автоматизации своимх процессов и предоставление услуг **SaaS**, с возможностью дальнейшего масшатбирования, на базе облачных сервисов. Основная платформа для настройки является [Yandex Cloud](https://yandex.cloud/). Также, с целью тестирования, будет развёрнута тестовая версия на платформке [VK Cloud](https://cloud.vk.com/).

### Используемое ПО:
Yandex Cloud:
- Ubuntu 24.04 LTS
- Yandex Cloud CLI 0.160.0
- 
VK Cloud:
- OpenStack
---
Предположим, что у нас уже создано отдельное облако и каталог в среде 
Yandex Cloud. Если нет, то как это сделать можно прочитать [здесь](https://yandex.cloud/ru/docs/resource-manager/operations/cloud/create). 

### 1. Установка серверов Yandex Cloud
#### 1.1 Создаём облачную сеть:
```
yc vpc network create `
  --name n8n-network `
  --description "Облачная сеть для n8n"
```
\* Здесь и далее приводятся команды PowerShell. Для Windows CMD и Bash потребуется небольшая адаптация.

Это же действие в консоли Yandex Cloud:

![Network Create в консоли Yandex Cloud](https://github.com/sword76/n8nYandexCloud/blob/main/assets/network_create.png)

#### 1.2 Создаём подсети в разных зонах доступности
Для этой задачи выбираем зоны **ru-central1-a** и **ru-central1-b**.
```
yc vpc subnet create `
  --name n8n-subnet-a `
  --zone ru-central1-a `
  --range 10.1.2.0/24 `
  --network-name n8n-network `
  --description "Подсеть A для n8n"
```
```
yc vpc subnet create `
  --name n8n-subnet-b `
  --zone ru-central1-b `
  --range 10.2.2.0/24 `
  --network-name n8n-network `
  --description "Подсеть B для n8n"
```

Yandex Cloud:

![Subnet Create действие в консоли Yandex Cloud](https://github.com/sword76/n8nYandexCloud/blob/main/assets/subnet_create.png)

#### 1.3 Создаём виртуальные машины

На начальном этапе воспользуемся минимальной конфигурацией:
- 2 vCPU
- 2 GB RAM
- 20 ГБ SSD
- Ubuntu 24.04 LTS

Создаём SSH-ключ для доступа к ВМ любым удобным способом. Подробности можно прочитать [здесь](https://yandex.cloud/ru/docs/baremetal/operations/servers/add-new-ssh-key).



