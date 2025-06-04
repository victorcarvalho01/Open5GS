# Open5GS
Este guia auxilia na instalação do Open5GS e UERANSIM em uma VM Ubuntu para simulação de redes 5G SA. Inclui a configuração do MongoDB, Node.js, Net-tools, plugin RLS para Wireshark e dependências necessárias para a compilação e execução dos componentes.

Excelente! Esse passo a passo está ótimo para um repositório GitHub — especialmente se for colocado em um arquivo `README.md` com formatação em **Markdown**. Abaixo, já converti seu conteúdo com a formatação adequada para o GitHub, com títulos, comandos destacados e blocos de código bem organizados.

Você pode copiar e colar esse conteúdo diretamente no GitHub ao criar um novo arquivo chamado `README.md`:

---

````markdown
# Instalação do Open5GS e UERANSIM em uma VM

## Atualização inicial e instalação de utilitários

sudo apt-get update
sudo apt-get install -y gnupg wget curl
````

---

## Instalação do MongoDB

```bash
wget -qO - https://www.mongodb.org/static/pgp/server-6.0.asc | sudo apt-key add -

echo "deb [ arch=amd64,arm64 ] https://repo.mongodb.org/apt/ubuntu focal/mongodb-org/6.0 multiverse" | sudo tee /etc/apt/sources.list.d/mongodb-org-6.0.list
```

### Você pode forçar a instalação do `libssl1.1` adicionando a fonte (repositório) do Ubuntu 20.04:

```bash
echo "deb http://security.ubuntu.com/ubuntu focal-security main" | sudo tee /etc/apt/sources.list.d/focal-security.list
sudo apt-get update && sudo apt-get install libssl1.1
```

### Em seguida, exclua o arquivo de lista `focal-security` que você acabou de criar:

```bash
sudo rm /etc/apt/sources.list.d/focal-security.list
```

### Instalar MongoDB:

```bash
sudo apt install -y mongodb-org mongodb-org-database
sudo systemctl start mongod
sudo systemctl enable mongod
sudo systemctl status mongod
```

---

## Instalação do Node.js

```bash
curl -fsSL https://deb.nodesource.com/setup_18.x | sudo -E bash -
sudo apt install nodejs
```

---

## Instalação do Open5GS

```bash
sudo apt-get install -y software-properties-common
sudo add-apt-repository ppa:open5gs/latest
sudo apt-get -y update && sudo apt install -y open5gs
```

### Verificar a instalação:

```bash
sudo service open5gs-* status
```

---

## Instalação do Net-tools

```bash
sudo apt-get install net-tools
ifconfig
```

---

## Instalação do WebUI (Interface de usuário Open5GS)

```bash
curl -fsSL https://open5gs.org/open5gs/assets/webui/install | sudo -E bash -
```

---

## Instalação do UERANSIM

```bash
sudo apt install make gcc g++ libsctp-dev lksctp-tools iproute2 git
sudo snap install cmake --classic

mkdir ueransim && cd ueransim
git clone https://github.com/aligungr/UERANSIM
cd UERANSIM/
make
cd config/
```

---

## Ver endereços IP usados pelo Open5GS

```bash
cd /etc/open5gs
grep -i 'addr' *.yaml
```

---

## Duplicar arquivos de configuração do UERANSIM para edição

```bash
cd ~/ueransim/UERANSIM/config
cp open5gs-gnb.yaml gnb1.yaml
cp open5gs-ue.yaml ue1.yaml

sudo gedit gnb1.yaml &
sudo gedit ue1.yaml &
```

### Editar `gnb1.yaml`

* `linkIp`: 127.0.0.101
* `ngapIp`: 127.0.0.100
* `gtpIp`: 127.0.0.200

### Editar `ue1.yaml`

* `imsi`: 999700000000001
* `gnbSearchList`: 127.0.0.101

---

## Editar arquivos do AMF no Open5GS

```bash
cd /etc/open5gs
sudo gedit amf.yaml
```

---

## Instalar plugin Wireshark para captura de pacotes

```bash
cd /home/simu5g/ueransim
git clone https://github.com/louisroyer/RLS-wireshark-dissector.git

cd /usr/lib/x86_64-linux-gnu/wireshark/plugins/
sudo cp -R /home/simu5g/ueransim/RLS-wireshark-dissector/ RLS-wireshark-dissector/
```

---

## Teste final

```bash
cd ~/ueransim/UERANSIM/config
../build/nr-gnb -c gnb1.yaml
```

> Em outro terminal:

```bash
../build/nr-ue -c ue1.yaml
```
