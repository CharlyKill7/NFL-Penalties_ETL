# W4 Project - ETL NFL Penalties

![portada](https://github.com/CharlyKill7/Database-Project/blob/main/images/videoclub.jpg)

## ⛓️ Índice

1. [Descripción](#descripción)
2. [Extracción](#extracción)
3. [Transformación](#transformación)
4. [Carga](#carga)
5. [BONUS: Consultas y conclusión](#consultas)


<a name="descripción"/>

## Descripción del proyecto

En este proyecto habremos de efectuar un proceso completo de ETL, las siglas en inglés de Extract, Transform y Load. Los datos a extraer serán de nuestra elección, pero habrá que cumplir ciertos requerimientos. En este caso, vamos a extraer datos y hacer un pequeño análisis acerca de las decisiones arbitrales en la NFL.

### Restricciones:
- Obtener la información de tres fuentes distintas (urls).
- Dos métodos distintos de extracción (csv, excel, api, rss, web scrapping...)

### Objetivo:
 
Nuestro objetivo es encontrar tres fuents de datos distintas sobre las señalizaciones arbitrales en la NFL, de tal modo que podamos generar un DataFrame rico y completo a través de cual podamos sacar conclusiones al respecto. En este sentido, trabajaremos en la transformación del dato crudo para su adaptación al resto de las fuentes, con el fin de establecer una base de datos SQL con los resultados, pudiendo lanzar queries que nos ayuden a confirmar o refutar las hipótesis que se nos ocurran.

 
 <a name="extracción"/>
 
## Extracción

En primer lugar hemos realizado un ejercicio analítico de cada uno de los siete CSV que nos han proporcionado utilizando las técnicas más comunes como son `.head`,`.tail`,`.info`, `.shape`, `.columns` y `.value_counts`  para obtener información general de cada CSV. El objetivo de esta tarea consiste en verificar que las columnas estén limpias, tengan sentido, y encontrar inconsistencias. Durante este proceso, encontramos las siguientes incongruencias:

<details>
<summary>¿ACTRIZ DUPLICADA?</summary>
<br>

 ![susan](https://github.com/CharlyKill7/Database-Project/blob/main/images/Susandavis.png)

</details>

<details>
<summary>¿RELEASE DATE INCORRECTO?</summary>
<br>

Notamos que la columna **release_year**, la fecha de estreno de las películas, indica **2006** para todas las pelis.
<br>
Pero en las fechas de los alquileres: 
```
year = []
for i in ren.rental_date:
    year.append(i[0:4])
set(year) 
```
**2005**
<br>
<br>
Al ser todos los alquileres previos a 2006, concluimos que la columna **release_year** está incorrectamente introducida y por tanto la vaciamos.


</details>

<br>

**¿Qué tenemos?**


Explorando la tabla **INVENTORY** vimos que había mil películas inventariadas, y a través de **film_id** descubrimos que se correspondían con las primeras **223** películas de la tabla **FILMS**. En otras palabras, en nuestro inventario **sólo había películas con títulos de la ‘A’ a la ‘D’**. Esto nos hizo sospechar que tal vez la información estuviera incompleta.


Poco después, durante el análisis de la tabla **RENTAL**, nos percatamos de que la columna **inventory_id** contenía valores por encima de los mil de la tabla **INVENTORY** (hasta el **4581**). Es decir, en nuestro videoclub se habían estado alquilando películas que no figuraban en inventario. Así nos convencimos de que nuestra hipótesis era correcta.


 <a name="transformación"/>
 
## Transformación

Nuestra intención siempre fue simplificar, además de profesionalizar, el manejo del videoclub. Para ello, decidimos quedarnos con las tablas que solo fueran indispensables, a pesar de que todas ellas proporcionaban alguna información valiosa. A continuación, detallamos el proceso de selección para nuestra base de datos:


<br>

<img src="https://github.com/CharlyKill7/Database-Project/blob/main/images/EERD_inicial.png" width="550" height="400" />

<a name="carga"/>

## Carga

<details>
<summary>INVENTORY_MASTER</summary>
<br>
 
 
<details>
<summary>Si alquilamos la película cuyo id de inventario es 1...</summary>
<br>
 
```
 INSERT INTO rental (rental_id, rental_date, inventory_id, customer_id, return_date, staff_id)
 
 VALUES (1002, '2005-05-31 00:55:00', 1, 588, '', 2);
  ```


</details>
 
![inventory_master](https://github.com/CharlyKill7/Database-Project/blob/main/images/inventory_master.png)

</details>

<details>
<summary>CUSTOMER_MASTER</summary>
<br>
 
 ```
SELECT customer.customer_id AS 'CUSTOMER ID', name AS 'NAME', lastname AS 'LAST NAME', telephone AS TELEPHONE, 
	   mail AS EMAIL, round(sum((rental_rate * (DATEDIFF(return_date, rental_date) + 1))), 2) AS 'TOTAL SPENT',
       GROUP_CONCAT(' ',title) AS 'FILMS RENTED'
       
FROM rental

LEFT JOIN inventory ON inventory.inventory_id = rental.inventory_id
LEFT JOIN films ON inventory.film_id = films.film_id
LEFT JOIN customer ON customer.customer_id = rental.customer_id

GROUP BY customer.customer_id, name , lastname, telephone, mail
;
 ```
![customer_master](https://github.com/CharlyKill7/Database-Project/blob/main/images/customer_master.png)

</details>


<a name="consultas"/>

## 📊 BONUS: Consultas y conclusión

<details>
<summary>LOS CLIENTES QUE MÁS ALQUILAN</summary>
<br>

 ```
SELECT  rental.customer_id, count(rental.customer_id) as Rentals
FROM rental
 
LEFT JOIN customer ON rental.customer_id = customer.customer_id
 
GROUP BY rental.customer_id
 
ORDER BY Rentals desc
LIMIT 5
 ```

![top_clientes_cantidad](https://github.com/CharlyKill7/Database-Project/blob/main/images/top_clientes_cantidad.PNG)
	
</details>

<details>
<summary>LOS CLIENTES QUE MÁS GASTAN</summary>
<br>

```
SELECT  `customer id`, round(sum(Income),2) as 'Total Spent'
FROM rental_master
 
GROUP BY `customer id`
 
ORDER BY 'Total Spent' desc
LIMIT 5 
 ```

![top_clientes_income2](https://github.com/CharlyKill7/Database-Project/blob/main/images/top_clientes_income2.PNG)

</details>

<details>
<summary>LAS PELÍCULAS QUE MÁS SE ALQUILAN</summary>
<br>

```
SELECT  title, count(title) as Alquileres
FROM rental_master
 
GROUP BY title
 
ORDER BY Alquileres DESC
LIMIT 5
 ```

![top_pelis](https://github.com/CharlyKill7/Database-Project/blob/main/images/top_pelis.png)

</details>
