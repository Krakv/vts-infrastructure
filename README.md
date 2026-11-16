# VoiceTaskSystem — Infrastructure

Инфраструктурная часть проекта [VoiceTaskSystem](https://github.com/Krakv/VoiceTaskSystem).

Репозиторий содержит Ansible-манифесты для автоматизированного развёртывания Kubernetes-кластера.

Основная задача — подготовить серверы, установить необходимые компоненты и объединить их в единый Kubernetes-кластер.

## Архитектура

По умолчанию проект разворачивает кластер из трёх узлов:

| Узел | Роль | IP-адрес |
|---|---|---|
| k8s-master | Control Plane | 192.168.56.10 |
| k8s-worker-01 | Worker | 192.168.56.11 |
| k8s-worker-02 | Worker | 192.168.56.12 |

Для настройки инфраструктуры используются:

- **Ansible** — автоматизация установки и конфигурации узлов.
- **containerd** — контейнерная среда выполнения.
- **kubeadm** — инициализация Kubernetes-кластера.
- **Flannel** — организация сетевого взаимодействия между Pod.
- **Local Path Provisioner** — предоставление локального хранилища.

## Как работает автоматизация

Развёртывание выполняется последовательно с помощью Ansible Playbook `site.yml`.

1. **Подготовка узлов (`common`)** — отключение swap.
2. **Установка containerd (`containerd`)** — установка containerd, runc и CNI Plugins, настройка SystemdCgroup.
3. **Подготовка Kubernetes (`kubernetes_common`)** — настройка модулей ядра и сетевых параметров, установка kubelet и kubeadm.
4. **Инициализация кластера (`control_plane`)** — выполнение `kubeadm init`, установка Flannel и Local Path Provisioner.
5. **Подключение рабочих узлов (`worker`)** — выполнение команды `kubeadm join` для добавления Worker Nodes.

## Требования

- Linux-серверы на базе Debian/Ubuntu.
- Ansible на управляющей машине.
- SSH-доступ ко всем узлам.
- SSH-ключ для подключения к серверам.
- Пользователь с возможностью выполнения команд через `sudo`.
- Доступ к интернету на узлах для загрузки необходимых компонентов.

Для Ansible также требуются коллекции:

```bash
ansible-galaxy collection install community.general ansible.posix
```

## Настройка Inventory

Основная конфигурация узлов находится в `ansible/inventory/hosts.yml`.

Перед развёртыванием необходимо указать собственные адреса серверов, пользователя и путь к приватному SSH-ключу.

Пример:

```yaml
all:
  children:
    control_plane:
      hosts:
        k8s-master:
          ansible_host: 192.168.56.10
          node_ip: 192.168.56.10
          ansible_user: ubuntu
          ansible_ssh_private_key_file: ~/.ssh/id_ed25519

    workers:
      hosts:
        k8s-worker-01:
          ansible_host: 192.168.56.11
          node_ip: 192.168.56.11
          ansible_user: ubuntu
          ansible_ssh_private_key_file: ~/.ssh/id_ed25519

        k8s-worker-02:
          ansible_host: 192.168.56.12
          node_ip: 192.168.56.12
          ansible_user: ubuntu
          ansible_ssh_private_key_file: ~/.ssh/id_ed25519

  vars:
    pod_network_cidr: 10.244.0.0/16
    kubernetes_version: "1.37"
    flannel_iface: eth1
    control_plane_endpoint: 192.168.56.10
```

Важно учитывать:

- `ansible_host` — адрес для SSH-подключения.
- `node_ip` — адрес, который будет использовать Kubernetes для узла.
- `ansible_user` — пользователь для подключения.
- `ansible_ssh_private_key_file` — путь к приватному SSH-ключу.
- `control_plane_endpoint` — адрес управляющего узла, доступный всем участникам кластера.
- `flannel_iface` — сетевой интерфейс, используемый Flannel.

При использовании собственных серверов необходимо изменить сетевые параметры в соответствии с их конфигурацией.

## Развёртывание

Все команды выполняются из директории `ansible`.

### 1. Первоначальная подготовка

```bash
ansible-playbook playbooks/bootstrap.yml
```

Playbook устанавливает Python 3 на узлы, если он отсутствует.

### 2. Развёртывание Kubernetes-кластера

```bash
ansible-playbook playbooks/site.yml
```

После выполнения команды Ansible автоматически подготовит все узлы, инициализирует Control Plane и подключит Worker Nodes.

### 3. Проверка кластера

```bash
kubectl get nodes -o wide
```

Все три узла должны находиться в состоянии `Ready`.

## Локальное развёртывание с Vagrant

В репозитории также предусмотрен `Vagrantfile`, позволяющий эмулировать инфраструктуру из трёх виртуальных машин.

Vagrant используется исключительно для удобства локальной разработки и тестирования Ansible-манифестов. Он не является обязательным требованием для развёртывания Kubernetes-кластера.

Для создания виртуальных машин используется:

- Debian 12 (`generic/debian12`);
- Libvirt в качестве провайдера виртуализации;
- приватная сеть `192.168.56.0/24`;
- статические IP-адреса, соответствующие Ansible Inventory;
- по 2 CPU и 2 GB RAM на каждую виртуальную машину.

Для запуска необходимо установить Vagrant и плагин `vagrant-libvirt`, затем выполнить из корня репозитория:

```bash
vagrant plugin install vagrant-libvirt
vagrant up --provider=libvirt
```

После создания виртуальных машин можно переходить к выполнению Ansible Playbook, описанных выше.

При использовании другой инфраструктуры Vagrant не требуется — достаточно настроить Inventory под реальные серверы.

---

**VoiceTaskSystem Infrastructure** — автоматизированное развёртывание Kubernetes-кластера с помощью Ansible.