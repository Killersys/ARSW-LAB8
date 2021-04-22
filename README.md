### Escuela Colombiana de Ingeniería
### Arquitecturas de Software - ARSW

## Escalamiento en Azure con Maquinas Virtuales, Sacale Sets y Service Plans

### Integrantes:
-[Jairo Pulido](https://github.com/Killersys)

### Dependencias
* Cree una cuenta gratuita dentro de Azure. Para hacerlo puede guiarse de esta [documentaciÃ³n](https://azure.microsoft.com/en-us/free/search/?&ef_id=Cj0KCQiA2ITuBRDkARIsAMK9Q7MuvuTqIfK15LWfaM7bLL_QsBbC5XhJJezUbcfx-qAnfPjH568chTMaAkAsEALw_wcB:G:s&OCID=AID2000068_SEM_alOkB9ZE&MarinID=alOkB9ZE_368060503322_%2Bazure_b_c__79187603991_kwd-23159435208&lnkd=Google_Azure_Brand&dclid=CjgKEAiA2ITuBRDchty8lqPlzS4SJAC3x4k1mAxU7XNhWdOSESfffUnMNjLWcAIuikQnj3C4U8xRG_D_BwE). Al hacerlo usted contarÃ¡ con $200 USD para gastar durante 1 mes.

![](images/dependencia.jpg)

### Parte 0 - Entendiendo el escenario de calidad

Adjunto a este laboratorio usted podrÃ¡ encontrar una aplicación totalmente desarrollada que tiene como objetivo calcular el enÃ©simo valor de la secuencia de Fibonnaci.

**Escalabilidad**
Cuando un conjunto de usuarios consulta un enésimo número (superior a 1000000) de la secuencia de Fibonacci de forma concurrente y el sistema se encuentra bajo condiciones normales de operaciÃ³n, todas las peticiones deben ser respondidas y el consumo de CPU del sistema no puede superar el 70%.

### Parte 1 - Escalabilidad vertical

1. Diríjase a el [Portal de Azure](https://portal.azure.com/) y a continuaciÃ³n cree una maquina virtual con las caracterÃ­sticas bÃ¡sicas descritas en la imÃ¡gen 1 y que corresponden a las siguientes:
    * Resource Group = SCALABILITY_LAB
    * Virtual machine name = VERTICAL-SCALABILITY
    * Image = Ubuntu Server 
    * Size = Standard B1ls
    * Username = scalability_lab
    * SSH publi key = Su llave ssh publica

![Imágen 1](images/part1/part1-vm-basic-config.png)

![](images/escalabilidad.jpg)

2. Para conectarse a la VM use el siguiente comando, donde las `x` las debe remplazar por la IP de su propia VM.

    `ssh scalability_lab@xxx.xxx.xxx.xxx`

![](images/ssh.jpg)

3. Instale node, para ello siga la sección *Installing Node.js and npm using NVM* que encontrarÃ¡ en este [enlace](https://linuxize.com/post/how-to-install-node-js-on-ubuntu-18.04/).
4. Para instalar la aplicación adjunta al Laboratorio, suba la carpeta `FibonacciApp` a un repositorio al cual tenga acceso y ejecute estos comandos dentro de la VM:

![](images/node.jpg)

    `git clone <your_repo>`

    `cd <your_repo>/FibonacciApp`

    `npm install`
    
![](images/vm.jpg)

5. Para ejecutar la aplicación puede usar el comando `npm FibinacciApp.js`, sin embargo una vez pierda la conexiÃ³n ssh la aplicaciÃ³n dejarÃ¡ de funcionar. Para evitar ese compartamiento usaremos *forever*. Ejecute los siguientes comando dentro de la VM.

    `npm install forever -g`

    `forever start FibinacciApp.js`
    
![](images/npm.jpg)

![](images/npm2.jpg)

6. Antes de verificar si el endpoint funciona, en Azure vaya a la secciÃ³n de *Networking* y cree una *Inbound port rule* tal como se muestra en la imÃ¡gen. Para verificar que la aplicaciÃ³n funciona, use un browser y user el endpoint `http://xxx.xxx.xxx.xxx:3000/fibonacci/6`. La respuesta debe ser `The answer is 8`.

![](images/part1/part1-vm-3000InboudRule.png)


![](images/network.jpg)

![](images/networkprueba.jpg)

7. La función que calcula en enésimo número de la secuencia de Fibonacci está muy mal construido y consume bastante CPU para obtener la respuesta. Usando la consola del Browser documente los tiempos de respuesta para dicho endpoint usando los siguintes valores:
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

8. Dírijase ahora a Azure y verifique el consumo de CPU para la VM. (Los resultados pueden tardar 5 minutos en aparecer).

![Imágen 2](images/part1/part1-vm-cpu.png)

![](images/consumocpu.jpg)

9. Ahora usaremos Postman para simular una carga concurrente a nuestro sistema. Siga estos pasos.
    * Instale newman con el comando `npm install newman -g`. Para conocer más de Newman consulte el siguiente [enlace](https://learning.getpostman.com/docs/postman/collection-runs/command-line-integration-with-newman/).
    * Diríjase hasta la ruta `FibonacciApp/postman` en una maquina diferente a la VM.
    * Para el archivo `[ARSW_LOAD-BALANCING_AZURE].postman_environment.json` cambie el valor del parámetro `VM1` para que coincida con la IP de su VM.
    * Ejecute el siguiente comando.

    ```
    newman run ARSW_LOAD-BALANCING_AZURE.postman_collection.json -e [ARSW_LOAD-BALANCING_AZURE].postman_environment.json -n 10 &
    newman run ARSW_LOAD-BALANCING_AZURE.postman_collection.json -e [ARSW_LOAD-BALANCING_AZURE].postman_environment.json -n 10
    ```

![](images/carga.jpg)

10. La cantidad de CPU consumida es bastante grande y un conjunto considerable de peticiones concurrentes pueden hacer fallar nuestro servicio. Para solucionarlo usaremos una estrategia de Escalamiento Vertical. En Azure diríjase a la sección *size* y a continuación seleccione el tamaño `B2ms`.

![Imágen 3](images/part1/part1-vm-resize.png)

Cantidad de CPU consumida:

![](images/porcentajecpu.jpg)


11. Una vez el cambio se vea reflejado, repita el paso 7, 8 y 9.

Repetición paso 7:

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

Repetición paso 8:

![](images/consumocpu2.jpg)

Repetición paso 9:

![](images/carga2.jpg)

12. Evalue el escenario de calidad asociado al requerimiento no funcional de escalabilidad y concluya si usando este modelo de escalabilidad logramos cumplirlo.
13. Vuelva a dejar la VM en el tamaño inicial para evitar cobros adicionales.

**Preguntas**

1. ¿Cuántos y cuáles recursos crea Azure junto con la VM?

![](images/recursos.jpg)

2. ¿Brevemente describa para qué sirve cada recurso?

- Red Virtual: Simulación de una red física con recursos de software y hardware
- Cuenta de almacenamiento: contiene todos los objetos de datos de Azure
- Máquina Virtual: Máquina virtual usada como tal
- Dirección IP pública: Permite saber cuál es la dirección IP que se usa para la conexión saliente
- Grupo de seguridad de red: Contiene reglas de seguridad que permiten o deniegan el tráfico de red entrante o saliente de recursos de Azure
- Interfaz de red: Permite a la maquina virtual comunicarse con recursos en internet.
- Disco:  Es la version virtualizada de la maquina que creamos, guarda el sistema operativo y otros componentes.


3. ¿Al cerrar la conexión ssh con la VM, por qué se cae la aplicación que ejecutamos con el comando `npm FibonacciApp.js`? ¿Por qué debemos crear un *Inbound port rule* antes de acceder al servicio?

Porque en ese momento se cierra la conexión necesaria para ejecutar la aplicación. Para tener una conexión por un puerto con la máquina y el servició que está proveyendo en todo momento.

4. Adjunte tabla de tiempos e interprete por qué la función tarda tando tiempo.

![](images/tablatiempo.jpg)

Porque resuelve la fórmula paso a paso, hace la sumatoria de todos y cada uno de los números anteriores al solicitado

5. Adjunte imágen del consumo de CPU de la VM e interprete por qué la función consume esa cantidad de CPU.

B1ls:

![](images/consumocpu.jpg)

B2ms:

![](images/consumocpu2.jpg)

Porque para el primer caso, toda la CPU está destinada a resolver esta tarea, para el segundo tan solo la mitad.

6. Adjunte la imagen del resumen de la ejecución de Postman. Interprete:

B1ls:

![](images/carga.jpg)

B2ms:

![](images/cargarr.jpg)

* Tiempos de ejecución de cada petición.

Los tiempos de ejecución son muy similares

* Si hubo fallos documentelos y explique.

Error ECONNRESET: Significa que el servidor cerró la conexión de una manera que probablemente no era normal.

7. ¿Cuál es la diferencia entre los tamaños `B2ms` y `B1ls` (no solo busque especificaciones de infraestructura)?

B1ls tiene: 1 vCPU, 0.5 GB de RAM, 2 discos de datos,  160 E/S,  4 GB de almacenamiento temporal

B2ms tiene: 2 vCPU,   8 GB de RAM, 4 discos de datos, 1920 E/S, 16 GB de almacenamiento temporal

8. ¿Aumentar el tamaño de la VM es una buena solución en este escenario?, ¿Qué pasa con la FibonacciApp cuando cambiamos el tamaño de la VM?

No, es cierto que la CPU se satura menos aumentando el tamaño, pero los tiempos de respuesta son iguales o peores.

9. ¿Qué pasa con la infraestructura cuando cambia el tamaño de la VM? ¿Qué efectos negativos implica?

Se presentan errores de tipo ECONNRESET por concurrencia de solicutudes a una CPU saturada, esta no logra responder exitosamente.

10. ¿Hubo mejora en el consumo de CPU o en los tiempos de respuesta? Si/No ¿Por qué?

Sí, se mejoró el consumo de CPU debido a que su capaciad de procesamiento es mayor, pero los tiempos no mejoraron debido a que el programa se sigue ejecutando secuencialmente, no es problema de capacidad sino del diseño del programa.

11. Aumente la cantidad de ejecuciones paralelas del comando de postman a `4`. ¿El comportamiento del sistema es porcentualmente mejor?

No, el comportamiento sigue siendo similar.

### Parte 2 - Escalabilidad horizontal

#### Crear el Balanceador de Carga

Antes de continuar puede eliminar el grupo de recursos anterior para evitar gastos adicionales y realizar la actividad en un grupo de recursos totalmente limpio.

1. El Balanceador de Carga es un recurso fundamental para habilitar la escalabilidad horizontal de nuestro sistema, por eso en este paso cree un balanceador de carga dentro de Azure tal cual como se muestra en la imágen adjunta.

![](images/part2/part2-lb-create.png)

2. A continuación cree un *Backend Pool*, guiese con la siguiente imágen.

![](images/part2/part2-lb-bp-create.png)

3. A continuación cree un *Health Probe*, guiese con la siguiente imágen.

![](images/part2/part2-lb-hp-create.png)

4. A continuación cree un *Load Balancing Rule*, guiese con la siguiente imágen.

![](images/part2/part2-lb-lbr-create.png)

5. Cree una *Virtual Network* dentro del grupo de recursos, guiese con la siguiente imágen.

![](images/part2/part2-vn-create.png)

#### Crear las maquinas virtuales (Nodos)

Ahora vamos a crear 3 VMs (VM1, VM2 y VM3) con direcciones IP públicas standar en 3 diferentes zonas de disponibilidad. Después las agregaremos al balanceador de carga.

1. En la configuración básica de la VM guíese por la siguiente imágen. Es importante que se fije en la "Avaiability Zone", donde la VM1 será 1, la VM2 será 2 y la VM3 será 3.

![](images/part2/part2-vm-create1.png)

2. En la configuración de networking, verifique que se ha seleccionado la *Virtual Network*  y la *Subnet* creadas anteriormente. Adicionalmente asigne una IP pública y no olvide habilitar la redundancia de zona.

![](images/part2/part2-vm-create2.png)

3. Para el Network Security Group seleccione "avanzado" y realice la siguiente configuración. No olvide crear un *Inbound Rule*, en el cual habilite el tráfico por el puerto 3000. Cuando cree la VM2 y la VM3, no necesita volver a crear el *Network Security Group*, sino que puede seleccionar el anteriormente creado.

![](images/part2/part2-vm-create3.png)

4. Ahora asignaremos esta VM a nuestro balanceador de carga, para ello siga la configuración de la siguiente imágen.

![](images/part2/part2-vm-create4.png)

![](images/balanceador.jpg)

5. Finalmente debemos instalar la aplicaciÃ³n de Fibonacci en la VM. para ello puede ejecutar el conjunto de los siguientes comandos, cambiando el nombre de la VM por el correcto

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

1. Porsupuesto el endpoint de acceso a nuestro sistema serÃ¡ la IP pÃºblica del balanceador de carga, primero verifiquemos que los servicios bÃ¡sicos estÃ¡n funcionando, consuma los siguientes recursos:

```
http://52.155.223.248/
http://52.155.223.248/fibonacci/1
```

Con la ip del balanceador (52.167.66.110):

![](images/balanceadorprueba.jpg)


2. Realice las pruebas de carga con `newman` que se realizaron en la parte 1 y haga un informe comparativo donde contraste: tiempos de respuesta, cantidad de peticiones respondidas con Ã©xito, costos de las 2 infraestrucruras, es decir, la que desarrollamos con balanceo de carga horizontal y la que se hizo con una maquina virtual escalada.

![](images/pruebacarga.jpg)

3. Agregue una 4 maquina virtual y realice las pruebas de newman, pero esta vez no lance 2 peticiones en paralelo, sino que incrementelo a 4. Haga un informe donde presente el comportamiento de la CPU de las 4 VM y explique porque la tasa de Ã©xito de las peticiones aumento con este estilo de escalabilidad.

```
newman run ARSW_LOAD-BALANCING_AZURE.postman_collection.json -e [ARSW_LOAD-BALANCING_AZURE].postman_environment.json -n 10 &
newman run ARSW_LOAD-BALANCING_AZURE.postman_collection.json -e [ARSW_LOAD-BALANCING_AZURE].postman_environment.json -n 10 &
newman run ARSW_LOAD-BALANCING_AZURE.postman_collection.json -e [ARSW_LOAD-BALANCING_AZURE].postman_environment.json -n 10 &
newman run ARSW_LOAD-BALANCING_AZURE.postman_collection.json -e [ARSW_LOAD-BALANCING_AZURE].postman_environment.json -n 10
```

[](images/pruebac.jpg)

![](images/pruebac2.jpg)

![](images/pruebac3.jpg)

La tasa de éxito aumentó porque ahora cada una de las máquinas virtuales es la encargada de responder a cada una de las peticiones lanzadas, es decir, cada una solo responde una petición secuencialmente y así disminuye la posibilidad de congestionarse y arrojar errores.




**Preguntas**

* Â¿CuÃ¡les son los tipos de balanceadores de carga en Azure y en quÃ© se diferencian?, Â¿QuÃ© es SKU, quÃ© tipos hay y en quÃ© se diferencian?, Â¿Por quÃ© el balanceador de carga necesita una IP pÃºblica?
* Â¿CuÃ¡l es el propÃ³sito del *Backend Pool*?
* Â¿CuÃ¡l es el propÃ³sito del *Health Probe*?
* Â¿CuÃ¡l es el propÃ³sito de la *Load Balancing Rule*? Â¿QuÃ© tipos de sesiÃ³n persistente existen, por quÃ© esto es importante y cÃ³mo puede afectar la escalabilidad del sistema?.
* Â¿QuÃ© es una *Virtual Network*? Â¿QuÃ© es una *Subnet*? Â¿Para quÃ© sirven los *address space* y *address range*?
* Â¿QuÃ© son las *Availability Zone* y por quÃ© seleccionamos 3 diferentes zonas?. Â¿QuÃ© significa que una IP sea *zone-redundant*?
* Â¿CuÃ¡l es el propÃ³sito del *Network Security Group*?
* Informe de newman 1 (Punto 2)
* Presente el Diagrama de Despliegue de la soluciÃ³n.




