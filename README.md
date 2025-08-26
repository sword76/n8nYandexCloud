# 🚀 Развертывание n8n в Yandex Cloud

Краткое руководство по установке и настройке серверной платформы для автоматизации процессов на базе **n8n** в Yandex Cloud. Идеально подходит для малого и среднего бизнеса с перспективой масштабирования до SaaS-решения.

---
**⏱ Время на чтение:** ~5 минут  
**⚙️ Время на реализацию:** Зависит от опыта

---

### 🧰 Используемый стек:

**Yandex Cloud:**
- Ubuntu 24.04 LTS
- Yandex Cloud CLI
- Docker & Docker Compose
- n8n
- PostgreSQL

**Для тестирования (опционально):**
- VK Cloud (на базе OpenStack)

---

## 🏗 Шаг 1: Подготовка инфраструктуры в Yandex Cloud

Предполагаем, что у вас уже есть облако и каталог в YC. Если нет — [создайте их]((https://yandex.cloud/ru/docs/resource-manager/operations/cloud/create)).

### 1.1. Создаем облачную сеть

Запустите в PowerShell (для Bash/CMD потребуется адаптация):

```powershell
yc vpc network create `
  --name n8n-network `
  --description "Сеть для n8n и всего такого"
```

Или через веб-консоль:

![Создание сети в консоли Yandex Cloud](https://github.com/sword76/n8nYandexCloud/blob/main/assets/network_create.png)

### 1.2. Создаем подсети в двух зонах доступности

**Зона A (ru-central1-a):**
```powershell
yc vpc subnet create `
  --name n8n-subnet-a `
  --zone ru-central1-a `
  --range 10.1.2.0/24 `
  --network-name n8n-network
```

**Зона B (ru-central1-b):**
```powershell
yc vpc subnet create `
  --name n8n-subnet-b `
  --zone ru-central1-b `
  --range 10.2.2.0/24 `
  --network-name n8n-network
```

Через консоль это выглядит так:

![Создание подсети](https://github.com/sword76/n8nYandexCloud/blob/main/assets/subnet_create.png)

### 1.3. Разворачиваем виртуальные машины

Сначала получим актуальный ID образа Ubuntu 24.04:
```bash
yc compute image get-latest-from-family ubuntu-2404-lts --folder-id standard-images
```

**Создаем ВМ с минимальной конфигурацией (2vCPU/2GB/20GB):**

```powershell
# Создаем первый инстанс в зоне А
yc compute instance create `
  --name n8n-vm1 `
  --zone ru-central1-a `
  --network-interface subnet-name=n8n-subnet-a,nat-ip-version=ipv4 `
  --create-boot-disk image-id=fd8tekibj7eld4eqtaed,size=20 ` # Используй актуальный ID из команды выше!
  --memory=2 `
  --cores=2 `
  --ssh-key ~/.ssh/n8n_ya.pub # Укажи верный путь к своему PUB-ключу
```

Повтори команду для `n8n-vm2`, заменив зону на `ru-central1-b` и имя подсети.

**Подключение к серверу:**
```bash
ssh -i ~/.ssh/n8n_ya yc-user@<ВНЕШНИЙ_IP_ВМ>
```
Узнать IP можно командой: `yc compute instance list`

---

## 🐳 Шаг 2: Устанавливаем Docker и Docker Compose

Выполни на каждой машине:

```bash
# Обновляем список пакетов и ставим зависимости
sudo apt-get update && sudo apt-get install -y ca-certificates curl

# Добавляем официальный GPG-ключ Docker
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc

# Добавляем репозиторий в источники APT
echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu $(. /etc/os-release && echo "${UBUNTU_CODENAME:-$VERSION_CODENAME}") stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

# Ставим Docker
sudo apt-get update && sudo apt-get install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin

# Запускаем и проверяем
sudo service docker start
sudo service docker status
```

---

## 🗃 Шаг 3: Настраиваем кластер PostgreSQL (опционально)

*Примечание: Для продакшена мы рекомендуем использовать управляемую БД (например, Yandex Managed Service for PostgreSQL) для большей надежности. Для теста можно развернуть свой экземпляр.*

Стандартная установка n8n использует SQLite, но для кластера ВМ нужна более надежная БД. Пример с развертыванием в Docker можно найти в официальной документации n8n.
