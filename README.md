# Домашнее задание к занятию «Helm»

> Репозиторий: hw-41\
> Выполнил: Асадбек Асадбеков\
> Дата: сентябрь 2025\
> Ссылка на манифесты: https://github.com/asad-bekov/hw-41/blob/main/manifests

## Задание 1. Подготовить Helm-чарт для приложения

1. Необходимо упаковать приложение в чарт для деплоя в разные окружения.
2. Каждый компонент приложения деплоится отдельным `deployment`ом или `statefulset` ом.
3. В переменных чарта измените образ приложения для изменения версии.


## Cоздание базового чарта

```bash
# Создать шаблон чарта
helm create myapp
```

**Скриншот 1 — структура созданного чарта:**
![helm create myapp](https://github.com/asad-bekov/hw-41/blob/main/img/1.PNG)

---

## Внесённые изменения в шаблон

Вместо одного монолитного шаблона были добавлены отдельные ресурсы для каждого компонента:

- `frontend-deployment.yaml` + `frontend-service.yaml`
- `backend-deployment.yaml` + `backend-service.yaml`
- `database-statefulset.yaml` + `database-service.yaml`

**Скриншот 2 — структура `templates/` после модификации:**
![tree myapp](https://github.com/asad-bekov/hw-41/blob/main/img/2.PNG

---

## Проверка чарта

Проверяем валидность чарта с помощью `lint` а Helm:

```bash
helm lint myapp
```

**Скриншот 3 — результат `helm lint`:**
![helm lint](https://github.com/asad-bekov/hw-41/blob/main/img/3.PNG)

---

## Задание 2. Запустить две версии в разных неймспейсах

1. Подготовив чарт, необходимо его проверить. Запуститe несколько копий приложения.
2. Одну версию в `namespace=app1`, вторую версию в том же неймспейсе, третью версию в `namespace=app2`.
3. Продемонстрируйте результат.

### 1) Создание неймспейсов

```bash
kubectl create namespace app1
kubectl create namespace app2
```

**Скриншот 4 — создание неймспейсов:**
![namespaces](https://github.com/asad-bekov/hw-41/blob/main/img/5.PNG)

### 2) Установка релизов (примеры)

Устанавливаем несколько релизов чарта `myapp` с разными тегами образов.

**Версия 1 в `app1`:**

```bash
helm install app1-v1 myapp -n app1 \
  --set frontend.image.tag="1.25" \
  --set backend.image.tag="7.2" \
  --set database.image.tag="15"
```

**Скриншот 5.1 — установка `app1-v1`:**
![`app1-v1`](https://github.com/asad-bekov/hw-41/blob/main/img/6.1.PNG)

**Версия 2 в `app1`:**

```bash
helm install app1-v2 myapp -n app1 \
  --set frontend.image.tag="1.24" \
  --set backend.image.tag="7.0" \
  --set database.image.tag="14"
```

**Скриншот 5.2 — установка `app1-v2`:**
![`app1-v2`](https://github.com/asad-bekov/hw-41/blob/main/img/6.2.PNG)

**Версия 1 в `app2`:**

```bash
helm install app2-v1 myapp -n app2 \
  --set frontend.image.tag="1.23" \
  --set backend.image.tag="6.2" \
  --set database.image.tag="13"
```

**Скриншот 5.3 — установка `app2-v1`:**
![`app2-v1`](https://github.com/asad-bekov/hw-41/blob/main/img/6.3.PNG)

---

## Демонстрация результатов и проверка

**Список всех релизов Helm (по всем namespace)**
![helm list -A](https://github.com/asad-bekov/hw-41/blob/main/img/7.PNG)

**Поды в namespace app1**
![kubectl get pods -n app1](https://github.com/asad-bekov/hw-41/blob/main/img/8.PNG)

**Поды в namespace app2**
![kubectl get pods -n app2](https://github.com/asad-bekov/hw-41/blob/main/img/9.PNG)

**Сервисы в namespace app1 и app2**
![kubectl get svc -n app1 ...app2](https://github.com/asad-bekov/hw-41/blob/main/img/10.PNG)

**Deployments и StatefulSets в app1**
![kubectl get deploy,statefulset -n app1](https://github.com/asad-bekov/hw-41/blob/main/img/11.PNG)


---

## Структура чарта (пример)

```
myapp/
├── Chart.yaml
├── values.yaml
└── templates/
    ├── frontend-deployment.yaml
    ├── frontend-service.yaml
    ├── backend-deployment.yaml
    ├── backend-service.yaml
    ├── database-statefulset.yaml
    ├── database-service.yaml
    ├── NOTES.txt
    └── _helpers.tpl
```

---

## Переменные чарта (в `values.yaml`)

Основные настраиваемые параметры:

- `frontend.image.repository` / `frontend.image.tag` — образ frontend
- `backend.image.repository` / `backend.image.tag` — образ backend
- `database.image.repository` / `database.image.tag` — образ БД
- `replicaCount` — количество реплик для деплойментов
- `service.type` / `service.port` — параметры сервисов
- Прочие параметры: ресурсы, переменные окружения, storage для БД и т.д.

---

**`values.yaml`**

<details>
<summary>Показать</summary> 

```yaml
global:
  environment: production
frontend:
  enabled: true
  image:
    repository: nginx
    tag: "1.25"
    pullPolicy: IfNotPresent
  replicaCount: 2
  service:
    type: ClusterIP
    port: 80
backend:
  enabled: true  
  image:
    repository: redis
    tag: "7.2"
    pullPolicy: IfNotPresent
  replicaCount: 1
  service:
    type: ClusterIP
    port: 6379
database:
  enabled: true
  statefulset: true
  image:
    repository: postgres
    tag: "15"
    pullPolicy: IfNotPresent
  replicaCount: 1
  service:
    type: ClusterIP
    port: 5432
serviceAccount:
create: false
```
</details>

---

## Результат

- Развернуто 3 версии приложения:
  - 2 релиза в namespace `app1` с разными версиями образов
  - 1 релиз в namespace `app2`
- Компоненты работают изолированно в своих неймспейсах
- Продемонстрирована возможность управления версиями через Helm

---

## Команды очистки (*при необходимости*)

# Удалить релиз

```bash
helm uninstall app1-v1 -n app1
helm uninstall app1-v2 -n app1
helm uninstall app2-v1 -n app2
```

# Удалить namespace 

```bash
kubectl delete namespace app1
kubectl delete namespace app2
```

---

