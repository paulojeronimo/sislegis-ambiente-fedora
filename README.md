# Ambiente Fedora para execução do SISLEGIS

## Resumo
Este projeto prepara um ambiente Fedora para a execução do SISLEGIS. Esse ambiente pode ser usado de duas formas:

1. Através do Vagrant (numa máquina com qualquer sistema operacional suportado por ele);
2. Através do Fedora instalado em tua própria máquina.

Neste momento, a versão suportada para o Fedora é a 21.

## Preparação

Para utilizar este ambiente, edite o arquivo hosts da tua máquina e adicione o nome ``sislegis.local`` ao ip 127.0.0.1.

Clonando este projeto:

```bash
git clone http://github.com/pensandoodireito/sislegis-ambiente-fedora
cd sislegis-ambiente-fedora
```

Crie o arquivo config a partir do exemplo existente e, se desejar (ou precisar), configure-o. Nesse arquivo de configuração você pode informar, por exemplo, que utilizará um proxy no ambiente. Também poderá configurar as variáveis que referenciam os projetos do sislegis para os teus próprios forks no GitHub, se estiver trabalhando neles ao invés de diretamente nos projetos da organização ``pensandoodireito``. Execute:
```bash
cp config.exemplo config
vim config
```

Se desejar (para carregar algumas variáveis de ambiente), solicite o carregamento do arquivo ``config`` através da inclusão da seguinte linha em teu arquivo ``~/.bashrc``
```
source <CAMINHO ATÉ ESTE PROJETO>/config
```

Se estiver utilizando um proxy, configure-o também:
```bash
cp .proxy.exemplo .proxy
vim .proxy
```

## Montando o ambiente através do Vagrant (alternativa 1)

Instale o plugin vbguest no Vagrant:
```
vagrant plugin install vagrant-vbguest
```

Inicie e provisione a máquina com os seguintes comandos:
```bash
vagrant up
vagrant ssh -c /vagrant/instalar
```

Ao término do provisionamento, faça um reload da máquina com o comando abaixo. Dessa forma, após a reinicialização, se houver alguma atualização de kernel, o plugin ``vagrant-vbguest`` ficará encarregado de instalar o [VirtualBox Guest Additions](https://www.virtualbox.org/manual/ch04.html) nesse novo kernel.
```bash
vagrant reload
```

Para eliminar o(s) kernel(s) antigo(s) e recuperar o espaço em disco ocupado(s) por ele(s), execute:
```bash
vagrant ssh -c 'sudo yum -y install yum-utils; sudo package-cleanup -y --oldkernels --count=1'
```

Após a montagem do ambiente, você pode acessar as URLs relativas a aplicação.

Você pode ter acesso ao ambiente montado executando o seguinte comando:
```bash
vagrant ssh
```

## Montando o ambiente numa instalação do Fedora (alternativa 2)

Execute o script instalar:
```bash
./instalar
```

Após a montagem do ambiente, você pode acessar as URLs relativas a aplicação.

Você pode ter acesso ao ambiente montado executando o seguinte comando:
```
sudo su - sislegis
```

## URLs relativas a aplicação

* Acesso a aplicação: http://sislegis.local:8080
    * Usuário/senha: sislegis/@dmin123
* Acesso a administração do Wildfly: http://sislegis.local:9990
    * Usuário/senha: sislegis/@dmin123
* Acesso a administração dos usuários da aplicação: http://localhost:8080/auth
    * Usuário/senha: admin/admin

## Salvando o ambiente

O salvamento do ambiente é interessante de ser realizado para que ele possa ser reconstruído de forma mais ágil numa próxima montagem. Para realizar essa operação, execute o seguinte script dentro do ambiente que estiver sendo executado (real ou o virtual executado pelo usuário ``vagrant``):

```bash
./salvar
```

Se o ambiente estiver sendo executado pelo ``vagrant``, também é possível executar o script de salvamente a partir do ``HOST`` da seguinte forma:

```bash
vagrant ssh -c /vagrant/salvar
```

## Migrando a execução do Wildfly da VM para seu HOST

A execução do SISLEGIS dentro de uma VM provisionada pelo Vagrant e executada pelo VirtualBox é suficiente para fins de demonstração de seu funcionamento. Entretanto, para trabalhar no desenvolvimento e na depuração do código do SISLEGIS, essa abordagem talvez não proveja recursos satisfatórios para um desenvolvedor Java. Sendo assim, é possível migrar a execução do SISLEGIS, da VM para o host em que ela executada, movendo o servidor de aplicações (Wildfly) de local. Obviamente é interessante, nessa situação, que o desenvolvedor já possua uma JVM e um IDE instalados no ``HOST`` para, simplesmente, configurá-los para executar o WildFly movido. O script [mover-wildfly-para-host]() pode ser utilizado para realizar essa operação.

Ao final da execução do script ``mover-wildfly-para-host`` o Wildfly estará disponível dentro do diretório ``sislegis-ambiente-comum``, localizado no mesmo nível do diretório deste projeto (``sislegis-ambiente-fedora``). Os códigos fonte dos projetos que possibilitam a execução do SISLEGIS também estarão disponíveis nos diretórios ``app`` e ``app_frontend``, abaixo do diretório ``sislegis-ambiente-comum/projetos``. O repositório Maven utilizado para a compilação de ``app`` estará em ``sislegis-ambiente-comum/.m2``. Dessa forma, basta o desenvolvedor fazer os ajustes necessários em suas ferramentas para poder compilar e depurar o código do SISLEGIS.

O script ``mover-wildfly-para-host`` não move o PostgreSQL utilizado como gerenciardor da base de dados do SISLEGIS. Contudo, pelo fato de ser retirada da VM a execução do Wildfly, o script altera o ``Vagrantfile`` desse projeto (e reinicia a VM) para que ela consuma menos memória (apenas a necessária para executar o PostgreSQL).

## Problemas conhecidos (e soluções)

### Execução no Vagrant no Windows 8.1

#### Instalação do Vagrant

A forma mais simples e rápida de se instalar o Vagrant (e o VirtualBox - que é uma dependência) é através do [Chocolatey](http://chocolatey.org). Então, após instalá-lo abra um prompt de comando como administrador e execute:

```
choco install -y virtualbox vagrant
```

#### Instalação do plugin vagrant-vbguest

Em algumas situações a execução do comando que instala o plugin ``vagrant-vbguest`` pode apresentar o seguinte problema:

```bash
$ vagrant plugin install vagrant-vbguest
Installing the 'vagrant-vbguest' plugin. This can take a few minutes...
C:/HashiCorp/Vagrant/embedded/lib/ruby/2.0.0/rubygems.rb:517:in `inflate': incorrect header check (Zlib::DataError)
        from C:/HashiCorp/Vagrant/embedded/lib/ruby/2.0.0/rubygems.rb:517:in `inflate'
        from C:/HashiCorp/Vagrant/embedded/gems/gems/bundler-1.7.11/lib/bundler/fetcher.rb:147:in `fetch_spec'
```

A solução definitiva para esse problema ainda não foi depurada.

A solução de contorno é fazer o download e a instalação desse plugin manualmente, através dos seguintes comandos: 

```bash
wget -c https://rubygems.org/downloads/vagrant-vbguest-0.10.0.gem
vagrant plugin install vagrant-vbguest-0.10.0.gem
```
