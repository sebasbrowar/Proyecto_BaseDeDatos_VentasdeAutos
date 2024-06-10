PROYECTO: BASE DE DATOS
Los datos que yo escogí son relacionados a ventas de autos, los obutve en Kaggle, este es el link: https://www.kaggle.com/datasets/syedanwarafridi/vehicle-sales-data

Como se puede observar, los datos ya vienen en una tabla y para este caso yo busco al menos dos tabla para de esta manera poder hacer lo que se me pide y asociarlas.
Entonces lo primero que hice fue abrir los los datos en excel y crear dos tablas.

Las columnas de los datos originales eran las siguientes:
year	make	model	trim	body	transmission	make	model	trim	body	transmission	vin	state	condition	odometer	color	interior	seller	mmr	sellingprice	saledate

Entonces cree primero una hoja en excel llamada cars_prices_temp con las siguientes columnas:
make	model	trim	body	transmission	vin	state	condition	odometer	color	interior	seller	mmr	sellingprice	saledate

Y otra hoja llamada car_models con las columnas restantes y a la cual le agregué una columna llamada id que va de 1 a n:
year	make	model	trim	body	transmission

La idea aquí fue separar los datos que describen a los modelos de auto (año, fabricante, modelo, trim, cuerpo y transmisión respectivamente) ya que en los datos estos vienen repetidos, o sea, hay muchos autos con su propio Vehicle Identification Number (vin) que son el mismo modelo. Entonces crearemos un indice llamado car_model_id el cual representará a un modelo de auto, esto facilitará el manejo de dato ya que el modelo será representado por un número y también hará que se vea mejor al no tener tantos repetidos.


Primero empezamos creando la tabla car_models, que tendrá la información de los distintos modelos de autos:

CREATE TABLE `car_models` (
  `id` int NOT NULL,
  `year` int DEFAULT NULL,
  `make` varchar(50) DEFAULT NULL,
  `model` varchar(50) DEFAULT NULL,
  `trim` varchar(100) DEFAULT NULL,
  `body` varchar(50) DEFAULT NULL,
  `transmission` varchar(50) DEFAULT NULL,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci;

Ahora importamos los datos del archivo excel 'car_models' que creamos anteriormente a la tabla car_models que acabamos de crear.
Esto se hace dandole click derecho a la tabla desde MySQL Workbench, seleccionamos 'Table Data Import Wizard', continuamos lo que nos dicen y listo.


Luego creamos la tabla 'cars_prices_temp', esta será una tabla temporal que utilizaremos para despues crear la final. Contiene el resto de columnas que dijimos previamente.

CREATE TABLE `cars_prices_temp` (
  `year` int DEFAULT NULL,
  `make` varchar(50) DEFAULT NULL,
  `model` varchar(50) DEFAULT NULL,
  `trim` varchar(100) DEFAULT NULL,
  `body` varchar(50) DEFAULT NULL,
  `transmission` varchar(50) DEFAULT NULL,
  `vin` varchar(100) DEFAULT NULL,
  `state` varchar(50) DEFAULT NULL,
  `car_condition` int DEFAULT NULL,
  `odometer` int DEFAULT NULL,
  `color` varchar(50) DEFAULT NULL,
  `interior` varchar(50) DEFAULT NULL,
  `seller` varchar(100) DEFAULT NULL,
  `mmr` int DEFAULT NULL,
  `sellingprice` int DEFAULT NULL,
  `saledate` varchar(100) DEFAULT NULL
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci;

Importamos los datos del archivo excel 'cars_prices_temp' que creamos anteriormente a la tabla cars_prices_temp que acabamos de crear.


Continuando, creamos la tabla que será la final, esta tabla ya tendrá el 'car_model_id' pero aun no la llenaremos con los datos. 

CREATE TABLE `car_prices` (
  `id` int NOT NULL AUTO_INCREMENT,
  `car_model_id` int DEFAULT NULL,
  `vin` varchar(100) DEFAULT NULL,
  `state` varchar(50) DEFAULT NULL,
  `car_condition` int DEFAULT NULL,
  `odometer` int DEFAULT NULL,
  `color` varchar(50) DEFAULT NULL,
  `interior` varchar(50) DEFAULT NULL,
  `seller` varchar(100) DEFAULT NULL,
  `mmr` int DEFAULT NULL,
  `sellingprice` int DEFAULT NULL,
  `saledate` varchar(100) DEFAULT NULL
) ENGINE=InnoDB AUTO_INCREMENT=327676 DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci;


Ahora sigue crear la conexión entre las tablas car_models y cars_prices_temp.

Con esta sentencia poblamos la tabla car_prices a partir de un JOIN entre las tablas car_models y cars_prices_temp.
cm = car_models, cpt = cars_prices_temp

INSERT INTO car_prices (car_model_id, vin, state, car_condition, odometer, color, 
interior, seller, mmr, sellingprice, saledate)
SELECT cm.id as car_model_id, cpt.vin, cpt.state, cpt.car_condition, cpt.odometer, cpt.color, 
cpt.interior, cpt.seller, cpt.mmr, cpt.sellingprice, cpt.saledate  FROM cars_prices_temp cpt
JOIN car_models cm 
ON cpt.year = cm.year and cpt.make = cm.make and cpt.model = cm.model
and cpt.trim = cm.trim and cpt.body = cm.body and cpt.transmission = cm.transmission


Por último hacemos que car_model_id sea una FOREIGN KEY, esta restricción asegura que los valores en las columnas coincidan con los valores en una columna de otra tabla.

ALTER TABLE car_prices
ADD CONSTRAINT fk_car_models
FOREIGN KEY (car_model_id) REFERENCES car_models(id);


Ahora nos queda llevar nuestros datos a python, intenté hacerlo desde un Jupyter Notebook pero no pude, así que lo hice desde PyCharm utilizando 'mysql-connector-python':

import mysql.connector

# Nos conectamos a la base de datos
conexion = mysql.connector.connect(
    host="localhost",
    user="root",
    password="---------",
    database="Proyecto"
)

# Creamos un cursor para ejecutar consultas
cursor = conexion.cursor()

# Ejecutar una consulta
cursor.execute("SELECT COUNT(*) FROM car_models")

# Obtenemos los resultados
resultados = cursor.fetchall()
for fila in resultados:
    print(fila)

# Cerrar cursor y conexión
cursor.close()
conexion.close()

Yo como ejemplo puse ("SELECT COUNT(*) FROM car_models") y me devolvió (26216,), respuesta que corresponde a la cantidad de datos.


