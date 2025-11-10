
# La Isla del Tesoro

**Documento Técnico – Versión 0.1 (Diseño Base)**

----------

## 1. Introducción general

### 1.1. Nombre del proyecto

**La Isla del Tesoro**

### 1.2. Género

-   Puzzle / Estrategia
    
-   Inspirado en **Buscaminas**, con:
    
    -   Progresión narrativa (modo Aventura)
        
    -   Sistema de objetos, trampas y recompensas
        
    -   Ambientación temática pirata
        

### 1.3. Descripción breve

> “La Isla del Tesoro” es un juego de tablero tipo Buscaminas ampliado: el jugador explora zonas de una isla misteriosa revelando casillas, evitando trampas y minas, descubriendo pasadizos secretos y recogiendo objetos útiles. El modo principal (Aventura) presenta una progresión narrativa a través de diferentes zonas de la isla; el modo Supervivencia ofrece un desafío infinito con dificultad creciente.

### 1.4. Plataformas objetivo

-   **Objetivo principal inicial**:
    
    -   PC (Windows)
        
-   **Deseable (sin comprometer la arquitectura)**:
    
    -   Compatibilidad futura con otros sistemas de escritorio (Linux, macOS)
        
-   Todas las decisiones técnicas deben evitar dependencias innecesarias específicas de Windows.
    

### 1.5. Tecnologías

-   **Lenguaje**: Java (Liberica JDK 25)
    
-   **UI / Motor gráfico**: JavaFX 25.0.1
    
-   **Persistencia / Configuración**: JSON mediante **Gson 2.13.2**
    
-   **Arquitectura**:
    
    -   Aplicación de escritorio JavaFX modular (con `module-info.java`)
        
    -   Organización por paquetes: `principal`, `controlador`, `vista`, `modelo`, `util`, `recursos` (detallado más adelante en secciones técnicas)
        

### 1.6. Estado del diseño

Este documento:

-   Recoge **decisiones firmes** acordadas.
    
-   Señala **puntos pendientes de definición fina** (etiquetados como `PENDIENTE`).
    
-   Sirve como **fuente de verdad** para las siguientes fases:
    
    -   Diseño técnico (diagramas, clases)
        
    -   Implementación escalonada sin romper coherencia.
        

----------

## 2. Diseño del juego

### 2.1. Estructura general

El juego se organiza en:

1.  **Modo Aventura (principal)**
    
2.  **Modo Supervivencia**
    
3.  **Sistema de dificultad temático (Día / Atardecer / Noche)**
    
4.  **Sistema de perfiles / guardado**
    
5.  **Capas de narrativa (lore) entre fases**
    

Todo se apoya sobre la misma base jugable:  
**tablero de casillas al estilo Buscaminas**, con variaciones mediante objetos, trampas y reglas especiales.

----------

### 2.2. Modo Aventura

#### 2.2.1. Concepto

-   Modo historia principal, **progresivo y finito**.
    
-   El jugador avanza por diferentes zonas de la isla, cada una representada por un tablero:
    
    -   Ejemplos de fases (orden tentativo): Playa → Jungla → Ruinas → Caverna → Tesoro.
        
-   En cada fase:
    
    -   Se muestra un breve texto de **lore** que contextualiza la situación.
        
    -   Se juega una partida sobre un tablero con reglas coherentes con el entorno.
        
    -   Al completar la fase (condición concreta a definir en Bloque 2: p.ej. encontrar salida / tesoro / objetivos), se pasa a la siguiente.
        

#### 2.2.2. Ambientación en Aventura

-   Aunque el sistema de dificultad visual se base en Día / Atardecer / Noche para el menú:
    
    -   **El modo Aventura tiene su propia progresión natural de luz y tono**:
        
        -   Empieza con ambiente más luminoso (día).
            
        -   Evoluciona hacia entornos más oscuros / peligrosos (noche) en fases avanzadas.
            
-   Esta progresión en Aventura:
    
    -   Es **independiente** de la skin elegida en el menú.
        
    -   Respeta la narrativa (sensación de viaje/descenso hacia el peligro).
        

#### 2.2.3. Relación con la dificultad

-   La **dificultad elegida (Día / Atardecer / Noche)** afecta:
    
    -   Parámetros mecánicos (número de trampas, minas, objetos, escudos iniciales, margen de error).
        
-   Pero en Modo Aventura:
    
    -   El **tono visual in-game** (fases) se guía por la historia, no por el tema del menú.
        
    -   No se fuerza que todo el juego se vea siempre como “noche” solo por elegir la máxima dificultad.
        

----------

### 2.3. Modo Supervivencia

#### 2.3.1. Concepto

-   Modo secundario pero importante.
    
-   **Infinito / por oleadas de tableros**, con dificultad creciente.
    
-   Objetivo: llegar lo más lejos posible, acumulando puntuación.
    

#### 2.3.2. Progresión

-   La dificultad **no depende de la elección inicial Día/Atardecer/Noche.**
    
-   Escala con:
    
    -   Tamaño del tablero.
        
    -   Densidad de minas y trampas.
        
    -   Menos objetos útiles, más trampas complejas.
        
-   La ambientación (visual/musical) puede:
    
    -   Ciclar entre Día → Atardecer → Noche → Día (bucle).
        
    -   Reflejar el aumento de tensión, pero sin romper la claridad visual del tablero.
        

_(Reglas exactas se fijarán en Bloque 2; la arquitectura debe permitir extender tablas de parámetros fácilmente.)_

----------

### 2.4. Dificultad & Ambientaciones (Día / Atardecer / Noche)

#### 2.4.1. Presentación al jugador

-   Nunca se muestra explícitamente “Fácil / Normal / Difícil”.
    
-   Se utiliza una presentación **narrativa y temática**:
    
    -   “¿Recuerdas en qué momento del día despertaste en la isla?”
        
        -   **Día** → Fácil
            
        -   **Atardecer** → Normal
            
        -   **Noche** → Difícil
            

#### 2.4.2. Elección inicial (primera ejecución)

Flujo acordado:

1.  No hay perfil ni partida previa → se muestra introducción narrativa.
    
2.  El juego pide:
    
    -   Nombre del protagonista (ese será el nombre del perfil).
        
    -   Momento del día en que despertó (Día / Atardecer / Noche).
        
3.  En base a esa elección:
    
    -   Se crea un **perfil** con:
        
        -   Dificultad asociada.
            
        -   Tema visual/musical asociado para menús.
            
        -   Parámetros base de juego (ej: escudos iniciales).
            
4.  A partir de ahí:
    
    -   Se muestra el menú principal ya tematizado.
        

#### 2.4.3. Tema visual/musical según dificultad

Aplica a:

-   Menú principal.
    
-   Submenús (Opciones, Créditos, Estadísticas, Perfil).
    

Cambios según elección:

-   **Día**:
    
    -   Colores cálidos y claros.
        
    -   Cielo despejado, mar visible, sensación de aventura inicial.
        
    -   Música más ligera y luminosa.
        
-   **Atardecer**:
    
    -   Tonos dorados/naranjas.
        
    -   Sensación de viaje avanzado, tensión moderada.
        
    -   Música más seria, pero calmada.
        
-   **Noche**:
    
    -   Tonos fríos, azulados/violáceos.
        
    -   Cielo nocturno, bruma, misterio.
        
    -   Música más oscura y tensa.
        

Transiciones:

-   El cambio de dificultad/tema desde el menú:
    
    -   Debe hacerse con **transiciones suaves** (fade de fondo, crossfade musical).
        
    -   Nada de cambios bruscos en seco.
        

#### 2.4.4. Impacto mecánico (resumen inicial)

Ejemplo acordado (aplicable tanto a Aventura como a parámetros iniciales generales):

-   **Día**:
    
    -   -   Escudos iniciales: **2**
            
    -   Menos minas / trampas.
        
-   **Atardecer**:
    
    -   -   Escudos iniciales: **1**
            
    -   Densidad media de minas/trampas.
        
-   **Noche**:
    
    -   -   Escudos iniciales: **0**
            
    -   Mayor castigo, más minas/trampas, menos ayudas.
        

_(Los valores concretos por fase se detallarán en tablas en fases posteriores de diseño.)_

----------

### 2.5. Narrativa (Lore)

Puntos clave:

-   Existe un **marco narrativo cohesivo**:
    
    -   Pirata legendario, mapa oculto, mensaje en una botella, naufragio, despertar en la isla.
        
-   Antes de algunas fases del modo Aventura:
    
    -   Se muestra un breve texto que:
        
        -   Explica **dónde estás**.
            
        -   Por qué esa zona es diferente.
            
        -   Justifica ciertas mecánicas (más trampas, presencia de ruinas, etc.).
            
-   Esta narrativa:
    
    -   No rompe el ritmo (mensajes cortos).
        
    -   Se integra visualmente en la UI temática.
        
    -   Refuerza el tono profesional y cuidado del juego.
        

_(Los textos concretos se definirán en un anexo de lore, separados del código para fácil localización/traducción.)_

----------

## 3. Mecánicas principales

> En esta sección se definen las reglas base del juego. Nada de implementación todavía, solo comportamiento esperado.  
> Si algo queda ambiguo, se marca como `PENDIENTE` para cerrarlo explícitamente en el siguiente bloque.

----------

### 3.1. Núcleo jugable: Buscaminas extendido

-   El jugador interactúa con un **tablero de celdas** usando únicamente el ratón:
    
    -   Click izquierdo → descubrir celda.
        
    -   (Opcional futuro) Click derecho → marcar/bandera (si se considera útil).
        
-   Cada celda puede contener:
    
    -   Nada (vacía).
        
    -   Número (indica peligro cercano).
        
    -   Mina.
        
    -   Trampa especial.
        
    -   Objeto.
        
    -   Elemento de progreso (tesoro, llave, salida, etc.).
        
-   Si se revela una mina/trampa letal **sin escudo disponible**:
    
    -   Fin de la partida (derrota) en la mayoría de modos/fases.
        
-   Si la celda es segura y sin contenido especial:
    
    -   Puede activar la **revelación en cascada** (como Buscaminas clásico) si no hay peligros adyacentes.
        
-   Objetivo por fase:
    
    -   `PENDIENTE`: se definirá para cada tipo de fase:
        
        -   Encontrar X tesoros.
            
        -   Llegar a una salida.
            
        -   Activar un mecanismo.
            
        -   etc.
            
    -   El diseño debe admitir distintas condiciones sin cambiar la base.
        

### 3.2. Sistema de dificultad (impacto en mecánicas)

La elección Día / Atardecer / Noche afecta:

-   Cantidad de minas.
    
-   Cantidad de trampas.
    
-   Frecuencia de objetos útiles.
    
-   Escudos iniciales.
    
-   Margen de error permitido (si existe concepto de “vidas” adicionales además del escudo).
    

Estos parámetros:

-   Estarán centralizados en una configuración por dificultad y modo,
    
-   Para permitir ajuste fino **sin** reescribir lógica.
    

_(Tablas exactas se definirán en la siguiente fase técnica.)_

----------

### 3.3. Sistema de Objetos

#### 3.3.1. Clasificación

Los objetos se dividen en:

1.  **Pasivos automáticos**
    
    -   Se aplican solos al obtenerlos.
        
2.  **Pasivos persistentes**
    
    -   Modifican reglas mientras el jugador los tenga (ej: brújula, mapa).
        
3.  **Activos de un solo uso**
    
    -   Se guardan en inventario.
        
    -   El jugador decide cuándo consumirlos.
        

Sin menús complejos: el inventario debe ser:

-   Claro.
    
-   Minimalista.
    
-   Fáciles de entender en contexto de un puzzle.
    

#### 3.3.2. Escudo (definido y aprobado)

-   Representa una **vida extra**.
    
-   Tipos:
    
    -   Escudo inicial según dificultad.
        
    -   Escudos adicionales encontrados en el tablero.
        
-   Funcionamiento:
    
    -   Si el jugador revela una mina o trampa letal:
        
        -   Si hay ≥1 escudo:
            
            -   No muere.
                
            -   Pierde 1 escudo.
                
            -   Se muestra feedback claro (visual + sonido).
                
        -   Si no hay escudos:
            
            -   Derrota.
                
-   Apilable:
    
    -   Puede tener varios.
        
    -   Límite máximo `PENDIENTE` (se definirá si es necesario equilibrar).
        

#### 3.3.3. Brújula (concepto aprobado)

-   Tipo: **Pasivo persistente**.
    
-   Efecto general (alto nivel, sin números aún):
    
    -   Ayuda a orientar hacia objetivos importantes (tesoro, salida, mecanismo clave…).
        
-   Implementación posible:
    
    -   Indicadores sutiles (UI) o pistas contextuales.
        
-   Importante:
    
    -   No rompe la lectura lógica del tablero.
        
    -   No convierte el juego en trivial.
        
-   Detalle exacto `PENDIENTE` (se definirá en Bloque 2, manteniendo coherencia con Buscaminas).
    

#### 3.3.4. Mapa (concepto aprobado)

-   Tipo: puede ser **activo** o **pasivo según diseño final**, pero:
    
    -   Su idea base es revelar información útil del tablero.
        
-   Posibles comportamientos viables:
    
    -   Revelar un área segura.
        
    -   Marcar probables celdas seguras.
        
    -   Resaltar rutas hacia el objetivo.
        
-   Debe:
    
    -   Mantener la filosofía de deducción, no reemplazarla.
        
-   Implementación exacta `PENDIENTE`, se acotará en la siguiente fase.
    

#### 3.3.5. Otros objetos

Criterios acordados:

-   Solo se incluirán objetos que:
    
    -   Sean comprensibles con la metáfora Buscaminas.
        
    -   No requieran movimiento de personaje.
        
    -   No bloqueen tablero de forma arbitraria e irreversible.
        
-   Cualquier nuevo objeto:
    
    -   Debe documentarse aquí antes de implementarse.
        
    -   No improvisar en código.
        

----------

### 3.4. Trampas

Trampas = variantes de “casillas peligrosas” distintas de la mina clásica.

Reglas generales:

-   Una trampa:
    
    -   Puede ser **letal** (requiere escudo o causa derrota).
        
    -   Puede ser **no letal** (penaliza, pero no mata directamente).
        
-   No se permite:
    
    -   Mecánicas incoherentes con tablero estático (ej: derrumbes que reconfiguren media parrilla de forma aleatoria sin lógica).
        
    -   Bloqueos permanentes que hagan la partida irresoluble sin que el jugador tenga culpa clara.
        

Ejemplos conceptuales (no todos cerrados aún):

-   Minas clásicas → letales.
    
-   Trampas que:
    
    -   Revelan información falsa temporalmente.
        
    -   Penalizan puntos.
        
    -   Activan un mecanismo (bueno o malo).
        
    -   Desgastan un escudo en lugar de matar.
        

Todas las trampas:

-   Deberán estar alineadas con la lectura de números/mecánica lógica del tablero.
    
-   Diseño definitivo `PENDIENTE` para Bloque 2 (se hará limpieza final antes de implementar).
    

----------

### 3.5. Pasadizos secretos y salas seguras

Decisiones tomadas:

-   Los pasadizos **sí existen**, pero:
    
    -   Su activación es **coherente**, no caótica.
        
-   Descubrimiento:
    
    -   Mediante mecanismos visibles en celdas especiales:
        
        -   Ejemplo: “Hay un mecanismo sospechoso. ¿Deseas accionarlo?”
            
    -   O mediante uso de ciertos objetos (ej: llave especial).
        
-   Comportamiento:
    
    -   Si se acciona:
        
        -   Puede llevar a sala segura con recompensas (oro, coleccionables, escudos, pistas).
            
        -   O puede desencadenar un efecto negativo (controlado, diseñado, no absurdo).
            
    -   Si se decide no accionar:
        
        -   Se pierde la oportunidad, no se queda como bug.
            
-   Resolución:
    
    -   Las consecuencias (buenas/malas) son **predefinidas o determinadas por RNG controlado**, pero siempre diseñadas.
        
-   Implementación:
    
    -   Se definirá en detalle una vez cerrado el set de trampas/objetos (Bloque 2 o 3).
