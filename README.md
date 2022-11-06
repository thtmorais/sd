# Sistemas distribuídos

Vamos demonstrar como funciona um sistema distribuído utilizando o [Rancher](https://docs.ranchermanager.rancher.io/).

O Rancher é uma ferramenta de gerenciamento do Kubernetes para implantar e executar clusters em qualquer lugar e em qualquer provedor.

O Rancher é uma plataforma completa de gerenciamento de contêineres para Kubernetes, oferecendo as ferramentas para executar o Kubernetes com sucesso em qualquer lugar.

## Instalando o Rancher

Para instalar o Rancher necessita de uma máquina para a sua instalação, iremos utlizar uma máquina de 2 vCPU e 4GB de RAM (e2-medium da GCP), com o sistema operacional Ubuntu 20.04 LTS.

O Rancher necessita também de algumas [portas abertas](https://docs.ranchermanager.rancher.io/getting-started/installation-and-upgrade/installation-requirements/port-requirements), mas para este exemplo abrimos todas as portas para maior facilidade.

Um IP fixo é recomendado (devido estarmos utilizando o GCP e pode correr o risco do IP ser mudado).

Após a criação é necessário a instalação do [Docker](https://docs.docker.com/), e podemos utilizar a instalação recomendada pelo próprio rancher executando:

```sh
curl https://releases.rancher.com/install-docker/20.10.sh | sh
```

Para não necessitar de executar `sudo`em todos os comandos de instalação com Docker:

```sh
sudo groupadd docker
sudo usermod -aG docker $USER
```

Com todos os passos anteriores executados vamos a instalação do Rancher:


```sh
docker run -d --restart=unless-stopped -p 80:80 -p 443:443 -v /opt/rancher:/var/lib/rancher --privileged rancher/rancher:latest
```

Após isto o Rancher estará instalado e assim seguimos para a criação de nosso cluster.

## Máquinas para o cluster

As máquinas do cluster também necessita do Docker instalado e portas abertas, repetindo os passos anteriores, mas as configurações das máquinas podem ser diferentes desde o seu tamanho (quantiade de CPU, RAM e outros), sua região, seu sistema operacional e diferenças similares a estes quesitos.

## Criando o cluster através do Rancher

Na interface do Rancher mesmo, iremos criar um novo cluster ("bare-metal") com as opções etc, control-plane e worker habilitadas.

Após estas informações o Rancher irá nos fornecer um comando similar a este:

```sh
sudo docker run -d --privileged --restart=unless-stopped --net=host -v /etc/kubernetes:/etc/kubernetes -v /var/run:/var/run  rancher/rancher-agent:v2.6.9 --server https://rancher.mathec.com.br --token abcdefghijklmnopqrstuvwxyz0123456789 --ca-checksum abcdefghijklmnopqrstuvwxyz0123456789 --etcd --controlplane --worker --node-name k8s-1
```

Este comando deverá ser executado nas máquinas do nosso cluster, não havendo limite superior de quantidade de máquinas.

### Algumas definições:

- **etcd:** com essa função, o contêiner etcd será executado nesses nós. Etcd mantém o estado do seu cluster e é o componente mais importante do seu cluster, única fonte de verdade do seu cluster. Embora você possa executar o etcd em apenas um nó, normalmente são necessários 3, 5 ou mais nós para criar uma configuração de alta disponibilidade. Etcd é um armazenamento de chave-valor confiável distribuído que armazena todo o estado do Kubernetes. 

- **control-plane:** com essa função, os componentes sem estado usados para implantar o Kubernetes serão executados nesses nós. Esses componentes são usados para executar o servidor de API, agendador e controladores.

- **worker:** com essa função, quaisquer cargas de trabalho ou pods implantados chegarão a esses nós.

Após a execução do comando em todas as maquinas desejadas, nosso cluster está praticamente finalizado, necessitando apenas de configurações adicionais de como o mesmo deverá funcionar para as nossas aplicações.
