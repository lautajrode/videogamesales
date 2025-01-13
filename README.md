Introducción al Dataset "Video Game Sales"

El dataset Video Game Sales de Kaggle contiene información sobre ventas de videojuegos a nivel global, abarcando distintos géneros, plataformas, y regiones. Este dataset es útil para realizar análisis de tendencias de ventas en la industria de videojuegos y comprender el impacto de diferentes factores en el éxito comercial de los títulos.
El mismo fue obtenido y descargado del sitio web: [www.kaggle.com ](https://www.kaggle.com/)

![imgkaggle](https://github.com/user-attachments/assets/8af3dea0-9e1e-4ce1-b143-9a60f51217a0)

---

Características principales del dataset:

- Nombre del juego: El título del videojuego.
- Plataforma: La consola o sistema en el que el juego fue lanzado (e.g., PS4, Xbox, PC).
- Año de lanzamiento: El año en que se lanzó el videojuego.
- Género: El género del videojuego (e.g., Acción, Deportes, Rol).
- Publisher: La compañía que publicó el videojuego.
- Ventas regionales: Ventas desglosadas por región:
- NA_Sales: Ventas en América del Norte (en millones de dólares).
- EU_Sales: Ventas en Europa (en millones de dólares).
- JP_Sales: Ventas en Japón (en millones de dólares).
- Other_Sales: Ventas en otras regiones.
- Global_Sales: Ventas totales globales.

---

A partir de los datos obtenidos se crea un plan de metricas:

![plan de metricas](https://github.com/user-attachments/assets/5f857861-d164-433c-856f-f9a49252cf41)

---

Plan de análisis:

- Cantidad de ventas de videojuegos por género:

Identificar cuáles géneros dominan en términos de ventas globales.
Comparar ventas por género en cada región (NA, EU, JP, Others).

- Suma de ventas por año:

Analizar las tendencias de ventas a lo largo de los años para identificar periodos de crecimiento o declive.

- Ventas por publisher:

Determinar cuáles son las empresas más exitosas en términos de ventas globales y regionales.

- Ventas por género geográficamente:

Examinar cómo varían las preferencias de género según la región y qué géneros dominan en mercados clave.

- Ventas por plataforma:

Identificar cuáles plataformas generaron mayores ingresos globales.
Verificar si la popularidad de una plataforma está relacionada con la cantidad de títulos publicados.

---

Luego se crea una capa silver del proyecto en GCP
Utilizamos el siguiente codigo:

```

SELECT * 
FROM `bronze_video_game_sales.raw_vgsales`
LIMIT 10;


-- Crear tabla curada (Capa Silver)
CREATE TABLE `trabajofinal-445014.silver_video_game_sales.silver_vgsales` AS
SELECT 
    Rank, 
    Name,
    Platform,
    SAFE_CAST(Year AS INT64) AS Year,  -- Convierte a INT64 de forma segura (valores no numéricos se convierten en NULL)
    Genre,
    Publisher,
    IFNULL(NA_Sales, 0) AS NA_Sales,  -- Reemplazar valores nulos con 0
    IFNULL(EU_Sales, 0) AS EU_Sales,
    IFNULL(JP_Sales, 0) AS JP_Sales,
    IFNULL(Other_Sales, 0) AS Other_Sales,
    IFNULL(Global_Sales, 0) AS Global_Sales
FROM `trabajofinal-445014.bronze_video_game_sales.raw_vgsales`
WHERE Name IS NOT NULL 
  AND Platform IS NOT NULL
  AND SAFE_CAST(Year AS INT64) IS NOT NULL;

```


![imgcapasilver](https://github.com/user-attachments/assets/ec0dcff3-c81c-4144-a459-648e69e5ff95)

---

Ya en Power BI, se crean las diferentes tablas metricas realizando sus correspondientes consultas en DAX:

![Imgtablas2](https://github.com/user-attachments/assets/35650fc8-70bc-48da-b5c2-ecd1ffac774a)


En el caso de la primer tabla, la consulta para el top 10 de videojuegos seria asi:

```
Top_10_videojuegos = 
TOPN(10,
     'silver_vgsales',
     'silver_vgsales'[Global_Sales],
     DESC)
```

Y dara como resultado la siguiente tabla:

![imgdaxtabla](https://github.com/user-attachments/assets/7855629d-502b-45cd-9246-2f263d94a380)

Para las demas tablas utilizaremos las siguientes consultas:

Top 5 publishers: 

```

Top_5_Publishers = 
TOPN(5,
     SUMMARIZE(
         'silver_vgsales',
         'silver_vgsales'[Publisher],
         "Total_Ventas", SUM('silver_vgsales'[Global_Sales])
     ),
     [Total_Ventas],
     DESC)

```

Ventas por año: 

```

Ventas_Por_Anio = 
SUMMARIZE(
    'silver_vgsales',
    'silver_vgsales'[Year],
    "Total_Ventas", SUM('silver_vgsales'[Global_Sales])
)

```

Ventas por genero:

```

Ventas_Por_Genero = 
SUMMARIZE(
    'silver_vgsales',
    'silver_vgsales'[Genre],
    "Total_Ventas", SUM('silver_vgsales'[Global_Sales])
)

```

Ventas por genero en distintas regiones:

```

Ventas_Por_Genero_Region = 
SUMMARIZE(
    'silver_vgsales',
    'silver_vgsales'[Genre],
    "Ventas_NA", SUM('silver_vgsales'[NA_Sales]),
    "Ventas_EU", SUM('silver_vgsales'[EU_Sales]),
    "Ventas_JP", SUM('silver_vgsales'[JP_Sales]),
    "Ventas_Other", SUM('silver_vgsales'[Other_Sales])
)

```

Ventas por plataforma: 

```

Ventas_Por_Plataforma = 
SUMMARIZE(
    'silver_vgsales',
    'silver_vgsales'[Platform],
    "Total_Ventas", SUM('silver_vgsales'[Global_Sales])
)

```

Distribucion regional ventas: 

```

Distribucion_Regional_Ventas = 
SUMMARIZE(
    'silver_vgsales',
    'silver_vgsales'[Name],
    "Ventas_NA", SUM('silver_vgsales'[NA_Sales]),
    "Ventas_EU", SUM('silver_vgsales'[EU_Sales]),
    "Ventas_JP", SUM('silver_vgsales'[JP_Sales]),
    "Ventas_Other", SUM('silver_vgsales'[Other_Sales])
)
```


A partir de nuestros datos, se eligen las distintas visualizaciones para crear nuestras metricas:

![datosyvisualizaciones2](https://github.com/user-attachments/assets/7ba00c14-eb4f-4857-a548-3a90f614f4f8)


Asi quedarian los dashboards

![dashboard1](https://github.com/user-attachments/assets/0497b8bb-0161-4f3f-a460-89a6dd208cdb)

![dashboard2](https://github.com/user-attachments/assets/f5e5deda-3523-46a7-a0d6-7435932a2e09)



Hipótesis principal:

Los géneros más populares, como Acción y Deportes, tienden a generar mayores ventas globales, especialmente en plataformas con una amplia base de usuarios, como PlayStation y Xbox.

Hipótesis secundarias:

Los videojuegos publicados por compañías grandes (como Nintendo y Electronic Arts) generan mayores ventas globales debido a su capacidad de marketing y distribución.
Las ventas de videojuegos tienen un pico en años específicos debido al lanzamiento de nuevas consolas o franquicias exitosas.
La preferencia por géneros varía por región: Japón muestra mayor interés por los géneros RPG y Aventura, mientras que América del Norte prefiere Acción y Deportes.
Las plataformas con mayor número de títulos exclusivos tienen ventajas significativas en ventas globales y regionales.

