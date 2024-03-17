# PaginaWebDeber1
# Containers and Virtual Machines 
<p>
In this assignment we will be comparing the usage and performance of both virtual machines and containers. For this activity, you’ll be working on groups of 2 people (max 3).
</p>

## Requirements: 
- The server: 1 computer running Vagrant and Docker
- The client: 1 computer with at least Vagrant

## Experiment 1:
<p>
For the server, write a Vagrant file that starts a Ubuntu VM, installs and configures a web server such as NGINX, and starts up the web server. The web server should host a very simple web page.

In the client, you’ll write a Vagrant file that starts Ubuntu and installs wrk. Then you’ll login into the VM, and use wrk to run a load test against the web server running in the server machine.

You can find wrk at: https://github.com/wg/wrk

Note that for this experiment to work, you will need to make sure that you configure the network of the VM in the right way. You can find tutorials online on how to do this.

Make sure that your load test runs for at least 3 minutes. Your load test should generate at least 100 requests per second. When your load test is running, you should take screenshots that show the CPU and memory utilization of the server machine.
</p>

## Procedure Experiment 1:
### 1.1 Configuración del Servidor en Vagrant

<p>
Para realizar el experimento primero se necesitó la creación de la máquina virtual donde se correrá el servidor, para ello utilizamos 
</p>

````
mkdir vagrantserver
cd vagrantserver
````

<p>
Con esto, creamos una carpeta "vagrantserver" donde guardaremos los datos de nuestra máquina virtual.

Para inicializar vagrant escribiremos el comando 
</p>

````
vagrant init
````

<p>
Veremos un mensaje tal que:
</p>

````
A `Vagrantfile` has been placed in this directory. You are now
ready to `vagrant up` your first virtual environment! Please read
the comments in the Vagrantfile as well as documentation on
`vagrantup.com` for more information on using Vagrant.
````

<p>
Buscaremos la carpeta "vagrantserver" para modificarla, terminando con este archivo:
</p>

````
Vagrant.configure("2") do |config|
  config.vm.box = "ubuntu/bionic64" 

  config.vm.network "private_network", ip: "192.168.33.10"  

  config.vm.provider "virtualbox" do |vb|
  end

  config.vm.provision "shell", inline: <<-SHELL
    # Actualizar los repositorios e instalar NGINX
    apt-get update
    apt-get install -y nginx

    # Iniciar el servicio de NGINX
    service nginx start

    # Configurar la página web de prueba
    echo "<h1>Welcome to my simple webpage</h1>" > /var/www/html/index.html
  SHELL
end
````

<p>
Con este archivo "vagrantfile" lograremos que nuestra máquina virtual se ejecute con ubuntu, así como también le asignaremos una dirección ip y nos aseguraremos que se corra utilizando virtualbox. 
A su vez, actualizaremos los repositorios e instalaremos Nginx que lo utilizaremos para correr nuestra página web. Finalmente, en la parte final del código configuraremos nuestra página web con el mensaje "Welcome to my simple webpage".
</p>

<p>
Ahora, utilizaremos el código 
</p>

````
vagrant up
````

<p>
Para correr nuestra máquina virtual. 
Y, cuando todos los procesos hayan finalizado, para ingresar a la misma utilizaremos 
</p>

````
vagrant ssh
````

<p>
Dentro de nuestra máquina virtual, utilizaremos el comando
</p>

````
sudo service nginx status
````

<p>
Para verificar el estado de nuestro servidor. Y, para conectarnos a este utilizaremos 
</p>

````
curl http://192.168.33.10
````

<p>
Donde deberiamos visualizar el mensaje "Welcome to my simple webpage" con el cual podremos verificar que nuestra página web se ejecuta con normalidad.
</p>
### 1.2 Configuración Cliente en Vagrant

<p>
Crearemos una nueva maquina virtual para el cliente
</p>

````
mkdir vagrantcliente
cd vagrantcliente
````

<p>
Inicializaremos esta nueva MV
</p>

````
vagrant init
````

<p>
Dentro del archivo "vagrantfiile" colocaremos: 
</p>

````
Vagrant.configure("2") do |config|
  config.vm.box = "ubuntu/bionic64" 

  config.vm.provider "virtualbox" do |vb|
    # Configuraciones específicas de VirtualBox
    vb.memory = "1024"  
    vb.cpus = 2         
  end

  config.vm.network "private_network", ip: "192.168.33.11"
  
  config.vm.provision "shell", inline: <<-SHELL
    apt-get update
    apt-get install -y build-essential libssl-dev git unzip

    git clone https://github.com/wg/wrk.git
    cd wrk
    make
    cp wrk /usr/local/bin
  SHELL
end
````

<p>
Con esto nos aseguraremos que en la MV de cliente se instale wrk que utilizaremos para el testeo de nuestra página web.
</p>

<p>
Una vez colocado esto en nuestro archivo "vagrantfile" correremos nuestra MV
</p>

````
vagrant up
````

<p>
Y para entrar en la MV utilizaremos 
</p>

````
vagrant ssh
````

<p>
Una vez dentro de la MV de cliente nos conectaremos a la página web a través del comando 
</p>

````
wrk -t2 -c100 -d180s http://192.168.33.10/
````

### 1.3 Resultados:

<p>
Posterior a conectarnos a nuestra MV de servidor con nuestra MV de cliente, hemos recibido los siguientes datos:
</p>

#### Antes de prueba de carga
<p>
Para observar los recursos dentro de la maquina virtual utilizamos el comando:
</p>

````
top
````

<p>
Y podemos observar los siguientes resultados:
</p>

![Imagen 1. Uso de CPU y memoria sin Carga](https://github.com/AndresProano/PaginaWebDeber1/blob/main/CapturasVagrant%20/Sin%20carga%20vagrant.png)

<p>
Mientras que dentor de monitor de actividad en la computadora host podemos observar:
</p>

![Imagen 2. Uso de CPU y memoria sin Carga](https://github.com/AndresProano/PaginaWebDeber1/blob/main/CapturasVagrant%20/Sin%20carga.png)

#### Durante la prueba de carga 

<p>
Para observar los recursos dentro de la maquina virtual utilizamos el comando:
</p>

````
top
````

<p>
Y podemos observar los siguientes resultados:
</p>

![Imagen 3. Uso de CPU y memoria con Carga](https://github.com/AndresProano/PaginaWebDeber1/blob/main/CapturasVagrant%20/Con%20carga.png)

<p>
Mientras que dentor de monitor de actividad en la computadora host podemos observar:
</p>

![Imagen 4. Uso de CPU y memoria con Carga](https://github.com/AndresProano/PaginaWebDeber1/blob/main/CapturasVagrant%20/Con%20carga%20vagrant.png)

<p>
Al observar los datos recopilados, podemos ver que existe un aumento significativo del uso del CPU. Sin carga podemos observar que tenemos un uso de sistema de 6.93% mientras que con carga tenemos un uso de sistema de 66.17% mostrando un aumento significativo.
</p>
