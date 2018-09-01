# Guía de uso Slurm

## Introducción a Slurm

Slurm  correponde el administrador de trabajos dentro del cluster, es el encargado de gestionar las colas. En la actualidad es utilizado en el 60%  de los supercomputadores del [top500](https://www.top500.org/). La versión disponible en el cluster es la [15.08.7](https://slurm.schedmd.com/archive/slurm-15.08-latest/).

#### Principales funciones
* #### Gestión de tareas en la cola
* #### Asignación de recursos
* #### Permite la modificación de tareas durante su ejecución

## ¿Dónde se encuentra?

Slurm actualmente se encuentra funcionando en:

* #### Caleuche
* #### Trauco

## Comandos básicos

Existe documentación para cada comando en la página oficial de Slurm. Algunos de los comandos básicos para el uso del scheduler se encuentran detallados a continuación.

### [sinfo](https://slurm.schedmd.com/sinfo.html)
Imprime información sobre las particiones del cluster y sus estados.
##### Ejemplo:
```
$ sinfo
PARTITION  AVAIL  TIMELIMIT  NODES  STATE NODELIST
ialab-low*    up 3-00:00:00      2  down* makemake,titan
ialab-low*    up 3-00:00:00      2   idle caleuche,tripio
ialab-high    up   infinite      3   idle hydra,icarus,kraken
zippedi       up   infinite      1  down* ahsoka
dipbd         up    6:00:00      2  down* makemake,titan
dipbd         up    6:00:00      3   idle caleuche,kraken,tripio
all           up 3-00:00:00      4  down* ahsoka,makemake,titan,trauco
all           up 3-00:00:00      5   idle caleuche,hydra,icarus,kraken,tripio
```

### [squeue](https://slurm.schedmd.com/squeue.html)
Permite ver el estado de los trabajos que se encuentran en la cola de Slurm.
Además es posile filtrar la información del trabajo por usuario usando el parametro ```-u [usuario]```.
##### Ejemplo:
```
$ squeue
JOBID   PARTITION NAME      USER      ST   TIME  NODES  NODELIST(REASON)
605_28  ialab-low trabajito username  CG   0:00      1  caleuche
605_35  ialab-low trabajito username  CG   0:00      1  tripio
606     ialab-low main-job  user_abc  CG   0:00      1  tripio

$ squeue -u user_abc
JOBID   PARTITION NAME      USER      ST   TIME  NODES  NODELIST(REASON)
606     ialab-low main-job  userabc   CG   0:00      1  tripio
```

### [srun](https://slurm.schedmd.com/srun.html)
Envía un trabajo iterativo a la cola de ejecución de Slurm.
Se debe definir el parametro ```-n [numero]``` para definir el numero de ejecucciones paralelas.
##### Ejemplo:
```
$ srun -n 2 python main.py
```

### [sbatch](https://slurm.schedmd.com/sbatch.html)
Permite añadir un trabajo a a cola de Slurm.

##### Ejemplo:
```
$ cat script.sh
#!/bin/bash
#SBATCH --job-name=compile
#SBATCH -t 0-2:00                    # tiempo maximo en el cluster (D-HH:MM)
#SBATCH -o c_job.out                 # STDOUT (A = )
#SBATCH -e c_job.err                 # STDERR
#SBATCH --mail-type=END,FAIL         # notificacion cuando el trabajo termine o falle
#SBATCH --mail-user=usuario@uc.cl    # mail donde mandar las notificaciones
#SBATCH --workdir=/user/miusuario    # direccion del directorio de trabajo
#
#SBATCH --nodes 1                    # numero de nodos a usar
#SBATCH --ntasks-per-node=24         # numero de trabajos (procesos) por nodo
#SBATCH --cpus-per-task=1            # numero de cpus (threads) por trabajo (proceso)  

gcc -o a.out main.c
echo "Finished with job $SLURM_JOBID"
```
```
$ cat script-array.sh
#!/bin/bash
#SBATCH --job-name=trabajito
#SBATCH -t 0-2:00                    # tiempo maximo en el cluster (D-HH:MM)
#SBATCH -o slurm-%a.out              # STDOUT
#SBATCH -e slurm-%a.err              # STDERR
#SBATCH --mail-type=ALL              # notificacion cuando el trabajo termine o falle
#SBATCH --mail-user=usuario@uc.cl    # mail donde mandar las notificaciones
#SBATCH --workdir=/user/miusuario    # direccion del directorio de trabajo
#
#SBATCH --ntasks 1                   # 1 trabajo
#SBATCH --array 1-100%10             # 100 procesos, 10 simultáneos

python3 main.py $SLURM_ARRAY_TASK_ID

$ sbatch script.sh
Submitted batch job 60
```

### [scancel](https://slurm.schedmd.com/scancel.html)
Permite cancelar un trabajo por su JobID
##### Ejemplo:
```
$ sbatch script.sh
Submitted batch job 14

$ scancel 14
```
### [scontrol](https://slurm.schedmd.com/scontrol.html)
Es usado para ver o modificar el estado de un trabajo.
##### Ejemplo:
```
$ sbatch script.sh
Submitted batch job 29

$ scontrol hold 29
```
El comando hold detendrá la ejecución mientras que release la retomará.
```
$ scontrol release 29
```

## Variables de entorno

Slurm establecera o preestablecerá las variables de entorno que puedan ser usadas en el script. En la siguiente tabla se muestras las mas usadas:

|Descripción      |Slurm                        |
|-----------------|-----------------------------|
|JobID            |$SLURM_JOBID                 |
|Submit Directory |$SLURM_SUBMIT_DIR (default)  |
|Submit Host      |$SLURM_SUBMIT_HOST           |
|Node List        |$SLURM_JOB_NODELIST          |
|Job Array Index  |$SLURM_ARRAY_TASK_ID         |


## Flags comunes

A continuación se muestra una lista de los flags más comunes que un usuario puede incluir en su trabajo, ya sean para solicitar recursos o características para el trabajo.

|Descripción      |Slurm                           |
|-----------------|--------------------------------|
|Nombre del trabajo       |#SBATCH --job-name=My-Job_Name  |
|Número de nodos solicitados |#SBATCH --nodes=1 |
|Número de cores por nodo solicitados |#SBATCH --ntasks-per-node=24        |
|Copia las variables de entorno del usuario  |#SBATCH --export=[ALL\|NONE\|Variables]  |
|Restricción de tiempo |#SBATCH --time=24:0:0   |
|Reiniciar un trabajo en caso de falla|#SBATCH --requeue  |
|Compartir los nodos |#SBATCH --shared |
|Reservar los nodos para uso exclusivo|#SBATCH --exclusive |
|Uso de un recurso específico |#SBATCH --constraint="XXX"|
|Uso de memoria |#SBATCH --mem=[mem \|M\|G\|T] o --mem-per-cpu |
|Email usuario |#SBATCH --mail-user=username@uc.cl |
|Solicitud nodo específico |#SBATCH --nodelist=Caleuche |


## Flags importantes en tu trabajo

Es importante conocer los flags que puedes utilizar para una adecuada administración de los recursos dado que su correcto uso traerá beneficios tanto para scheduler como para tu trabajo.

#### Restricción de tiempo

Para restringir el tiempo que correrá un trabajo se hace uso de [--time](https://slurm.schedmd.com/sbatch.html)  donde se específica el tiempo mínimo y máximo que tiene permitido correr dentro del cluster. Por un lado, si el tiempo solicitado no es suficiente el programa será cortado y los resultados se perderán.  Por otro lado, si el tiempo específicado es demasiado el trabajo permanecerá en la cola por mucho tiempo mientras el scheduler busca los recursos que solicitó, además, una vez asignados los recursos otros programas no podrán acceder a ellos afectando la eficiencia de la administración de Slurm.

##### Ejemplo:
Al incluir la siguiente línea en el script el tiempo máximo serán dos horas.

```
#SBATCH -t 0-2:00                               # time (D-HH:MM)
```

#### Nodos

Reservar un nodo para uso es posible con el comando [--exlusive](https://slurm.schedmd.com/sbatch.html), sin embargo el uso de este flags se recomienda ser usado en el caso que el programa que se desea correr depende en gran parte de la transferencia de datos entre los trabajos, esto justificaría la asignación a un solo usuario de un nodo completo.En la mayoŕía de los casos los trabajos son relativamente pequeños permitiendo que puedan compartir un mismo nodo, para todas estas situaciones se mantendrá la configuración determinada [--shared](https://slurm.schedmd.com/sbatch.html) en el nodo. 

El número de nodos que se desea utilizar es definidos con [--nodes o -N](https://slurm.schedmd.com/sbatch.html) donde es posible usar un intervalo de nodos necesarios como 2-4  para este caso el scheduler correra el programa cuando encuentre al menos 2 nodos disponibles si encuentra los 4 correrá utilizando los 4 sugeridos, en cambio si específica un número de nodos específicos como 5 no correrá hasta hallar disponibles 5. Se recomienda verificar las disponibilidad de los nodos antes de lanzar el trabajo utilizando el comando [--sinfo](https://slurm.schedmd.com/sinfo.html) como se mencionó anteriormente.

### Uso de GPU

A pesar que hay muchos cores dentro de la GPU estos no son compatibles con el estándar intel x86, es por esto que el código debe ser escrito utilizando [CUDA](https://www.nvidia.es/object/cuda-parallel-computing-es.html).

Para solicitar el uso de gpu en tu trabajo se utilizan --gres=gpu ó --gres=gpu:N donde N es el número de gpus por nodo.

##### Ejemplo:
```
#SBATCH -t 0-2:00                               # time (D-HH:MM)
#SBATCH -N 4
#SBATCH --gres=gpu

cd /slurm/gpuExamples
./run_my_gpu_code

```
### Scripts básicos
En este [documento](/doc/slurm_samples.md) podrás encontrar algunos scripts básicos para el uso de slurm en determinados casos.