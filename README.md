Este repositorio contiene un script de R realizado por Esteban Omar Munguía-Soto diseñado como material didáctico para el análisis de redes de interacción ecológica utilizando el paquete bipartite (Dormann, Gruber & Fründ, 2008). El código está orientado a estudiantes de licenciatura y posgrado con interés en ecología de comunidades, redes mutualistas y ecología de polinizadores.
El análisis emplea matrices de interacción cuantitativas (e.g., plantas–polinizadores, huéspedes–parásitos) para calcular métricas a nivel de red y de especie que describen la estructura y especialización de las interacciones. Entre las métricas abordadas se incluyen conectancia, anidamiento (NODF), modularidad, índice de especialización H₂ y métricas de nivel de especie como d' y grado de conectividad (k). El script incluye la visualización de las redes bipartitas mediante grafos y matrices de interacción.
El material está estructurado de manera progresiva: carga y preparación de datos, construcción de la matriz de interacción, cálculo de índices ecológicos y generación de figuras publicables. Se incluyen comentarios explicativos en cada sección para facilitar la comprensión de los procedimientos estadísticos y ecológicos.

This repository contains an R script developed by Esteban Omar Munguía-Soto, designed as didactic material for the analysis of ecological interaction networks using the bipartite package (Dormann, Gruber & Fründ, 2008). The code is intended for undergraduate and graduate students with an interest in community ecology, mutualistic networks, and pollination ecology.
The analysis employs quantitative interaction matrices (e.g., plant–pollinator, host–parasite) to compute network- and species-level metrics that characterize the structural organization and degree of specialization of biotic interactions. Metrics covered include connectance, nestedness (NODF), modularity, the network-level specialization index H₂, and species-level metrics such as the specialization index d' and linkage level (k). The script further incorporates the visualization of bipartite networks through interaction graphs and interaction matrices.
The material is organized progressively, encompassing the following analytical stages: data loading and pre-processing, construction of the interaction matrix, computation of ecological indices, and generation of publication-quality figures. Explanatory annotations are provided throughout each section to support the comprehension of both the statistical procedures and their ecological interpretation.

Referencias
(Dormann, Gruber & Fründ, 2008. Introducing the bipartite Package: Analysing Ecological Networks. R News, 8(2), 8–11). 

```r

# ============================================================================
# ANÁLISIS DE REDES DE INTERACCIÓN BIPARTITAS
# Autor del script documentado: Esteban O. Munguía-Soto
# Fecha: 10/04/2026
# ============================================================================
# 1. INSTALACIÓN Y CARGA DE PAQUETES
# ============================================================================

# Instalar el paquete (solo necesario la primera vez)
 install.packages("bipartite")

# Cargar el paquete
library(bipartite)

# ============================================================================
# 2. CARGA DE DATOS
# OPCIÓN A: Cargar desde archivo .txt delimitado por tabulaciones
# ------------------------------------------------------------------------
# ARGUMENTOS de read.delim():
#   - file: nombre del archivo (entre comillas)
#   - row.names: número de columna que contiene los nombres de filas (1 = primera columna)
#   - header: TRUE si la primera fila contiene nombres de columnas

 Tabla_1 <- read.delim("file", row.names = 1, header = TRUE)
 Tabla_1 <- as.matrix(Tabla_1)  # Convertir a matriz

# ------------------------------------------------------------------------
# OPCIÓN B: Crear matriz directamente en R
# ARGUMENTOS de matrix():
#   - c(...): vector de valores (números de interacciones)
#   - nrow: número de filas (especies del nivel trófico inferior, ej. plantas)
#   - byrow: TRUE = llenar por filas, FALSE = llenar por columnas
#   - dimnames: lista con nombres de filas y columnas

Tabla_1 <- matrix(
  c(
    26, 0, 0, 0, 15, 4, 17, 8, 4, 27, 22, 0, 23, 0,
    0, 1, 3, 6, 0, 4, 2, 10, 10, 16, 4, 17, 0, 0,
    31, 48, 45, 0, 0, 0, 34, 0, 10, 16, 46, 0, 50, 0,
    35, 18, 66, 28, 0, 6, 52, 9, 9, 55, 0, 16, 30, 51,
    116, 0, 44, 0, 104, 0, 103, 0, 0, 117, 0, 0, 4, 0,
    0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0
  ),
  nrow = 6, byrow = TRUE,
  dimnames = list(
    c("Planta_01", "Planta_02", "Planta_03", "Planta_04", "Planta_05", "Planta_06"),
    c("Polinizador_01","Polinizador_02","Polinizador_03","Polinizador_04",
      "Polinizador_05","Polinizador_06","Polinizador_07","Polinizador_08",
      "Polinizador_09","Polinizador_10","Polinizador_11","Polinizador_12",
      "Polinizador_13","Polinizador_14")
  )
)

# Verificar la estructura de los datos
print("Estructura de la matriz:")
print(Tabla_1)
print(paste("Dimensiones: ", nrow(Tabla_1), " plantas x ", ncol(Tabla_1), " polinizadores"))

# ============================================================================
# 3. VISUALIZACIÓN DE LA MATRIZ DE INTERACCIONES
# ARGUMENTOS de visweb():
#   - type: tipo de ordenamiento ("nested" = anidado, "diagonal" = diagonal)
#   - plotsize: tamaño del gráfico (número positivo)
#   - square: qué representan los cuadrados ("interaction" = interacciones, "b" = binario)
#   - text: qué texto mostrar ("interaction" = número de interacciones, "no" = sin texto)
#   - textcol: color del texto
#   - frame: TRUE = mostrar marco alrededor de la matriz
#   - clear: TRUE = limpiar dispositivo gráfico antes de dibujar
#   - labsize: tamaño de las etiquetas (1 = normal, >1 = más grande)

visweb(Tabla_1,
       type = "nested",           # Ordenar según anidamiento
       plotsize = 16,             # Tamaño del gráfico
       square = "interaction",    # Cuadrados muestran interacciones
       text = "interaction",      # Mostrar número de interacciones
       textcol = "red",           # Color del texto
       frame = TRUE,              # Mostrar marco
       clear = TRUE,              # Limpiar gráfico anterior
       labsize = 1.2)             # Tamaño de etiquetas

#Interpretación 
# El gráfico muestra la interacción entre plantas y polinizadores en forma de matriz.
# Las especies con mayor interacción tienen colores más oscuros; las de menor interacción,
# colores más claros

# ============================================================================
# 4. VISUALIZACIÓN DE LA RED BIPARTITA
# ARGUMENTOS de plotweb():
#   - method: método de visualización ("normal" = barras proporcionales, "cca" = análisis CCA)
#   - adj.high: ajuste vertical del nivel superior (plantas) [0-1]
#   - adj.low: ajuste vertical del nivel inferior (polinizadores) [0-1]
#   - text.rot: rotación del texto en grados (90 = vertical, 270 = invertido)
#   - labsize: tamaño de etiquetas
#   - col.interaction: color de las líneas de interacción
#   - col.high: color de las barras del nivel superior
#   - col.low: color de las barras del nivel inferior

plotweb(sortweb(Tabla_1, sort.order = "inc"),  # sortweb ordena la matriz
        method = "normal",          # Método de visualización
        adj.high = 0.8,            # Posición vertical superior
        adj.low = 0.1,             # Posición vertical inferior
        text.rot = 270,            # Rotación del texto
        labsize = 1.03,            # Tamaño de etiquetas
        col.interaction = "gray",  # Color de interacciones
        col.high = "black",        # Color nivel superior (plantas)
        col.low = "black")         # Color nivel inferior (polinizadores)

#Interpretación
# El gráfico muestra la red bipartita. El tamaño de las barras representa la abundancia
# o frecuencia de cada especie (plantas: visitas recibidas; polinizadores: visitas realizadas).
# Las líneas representan las interacciones observadas; su grosor es proporcional al número
# de visitas (líneas más gruesas = mayor interacción)

# ============================================================================
# 5. MÉTRICAS DE RED - NIVEL DE RED (networklevel)
networklevel(Tabla_1)   # Todas las metricas a nivel red disponibles en la libreria bipartite

# Calcular SOLO métricas específicas
metricas_red <- networklevel(Tabla_1, 
                             index = c("NODF", "connectance", "weighted connectance",
                                       "H2", "weighted cluster coefficient",
                                       "modularity", "extinction slope")) # index= nos permite visualizaro solo las metricas de interes

print(metricas_red)

#como:
#ANIDAMIENTO (NODF)
cat(paste("   Valor:", round(metricas_red["NODF"], 3), "\n")) #muestra el valor NODF

#Rango:
#  0 = red completamente no anidada (cada especie interactúa con diferentes especies)
#  100 = red perfectamente anidada (especialistas interactúan con subconjuntos de generalistas)

#Interpretación
#Valores <30 se puede conciderar como una red con bajo anidamiento
#Valores < 60 se puede conciderar como una red con anidamiento moderado
#Valores >60 se puede conciderar como una red con alto anidamiento 

# ============================================================================
# 2. CONECTANCIA
cat(paste("   Valor:", round(metricas_red["connectance"], 3), "\n")) #muestra el valor de conectancia
#Rango:
# 0 - 1
#Interpretación
# 0 = ninguna interacción 
# 1 = todas las especies interactúan entre sí
#Ejemplo
cat(paste("   RESULTADO: ", round(metricas_red["connectance"]*100, 1), "% de interacciones posibles realizadas\n", sep="")) #Nos arroja el porcentaje de conectancia
#RESULTADO: 64.3% de interacciones posibles realizadas

# ============================================================================
# 3. ESPECIALIZACIÓN DE LA RED (H2)
cat(paste("   Valor:", round(metricas_red["H2"], 3), "\n")) #muestra el valor de Especializacion de la red
#Rango:
# 0 - 1
#Interpretación
#Mide cuán especializada es la red completa
# 0 = red generalista (interacciones distribuidas uniformemente)
# 1 = red altamente especializada (cada especie interactúa con pocas especies específicas)
#Valores <0.3 Red generalista
#Valores <0.7 Red con especialización moderada
#Valores >0.7 altamente especializada

# ============================================================================
# 4. MODULARIDAD
cat(paste("   Valor:", round(metricas_red["modularity"], 3), "\n"))
#Rango:
# 0 - 1
#Interpretación
#Mide si la red está dividida en módulos/compartimentos
# 0 = sin estructura modular
# 1 = estructura perfectamente modular
# Valores > 0.4 generalmente indican modularidad significativa
# Valores < 0.3 Sin estructura modular clara

# ============================================================================

# ============================================================================
# 6. MÉTRICAS A NIVEL DE ESPECIE (specieslevel)
specieslevel(Tabla_1)   # Métricas a nivel de especie

# ARGUMENTOS de specieslevel():
#   - web: matriz de interacciones
#   - index: métricas a calcular ("degree", "species strength", "d", "betweenness", etc.)
#   - level: nivel trófico ("both" = ambos, "higher" = superior, "lower" = inferior)

metricas_especies <- specieslevel(Tabla_1, 
                                  index = c("degree", "species strength", "d", "betweenness",
                                            "species.specificity.index", "closeness"),
                                  level = "both") #Selecciona algunas metricas a nivel de especies

# degree: número de especies con las que interactúa
# species strength: suma ponderada de dependencias mutuas
# Mostrar métricas de plantas (nivel superior)
#d': especialización de la especie (0=generalista, 1=especialista)
#closeness: qué tan conectada está una especie con el resto de la red


# ============================================================================
# 7. ANÁLISIS NÚCLEO-PERIFERIA
# ============================================================================
# Referencia: Dáttilo W., Guimaraes P.R., Izzo T.J. 2013. 
# Spatial structure of ant-plant mutualistic networks. Oikos 122: 1643-1648.
# Función para calcular posición núcleo-periferia del nivel SUPERIOR (plantas)
# ------------------------------------------------------------------------
# EXPLICACIÓN DEL ALGORITMO:
#   1. Calcula el grado (degree) de cada especie normalizado por el total de especies del otro nivel
#   2. Estandariza usando z-score: (valor - media) / desviación estándar
#   3. Valores positivos = núcleo, valores negativos = periferia
#   4. Ordena de mayor a menor

nucleohigher <- function(x) {
  # ARGUMENTOS:
  #   x: matriz de interacciones (filas = nivel superior, columnas = nivel inferior)
  
  # Calcular grado normalizado
  kh <- specieslevel(x, index = "degree", level = "higher") / nrow(x)
  
  # Estandarizar
  kmean <- mean(kh[,1])
  ksd <- sd(kh[,1])
  nucleo <- (kh[,1] - kmean) / ksd
  
  # Retornar data frame ordenado
  data.frame(especies = rownames(kh), nucleo)[order(nucleo, decreasing = TRUE), ]
}

# Función para calcular posición núcleo-periferia del nivel INFERIOR (polinizadores)
nucleolower <- function(x) {
  # ARGUMENTOS:
  #   x: matriz de interacciones
  
  kl <- specieslevel(x, index = "degree", level = "lower") / ncol(x)
  kmean <- mean(kl[,1])
  ksd <- sd(kl[,1])
  nucleo <- (kl[,1] - kmean) / ksd
  data.frame(especies = rownames(kl), nucleo)[order(nucleo, decreasing = TRUE), ]
}

# Calcular núcleo-periferia PLANTAS (nivel superior)
nucleo_plantas <- nucleohigher(Tabla_1)
print(nucleo_plantas)

# Calcular núcleo-periferia POLINIZADORES (nivel inferior)
nucleo_polinizadores <- nucleolower(Tabla_1)
print(nucleo_polinizadores)

#Interpretación
# Valores > 0: especies del NÚCLEO (más conectadas que el promedio)
# Valores < 0: especies de la PERIFERIA (menos conectadas que el promedio)
```
# Valores > 0: especies del NÚCLEO (más conectadas que el promedio)
# Valores < 0: especies de la PERIFERIA (menos conectadas que el promedio)
