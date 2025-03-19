# Limpieza de los shapefiles de WindHub

## Importar librerías y definir directorio de exportación


```python
import pandas as pd
import geopandas as gpd
import re


def filtrar_shp(shape: gpd.GeoDataFrame, clase: str) -> gpd.GeoDataFrame:
    return shape[shape.clase == clase].reset_index(drop=True)


def negate(boolean_list: list[bool]) -> list[bool]:
    return [not elem for elem in boolean_list]


folder = "Capas combinadas - Limpias"
```

## Limpiar capa de construcciones


```python
# Cargar shapefile
construccion = gpd.read_file("Construccion_P.shp")

# Seleccionar y renombrar columnas relevantes
construccion = construccion[['fid', 'OBJECTID', "CODIGO_USO", "PROYECTO",
                             "FECHA", "NOMBRE_GEO", "path", "geometry"]]

construccion.columns = ["fid", "OBJECTID", "codigo_uso", "proyecto", "fecha",
                        "tipo", "path", "geometry"]

# Eliminar puntos sin el dato de nombre
construccion = construccion[construccion.tipo.notna()]

# Identificar puntos con nombres no inteligibles o en código)
filter_construccion = [
    True
    if re.sub("-?[0-9]?-?", "", nombre)
    and len(re.sub("-?[0-9]?-?", "", nombre)) > 2
    and not re.sub("-?[0-9]?-?", "", nombre).startswith("_")
    and not re.sub("-?[0-9]?-?", "", nombre) == "<Null>"
    else False
    for nombre in construccion.tipo
]

# Filtrar con base en la identificación anterior
construccion[filter_construccion].tipo

# Homogeneizar nombres
construccion.tipo = ["Alberca"
                     if nombre == "alberca"
                     else "Corral"
                     if nombre == "corral"
                     else "Torre de medición"
                     if nombre == "Torre de MediciÃ³n"
                     else "Torre de medición"
                     if nombre == "Torre de mediciÃ³n Guerrero"
                     else "Torre de medición"
                     if nombre == "Torre de mediciÃ³n Pattain"
                     else nombre
                     for nombre in construccion.tipo
                     ]

construcciones = [  # 'Atay',
    'Alberca', 'Cementerio', 'Cocina', 'Corral', 'Enramada', 'Escuela',
    'Habitable', 'Hogar', 'Huerta', 'Iglesia', 'Molino', 'Pista de caballos',
    'Pozo', 'Roza', 'Santana', 'Torre de medición'
]

for cn in construcciones:
    construccion.tipo = [
        cn
        if nombre.startswith(cn)
        else nombre
        for nombre in construccion.tipo
    ]

# Seleccionar tipos específicos
construccion = construccion[construccion.tipo.isin(construcciones)
                            ].reset_index(drop=True)

# construccion.to_file(f"{folder}/Construccion_limpio.shp")
```

    c:\Users\JuanBetancur\anaconda3\Lib\site-packages\pyogrio\raw.py:198: UserWarning: Measured (M) geometry types are not supported. Original type 'Measured 3D Point' is converted to 'Point Z'
      return ogr_read(
    

## Limpiar capa de equipamientos


```python
# Cargar shapefile
equipamiento = gpd.read_file("Equipamiento_P.shp")

# Seleccionar y renombrar columnas relevantes
equipamiento = equipamiento[['OBJECTID', "NOMBRE", "path", "geometry"]]

equipamiento.columns = ["OBJECTID", "tipo", "path", "geometry"]

# Remover puntos sin el dato de nombre
equipamiento = equipamiento[equipamiento.tipo.notna()]

# Remover puntos con nombres no inteligibles o en código
filter_equipamiento = [
    True
    if re.sub("-?[0-9]?-?", "", nombre)
    and len(re.sub("-?[0-9]?-?", "", nombre)) > 2
    and not re.sub("-?[0-9]?-?", "", nombre).startswith("_")
    and not re.sub("-?[0-9]?-?", "", nombre) == "<Null>"
    else False
    for nombre in equipamiento.tipo
]

equipamiento = equipamiento[filter_equipamiento]

# Limpiar nombres de números y guiones
equipamiento.tipo = [
    re.sub("-?[0-9]?-?", "", nombre).strip()
    for nombre in equipamiento.tipo
]

# Seleccionar categorías para limpiar
equipamientos = [  # Mercado,
    "Vivienda", "ConstrucciÃ³n Anexa", "Corral", "Alberca", 'Enramada',
    'Pozo', 'UCA', 'Huerta', 'Torre de mediciÃ³n', 'Jaguey', 'JagÃ¼ey',
    'Iglesia', 'Escuela', 'Enramada', 'Roza', "Laguna", "Bienes y servicios",
    "AlcaldÃ\x83Â\xada", "La Mezquita", "Universidad", "Hospital", "Clínica",
    "Parque", "Cancha", "Complejo deportivo", "Colegio",
    "InstituciÃ\x83Â³n Educativa", "Liceo Moderno", "Aguas de", "Electricaribe",
    "Casa de la cultura", "Corrales", "pozo y molino", "RancherÃ\x83Â\xada",
    "Cisterna", "IE Elemental", "Centro de salud", "alberca"
]

for eq in equipamientos:
    equipamiento.tipo = [
        eq
        if nombre.startswith(eq)
        else nombre
        for nombre in equipamiento.tipo
    ]

# Limpiar
replace_equip = dict(zip(
    ['Vivienda', 'ConstrucciÃ³n Anexa', 'Corral', 'Alberca', 'Enramada', 'Pozo',
     'UCA', 'Huerta', 'Torre de mediciÃ³n', 'Jaguey', 'JagÃ¼ey', 'Iglesia',
     'Escuela', 'Roza', 'Laguna', 'Bienes y servicios', 'AlcaldÃ\x83Â\xada',
     'La Mezquita', 'Universidad', 'Hospital', 'Clinica de Maicao', 'Parque',
     'Cancha', 'Complejo deportivo', 'Colegio', 'InstituciÃ\x83Â³n Educativa',
     'Liceo Moderno', 'Aguas de', 'Electricaribe', 'Casa de la cultura',
     'pozo y molino', 'RancherÃ\x83Â\xada', 'Cisterna', 'IE Elemental',
     'Centro de Salud', 'alberca'],
    ['Vivienda', 'Construcción anexa', 'Corral', 'Alberca', 'Enramada', 'Pozo',
     'UCA', 'Huerta', 'Torre de medición', 'Jagüey', 'Jagüey', 'Iglesia',
     'Escuela', 'Roza', 'Laguna', 'Mercado', 'Alcaldía', 'Mezquita',
     'Universidad', 'Hospital', 'Clínica', 'Parque', 'Espacio deportivo',
     'Espacio deportivo', 'Colegio', 'Colegio', 'Colegio',
     'Oficina servicios públicos', 'Oficina servicios públicos',
     'Casa de cultura', 'Molino', 'Ranchería', 'Cisterna', 'Colegio',
     'Centro de salud', 'Alberca']
))

equipamiento.tipo = [
    replace_equip[eq]
    for eq in equipamiento.tipo
]

# Filtrar tipos de equipamiento deseados
equipamiento = equipamiento[equipamiento.tipo.isin(equipamientos)
                            ].reset_index(drop=True)

# equipamiento.to_file(f"{folder}/Equipamiento_limpio.shp")
```

## Limpiar capa de sitios de interés cultural


```python
# Cargar shapefile
sic = gpd.read_file("SitioInteresCultural_P.shp")

# Seleccionar y renombrar columnas relevantes
sic = sic[['OBJECTID', "NOMBRE", "path", "geometry"]]

sic.columns = ["OBJECTID", "tipo", "path", "geometry"]

# Remover puntos sin el dato de nombre
sic = sic[sic.tipo.notna()]

# Remover puntos con nombres ininteligibles o en código
filter_sic = [
    True
    if re.sub("-?[0-9]?-?", "", nombre)
    and len(re.sub("-?[0-9]?-?", "", nombre)) > 2
    and not re.sub("-?[0-9]?-?", "", nombre).startswith("_")
    and not re.sub("-?[0-9]?-?", "", nombre) == "<Null>"
    else False
    for nombre in sic.tipo
]

sic = sic[filter_sic]

# Limpiar nombres
sic.tipo = [
    re.sub("-?[0-9]?-?", "", nombre).strip()
    for nombre in sic.tipo
]

# Seleccionar tipos de sitio para limpiar
sics = [
    "Cementerio", "Iglesia", "RancherÃ\xada", "Escuela", "Pista de caballos",
    "Molino", "Roza", "Corral", "Jaguey", "JagÃ¼ey", "Centro Capacitacion",
    "La Mezquita", "Casa de la cultura", "cementerio", "Simbolo religioso"
]

for s in sics:
    sic.tipo = [
        s
        if nombre.startswith(s)
        else nombre
        for nombre in sic.tipo
    ]

# Limpiar tipos de sitio
replace_sic = dict(zip(
    ["Cementerio", "Iglesia", "RancherÃ\xada", "Escuela", "Pista de caballos",
     "Molino", "Roza", "Corral", "Jaguey", "JagÃ¼ey", "Centro Capacitacion",
     "La Mezquita", "Casa de la cultura", "cementerio", "Simbolo religioso",
     "Homenaje Carlos Epiayu"],
    ["Cementerio", "Iglesia", "Ranchería", "Escuela", "Pista de caballos",
     "Molino", "Roza", "Corral", "Jagüey", "Jagüey", "Centro de capacitación",
     "Mezquita", "Casa de cultura", "Cementerio", "Símbolo religioso",
     "Homenaje"]
))

sic.tipo = [
    replace_sic[s]
    for s in sic.tipo
]

# sic.to_file(f"{folder}/SitioInteresCultural_limpio.shp")
```

# Limpiar capa de los proyectos faltantes


```python
# Cargar shapefile -- WCEA2 = Windpeshi, Casa Eléctrica,
WCEA2 = gpd.read_file("CapasCombinadas_WCEA2.shp")

# Seleccionar y renombrar columnas relevantes
WCEA2 = WCEA2[["NOMBRE", "path", "geometry"]]

WCEA2.columns = ["tipo", "path", "geometry"]

# Remover puntos sin el dato de nombre
WCEA2 = WCEA2[WCEA2.tipo.notna()]

# Remover puntos con nombres ininteligibles o en código
filter_WCEA2 = [
    True
    if re.sub("-?[0-9]?-?", "", nombre)
    and len(re.sub("-?[0-9]?-?", "", nombre)) > 2
    and not re.sub("-?[0-9]?-?", "", nombre).startswith("_")
    and not re.sub("-?[0-9]?-?", "", nombre) == "<Null>"
    else False
    for nombre in WCEA2.tipo
]

WCEA2 = WCEA2[filter_WCEA2]

# Limpiar nombres
WCEA2.tipo = [
    re.sub("-?[0-9]?-?", "", nombre).strip().title()
    for nombre in WCEA2.tipo
]

WCEA2.loc[571:601, "tipo"] = "Jagüey"

WCEA2.loc[WCEA2.tipo == "Baã±O", "tipo"] = "Baño"

WCEA2.tipo = [
    "Torre de medición"
    if nombre.startswith("Torre")
    else nombre
    for nombre in WCEA2.tipo
]

WCEA2.tipo = [
    "UCA"
    if nombre.startswith("Uca") or nombre.startswith("Unidad Comunitaria")
    else nombre
    for nombre in WCEA2.tipo
]

# Seleccionar tipos de sitio para limpiar
WCEA2s = [
    "Alambique", "Aljibe", "Aula Escolar", "Base Militar", "Cancha Deportiva",
    "Casa", "  Iperrain", "Escuela", "Establecimiento Educativo", "Huerta",
    "Pozo", "Tanque", "Baño", "Torre de medición", "UCA",
    "Cementerio", "Iglesia", "RancherÃ\xada", "Escuela", "Pista de caballos",
    "Molino", "Roza", "Corral", "Jagüey", "Centro Capacitacion",
    "La Mezquita", "Casa de la cultura", "cementerio", "Simbolo religioso"
]

for s in WCEA2s:
    WCEA2.tipo = [
        s
        if nombre.startswith(s)
        else nombre
        for nombre in WCEA2.tipo
    ]

# Seleccionar tipos específicos
WCEA2 = WCEA2[WCEA2.tipo.isin(WCEA2s)
              ].reset_index(drop=True)

for palabra in ['Alambique', 'Aljibe', 'Aula Escolar', 'Base Militar', 'Baño',
                'Cancha Deportiva', 'Casa', 'Establecimiento Educativo',
                'Tanque']:
    WCEA2.tipo = [
        nombre.capitalize()
        if nombre == palabra
        else nombre
        for nombre in WCEA2.tipo
    ]

WCEA2
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>tipo</th>
      <th>path</th>
      <th>geometry</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>Iglesia</td>
      <td>C:\Users\yonai\OneDrive\Escritorio\SEI 2025\GD...</td>
      <td>POINT (5.11e+06 2.9e+06)</td>
    </tr>
    <tr>
      <th>1</th>
      <td>Iglesia</td>
      <td>C:\Users\yonai\OneDrive\Escritorio\SEI 2025\GD...</td>
      <td>POINT (5.11e+06 2.9e+06)</td>
    </tr>
    <tr>
      <th>2</th>
      <td>Iglesia</td>
      <td>C:\Users\yonai\OneDrive\Escritorio\SEI 2025\GD...</td>
      <td>POINT (5.1e+06 2.9e+06)</td>
    </tr>
    <tr>
      <th>3</th>
      <td>Iglesia</td>
      <td>C:\Users\yonai\OneDrive\Escritorio\SEI 2025\GD...</td>
      <td>POINT (5.1e+06 2.9e+06)</td>
    </tr>
    <tr>
      <th>4</th>
      <td>Iglesia</td>
      <td>C:\Users\yonai\OneDrive\Escritorio\SEI 2025\GD...</td>
      <td>POINT (5.11e+06 2.9e+06)</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <th>541</th>
      <td>Cementerio</td>
      <td>C:\Users\yonai\OneDrive\Escritorio\SEI 2025\GD...</td>
      <td>POINT (5.11e+06 2.9e+06)</td>
    </tr>
    <tr>
      <th>542</th>
      <td>Cementerio</td>
      <td>C:\Users\yonai\OneDrive\Escritorio\SEI 2025\GD...</td>
      <td>POINT (5.11e+06 2.9e+06)</td>
    </tr>
    <tr>
      <th>543</th>
      <td>Cementerio</td>
      <td>C:\Users\yonai\OneDrive\Escritorio\SEI 2025\GD...</td>
      <td>POINT (5.11e+06 2.9e+06)</td>
    </tr>
    <tr>
      <th>544</th>
      <td>Cementerio</td>
      <td>C:\Users\yonai\OneDrive\Escritorio\SEI 2025\GD...</td>
      <td>POINT (5.1e+06 2.9e+06)</td>
    </tr>
    <tr>
      <th>545</th>
      <td>Cementerio</td>
      <td>C:\Users\yonai\OneDrive\Escritorio\SEI 2025\GD...</td>
      <td>POINT (5.11e+06 2.9e+06)</td>
    </tr>
  </tbody>
</table>
<p>546 rows × 3 columns</p>
</div>




```python
for i in sorted(WCEA2.tipo.unique()):
    print(i)
```

    Alambique
    Aljibe
    Aula escolar
    Base militar
    Baño
    Cancha deportiva
    Casa
    Cementerio
    Corral
    Escuela
    Establecimiento educativo
    Huerta
    Iglesia
    Jagüey
    Molino
    Pozo
    Roza
    Tanque
    Torre de medición
    UCA
    

## Análisis de coincidencia de tipos de sitio


```python
# Identificar cuáles son los tipos únicos en cada una de las capas
tipos_construccion: set = set(sorted(construccion.tipo.unique()))

tipos_equipamiento: set = set(sorted(equipamiento.tipo.unique()))

tipos_sic: set = set(sorted(sic.tipo.unique()))

tipos_faltantes: set = set(sorted(WCEA2.tipo.unique()))

# Calcular las intersecciones (elementos en común) entre los tipos de lugar en
# cada capa
print(f"""
A. Tipos en construccion ({len(tipos_construccion)}): {tipos_construccion}    
B. Tipos en equipamiento ({len(tipos_equipamiento)}): {tipos_equipamiento}
C. Tipos sitios IC ({len(tipos_sic)}): {tipos_sic}
      
AnB = {sorted(tipos_construccion.intersection(tipos_equipamiento))}
AnC = {sorted(tipos_construccion.intersection(tipos_sic))}
BnC = {sorted(tipos_equipamiento.intersection(tipos_sic))}
      
AnBnC = {sorted(
    tipos_construccion.intersection(tipos_equipamiento).intersection(tipos_sic)
)}
                
AuBuC = {sorted(
    tipos_construccion.union(tipos_equipamiento).union(tipos_sic)
)}
n AuBuC = {len(
    tipos_construccion.union(tipos_equipamiento).union(tipos_sic)
)}

AuBuCuF = {sorted(
    tipos_construccion.union(tipos_equipamiento).union(tipos_sic)
    .union(tipos_faltantes)
)}
n AuBuCuF = {len(
    tipos_construccion.union(tipos_equipamiento).union(tipos_sic)
    .union(tipos_faltantes)
)}
nuevos tipos: {sorted(
    tipos_faltantes
    .difference(tipos_construccion.union(tipos_equipamiento).union(tipos_sic))
)}

""")
```

    
    A. Tipos en construccion (16): {'Corral', 'Enramada', 'Huerta', 'Escuela', 'Hogar', 'Pista de caballos', 'Iglesia', 'Habitable', 'Molino', 'Roza', 'Santana', 'Pozo', 'Cocina', 'Torre de medición', 'Cementerio', 'Alberca'}    
    B. Tipos en equipamiento (18): {'Corral', 'Enramada', 'Colegio', 'Centro de salud', 'Huerta', 'Escuela', 'Cisterna', 'Laguna', 'Universidad', 'Vivienda', 'Iglesia', 'Clínica', 'Hospital', 'Parque', 'Roza', 'UCA', 'Pozo', 'Alberca'}
    C. Tipos sitios IC (14): {'Corral', 'Símbolo religioso', 'Centro de capacitación', 'Casa de cultura', 'Escuela', 'Pista de caballos', 'Homenaje', 'Iglesia', 'Mezquita', 'Molino', 'Roza', 'Ranchería', 'Jagüey', 'Cementerio'}
          
    AnB = ['Alberca', 'Corral', 'Enramada', 'Escuela', 'Huerta', 'Iglesia', 'Pozo', 'Roza']
    AnC = ['Cementerio', 'Corral', 'Escuela', 'Iglesia', 'Molino', 'Pista de caballos', 'Roza']
    BnC = ['Corral', 'Escuela', 'Iglesia', 'Roza']
          
    AnBnC = ['Corral', 'Escuela', 'Iglesia', 'Roza']
                    
    AuBuC = ['Alberca', 'Casa de cultura', 'Cementerio', 'Centro de capacitación', 'Centro de salud', 'Cisterna', 'Clínica', 'Cocina', 'Colegio', 'Corral', 'Enramada', 'Escuela', 'Habitable', 'Hogar', 'Homenaje', 'Hospital', 'Huerta', 'Iglesia', 'Jagüey', 'Laguna', 'Mezquita', 'Molino', 'Parque', 'Pista de caballos', 'Pozo', 'Ranchería', 'Roza', 'Santana', 'Símbolo religioso', 'Torre de medición', 'UCA', 'Universidad', 'Vivienda']
    n AuBuC = 33
    
    AuBuCuF = ['Alambique', 'Alberca', 'Aljibe', 'Aula escolar', 'Base militar', 'Baño', 'Cancha deportiva', 'Casa', 'Casa de cultura', 'Cementerio', 'Centro de capacitación', 'Centro de salud', 'Cisterna', 'Clínica', 'Cocina', 'Colegio', 'Corral', 'Enramada', 'Escuela', 'Establecimiento educativo', 'Habitable', 'Hogar', 'Homenaje', 'Hospital', 'Huerta', 'Iglesia', 'Jagüey', 'Laguna', 'Mezquita', 'Molino', 'Parque', 'Pista de caballos', 'Pozo', 'Ranchería', 'Roza', 'Santana', 'Símbolo religioso', 'Tanque', 'Torre de medición', 'UCA', 'Universidad', 'Vivienda']
    n AuBuCuF = 42
    nuevos tipos: ['Alambique', 'Aljibe', 'Aula escolar', 'Base militar', 'Baño', 'Cancha deportiva', 'Casa', 'Establecimiento educativo', 'Tanque']
    
    
    

## Reclasificar los puntos en diferentes capas

### Consolidación en un solo shape


```python
# Usando una tabla de clasificación para identificar cómo distribuir los puntos
# entre varios shapefiles

# Esta tabla se usó sólo una vez y se deja como comentario para:
# 1) dejar registro de la operación.
# 2) evitar reemplazar la tabla, que será cargada más adelante, y que fue
#    clasidficada manualmente.

# clasificacion = pd.DataFrame(
#     {"tipo": list(tipos_construccion.union(tipos_equipamiento).union(tipos_sic)),
#      "clasificacion": ""})


#
# clasificacion.to_excel("Windhub - Tipos de construcción.xlsx", index=False)
```


```python
# Tabla utilizada para clasificar
clasificacion = pd.read_excel("Windhub - Tipos de construcción.xlsx")

# Mostrando un ejemplo de tipo dentro de cada clasificación
clasificacion.groupby("clase").first().reset_index()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>clase</th>
      <th>tipo</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>Base militar</td>
      <td>Base militar</td>
    </tr>
    <tr>
      <th>1</th>
      <td>Educación, salud y cultura</td>
      <td>Aula escolar</td>
    </tr>
    <tr>
      <th>2</th>
      <td>Infraestructura básica y comunitaria</td>
      <td>Alambique</td>
    </tr>
    <tr>
      <th>3</th>
      <td>Infraestructura productiva</td>
      <td>Corral</td>
    </tr>
    <tr>
      <th>4</th>
      <td>Sitios de importancia espiritual</td>
      <td>Cementerio</td>
    </tr>
    <tr>
      <th>5</th>
      <td>Torre de medición</td>
      <td>Torre de medición</td>
    </tr>
    <tr>
      <th>6</th>
      <td>Vivienda y asentamientos</td>
      <td>Enramada</td>
    </tr>
  </tbody>
</table>
</div>




```python
# Consolidar los diferentes shapefiles en uno solo
consolidado = pd.concat([construccion, equipamiento, sic, WCEA2])

# Clasificar el shapefile consolidado según la tabla de clasificación
orden_columnas = ["clase", "tipo", "codigo_uso", "fecha", "geometry"]

consolidado = consolidado.merge(clasificacion, on="tipo")[orden_columnas]

consolidado.head()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>clase</th>
      <th>tipo</th>
      <th>codigo_uso</th>
      <th>fecha</th>
      <th>geometry</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>Infraestructura productiva</td>
      <td>Corral</td>
      <td>4101</td>
      <td>2020/11/30 00:00:00.000</td>
      <td>POINT Z (5.11e+06 2.84e+06 0)</td>
    </tr>
    <tr>
      <th>1</th>
      <td>Infraestructura productiva</td>
      <td>Corral</td>
      <td>4101</td>
      <td>2020/11/30 00:00:00.000</td>
      <td>POINT Z (5.11e+06 2.84e+06 0)</td>
    </tr>
    <tr>
      <th>2</th>
      <td>Infraestructura productiva</td>
      <td>Corral</td>
      <td>4101</td>
      <td>2020/11/30 00:00:00.000</td>
      <td>POINT Z (5.11e+06 2.84e+06 0)</td>
    </tr>
    <tr>
      <th>3</th>
      <td>Infraestructura productiva</td>
      <td>Corral</td>
      <td>4101</td>
      <td>2020/11/30 00:00:00.000</td>
      <td>POINT Z (5.11e+06 2.84e+06 0)</td>
    </tr>
    <tr>
      <th>4</th>
      <td>Infraestructura productiva</td>
      <td>Corral</td>
      <td>4101</td>
      <td>2020/11/30 00:00:00.000</td>
      <td>POINT Z (5.11e+06 2.84e+06 0)</td>
    </tr>
  </tbody>
</table>
</div>



### Redistribución en otros shapes


```python
print(f"Clases: {clasificacion.clase.unique()}")
```

    Clases: ['Base militar' 'Educación, salud y cultura'
     'Infraestructura básica y comunitaria' 'Infraestructura productiva'
     'Sitios de importancia espiritual' 'Torre de medición'
     'Vivienda y asentamientos']
    


```python
# Separar puntos relevantes
base_militar = filtrar_shp(consolidado, "Base militar")
edu_salu_cult = filtrar_shp(consolidado, "Educación, salud y cultura")
infra_basica = filtrar_shp(consolidado, "Infraestructura básica y comunitaria")
infra_productiva = filtrar_shp(consolidado, "Infraestructura productiva")
import_espiritual = filtrar_shp(
    consolidado, "Sitios de importancia espiritual")
torre_medicion = filtrar_shp(consolidado, "Torre de medición")
vivienda = filtrar_shp(consolidado, "Vivienda y asentamientos")

# Identificar puntos duplicados
param_duplicates = {"subset": "geometry", "keep": "first"}

esc_duplicates = edu_salu_cult.duplicated(**param_duplicates)
infra_b_duplicates = infra_basica.duplicated(**param_duplicates)
infra_p_duplicates = infra_productiva.duplicated(**param_duplicates)
espiritual_duplicates = import_espiritual.duplicated(**param_duplicates)
torre_duplicates = torre_medicion.duplicated(**param_duplicates)
vivienda_duplicates = vivienda.duplicated(**param_duplicates)

# Remover puntos duplicados
edu_salu_cult = edu_salu_cult[negate(esc_duplicates)]
infra_basica = infra_basica[negate(infra_b_duplicates)]
infra_productiva = infra_productiva[negate(infra_p_duplicates)]
import_espiritual = import_espiritual[negate(espiritual_duplicates)]
torre_medicion = torre_medicion[negate(torre_duplicates)]
vivienda = vivienda[negate(vivienda_duplicates)]

# Exportar cada clase en un shapefile independiente
shapes = [
    base_militar, edu_salu_cult, infra_basica,
    infra_productiva, import_espiritual, torre_medicion,
    vivienda]
nombre_archivo = [
    "base_militar", "edu_salud_cult", "infra_basica",
    "infra_productiva", "importancia_espiritual", "torre_medición",
    "vivienda"]

for shp, nombre in zip(shapes, nombre_archivo):
    path = f"{folder}/Nuevas/{nombre}.shp"
    shp.to_file(path)
    print(f"Archivo correctamente exportado a {nombre}.shp")
```

    Archivo correctamente exportado a base_militar.shp
    Archivo correctamente exportado a edu_salud_cult.shp
    Archivo correctamente exportado a infra_basica.shp
    Archivo correctamente exportado a infra_productiva.shp
    Archivo correctamente exportado a importancia_espiritual.shp
    Archivo correctamente exportado a torre_medición.shp
    Archivo correctamente exportado a vivienda.shp
    
