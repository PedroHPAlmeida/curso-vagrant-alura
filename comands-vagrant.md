## Aula 01 - Instalação e a primeira VM

```vagrant version```: exibe a versão instalada do Vagrant.  

```vagrant init <box>```: cria o Vagrantfile. A box é baixada da internet e possui a imagem do sistema operacional, entre outras configurações.

```vagrant up```: sobe a máquina virtual e aplica as configurações.

```vagrant status```: mostra o status atual da máquina, se está executando ou parada por exemplo.

```vagrant halt```: para a máquina virtual.

```vagrant ssh```: por meio deste é possível conectar-se a VM criada.

VagrantFile é o arquivo que irá conter todas as informações referentes a nossa máquina virtual, tais como: rede, memória, SO, processador, etc. 

```vagrant ssh-config```: exibe as informações que o vagrant está utilizando para poder conectar-se a VM por meio do ssh. 
_____________
### Anotações deixadas pelo professor
* VirtualBox, VMware, Hyper-V, entre outros, são Hypervisors.
* Um Hypervisor emula o hardware do computador para criar e executar máquinas virtuais.
* O Vagrant é uma ferramenta que controla o Hypervisor a partir de um arquivo simples, o Vagrantfile.
* O Vagrantfile define detalhes da máquina virtual, como o sistema operacional, a rede, software utilizado, etc.
* Para se conectar com a máquina virtual, usamos a ferramenta SSH.
_____________
## Aula 02 - Configuração de rede

* Site para pesquisar ambientes prontos: https://app.vagrantup.com/boxes/search
_____________
### Forwarded Port

Para direcionarmos uma requisição em uma porta do Windows para a VM, devemos configurar isso no arquivo Vagrantfile. Adicionamos a linha que está em destaque, onde ```host: numero-porta``` é quem receberá a requisição em nossa máquina Windows, e ```guest: numero-porta``` é para onde será direcionada a requisição na VM. 

```
Vagrant.configure("2") do |config|
    config.vm.box = "ubuntu/bionic64"
    config.vm.box_version = "20190212.1.0"
    config.vm.network "forwarded_port", guest: 80, host: 8089 // configuração de rede
end
```
_____________
### IP na rede privada

Para criar um IP estático para a nossa VM, adicionamos a seguinte linha de código ao nosso Vagrantfile:

```
Vagrant.configure("2") do |config|
    config.vm.network "private_network", ip: "192.168.50.4" 
end
```
Agora podemos acessar serviços dentro da nossa VM diretamente pelo IP configurado. Por exemplo, abrindo o navegador e digitando o IP. 
_____________
### Conexão via DHCP

Para se conectar a máquina virtual via DHCP, adicionamos a seguinte linha de código ao nosso arquivo Vagrantfile:

```
Vagrant.configure("2") do |config|
    config.vm.network "private_network", type: "dhcp"
end
```
Obs.: no meu caso isso não funcionou, pesquisei soluções na internet e a seguinte parece ter dado certo (me faltam conhecimentos sobre redes):

Adicionar essas linhas de código ao Vagrantfile 
```
class VagrantPlugins::ProviderVirtualBox::Action::Network
    def dhcp_server_matches_config?(dhcp_server, config)
      true
    end
end
```

```vagrant reload```: derruba a VM e sobe ela novamente aplicando as alterações adicionadas ao Vagrantfile. Nem sempre esse comando funcionará, sendo por vezes necessário destruir a VM e constrí-la novamente.

```vagrant destroy```: esse comando destrói por completo a VM.
_____________
### IP na rede pública (bridge)
Neste passo veremos sobre as conexões com chave pública ou Public Network, que possibilita o acesso à máquina virtual por diversos computadores em uma única rede pública, enquanto no Private Network só é possível acessá-la através do host, ou seja, a partir de um computador específico.

No arquivo "Vagrantfile" no editor de texto, substituímos a sentença ```config.vm.network "private_network", type: "dhcp"``` por:

```
Vagrant.configure("2") do |config|
    config.vm.network "public_network"
end
```
Dessa forma, todos os computadores conectados na mesma rede que o host poderão acessa a VM por meio de seu endereço de IP.
_____________
### Resumo do professor
* Existem 3 formas para configurar a rede:
    - Forwarded Port
    - Private Network
    - Public Network
* Na configuração Forwarded Port, mapeamos uma porta do host para o guest, por exemplo:
```
config.vm.network "forwarded_port", guest: 80, host: 8080, host: 8080
```
* Na Private Network (static ou dhcp) é usado um endereço privado que não é acessível na sua rede pública (por exemplo, a rede empresarial).
* Na Public Network (static ou dhcp), usamos um endereço que faz parte da sua rede pública (por exemplo, da rede empresarial).
* Com o comando ```vagrant halt``` podemos parar a execução da máquina virtual.
* O comando ```vagrant reload``` recarrega a configuração da máquina virtual.
_____________
## Aula 03 - Lidando com o SSH

Neste passo, veremos como gerar nossa própria chave privada para permitir o acesso de outros desenvolvedores à máquina virtual.

```ssh-keygen -t rsa```: esse comando irá gerar uma chave pública na pasta que você especificar.

Agora precisamos copiar nossa chave pública para a máquina virtual; começamos por digitar ```vagrant ssh``` e acessar a pasta com ```ls /vagrant/``` que é automaticamente montada no Windows.

Copiamos a chave publica da pasta "vagrant" para a máquina virtual escrevendo ```cp /vagrant/id_bionic.pub .```. Precisamos adicioná-la ao arquivo "authorized_keys" digitando ```cat id_bionic.pub >> .ssh/authorized_keys```. Veja a nova chave adicionada com ```cat .ssh/authorized_keys```. Finalize com ```exit``` para retornar ao Windows.

Agora, usaremos a chave privada no Windows para nos conectar usando ```ssh -i id_bionic vagrant@192.168.1.24```, lembrando que o seu IP pode variar. É possível assim se conectar remotamente com sua máquina virtual baseado em uma chave pública/privada desde que as tenha associadas com a chave privada dentro da MV.
_____________
### Resumo do professor
* O comando vagrant ssh-config lista as configurações SSH que o comando vagrant ssh usará.
* O Vagrant gera automaticamente um par de chaves SSH.
* A chave pública fica na máquina virtual (guest), a chave privada fica no host.
* No arquivo .ssh/known_host, fica guardado o fingerprint de cada máquina com qual o SSH se conectou.
* Como criamos máquinas através do Vagrant com frequência, é preciso limpar esse arquivo .ssh/known_host (ou apagar) de tempos e tempos.
* Para gerar um par de chaves SSH, existe a ferramenta ssh-keygen.
* A chave pública deve ficar dentro do arquivo .ssh/authorized_keys da máquina virtual.

## Aula 04 - Provisionando a máquina

### Shell Provisioner
No Linux, o Shell - ou também o Batch - é o local onde se executa os comandos, e através do Vagrant podemos definir quais serão estes.
```
Vagrant.configure("2") do |config|
    config.vm.provision "shell",
        inline: "echo hello, World"
end
```
Após salvar, vá ao terminal com a máquina virtual rodando verificada com ```vagrant status``` para recarregar com ```vagrant reload```. Apesar de não haver alterações, aparece uma linha escrita "Machine already provisioned", indicando que esta já foi provisionada e que devemos inserir o comando ```vagrant provision```.


```vagrant provision```: chama a execução de todos os provisionadores configurados no Vagrantfile.

Desta forma, podemos executar comandos Batch-Shell através dessas configurações, nos possibilitando mais recursos sofisticados como a instalação do Nginx ou MySQL por exemplo.

#### O que significa provisionar:
___Provisionar significa fornecer a rede, CPU, memória, espaço, mas também o sistema operacional e pacotes, além da implantação em si. Tudo o que for preciso para rodar/executar o serviço. Melhor ainda, fica automatizado e pode ser repetido a qualquer momento.___
_____________
### Synced Folder e mais shell
Para sincornizar suas pastas locais (host) com pastas na máquina virtual, adicionamos a sentença abaixo ao nosso Vagrantfile:
```
Vagrant.configure("2") do |config|
    config.vm.synced_folder "./pasta_no_host", "/nome_pasta_vm"
end
```
Também é interessante não deixar a pasta "vagrant" acessível com o arquivo "Vagrantfile" à todos pela máquina virtual. Para desabilitar esta pasta padrão, vemos na parte "Disabling" da documentação o código que devemos inserir:
```
Vagrant.configure("2") do |config|
    config.vm.synced_folder ".", "/vagrant", disabled: true
end
```
Agora, automatizamos a adição da chave pública que geramos anteriormente substituindo o trecho "echo Hello, World >> hello.txt por cat /configs/id_bionic.pub >> .ssh/authorized_keys para imprimir o conteúdo e jogar a saída para a pasta "Authorized_keys" ao mesmo tempo que garante a presença da chave padrão, obtendo no arquivo "Vagrantfile":

```
Vagrant.configure("2") do |config|
    config.vm.provision "shell",
        inline: "cat /configs/id_bionic.pub >> .ssh authorized_keys"
end
```
Com isso, automatizamos o processo de criação da pasta "configs" onde será guardada nossa chave pública de ssh, e copiamos o conteúdo da chave para o arquivo "authorized_keys".
_____________
### Provisionando o MySQL
Muito conteúdo! Assista a aula.
_____________
Para adicionar uma variável contendo um script shell no arquivo Vagrantfile, fazemos o seguinte:
```
$nome_script = <<-SCRIPT
    comandos do script
SCRIPT
```
Depois chamamos a execução desse script:
```
config.vm.provision "shell", inline: $nome_script
```
_____________
### Resumo do professor
* O provisionador mais simples é o Shell Provisioner.
* Provisionamento significa instalar e configurar tudo o que for necessário para rodar algum serviço ou aplicação.
* Para usar o Shell Provisionar, basta definir um script com os passos de instalação:
```
Vagrant.configure("2") do |config|
    config.vm.provision "shell", path: "script.sh"
end
```
* Os comandos do Shell Provisioner também podem ser usados de maneira inline ou remoto:
```
$script = <<-SCRIPT
  echo Instalando MySQL
SCRIPT

Vagrant.configure("2") do |config|
  config.vm.provision "shell", inline: $script
end
```
Ou:
```
Vagrant.configure("2") do |config|
  config.vm.provision "shell", path: "https://seu-servidor/script.sh"
end
```
* O Vagrant automaticamente compartilha uma pasta entre o host e o guest (Synced Folder).
* Por padrão, é compartilhada a pasta onde se encontra o Vagrantfile.
* Na máquina guest, podemos acessar a pasta pelo caminho /vagrant.
* A pasta compartilhada pode ser reconfigurada no Vagrantfile:
```config.vm.synced_folder "src/", "/public"```
_____________
## Aula 05 - Conhecendo o Puppet

Para criarmos várias máquinas no mesmo arquivos basta adicionarmos as seguintes linhas de código:
```
config.vm.define "nome-vm" do |nome-obj-vm|
    nome-obj-vm.vm.network "forwarded_port", guest: 80, host: 8082
    nome-obj-vm.vm.network "public_network", ip: '192.168.0.115'
    // demais configs
end
```

__Consultar docs__: https://www.vagrantup.com/docs/multi-machine
_____________
### Código puppet que faz a instalação do php na máquina virtual
```
# execute 'apt-get update'
exec { 'apt-update':
    command => '/usr/bin/apt-get update' 
}

package { ['php7.2' ,'php7.2-mysql'] :
    require => Exec['apt-update'],
    ensure => installed,
}

exec { 'run-php7':
    require => Package['php7.2'],
    command => '/usr/bin/php -S 192.168.1.25:8888 -t /vagrant/src &'
}
```
Assista essa aula para entender: https://cursos.alura.com.br/course/vagrant-gerenciando-maquinas-virtuais/task/52090

Para usar o puppet como provisionador no nosso Vagranfile, adicionamos o seguinte:
```
config.vm.define "phpweb" do |phpweb|
      phpweb.vm.network "forwarded_port", guest: 8888, host: 8888
      phpweb.vm.network "public_network", ip: '192.168.0.115'
      phpweb.vm.provision "shell", inline: "apt-get update && apt-get install -y puppet"

      phpweb.vm.provision "puppet" do |puppet|
        puppet.manifests_path = "./configs/manifests"
        puppet.manifest_file = "phpweb.pp"
      end
end
```
Neste exemplo o puppet está provisionando a instalação do php na VM "phpweb". 

```vagrant validate```: valida o arquivo Vagrantfile.
_____________
### Resumo do professor:
* No mesmo Vagrantfile, podemos configurar várias máquinas, separando as configurações (Multi-Machine).
* O Puppet é uma ferramenta popular para provisionar uma máquina
Provisionamento significa instalar e configurar tudo o que for necessário para rodar algum serviço ou aplicação.
* Com Puppet, podemos definir os passos de instalação de mais alto nível, facilitando a manutenção.
* Os passos de instalação são configurados em um arquivo manifest, com a extensão ```.pp```.
* Para rodar o Puppet, é preciso instalar um cliente na máquina virtual.
* O Vagrant integra e consegue chamar o Puppet a partir do comando vagrant provision.
* Ao rodar o comando ```vagrant up``` pela primeira vez, ele também roda o provisionamento.
* Para configurar o Puppet dentro do Vagrantfile, basta usar:
```
config.vm.provision "puppet" do |puppet|
  puppet.manifests_path = "manifests"
  puppet.manifest_file = "wep.pp"
end
```
_____________
## Aula 06 - Usando o Ansible

O Ansible não funciona no Windows, então se você estiver usando uma máquina com Windows como host, deverá subir um VM apenas para executar o Ansible e provisionar as outras VM's. Essa VM com Ansible irá ler o arquivo "playbook.yml" direto da máquina host para poder provisionar todas as demais.

Presisamos usar o shell para instalar o Ansible na VM, o único pré-requisito é ter o Python instalado na guest que será provisionada, porém como a VM é Ubuntu geralmente já há uma versão do Python.

O seguinte código cria uma máquina e instala o Ansible:

```
    config.vm.define "ansible" do |ansible|
        ansible.vm.network "public_network", ip: "192.168.1.26"
        ansible.vm.provision "shell",
            inline: "apt-get update && \
                     apt-get install -y software-properties-common && \
                     apt-add-repository --yes --update ppa:ansible/ansible && \
                     apt-get install -y ansible"
        end
```

Para verificar a instalação do Ansible na VM, acessamos pelo ssh e utilizamos o comando ```ansible-playbook --version```.

Precisamos copiar a nossa chave pública para a VM que o Ansible irá provisionar, nesse exemplo copiamos a chave para uma VM que executará o MySQL Server:

```
mysqlserver.vm.provision "shell", inline: "cat /vagrant/configs/id_bionic.pub >> .ssh/authorized_keys"
```
Após isso, na VM que executa o Ansible precisamos copiar a chave privada da pasta ```/vagrant/id_bionic``` para a pasta ```/home/vagrant```, também precisamos mudar as permissões do arquivo e mudar o dono (owner) do arquivo para "vagrant", o código fica da seguinte maneira:

```
ansible.vm.provision "shell", 
        inline: "cp /vagrant/id_bionic /home/vagrant && \
                 chmod 600 /home/vagrant/id_bionic
                 chown vagrant:vagrant /home/vagrant/id_bionic"
```
_____________
### Fazendo o Ansible funcionar
Devemos definir dois arquivos necessários para o Ansible funcionar corretamente:
* Arquivo "hosts":

```
[mysqlserver]
192.168.1.22

[mysqlserver:vars]
ansible_user=vagrant
ansible_ssh_private_key_file=/home/vagrant/id_bionic
ansible_python_interpreter=/usr/bin/python3
ansible_ssh_common_args='-o StrictHostKeyChecking=no'
```

* Arquivo "playbook.yml":
```
- hosts: all
  handlers:
    - name: restart mysql
      service:
        name: mysql
        state: restarted
      become: yes

  tasks:
    - name: 'Instalar MySQL Server'
      apt:
        update_cache: yes
        cache_valid_time: 3600 #1 hora
        name: ["mysql-server-5.7", "python3-mysqldb"]
        state: latest
      become: yes

    - name: 'Criar usuario no MySQL'
      mysql_user:
        login_user: root
        name: phpuser
        password: pass
        priv: '*.*:ALL'
        host: '%'
        state: present
      become: yes

    - name: 'Copiar arquivo mysqld.cnf'
      copy:
        src: /vagrant/configs/mysqld.cnf
        dest: /etc/mysql/mysql.conf.d/mysqld.cnf
        owner: root
        group: root
        mode: 0644
      become: yes
      notify:
        - restart mysql
```
__Obs.: Assitir o curso de Ansible para entender melhor.__
_____________
Agora devemos automatizar a execução do Ansible a partir do nosso Vagrantfile. Como no nosso caso a máquina host é Windows, a documentação do Vagrant não irá nos ajudar, então deveremos rodar um shell a Vagrantfile para executar o comando que executará o Ansible na VM "ansible":

```
ansible.vm.provision "shell", 
      inline: "ansible-playbook -i /vagrant/configs/ansible/hosts \
               /vagrant/configs/ansible/playbook.yml"
```
Esse comando aponta o local está o nosso arquivo "hosts" e onde está o "playbook.yml" para que o Ansible possa executá-los.

Pronto!!!
_____________
### Resumo do professor:
* O Ansible, assim como o Puppet, é uma ferramenta para provisionar uma máquina.
* O Ansible envia comandos SSH para a máquina a ser configurada, e não precisa de um cliente instalado (apenas o Python).
* Os passos de instalação são configurados de alto nível, dentro de um playbook.
* O arquivo de inventário (hosts) define os alvos da instalação.
* O Vagrant integra o Ansible e tem uma configuração dedicada:
```
config.vm.provision "ansible" do |ansible|
 ansible.inventory_path = "hosts"
 ansible.playbook = "playbook.yml"
end
```
_____________
## Aula 07 - Configurações do provider

Provider é a VirtualBox (ou outro provedor que você escolher usar), vamos definir algumas configurações dele. Ose guinte código define que todas as máquinas serão criadas com 512 mb de RAM e 1 núcleo de CPU:

```
  config.vm.provider "virtualbox" do |vb|
  vb.memory = 512
  vb.cpus = 1
```
Para definir configuração específicas para cada máquina, basta colocarmos o código dentro do escopo dela. O seguinte código define o nome da máquina que armazenará o PHP: 

```
config.vm.define "phpweb" do |phpweb|
  phpweb.vm.provider "virtualbox" do |vb|
    vb.name = "ubuntu_bionic_php7"
  end
end
```
_____________
### Criar um box específico para uma VM

O código a seguir configura um box com centos7 para a VM especificada:
```
config.vm.define "memcached" do |memcached|
        memcached.vm.box = "centos/7"
end
```

```vagrant global-status```: recebemos a saída com todas os ambientes existentes atualmente em nosso computador.

Podemos executar ```halt``` a partir de qualquer pasta adicionando a ID apresentada na tabela, como ```vagrant halt bc0947c``` (para visualizar o ID use o comando acima).

```vagrant box list```: exibe todos os boxes instalados no sistema (host).

```vagrant box remove pasta-box/nome-box```: exclui o box especificado.

```vagrant box prune```: remove boxes duplicados.

__Caminho onde estão os boxes usados durante o curso: ```C:\Users\Pichau\.vagrant.d\boxes```.__
_____________
### Resumo do professor
* No Vagrantfile, podemos definir configurações específicas do provedor (hypervisor).
* As configurações são referente à memória, CPU, rede ou interface gráfica, entre outras opções.
* Para listar todos as boxes baixadas, use o comando: ```vagrant box list```.
* Para remover as boxes desatualizadas: ```vagrant box prune``` ou ```vagrant box remove <nome>```.
* Para listar todas as máquinas que foram criadas no host, use: ```vagrant global-status --prune```.
* Através do ID da máquina, podemos controlar a máquina virtual fora da pasta do projeto, por exemplo: ```vagrant destroy -f <ID-da-VM>```.
_____________
## Aula 08 - Virtualização vs Conteiners
Como o Docker só funciona em Linux, podemos subir uma VM através do Vagrant e lá instalar o Docker. O código a seguir faz isso:

```
config.vm.define "dockerhost" do |dockerhost|
  dockerhost.vm.provider "virtualbox" do |vb|
    vb.name = "ubuntu_dockerhost"
  end

  dockerhost.vm.provision "shell",
    inline: "apt-get update && apt-get install -y docker.io"
end
```
Agora temos um ambiente com o Docker instalado para podermos realizar testes.
_____________
### Resumo do professor
* O Docker é uma tecnologia para criar, rodar e administrar containers, baseado no Linux.
* Containers virtualizam o sistema operacional.
* Máquinas virtuais virtualizam o hardware.
* Containers são mais leves do que máquinas virtuais.
* Ambos, containers e máquinas virtuais, servem para rodar e isolar processos e aplicações.
_____________