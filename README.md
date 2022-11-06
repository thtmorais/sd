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

Após a execução do comando em todas as maquinas desejadas, nosso cluster está praticamente finalizado, necessitando apenas da instalação do Kubernetes nas máquinas e de configurações adicionais de como o mesmo deverá funcionar para as nossas aplicações.

## Instalação do Kubernetes

[Kubernetes](https://kubernetes.io/pt-br/docs/home/) é um plataforma de código aberto, portável e extensiva para o gerenciamento de cargas de trabalho e serviços distribuídos em contêineres, que facilita tanto a configuração declarativa quanto a automação. Ele possui um ecossistema grande, e de rápido crescimento. Serviços, suporte, e ferramentas para Kubernetes estão amplamente disponíveis.

Para instalar o kubectl que é a CLI do Kubernetes, que através dele iremos interagir com o cluster, iremos necessitar acessar a máquina em que o rancher está instalada e executar:

```sh
sudo apt-get update && sudo apt-get install -y apt-transport-https gnupg2
curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add -
echo "deb https://apt.kubernetes.io/ kubernetes-xenial main" | sudo tee -a /etc/apt/sources.list.d/kubernetes.list
sudo apt-get update
sudo apt-get install -y kubectl
```

Com o kubectl instalado, iremos capturar as credenciais de acesso no Rancher e configurar o kubectl.

Na interface do Rancher, vamos capturar o KubeConfig no menu superior e executar na mesma máquina:

```sh
mkdir ~/.kube
vi ~/.kube/config
```
E colar o código neste arquivo.

Após isto podemos executar o comando abaixo e verificar os nossos nós em funcionamento:

```sh
kubectl get nodes
```

## Persistência dos dados

Com o Rancher, para a persistência de dados, iremos utilizar o [Longhorn](https://longhorn.io/).

A instalação do Longhorn pode ser feito através da interface gŕafica, que irá fazer todo o deployment de forma automática.

Como o Longhorn replica todas as alterações em todas as máquinas, isto possui um custo de CPU e RAM.
