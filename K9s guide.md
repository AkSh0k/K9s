# Описание
**K9s** — это **терминальный интерфейс (TUI) для Kubernetes**, который позволяет удобно управлять кластерами прямо из терминала.  
Он позволяет графически отслеживать состояние всех ресурсов (pods, deployments, services и т.д.) в реальном времени и даёт быстрый доступ к логам, описаниям, метрикам, exec в контейнер и другим операциям.

Стоит отметить, что K9s работает поверх `kubectl` (официальный CLI-клиент Kubernetes) и использует уже готовый `kubeconfig`.
Данный TUI-интерфейс не заменяет `kubectl`, а просто предоставляет более наглядный и интерактивный способ управления ресурсами кластера.
### Основные возможности:
- просмотр всех ресурсов Kubernetes в режиме реального времени
- запуск команд типа `kubectl describe`, `kubectl logs`, `kubectl exec` без ручного ввода (через терминальный UI)
- фильтры, поиск, горячие клавиши
- управление несколькими контекстами
- перезапуск, удаление, отладка pod’ов
---
# Установка
## Предварительные требования
#### Перед установкой K9s необходимо иметь:
1. Доступ к Kubernetes-кластеру  
	- локальный (minikube, kind)
	- удалённый (bare metal / cloud)
2. Установленный `kubectl`
3. Корректный `kubeconfig`

K9s **не выполняет настройку кластера** и использует существующий `kubeconfig` для подключения к *Kubernetes API*.

Для тестирования K9s был использован **Minikube** — инструмент для запуска локального Kubernetes-кластера на одном компьютере.
### Обновление системы
Перед установкой любых компонентов необходимо обновить и установить базовые утилиты:
```bash
sudo dnf update -y
sudo dnf install -y curl tar podman slirp4netns fuse-overlayfs conntrack socat
```

!()[CleanShot 2025-12-18 at 12.32.12@2x.png]
### Установка `kubectl`
#### Проверка — вдруг уже установлен
```bash
kubectl version --client
```
Если команда не найдена — устанавливаем.
#### Установка официального бинарника Kubernetes
```bash
curl -LO https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl
```
(для ARM — `/bin/linux/arm64/kubectl`)

Выполнение установки и перемещение в соответствующую директорию:
```bash
chmod +x kubectl
sudo mv kubectl /usr/local/bin/
```

Проверка:
```bash
kubectl version --client
```

![[CleanShot 2025-12-18 at 12.34.44@2x.png]]
### Установка и настройка Minikube
```bash
curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64
```
(для ARM — `minikube-linux-arm64`)

#### Выполнение установки и перемещение в соответствующую директорию:
```bash
chmod +x minikube-linux-amd64
sudo mv minikube-linux-amd64 /usr/local/bin/minikube
```

#### Выбор Docker-драйвера и включение режима "Rootless":
```bash
minikube config set driver podman
minikube config set rootless true
```

#### Запуск Minikube с обходом демона Docker (для Podman Rootless):

```bash
minikube start --container-runtime=containerd
```
#### Проверка корректности работы кластера Kubernetes
```bash
kubectl get nodes
```

Если в списке есть нода Minikube, то установка и настройка прошла успешно!
![[CleanShot 2025-12-18 at 17.21.04@2x.png]]
Далее можно переходить к установке самого K9s.
## Установка K9s
### 1. Скачать последнюю версию
```bash
curl -LO https://github.com/derailed/k9s/releases/latest/download/k9s_Linux_amd64.tar.gz
```
(если ARM — `k9s_Linux_arm64.tar.gz`)
### 2. Распаковать
```bash
tar -xzf k9s_Linux_amd64.tar.gz
```
### 3. Переместить бинарник
```bash
sudo mv k9s /usr/local/bin/ 
sudo chmod +x /usr/local/bin/k9s
```
### 4. Проверка
```bash
k9s version
```

---
# Первый запуск K9s
```bash
k9s
```
Что происходит:
- читается `~/.kube/config`
- подключение к текущему context
- загрузка ресурсов кластера  
## Структура интерфейса K9s
![[CleanShot 2025-12-18 at 17.25.40@2x.png]]
### Интерфейс состоит из:
1. **Главная таблица ресурсов**
2. **Status bar (внизу)**
3. **Command prompt (:)**
4. **Hotkeys**

### Навигация:
- ↑ ↓ — перемещение
- Enter — переход внутрь ресурса
- Esc — назад

---

## Основные режимы и команды
Для ввода команд необходимо нажать `:`, после чего откроется строка запросов (*Command Prompt*):
![[CleanShot 2025-12-18 at 17.34.39@2x.png]]

Ниже представлен список всех поддерживаемых команд для отображения таблиц ресурсов и прочих данных:

| Команда   | Что отображает         |
| --------- | ---------------------- |
| `:po`     | Pods                   |
| `:deploy` | Deployments            |
| `:svc`    | Services               |
| `:ns`     | Namespaces             |
| `:ctx`    | Contexts               |
| `:node`   | Nodes                  |
| `:rs`     | ReplicaSets            |
| `:sts`    | StatefulSets           |
| `:ds`     | DaemonSets             |
| `:job`    | Jobs                   |
| `:cj`     | CronJobs               |
| `:ing`    | Ingress                |
| `:cm`     | ConfigMaps             |
| `:sec`    | Secrets                |
| `:pvc`    | PersistentVolumeClaims |
| `:pv`     | PersistentVolumes      |

---

## 6.2 Работа с Pod (самый важный раздел)

Находясь в списке pod’ов:

|Клавиша|Действие|
|---|---|
|`l`|Logs|
|`s`|Shell (exec внутрь контейнера)|
|`d`|Describe|
|`y`|YAML|
|`e`|Edit|
|`r`|Restart|
|`x`|Delete|
|`c`|Переключение контейнера|
|`Ctrl+l`|Очистить экран|

Пример:

`:po → выбрать pod → l`

---

## 6.3 Логи (Logs view)

В режиме логов:

- live-обновление (`<0>` — tail)
    
- фильтрация
    
- переключение контейнеров
    

Полезно:

- `f` — follow
    
- `/` — поиск
    
- `Esc` — назад
    

---

## 6.4 Exec / Shell

`s` — вход в контейнер:

- `/bin/sh`
    
- `/bin/bash` (если есть)
    

Работает как:

`kubectl exec -it pod -- sh`

---

# 7. Namespaces и Contexts

## Namespaces

`:ns`

- `Enter` — выбрать namespace
    
- `a` — показать все
    
- `Ctrl+a` — фильтр
    

## Contexts

`:ctx`

Позволяет:

- переключаться между кластерами
    
- dev / stage / prod
    

---

# 8. Фильтрация и поиск

## Фильтр

`/nginx`

Фильтрует таблицу **на лету**.

## Label selector

`-l app=backend`

---

# 9. Редактирование ресурсов

### Просмотр YAML

`y`

### Inline-редактирование

`e`

Работает как:

`kubectl edit`

---

# 10. RBAC и безопасность

K9s **не обходит RBAC**.

Если пользователь не имеет прав:

- ресурсы не отображаются
    
- операции запрещены
    

Это важно указать в статье:

> K9s полностью подчиняется Kubernetes RBAC.

---

# 11. Конфигурация K9s

Хранится в:

`~/.config/k9s/`

Основные файлы:

- `config.yml`
    
- `skins/`
    
- `views.yml`
    

Пример параметров:

- refresh rate
    
- default namespace
    
- skin / theme
    

---

# 12. Темы (Skins)

`~/.config/k9s/skins/`

Можно:

- менять цвета
    
- адаптировать под dark/light
    
- корпоративные темы
    

---

# 13. Плагины

K9s поддерживает плагины:

- helm
    
- kustomize
    
- внешние команды
    

Настраиваются в `plugin.yml`.

---

# 14. Типовые сценарии использования (обязательно в статье)

1. Отладка упавшего pod
    
2. Просмотр логов в real-time
    
3. Быстрый exec внутрь контейнера
    
4. Рестарт deployment
    
5. Переключение между окружениями
    
6. Мониторинг ресурсов без Grafana
    

---

# 15. Ограничения K9s (важно честно указать)

- не заменяет kubectl
    
- не предназначен для CI/CD
    
- не визуализирует сложные зависимости
    
- не делает автоматические исправления
    

---

# 16. Краткое резюме для статьи

> K9s — это мощный терминальный инструмент для интерактивной работы с Kubernetes.  
> Он упрощает навигацию по ресурсам, ускоряет диагностику проблем и повышает продуктивность инженеров, работающих с кластерами.

---

## Что могу сделать дальше

- адаптировать текст под **учебную работу / отчёт**
    
- сократить до нужного объёма
    
- добавить раздел «K9s vs kubectl»
    
- сделать чек-лист для экзамена/защиты
    
- связать K9s с **RED OS + kubeadm**
    

Скажи, **в каком формате и объёме тебе нужен финальный вариант статьи**.



# Возможные проблемы
## 1. Если на ВМ или компьютере меньше 3000 Мб ОЗУ или 2 ядер ЦПУ, то Minikube будет ругаться


Minikube требует:
- **минимум 2 CPU**
- **минимум ~1800 MiB RAM**


### Очистка контейнеров Minikube
```bash
minikube delete -p minikube
podman rm -af
podman volume rm -f minikube
```

### Запуск с явным указанием объёма памяти и количества ядер ЦПУ
```bash
minikube start --container-runtime=containerd --cpus=2 --memory=1800
```

