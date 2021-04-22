### Escuela Colombiana de Ingenier�a
### Arquitecturas de Software - ARSW

## Escalamiento en Azure con Maquinas Virtuales, Sacale Sets y Service Plans

### Integrantes:
-[Jairo Pulido](https://github.com/Killersys)

### Dependencias
* Cree una cuenta gratuita dentro de Azure. Para hacerlo puede guiarse de esta [documentación](https://azure.microsoft.com/en-us/free/search/?&ef_id=Cj0KCQiA2ITuBRDkARIsAMK9Q7MuvuTqIfK15LWfaM7bLL_QsBbC5XhJJezUbcfx-qAnfPjH568chTMaAkAsEALw_wcB:G:s&OCID=AID2000068_SEM_alOkB9ZE&MarinID=alOkB9ZE_368060503322_%2Bazure_b_c__79187603991_kwd-23159435208&lnkd=Google_Azure_Brand&dclid=CjgKEAiA2ITuBRDchty8lqPlzS4SJAC3x4k1mAxU7XNhWdOSESfffUnMNjLWcAIuikQnj3C4U8xRG_D_BwE). Al hacerlo usted contará con $200 USD para gastar durante 1 mes.

![](images/dependencia.jpg)

### Parte 0 - Entendiendo el escenario de calidad

Adjunto a este laboratorio usted podrá encontrar una aplicaci�n totalmente desarrollada que tiene como objetivo calcular el enésimo valor de la secuencia de Fibonnaci.

**Escalabilidad**
Cuando un conjunto de usuarios consulta un en�simo n�mero (superior a 1000000) de la secuencia de Fibonacci de forma concurrente y el sistema se encuentra bajo condiciones normales de operación, todas las peticiones deben ser respondidas y el consumo de CPU del sistema no puede superar el 70%.

### Parte 1 - Escalabilidad vertical

1. Dir�jase a el [Portal de Azure](https://portal.azure.com/) y a continuación cree una maquina virtual con las características básicas descritas en la imágen 1 y que corresponden a las siguientes:
    * Resource Group = SCALABILITY_LAB
    * Virtual machine name = VERTICAL-SCALABILITY
    * Image = Ubuntu Server 
    * Size = Standard B1ls
    * Username = scalability_lab
    * SSH publi key = Su llave ssh publica

![Im�gen 1](images/part1/part1-vm-basic-config.png)

![](images/escalabilidad.jpg)

2. Para conectarse a la VM use el siguiente comando, donde las `x` las debe remplazar por la IP de su propia VM.

    `ssh scalability_lab@xxx.xxx.xxx.xxx`

![](images/ssh.jpg)

3. Instale node, para ello siga la secci�n *Installing Node.js and npm using NVM* que encontrará en este [enlace](https://linuxize.com/post/how-to-install-node-js-on-ubuntu-18.04/).
4. Para instalar la aplicaci�n adjunta al Laboratorio, suba la carpeta `FibonacciApp` a un repositorio al cual tenga acceso y ejecute estos comandos dentro de la VM:

![](images/node.jpg)

    `git clone <your_repo>`

    `cd <your_repo>/FibonacciApp`

    `npm install`
    
![](images/vm.jpg)

5. Para ejecutar la aplicaci�n puede usar el comando `npm FibinacciApp.js`, sin embargo una vez pierda la conexión ssh la aplicación dejará de funcionar. Para evitar ese compartamiento usaremos *forever*. Ejecute los siguientes comando dentro de la VM.

    `npm install forever -g`

    `forever start FibinacciApp.js`
    
![](images/npm.jpg)

![](images/npm2.jpg)

6. Antes de verificar si el endpoint funciona, en Azure vaya a la sección de *Networking* y cree una *Inbound port rule* tal como se muestra en la imágen. Para verificar que la aplicación funciona, use un browser y user el endpoint `http://xxx.xxx.xxx.xxx:3000/fibonacci/6`. La respuesta debe ser `The answer is 8`.

![](images/part1/part1-vm-3000InboudRule.png)


![](images/network.jpg)

![](images/networkprueba.jpg)

7. La funci�n que calcula en en�simo n�mero de la secuencia de Fibonacci est� muy mal construido y consume bastante CPU para obtener la respuesta. Usando la consola del Browser documente los tiempos de respuesta para dicho endpoint usando los siguintes valores:
    * 1000000

	![](images/1000000.jpg)

    * 1010000

	![](images/1010000.jpg)

    * 1020000

	![](images/1020000.jpg)

    * 1030000

	![](images/1030000.jpg)

    * 1040000

	![](images/1040000.jpg)

    * 1050000

	![](images/1050000.jpg)

    * 1060000

	![](images/1060000.jpg)

    * 1070000

	![](images/1070000.jpg)

    * 1080000

	![](images/1080000.jpg)

    * 1090000    

	![](images/1090000.jpg)

8. D�rijase ahora a Azure y verifique el consumo de CPU para la VM. (Los resultados pueden tardar 5 minutos en aparecer).

![Im�gen 2](images/part1/part1-vm-cpu.png)

![](images/consumocpu.jpg)

9. Ahora usaremos Postman para simular una carga concurrente a nuestro sistema. Siga estos pasos.
    * Instale newman con el comando `npm install newman -g`. Para conocer m�s de Newman consulte el siguiente [enlace](https://learning.getpostman.com/docs/postman/collection-runs/command-line-integration-with-newman/).
    * Dir�jase hasta la ruta `FibonacciApp/postman` en una maquina diferente a la VM.
    * Para el archivo `[ARSW_LOAD-BALANCING_AZURE].postman_environment.json` cambie el valor del par�metro `VM1` para que coincida con la IP de su VM.
    * Ejecute el siguiente comando.

    ```
    newman run ARSW_LOAD-BALANCING_AZURE.postman_collection.json -e [ARSW_LOAD-BALANCING_AZURE].postman_environment.json -n 10 &
    newman run ARSW_LOAD-BALANCING_AZURE.postman_collection.json -e [ARSW_LOAD-BALANCING_AZURE].postman_environment.json -n 10
    ```

![](images/carga.jpg)

10. La cantidad de CPU consumida es bastante grande y un conjunto considerable de peticiones concurrentes pueden hacer fallar nuestro servicio. Para solucionarlo usaremos una estrategia de Escalamiento Vertical. En Azure dir�jase a la secci�n *size* y a continuaci�n seleccione el tama�o `B2ms`.

![Im�gen 3](images/part1/part1-vm-resize.png)

Cantidad de CPU consumida:

![](images/porcentajecpu.jpg)


11. Una vez el cambio se vea reflejado, repita el paso 7, 8 y 9.

Repetici�n paso 7:

* 1000000

![](images/1000000b.jpg)

* 1010000

![](images/1010000b.jpg)

* 1020000

![](images/1020000b.jpg)
	
* 1030000

![](images/1030000b.jpg)

* 1040000

![](images/1040000b.jpg)

* 1050000

![](images/1050000b.jpg)

* 1060000

![](images/1060000b.jpg)

* 1070000

![](images/1070000b.jpg)

* 1080000

![](images/1080000b.jpg)

* 1090000    

![](images/1090000b.jpg)

Repetici�n paso 8:

![](images/consumocpu2.jpg)

Repetici�n paso 9:

![](images/carga2.jpg)

12. Evalue el escenario de calidad asociado al requerimiento no funcional de escalabilidad y concluya si usando este modelo de escalabilidad logramos cumplirlo.
13. Vuelva a dejar la VM en el tama�o inicial para evitar cobros adicionales.

**Preguntas**

1. �Cu�ntos y cu�les recursos crea Azure junto con la VM?

![](images/recursos.jpg)

2. �Brevemente describa para qu� sirve cada recurso?

- Red Virtual: Simulaci�n de una red f�sica con recursos de software y hardware
- Cuenta de almacenamiento: contiene todos los objetos de datos de Azure
- M�quina Virtual: M�quina virtual usada como tal
- Direcci�n IP p�blica: Permite saber cu�l es la direcci�n IP que se usa para la conexi�n saliente
- Grupo de seguridad de red: Contiene reglas de seguridad que permiten o deniegan el tr�fico de red entrante o saliente de recursos de Azure
- Interfaz de red: Permite a la maquina virtual comunicarse con recursos en internet.
- Disco:  Es la version virtualizada de la maquina que creamos, guarda el sistema operativo y otros componentes.


3. �Al cerrar la conexi�n ssh con la VM, por qu� se cae la aplicaci�n que ejecutamos con el comando `npm FibonacciApp.js`? �Por qu� debemos crear un *Inbound port rule* antes de acceder al servicio?

Porque en ese momento se cierra la conexi�n necesaria para ejecutar la aplicaci�n. Para tener una conexi�n por un puerto con la m�quina y el servici� que est� proveyendo en todo momento.

4. Adjunte tabla de tiempos e interprete por qu� la funci�n tarda tando tiempo.

![](images/tablatiempo.jpg)

Porque resuelve la f�rmula paso a paso, hace la sumatoria de todos y cada uno de los n�meros anteriores al solicitado

5. Adjunte im�gen del consumo de CPU de la VM e interprete por qu� la funci�n consume esa cantidad de CPU.

B1ls:

![](images/consumocpu.jpg)

B2ms:

![](images/consumocpu2.jpg)

Porque para el primer caso, toda la CPU est� destinada a resolver esta tarea, para el segundo tan solo la mitad.

6. Adjunte la imagen del resumen de la ejecuci�n de Postman. Interprete:

B1ls:

![](images/carga.jpg)

B2ms:

![](images/cargarr.jpg)

* Tiempos de ejecuci�n de cada petici�n.

Los tiempos de ejecuci�n son muy similares

* Si hubo fallos documentelos y explique.

Error ECONNRESET: Significa que el servidor cerr� la conexi�n de una manera que probablemente no era normal.

7. �Cu�l es la diferencia entre los tama�os `B2ms` y `B1ls` (no solo busque especificaciones de infraestructura)?

B1ls tiene: 1 vCPU, 0.5 GB de RAM, 2 discos de datos,  160 E/S,  4 GB de almacenamiento temporal

B2ms tiene: 2 vCPU,   8 GB de RAM, 4 discos de datos, 1920 E/S, 16 GB de almacenamiento temporal

8. �Aumentar el tama�o de la VM es una buena soluci�n en este escenario?, �Qu� pasa con la FibonacciApp cuando cambiamos el tama�o de la VM?

No, es cierto que la CPU se satura menos aumentando el tama�o, pero los tiempos de respuesta son iguales o peores.

9. �Qu� pasa con la infraestructura cuando cambia el tama�o de la VM? �Qu� efectos negativos implica?

Se presentan errores de tipo ECONNRESET por concurrencia de solicutudes a una CPU saturada, esta no logra responder exitosamente.

10. �Hubo mejora en el consumo de CPU o en los tiempos de respuesta? Si/No �Por qu�?

S�, se mejor� el consumo de CPU debido a que su capaciad de procesamiento es mayor, pero los tiempos no mejoraron debido a que el programa se sigue ejecutando secuencialmente, no es problema de capacidad sino del dise�o del programa.

11. Aumente la cantidad de ejecuciones paralelas del comando de postman a `4`. �El comportamiento del sistema es porcentualmente mejor?

No, el comportamiento sigue siendo similar.

### Parte 2 - Escalabilidad horizontal

#### Crear el Balanceador de Carga

Antes de continuar puede eliminar el grupo de recursos anterior para evitar gastos adicionales y realizar la actividad en un grupo de recursos totalmente limpio.

1. El Balanceador de Carga es un recurso fundamental para habilitar la escalabilidad horizontal de nuestro sistema, por eso en este paso cree un balanceador de carga dentro de Azure tal cual como se muestra en la im�gen adjunta.

![](images/part2/part2-lb-create.png)

2. A continuaci�n cree un *Backend Pool*, guiese con la siguiente im�gen.

![](images/part2/part2-lb-bp-create.png)

3. A continuaci�n cree un *Health Probe*, guiese con la siguiente im�gen.

![](images/part2/part2-lb-hp-create.png)

4. A continuaci�n cree un *Load Balancing Rule*, guiese con la siguiente im�gen.

![](images/part2/part2-lb-lbr-create.png)

5. Cree una *Virtual Network* dentro del grupo de recursos, guiese con la siguiente im�gen.

![](images/part2/part2-vn-create.png)

#### Crear las maquinas virtuales (Nodos)

Ahora vamos a crear 3 VMs (VM1, VM2 y VM3) con direcciones IP p�blicas standar en 3 diferentes zonas de disponibilidad. Despu�s las agregaremos al balanceador de carga.

1. En la configuraci�n b�sica de la VM gu�ese por la siguiente im�gen. Es importante que se fije en la "Avaiability Zone", donde la VM1 ser� 1, la VM2 ser� 2 y la VM3 ser� 3.

![](images/part2/part2-vm-create1.png)

2. En la configuraci�n de networking, verifique que se ha seleccionado la *Virtual Network*  y la *Subnet* creadas anteriormente. Adicionalmente asigne una IP p�blica y no olvide habilitar la redundancia de zona.

![](images/part2/part2-vm-create2.png)

3. Para el Network Security Group seleccione "avanzado" y realice la siguiente configuraci�n. No olvide crear un *Inbound Rule*, en el cual habilite el tr�fico por el puerto 3000. Cuando cree la VM2 y la VM3, no necesita volver a crear el *Network Security Group*, sino que puede seleccionar el anteriormente creado.

![](images/part2/part2-vm-create3.png)

4. Ahora asignaremos esta VM a nuestro balanceador de carga, para ello siga la configuraci�n de la siguiente im�gen.

![](images/part2/part2-vm-create4.png)

![](images/balanceador.jpg)

5. Finalmente debemos instalar la aplicación de Fibonacci en la VM. para ello puede ejecutar el conjunto de los siguientes comandos, cambiando el nombre de la VM por el correcto

```
git clone https://github.com/daprieto1/ARSW_LOAD-BALANCING_AZURE.git

curl -o- https://raw.githubusercontent.com/creationix/nvm/v0.34.0/install.sh | bash
source /home/vm1/.bashrc
nvm install node

cd ARSW_LOAD-BALANCING_AZURE/FibonacciApp
npm install

npm install forever -g
forever start FibonacciApp.js
```

Realice este proceso para las 3 VMs, por ahora lo haremos a mano una por una, sin embargo es importante que usted sepa que existen herramientas para aumatizar este proceso, entre ellas encontramos Azure Resource Manager, OsDisk Images, Terraform con Vagrant y Paker, Puppet, Ansible entre otras.

![](images/vms.jpg)

#### Probar el resultado final de nuestra infraestructura

1. Porsupuesto el endpoint de acceso a nuestro sistema será la IP pública del balanceador de carga, primero verifiquemos que los servicios básicos están funcionando, consuma los siguientes recursos:

```
http://52.155.223.248/
http://52.155.223.248/fibonacci/1
```

Con la ip del balanceador (52.167.66.110):

![](images/balanceadorprueba.jpg)


2. Realice las pruebas de carga con `newman` que se realizaron en la parte 1 y haga un informe comparativo donde contraste: tiempos de respuesta, cantidad de peticiones respondidas con éxito, costos de las 2 infraestrucruras, es decir, la que desarrollamos con balanceo de carga horizontal y la que se hizo con una maquina virtual escalada.

![](images/pruebacarga.jpg)

3. Agregue una 4 maquina virtual y realice las pruebas de newman, pero esta vez no lance 2 peticiones en paralelo, sino que incrementelo a 4. Haga un informe donde presente el comportamiento de la CPU de las 4 VM y explique porque la tasa de éxito de las peticiones aumento con este estilo de escalabilidad.

```
newman run ARSW_LOAD-BALANCING_AZURE.postman_collection.json -e [ARSW_LOAD-BALANCING_AZURE].postman_environment.json -n 10 &
newman run ARSW_LOAD-BALANCING_AZURE.postman_collection.json -e [ARSW_LOAD-BALANCING_AZURE].postman_environment.json -n 10 &
newman run ARSW_LOAD-BALANCING_AZURE.postman_collection.json -e [ARSW_LOAD-BALANCING_AZURE].postman_environment.json -n 10 &
newman run ARSW_LOAD-BALANCING_AZURE.postman_collection.json -e [ARSW_LOAD-BALANCING_AZURE].postman_environment.json -n 10
```

[](images/pruebac.jpg)

![](images/pruebac2.jpg)

![](images/pruebac3.jpg)

La tasa de �xito aument� porque ahora cada una de las m�quinas virtuales es la encargada de responder a cada una de las peticiones lanzadas, es decir, cada una solo responde una petici�n secuencialmente y as� disminuye la posibilidad de congestionarse y arrojar errores.




**Preguntas**

* ¿Cuáles son los tipos de balanceadores de carga en Azure y en qué se diferencian?, ¿Qué es SKU, qué tipos hay y en qué se diferencian?, ¿Por qué el balanceador de carga necesita una IP pública?
* ¿Cuál es el propósito del *Backend Pool*?
* ¿Cuál es el propósito del *Health Probe*?
* ¿Cuál es el propósito de la *Load Balancing Rule*? ¿Qué tipos de sesión persistente existen, por qué esto es importante y cómo puede afectar la escalabilidad del sistema?.
* ¿Qué es una *Virtual Network*? ¿Qué es una *Subnet*? ¿Para qué sirven los *address space* y *address range*?
* ¿Qué son las *Availability Zone* y por qué seleccionamos 3 diferentes zonas?. ¿Qué significa que una IP sea *zone-redundant*?
* ¿Cuál es el propósito del *Network Security Group*?
* Informe de newman 1 (Punto 2)
* Presente el Diagrama de Despliegue de la solución.




