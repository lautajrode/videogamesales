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


Se crea una portada y luego 3 dashboards

![dash1](https://github.com/user-attachments/assets/23e0d1b8-0b01-4818-b067-3e21dc1c7379)

![dash2](https://github.com/user-attachments/assets/84613a42-5d18-4987-bc78-42a421537936)

![dash3](https://github.com/user-attachments/assets/471e6887-f7e3-4e8e-83cd-6dee7f74277d)

![dash5](https://github.com/user-attachments/assets/284f50e7-69ab-4141-ac49-7594b0719f13)




Hipótesis principal:

Los géneros más populares, como Acción y Deportes, tienden a generar mayores ventas globales, especialmente en plataformas con una amplia base de usuarios, como PlayStation y Xbox.

![hipotesis1](https://github.com/user-attachments/assets/c2e5f723-5f67-4d93-a4b0-e39355356926)


Hipótesis secundarias:

Los videojuegos publicados por compañías grandes (como Nintendo y Electronic Arts) generan mayores ventas globales debido a su capacidad de marketing y distribución.

![hipotesis2](https://github.com/user-attachments/assets/1e563f3a-3505-44d3-8c51-e0091d3a7b21)

Las ventas de videojuegos tienen un pico en años específicos debido al lanzamiento de nuevas consolas o franquicias exitosas.
Consolas Exitosas
PlayStation 2 (PS2) - 2000
Nintendo DS - 2004
Wii - 2006
Xbox 360 - 2005
PlayStation 3 (PS3) - 2006
Franquicias de Juegos Exitosas
Grand Theft Auto (GTA) - Vice City (2002), San Andreas (2004), GTA IV (2008)
Call of Duty - Modern Warfare (2007), Black Ops (2010)
Mario - Super Mario Galaxy (2007), New Super Mario Bros. (2006)
Pokémon - Ruby and Sapphire (2003), Diamond and Pearl (2006)
The Legend of Zelda - Twilight Princess (2006), Skyward Sword (2011)
Halo - Halo 3 (2007), Halo: Reach (2010)
Final Fantasy - Final Fantasy X (2001), Final Fantasy XIII (2009)
Guitar Hero - Guitar Hero III: Legends of Rock (2007)

![hipotesis3](https://github.com/user-attachments/assets/f591ec96-60c0-48f2-981f-c70c4ce2fdb6)

Las plataformas con mayor número de títulos exclusivos tienen ventajas significativas en ventas globales y regionales.
PlayStation 2 (PS2)
Gran Turismo 3: A-Spec (2001)
Gran Turismo 4 (2004)
Final Fantasy X (2001)
Metal Gear Solid 2: Sons of Liberty (2001)
Kingdom Hearts (2002)
Xbox 360
Halo 3 (2007)
Gears of War (2006)
Gears of War 2 (2008)
Forza Motorsport 2 (2007)
Fable II (2008)
PlayStation 3 (PS3)
Uncharted 2: Among Thieves (2009)
Gran Turismo 5 (2010)
The Last of Us (2013)
LittleBigPlanet (2008)
God of War III (2010)
Wii
Wii Sports (2006)
Mario Kart Wii (2008)
New Super Mario Bros. Wii (2009)
Super Mario Galaxy (2007)
Super Smash Bros. Brawl (2008)

![hipotesis4](https://github.com/user-attachments/assets/460e8dce-7094-4a5b-b622-c0c81d8d3907)


