# Algoritmos de Búsqueda

**Autor:** Camilo Tarazona Orozco
**Materia:** Lógica de Programación 2
**Tema:** Tercer corte – Algoritmos de búsqueda

> Notación: `n` = número de elementos. Complejidades expresadas en notación Big-O.

---

## Tabla resumen

| Algoritmo | ¿Requiere datos ordenados? | Mejor caso | Caso promedio | Peor caso | Espacio extra |
|---|---|---|---|---|---|
| Lineal | No | O(1) | O(n) | O(n) | O(1) |
| Binaria | Sí | O(1) | O(log n) | O(log n) | O(1) iterativa / O(log n) recursiva |
| Jump Search | Sí | O(1) | O(√n) | O(√n) | O(1) |
| Interpolation | Sí (y distribución uniforme) | O(1) | O(log log n) | O(n) | O(1) |
| Exponential | Sí | O(1) | O(log i)* | O(log i)* | O(1) |
| BST Balanceado (AVL / Rojo-Negro) | Se mantiene ordenado por estructura | O(1) | O(log n) | O(log n) | O(n) para la estructura |
| Hash Table | No | O(1) | O(1) | O(n)** | O(n) |

\* `i` es la posición del elemento buscado.
\** Con muchas colisiones. En `HashMap` de Java 8+ los buckets largos se convierten en árboles, llevando el peor caso a O(log n).

---

## 1. Búsqueda Lineal (Linear Search)

### Definición
Recorre la colección elemento por elemento, desde el inicio hasta el final, comparando cada uno con el valor buscado. Se detiene al encontrarlo o al terminar la colección. Es el algoritmo de búsqueda más simple y no exige ningún orden previo.

### Eficiencia
- **Tiempo:** mejor caso O(1) (el elemento está primero), promedio y peor caso O(n).
- **Espacio:** O(1).
- Realiza hasta `n` comparaciones; en promedio, `n/2` si el elemento existe.

### Casos de uso
- Colecciones pequeñas, donde el costo de ordenar o construir otra estructura no se justifica.
- Datos **sin ordenar** que se consultan una sola vez.
- Listas enlazadas (no permiten acceso directo por índice).
- Buscar por condiciones complejas (por ejemplo, "primer evento cuya fecha sea posterior a X").

### Ejemplo en Java
```java
public static int busquedaLineal(int[] arr, int objetivo) {
    for (int i = 0; i < arr.length; i++) {
        if (arr[i] == objetivo) {
            return i;
        }
    }
    return -1; // no encontrado
}
```

---

## 2. Búsqueda Binaria (Binary Search)

### Definición
Algoritmo de tipo *divide y vencerás* que trabaja sobre una colección **ordenada**. Compara el valor buscado con el elemento central: si es igual, termina; si es menor, continúa en la mitad izquierda; si es mayor, en la mitad derecha. En cada paso descarta la mitad de los datos.

### Eficiencia
- **Tiempo:** mejor caso O(1), promedio y peor caso O(log n). Para 1.000.000 de elementos, máximo ~20 comparaciones.
- **Espacio:** O(1) en la versión iterativa; O(log n) en la recursiva (pila de llamadas).
- Requiere acceso aleatorio O(1) (arreglos); en listas enlazadas pierde su ventaja.

### Casos de uso
- Buscar en arreglos o listas ordenadas grandes (diccionarios, catálogos, registros ordenados por ID).
- Encontrar límites (primer/último elemento que cumple una condición).
- Búsqueda de raíces o puntos de corte en funciones monótonas ("búsqueda binaria sobre la respuesta").
- Herramientas como `git bisect` para ubicar el commit que introdujo un error.

### Ejemplo en Java
```java
public static int busquedaBinaria(int[] arr, int objetivo) {
    int izq = 0, der = arr.length - 1;
    while (izq <= der) {
        int medio = izq + (der - izq) / 2; // evita desbordamiento
        if (arr[medio] == objetivo) return medio;
        if (arr[medio] < objetivo) izq = medio + 1;
        else der = medio - 1;
    }
    return -1;
}
```

---

## 3. Jump Search

### Definición
Búsqueda sobre una colección **ordenada** que avanza dando "saltos" de tamaño fijo, normalmente `√n`. Cuando el elemento al que salta es mayor o igual al objetivo, retrocede al bloque anterior y hace una búsqueda lineal dentro de ese bloque.

### Eficiencia
- **Tiempo:** O(√n) con el tamaño de salto óptimo `√n`. Es mejor que la lineal, pero peor que la binaria.
- **Espacio:** O(1).
- Ventaja: solo retrocede una vez y en un único bloque.

### Casos de uso
- Sistemas donde **retroceder es costoso** (por ejemplo, lectura secuencial en cinta o almacenamiento donde saltar hacia adelante es barato pero ir hacia atrás no).
- Cuando la búsqueda binaria resulta compleja de implementar o los saltos del "medio" son caros.
- Fines didácticos: muestra un punto medio entre lineal y binaria.

### Ejemplo en Java
```java
public static int jumpSearch(int[] arr, int objetivo) {
    int n = arr.length;
    int salto = (int) Math.sqrt(n);
    int anterior = 0;

    while (anterior < n && arr[Math.min(salto, n) - 1] < objetivo) {
        anterior = salto;
        salto += (int) Math.sqrt(n);
        if (anterior >= n) return -1;
    }
    for (int i = anterior; i < Math.min(salto, n); i++) {
        if (arr[i] == objetivo) return i;
    }
    return -1;
}
```

---

## 4. Interpolation Search (Búsqueda por interpolación)

### Definición
Variante de la búsqueda binaria para datos **ordenados y distribuidos de forma uniforme**. En lugar de ir siempre al medio, **estima la posición** del valor usando una fórmula de interpolación, como cuando buscamos una palabra cerca del final de un diccionario y abrimos directamente por esa zona:

```
pos = izq + ((objetivo - arr[izq]) * (der - izq)) / (arr[der] - arr[izq])
```

### Eficiencia
- **Tiempo:** promedio O(log log n) si los datos son uniformes; peor caso O(n) si la distribución es muy irregular (por ejemplo, crecimiento exponencial).
- **Espacio:** O(1).
- Depende fuertemente de la distribución de los datos.

### Casos de uso
- Grandes conjuntos de datos numéricos ordenados y uniformes (IDs consecutivos, teléfonos, códigos o timestamps espaciados regularmente).
- Búsqueda en bases de datos o índices con claves numéricas distribuidas de forma pareja.
- No recomendable si los datos están muy sesgados o no son numéricos.

### Ejemplo en Java
```java
public static int interpolacion(int[] arr, int objetivo) {
    int izq = 0, der = arr.length - 1;
    while (izq <= der && objetivo >= arr[izq] && objetivo <= arr[der]) {
        if (arr[der] == arr[izq]) {
            return arr[izq] == objetivo ? izq : -1;
        }
        int pos = izq + (int) (((long) (objetivo - arr[izq]) * (der - izq))
                / (arr[der] - arr[izq]));
        if (arr[pos] == objetivo) return pos;
        if (arr[pos] < objetivo) izq = pos + 1;
        else der = pos - 1;
    }
    return -1;
}
```

---

## 5. Exponential Search (Búsqueda exponencial)

### Definición
Trabaja sobre datos **ordenados** en dos fases: (1) **encuentra un rango** donde puede estar el elemento, duplicando el índice (1, 2, 4, 8, 16...) hasta superar el objetivo; (2) aplica **búsqueda binaria** dentro de ese rango.

### Eficiencia
- **Tiempo:** O(log i), donde `i` es la posición del elemento. Si el elemento está cerca del inicio, es más rápida que la binaria clásica.
- **Espacio:** O(1) (con binaria iterativa).
- En el peor caso equivale a O(log n).

### Casos de uso
- Colecciones **de tamaño desconocido o infinito** (flujos de datos, listas "sin límite").
- Cuando se espera que el elemento esté cerca del principio.
- Arreglos muy grandes en los que el objetivo suele aparecer en las primeras posiciones.

### Ejemplo en Java
```java
public static int busquedaExponencial(int[] arr, int objetivo) {
    if (arr.length == 0) return -1;
    if (arr[0] == objetivo) return 0;

    int i = 1;
    while (i < arr.length && arr[i] <= objetivo) {
        i *= 2;
    }
    return java.util.Arrays.binarySearch(arr, i / 2, Math.min(i, arr.length), objetivo);
}
```
> Nota: `Arrays.binarySearch` devuelve un valor negativo si no encuentra el elemento.

---

## 6. BST Balanceado (Árbol Binario de Búsqueda Balanceado)

### Definición
Un **árbol binario de búsqueda (BST)** guarda cada nodo con la regla: *todos los valores del subárbol izquierdo son menores y los del derecho mayores*. Un BST **balanceado** (AVL, Rojo-Negro) mantiene automáticamente su altura cercana a `log n` mediante **rotaciones** tras insertar o eliminar, evitando que degenere en una lista.

### Eficiencia
- **Tiempo:** búsqueda, inserción y eliminación en O(log n), tanto en promedio como en el peor caso.
- **Espacio:** O(n) para almacenar el árbol.
- Un BST **sin balancear** puede degenerar a O(n) si se insertan datos ya ordenados.
- AVL: más estricto (búsquedas más rápidas). Rojo-Negro: menos rotaciones (inserciones/eliminaciones más baratas).

### Casos de uso
- Colecciones **dinámicas** que cambian constantemente y deben mantenerse ordenadas.
- Búsquedas por rango (por ejemplo, "eventos entre dos fechas") y recorridos en orden (inorden).
- Implementaciones de `TreeMap` y `TreeSet` en Java (árbol Rojo-Negro).
- Índices de bases de datos y sistemas de archivos (variantes como B-Tree).
- Encontrar mínimo, máximo, predecesor y sucesor eficientemente.

### Ejemplo en Java (búsqueda en BST)
```java
class Nodo {
    int dato;
    Nodo izquierda, derecha;
    Nodo(int dato) { this.dato = dato; }
}

public static boolean buscar(Nodo raiz, int objetivo) {
    if (raiz == null) return false;
    if (objetivo == raiz.dato) return true;
    return objetivo < raiz.dato
            ? buscar(raiz.izquierda, objetivo)
            : buscar(raiz.derecha, objetivo);
}
```
> En un árbol balanceado, la inserción añade además rotaciones (simples o dobles) para mantener el equilibrio.

---

## 7. Hash Table (Tabla Hash)

### Definición
Estructura que asocia **claves con valores** usando una **función hash**: esta convierte la clave en un índice de un arreglo interno (bucket) donde se almacena el valor. Permite acceder directamente a la posición sin recorrer la colección. Cuando dos claves producen el mismo índice ocurre una **colisión**, que se resuelve con técnicas como *encadenamiento* (lista por bucket) o *direccionamiento abierto* (sondeo lineal, cuadrático, doble hash).

### Eficiencia
- **Tiempo:** búsqueda, inserción y eliminación en O(1) en promedio; O(n) en el peor caso si todas las claves colisionan.
- **Espacio:** O(n), más el espacio de reserva del arreglo.
- Depende de una buena función hash y del **factor de carga** (`elementos / buckets`); cuando es muy alto se hace *rehashing* (se agranda la tabla).
- No mantiene los datos ordenados.

### Casos de uso
- Diccionarios y mapas: `HashMap`, `HashSet`, `Hashtable` en Java.
- Cachés y memoización.
- Contar frecuencias (palabras, votos, ocurrencias).
- Detectar duplicados y verificar pertenencia rápidamente.
- Tablas de símbolos en compiladores, índices de bases de datos, sesiones de usuario.
- **No es adecuada** para búsquedas por rango ni para recorrer en orden.

### Ejemplo en Java
```java
import java.util.HashMap;

HashMap<Integer, String> eventos = new HashMap<>();
eventos.put(1, "Conferencia");
eventos.put(2, "Taller");

String resultado = eventos.get(2);           // O(1) promedio
boolean existe = eventos.containsKey(3);     // false
```

---

## Comparación y criterios de selección

| Situación | Algoritmo recomendado |
|---|---|
| Datos pequeños o sin ordenar, una sola consulta | Lineal |
| Arreglo ordenado, uso general | Binaria |
| Ordenado y retroceder es costoso | Jump Search |
| Ordenado, numérico y uniforme | Interpolation |
| Ordenado, tamaño desconocido o objetivo cerca del inicio | Exponential |
| Datos dinámicos que deben mantenerse ordenados / búsquedas por rango | BST Balanceado |
| Búsqueda por clave lo más rápida posible, sin importar el orden | Hash Table |

---

## Conclusiones

1. No existe un algoritmo "mejor" en todos los casos: la elección depende de si los datos están ordenados, de su distribución, de qué tan dinámicos son y de las operaciones que se necesiten (rango, orden, solo existencia).
2. La búsqueda lineal es la única que no exige preparación previa, pero la más costosa en volumen.
3. Binaria, Jump, Interpolation y Exponential exigen datos ordenados; ordenar tiene un costo (O(n log n)) que solo se compensa si se hacen muchas búsquedas.
4. BST balanceado y Hash Table son **estructuras de datos**, no solo algoritmos: ofrecen búsqueda rápida y además permiten inserciones y eliminaciones eficientes.

## Referencias
- Cormen, T. H., Leiserson, C. E., Rivest, R. L., & Stein, C. *Introduction to Algorithms* (MIT Press).
- Sedgewick, R., & Wayne, K. *Algorithms* (Addison-Wesley).
- Documentación oficial de Java: `java.util.Arrays`, `java.util.HashMap`, `java.util.TreeMap`.
- GeeksforGeeks – Searching Algorithms: https://www.geeksforgeeks.org/searching-algorithms/
