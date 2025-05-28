# Documentação de Instalação e Configuração do GeoServer em Ubuntu Server

## Sumário
1. [Pré-requisitos](#pré-requisitos)
2. [Instalação](#instalação)
3. [Configuração](#configuração)
4. [Gerenciamento do Serviço](#gerenciamento-do-serviço)
5. [Acesso Remoto](#acesso-remoto)
6. [Integração com QGIS](#integração-com-qgis)
7. [Solução de Problemas](#solução-de-problemas)
8. [Referências](#referências)

## Pré-requisitos

- Ubuntu Server 20.04/22.04 LTS
- Usuário com privilégios sudo
- Conexão à internet para download de pacotes
- Java JDK 11 instalado

## Instalação

### 1. Instalar Java JDK 11

```bash
sudo apt update
sudo apt install openjdk-11-jdk -y
```

Verifique a instalação:
```bash
java -version
```

### 2. Baixar e instalar o GeoServer

```bash
wget https://sourceforge.net/projects/geoserver/files/GeoServer/2.24.0/geoserver-2.24.0-bin.zip
sudo unzip geoserver-2.24.0-bin.zip -d /opt
sudo mv /opt/geoserver-2.24.0 /opt/geoserver
```

### 3. Criar usuário dedicado

```bash
sudo adduser --system --group --disabled-password --home /opt/geoserver geo
sudo chown -R geo:geo /opt/geoserver
```

### 4. Criar diretório de dados

```bash
sudo mkdir /home/geo/geoserver_data
sudo chown -R geo:geo /home/geo/geoserver-standalone
```

## Configuração

### 1. Configurar arquivo start.ini

```bash
sudo nano /opt/geoserver/start.ini
```

Altere para:
```
jetty.host=0.0.0.0
jetty.port=8080
```

### 2. Configurar serviço systemd

Crie o arquivo de serviço:
```bash
sudo nano /etc/systemd/system/geoserver.service
```

Com o conteúdo:
```ini
[Unit]
Description=GeoServer
After=network.target

[Service]
Type=simple
User=geo
Group=geo
WorkingDirectory=/opt/geoserver
Environment="JAVA_HOME=/usr/lib/jvm/java-11-openjdk-amd64"
ExecStart=/usr/bin/java -Xms128m -Xmx2g -DGEOSERVER_DATA_DIR=/home/geo/geoserver-standalone -jar /opt/geoserver/start.jar
Restart=on-failure
RestartSec=5
TimeoutStartSec=30

[Install]
WantedBy=multi-user.target
```

### 3. Habilitar e iniciar o serviço

```bash
sudo systemctl daemon-reload
sudo systemctl enable geoserver
sudo systemctl start geoserver
```

Verifique o status:
```bash
sudo systemctl status geoserver
```

## Gerenciamento do Serviço

- Iniciar: `sudo systemctl start geoserver`
- Parar: `sudo systemctl stop geoserver`
- Reiniciar: `sudo systemctl restart geoserver`
- Ver status: `sudo systemctl status geoserver`
- Ver logs: `journalctl -u geoserver -f`

## Acesso Remoto

### 1. Configurar firewall

```bash
sudo ufw allow 8080/tcp
sudo ufw reload
```

### 2. Acessar a interface web

- URL: `http://<ip-do-servidor>:8080/geoserver`
- Credenciais padrão:
  - Usuário: admin
  - Senha: geoserver

### 3. Configurar acesso por nome (opcional)

Edite o arquivo `/etc/hosts` nas máquinas clientes:
```
192.168.1.77  geo.local
```

## Integração com QGIS

1. No QGIS, vá para:
   - "Camada" > "Adicionar Camada" > "Adicionar Serviço WMS/WMTS"
2. URL do serviço: `http://geo.local:8080/geoserver/wms`
3. Clique em "Conectar" e selecione as camadas desejadas

## Solução de Problemas

### 1. Serviço não inicia

Verifique os logs:
```bash
journalctl -u geoserver -b --no-pager
```

### 2. Problemas de conexão

Teste a conectividade:
```bash
# Do cliente
telnet 192.168.1.77 8080
```

### 3. Erros de permissão

Corrija as permissões:
```bash
sudo chown -R geo:geo /opt/geoserver
sudo chown -R geo:geo /home/geo/geoserver_data
```

## Referências

- [Documentação Oficial GeoServer](https://docs.geoserver.org/)
- [Guia de Instalação GeoServer](https://github.com/geoserver/geoserver)
- [Configuração Systemd](https://www.freedesktop.org/wiki/Software/systemd/)

## Estrutura do Repositório Git

```
/geoserver-ubuntu-install/
│
├── docs/
│   ├── INSTALL.md          # Este documento
│   └── TROUBLESHOOTING.md  # Guia de solução de problemas
│
├── scripts/
│   ├── install.sh          # Script de instalação automatizada
│   └── configure.sh        # Script de configuração
│
└── README.md               # Visão geral do projeto
```

Para criar este repositório:

```bash
mkdir geoserver-ubuntu-install
cd geoserver-ubuntu-install
mkdir docs scripts
touch docs/INSTALL.md docs/TROUBLESHOOTING.md scripts/install.sh scripts/configure.sh README.md
```

Copie este conteúdo para os respectivos arquivos e faça commit no Git.
