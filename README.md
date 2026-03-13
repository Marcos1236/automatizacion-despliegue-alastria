# Automatización del Despliegue de un Nodo Alastria Red B

Trabajo de Fin de Grado — Universidad de Zaragoza (UNIZAR)

Este repositorio contiene la infraestructura como código necesaria para desplegar de forma automatizada y reproducible un nodo regular en la red blockchain Alastria Red B (tecnología Hyperledger Besu), utilizando Ansible como herramienta de automatización y Docker como entorno de ejecución del nodo.

---

## Descripción

Alastria Red B es una red blockchain pública-permisionada basada en [Hyperledger Besu](https://besu.hyperledger.org/) con consenso IBFT 2.0, gestionada por los socios de [Alastria](https://alastria.io/). Para unirse a la red, un nodo debe ser desplegado correctamente y posteriormente permisionado por Alastria.

Este proyecto automatiza completamente la fase de instalación y configuración del nodo, eliminando la intervención manual y garantizando que el despliegue sea idéntico independientemente del servidor destino.

---

## Requisitos previos

### Para pruebas en local
- [Vagrant](https://www.vagrantup.com/) >= 2.3
- [VirtualBox](https://www.virtualbox.org/) >= 6.1
- [Ansible](https://docs.ansible.com/) >= 2.12

### Para despliegue en producción (AWS u otro proveedor)
- Instancia con Ubuntu 22.04 LTS (Jammy) de 64 bits
- Mínimo 2 vCPUs y 4 GB de RAM (recomendado 4 vCPUs y 8 GB)
- Mínimo 512 GB de disco SSD
- Acceso SSH con clave privada
- Ansible >= 2.12 instalado en la máquina local

---

## Estructura del proyecto

```
.
├── Vagrantfile                  # Definición de la VM local para pruebas
├── ansible.cfg                  # Configuración global de Ansible
├── playbook.yml                 # Playbook principal
├── inventory/
│   └── hosts.yml                # Inventario de servidores destino
└── roles/
    ├── common/                  # Dependencias base y firewall UFW
    ├── docker/                  # Instalación de Docker CE
    └── besu_node/               # Despliegue del nodo Alastria Red B
        ├── tasks/main.yml
        ├── templates/.env.j2
        └── vars/main.yml
```

---

## Uso

### 1. Pruebas en local con Vagrant

Levantar la máquina virtual:
```bash
vagrant up
```

Ejecutar el playbook contra la VM:
```bash
ansible-playbook playbook.yml
```

Verificar que el nodo está corriendo en Alastria Red B (chainId 2020):
```bash
vagrant ssh -c "curl -s -X POST --data '{\"jsonrpc\":\"2.0\",\"method\":\"eth_chainId\",\"params\":[],\"id\":1}' http://127.0.0.1:8545"
```

Ver los logs del nodo:
```bash
vagrant ssh -c "sudo docker logs -f REG_UNIZAR_B_2_4_00"
```

Apagar la VM cuando no se use:
```bash
vagrant halt
```

### 2. Despliegue en producción (AWS)

Editar `inventory/hosts.yml` con los datos del servidor real:
```yaml
nodo_objetivo:
  ansible_host: X.X.X.X                        # IP pública de la instancia
  ansible_user: ubuntu                          # Usuario por defecto en Ubuntu AWS
  ansible_ssh_private_key_file: ~/.ssh/clave.pem
```

Editar `roles/besu_node/vars/main.yml` con el nombre del nodo siguiendo la convención de Alastria:
```yaml
node_name: "REG_UNIZAR_B_<CPUs>_<RAMgb>_<NN>"
node_type: "regular"
node_branch: "main"
```

Ejecutar el playbook:
```bash
ansible-playbook playbook.yml
```

---

## Configuración del nodo

Los parámetros del nodo se centralizan en `roles/besu_node/vars/main.yml`:

| Variable | Descripción | Ejemplo |
|---|---|---|
| `node_name` | Nombre del nodo (convención Alastria) | `REG_UNIZAR_B_2_4_00` |
| `node_type` | Tipo de nodo | `regular` |
| `node_branch` | Rama del directorio de Alastria | `main` |

La convención del nombre es: `REG_<EMPRESA>_B_<CPUs>_<RAMgb>_<NN>`

---

## Puertos utilizados

| Puerto | Protocolo | Uso |
|---|---|---|
| 30303 | TCP/UDP | Comunicación P2P entre nodos Ethereum |
| 8545 | TCP | API JSON-RPC HTTP |
| 8546 | TCP | API JSON-RPC WebSocket |
| 9545 | TCP | Métricas Prometheus |

---

## Proceso de permisionamiento

Una vez desplegado el nodo, es necesario solicitar el permisionamiento a Alastria para poder conectarse a la red. Para ello hay que rellenar el [formulario oficial](https://forms.gle/BiRqqgg2V7zbxF3c7) con:

1. **ENODE** del nodo:
```bash
curl -s -X POST --data '{"jsonrpc":"2.0","method":"admin_nodeInfo","params":[],"id":1}' http://127.0.0.1:8545
```

2. **IP pública** del servidor:
```bash
curl https://ifconfig.me/
```

3. **Detalles del sistema**: proveedor de hosting, número de vCPUs, RAM y tamaño de disco.

Una vez aceptada la solicitud, el nodo comenzará a conectarse a sus peers y a sincronizar la blockchain (proceso que puede tardar horas o días).

---

## Verificación del despliegue

| Comprobación | Comando | Resultado esperado |
|---|---|---|
| Nodo corriendo | `docker ps` | Contenedor `Up` |
| Red correcta | `eth_chainId` | `0x7e4` (2020 en decimal) |
| Peers conectados | `admin_peers` | Al menos 1 peer (tras permisionamiento) |
| Sincronización | `eth_syncing` | `false` cuando termina |

---

## Tecnologías

- [Hyperledger Besu 22.10.3](https://besu.hyperledger.org/)
- [Ansible](https://docs.ansible.com/)
- [Docker](https://www.docker.com/)
- [Vagrant](https://www.vagrantup.com/) (entorno de pruebas)
- [Alastria Red B](https://github.com/alastria/alastria-node-besu)

---

## Referencias

- [Repositorio oficial alastria-node-besu](https://github.com/alastria/alastria-node-besu)
- [Documentación Hyperledger Besu](https://besu.hyperledger.org/en/stable/)
- [Formulario de permisionamiento Alastria](https://forms.gle/BiRqqgg2V7zbxF3c7)
- [Explorador de bloques Red B](https://b-network.alastria.izer.tech/)