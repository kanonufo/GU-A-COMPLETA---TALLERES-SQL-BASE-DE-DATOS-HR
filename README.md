# 📘 GUÍA COMPLETA - TALLERES SQL BASE DE DATOS HR
## Preparación para exposición - Preguntas y respuestas del profesor

```
Autor: Juan Pablo Barrero, Pawel Antonio Lobo, Jhonatan Giraldo
Base de datos: HR (MySQL 8.0.46)
Puerto: 3307 | Usuario: root | Password: 9312
```

---

# 📋 ÍNDICE

1. [ESTRUCTURA DE LA BASE DE DATOS HR](#1-estructura-de-la-base-de-datos-hr)
2. [TALLER 13 - JOINS Y OPERADORES DE CONJUNTOS](#2-taller-13-joins-y-operadores-de-conjuntos)
3. [TALLER 14 - SUBCONSULTAS, TABLAS DERIVADAS Y CTE](#3-taller-14-subconsultas-tablas-derivadas-y-cte)
4. [TALLER 15 Y 16 - OPTIMIZACIÓN DE CONSULTAS](#4-taller-15-y-16-optimización-de-consultas)
5. [POSIBLES PREGUNTAS DEL PROFESOR](#5-posibles-preguntas-del-profesor)
6. [COMANDOS RÁPIDOS PARA LA EXPOSICIÓN](#6-comandos-rápidos-para-la-exposición)

---

# 1. ESTRUCTURA DE LA BASE DE DATOS HR

## 1.1 Diagrama del Modelo Relacional

```
┌──────────────┐       ┌──────────────┐
│   REGIONS    │1───N→│  COUNTRIES   │
│──────────────│       │──────────────│
│ PK region_id │       │ PK country_id│
│   region_name│       │   country_name│
└──────────────┘       │ FK region_id │
                       └──────┬───────┘
                              │1
                              │
                              │N
                       ┌──────┴───────┐
                       │  LOCATIONS   │
                       │──────────────│
                       │ PK location_id│
                       │   city       │
                       │ FK country_id│
                       └──────┬───────┘
                              │1
                              │
                              │N
                       ┌──────┴───────┐
          ┌────────────│ DEPARTMENTS  │
          │            │──────────────│
          │            │ PK department_id│
          │            │   department_name│
          │            │ FK location_id│
          │            │ FK manager_id│──┐
          │            └──────┬───────┘  │
          │                   │1         │
          │                   │          │
          │                   │N         │
          │            ┌──────┴───────┐  │
          │            │  EMPLOYEES   │  │
          │            │──────────────│  │
          │            │ PK employee_id│ │
          │            │   first_name │  │
          │            │   last_name  │  │
          │            │   email      │  │
          │            │   salary     │  │
          │            │ FK job_id    │  │
          │            │ FK dept_id   │  │
          │            │ FK manager_id│──┘ (Auto-referencia)
          │            └──────┬───────┘
          │                   │1
          │                   │
          │                   │N
          │            ┌──────┴───────┐
          │            │ JOB_HISTORY  │
          │            │──────────────│
          │            │ PK emp_id    │
          │            │ PK start_date│
          │            │   end_date   │
          │            │ FK job_id    │
          │            │ FK dept_id   │
          └────────────┘──────────────┘

     ┌──────────────┐
     │     JOBS     │──1──┐
     │──────────────│     │N
     │ PK job_id    │     │
     │   job_title  │     │
     │   min_salary │     │
     │   max_salary │     │
     └──────────────┘     │
                          │N
                     ┌────┴────┐
                     │ EMPLOYEES│
                     │ JOB_HIST │
                     └─────────┘
```

## 1.2 Tablas y sus relaciones

| Tabla | PK | FK | Descripción |
|-------|----|----|-------------|
| **regions** | region_id | - | Regiones geográficas (Europe, Americas, etc.) |
| **countries** | country_id | region_id → regions | Países por región |
| **locations** | location_id | country_id → countries | Ubicaciones/direcciones |
| **departments** | department_id | location_id → locations, manager_id → employees | Departamentos de la empresa |
| **jobs** | job_id | - | Cargos/trabajos con rango salarial |
| **employees** | employee_id | job_id → jobs, department_id → departments, manager_id → employees (self) | Empleados |
| **job_history** | (employee_id, start_date) | employee_id → employees, job_id → jobs, department_id → departments | Historial laboral |

## 1.3 Datos cargados

```
regions:     4  (Europe, Americas, Asia, Middle East and Africa)
countries:  25  (US, UK, IT, JP, CA, etc.)
locations:  23  (Seattle, Toronto, London, Tokyo, etc.)
departments:27  (Executive, IT, Sales, Shipping, etc.)
jobs:       19  (President, Programmer, Sales Manager, etc.)
employees: 107  (Steven King, Neena Kochhar, etc.)
job_history: 0  (vacía, para futuros cambios de cargo)
```

### Pregunta posible del profesor:
**¿Por qué job_history está vacía?**

> Porque el trigger `update_job_history` solo se activa cuando un empleado cambia de departamento o cargo, y aún no hemos realizado cambios desde que se insertaron los datos iniciales.

---

# 2. TALLER 13 - JOINS Y OPERADORES DE CONJUNTOS

## 2.1 INNER JOIN (Actividad 3)

### Consulta 3.1 - Empleados y nombre del departamento
```sql
SELECT e.employee_id,
       CONCAT(e.first_name, ' ', e.last_name) AS empleado,
       d.department_name
FROM employees e
INNER JOIN departments d ON e.department_id = d.department_id
ORDER BY e.employee_id;
```

**🔍 ¿Qué hace?**
Muestra cada empleado con el nombre del departamento al que pertenece.

**❓ Posible pregunta: ¿Qué pasa si un empleado no tiene departamento?**
> No aparecería en el resultado, porque INNER JOIN solo muestra registros que coinciden en AMBAS tablas.

**❓ ¿Cuántos registros devuelve?**
> 106 filas (todos los empleados que tienen department_id no nulo).

**❓ ¿Por qué se usa `CONCAT()`?**
> Para unir el nombre y apellido en una sola columna más legible.

**❓ ¿Qué significa el alias `e` y `d`?**
> Son alias para no tener que escribir el nombre completo de la tabla `employees` cada vez.

---

### Consulta 3.2 - Empleados y cargo asignado
```sql
SELECT e.employee_id,
       CONCAT(e.first_name, ' ', e.last_name) AS empleado,
       j.job_title
FROM employees e
INNER JOIN jobs j ON e.job_id = j.job_id
ORDER BY e.employee_id;
```

**🔍 ¿Qué hace?**
Vincula cada empleado con su cargo (job_title).

**❓ ¿Qué relación hay entre employees y jobs?**
> Es 1:N - Un cargo puede tener muchos empleados, pero un empleado tiene solo un cargo actual.

---

### Consulta 3.3 - Empleados y ciudad donde trabajan
```sql
SELECT e.employee_id,
       CONCAT(e.first_name, ' ', e.last_name) AS empleado,
       l.city
FROM employees e
INNER JOIN departments d ON e.department_id = d.department_id
INNER JOIN locations l ON d.location_id = l.location_id
ORDER BY e.employee_id;
```

**🔍 ¿Qué hace?**
Muestra la ciudad donde trabaja cada empleado, recorriendo la cadena: employees → departments → locations.

**❓ ¿Cuántos JOINs se necesitan para llegar a city?**
> Dos JOINs: employees → departments y departments → locations.

---

### Consulta 3.4 - Empleados, país y región
```sql
SELECT e.employee_id,
       CONCAT(e.first_name, ' ', e.last_name) AS empleado,
       c.country_name, r.region_name
FROM employees e
INNER JOIN departments d ON e.department_id = d.department_id
INNER JOIN locations l ON d.location_id = l.location_id
INNER JOIN countries c ON l.country_id = c.country_id
INNER JOIN regions r ON c.region_id = r.region_id
ORDER BY e.employee_id;
```

**❓ Posible pregunta: ¿Cuántas tablas se están uniendo?**
> 5 tablas: employees, departments, locations, countries y regions.

**❓ ¿Cuál es el camino lógico de las relaciones?**
> employees → departments → locations → countries → regions

---

### Consulta 3.5 - Historial laboral de empleados
```sql
SELECT CONCAT(e.first_name, ' ', e.last_name) AS empleado,
       h.start_date, h.end_date,
       j.job_title, d.department_name
FROM job_history h
INNER JOIN employees e ON h.employee_id = e.employee_id
INNER JOIN jobs j ON h.job_id = j.job_id
INNER JOIN departments d ON h.department_id = d.department_id
ORDER BY e.employee_id, h.start_date;
```

**❓ Posible pregunta: ¿Por qué no devuelve resultados?**
> Porque `job_history` está vacía. No hay cambios de cargo registrados aún.

---

## 2.2 LEFT JOIN (Actividad 5)

### Consulta 5.1 - Departamentos sin empleados
```sql
SELECT d.department_id, d.department_name
FROM departments d
LEFT JOIN employees e ON d.department_id = e.department_id
WHERE e.employee_id IS NULL
ORDER BY d.department_id;
```

**🔍 ¿Qué hace?**
Encuentra departamentos que NO tienen ningún empleado asignado.

**❓ ¿Por qué LEFT JOIN y no INNER JOIN?**
> Porque queremos TODOS los departamentos (incluyendo los que no tienen empleados), no solo los que tienen. Con LEFT JOIN, los departamentos sin empleados aparecen con NULL en las columnas de employees, y el WHERE e.employee_id IS NULL los filtra.

**❓ ¿Cuántos departamentos no tienen empleados?**
> 16 departamentos (Treasury, Corporate Tax, Control And Credit, etc.)

---

### Consulta 5.2 - Empleados sin historial laboral
```sql
SELECT e.employee_id,
       CONCAT(e.first_name, ' ', e.last_name) AS empleado
FROM employees e
LEFT JOIN job_history h ON e.employee_id = h.employee_id
WHERE h.employee_id IS NULL
ORDER BY e.employee_id;
```

**❓ ¿Qué devuelve esta consulta?**
> Todos los empleados que nunca han cambiado de cargo (no tienen registros en job_history).

---

## 2.3 RIGHT JOIN (Actividad 6)

### Consulta Complementaria B
```sql
SELECT e.employee_id,
       CONCAT(e.first_name, ' ', e.last_name) AS empleado,
       d.department_name
FROM employees e
RIGHT JOIN departments d ON e.department_id = d.department_id
ORDER BY d.department_id, e.employee_id;
```

**🔍 ¿Qué hace?**
Muestra TODOS los departamentos, incluso aquellos sin empleados (salen con NULL).

**❓ Diferencia entre LEFT JOIN y RIGHT JOIN:**
> LEFT JOIN conserva TODAS las filas de la tabla IZQUIERDA.
> RIGHT JOIN conserva TODAS las filas de la tabla DERECHA.
> 
> En este caso: `employees RIGHT JOIN departments` = `departments LEFT JOIN employees`
> Son equivalentes, solo cambia el orden de las tablas.

---

## 2.4 SELF JOIN (Actividad 8) - Relación Empleado-Gerente

### Consulta 8.1 - Empleados y sus gerentes
```sql
SELECT CONCAT(e.first_name, ' ', e.last_name) AS empleado,
       CONCAT(m.first_name, ' ', m.last_name) AS gerente
FROM employees e
LEFT JOIN employees m ON e.manager_id = m.employee_id
ORDER BY gerente, empleado;
```

**🔍 ¿Qué hace?**
Muestra cada empleado con su respectivo jefe.

**❓ ¿Qué es un SELF JOIN?**
> Es un JOIN de una tabla consigo misma. Aquí, `employees` se une con `employees` para relacionar a cada empleado con su manager. Usamos alias diferentes (`e` y `m`) para distinguir los dos roles.

**❓ ¿Por qué se usa LEFT JOIN y no INNER JOIN?**
> Porque Steven King (employee_id=100) no tiene manager (manager_id=NULL). Con LEFT JOIN, él aparece con gerente=NULL. Con INNER JOIN, desaparecería.

---

### Consulta 8.3 - Cantidad de empleados por gerente
```sql
SELECT CONCAT(m.first_name, ' ', m.last_name) AS gerente,
       COUNT(e.employee_id) AS cantidad_empleados
FROM employees m
LEFT JOIN employees e ON e.manager_id = m.employee_id
GROUP BY m.employee_id, m.first_name, m.last_name
HAVING COUNT(e.employee_id) > 0
ORDER BY cantidad_empleados DESC, gerente;
```

**❓ Posible pregunta: ¿Quién tiene más empleados a cargo?**
> Steven King es el gerente con más empleados directos.

---

## 2.5 CROSS JOIN (Actividad 9)

```sql
SELECT j.job_title, d.department_name
FROM jobs j
CROSS JOIN departments d
ORDER BY j.job_title, d.department_name;
```

**🔍 ¿Qué hace?**
Producto cartesiano: cada cargo se combina con CADA departamento.

**❓ ¿Cuántos registros genera?**
> 19 jobs × 27 departments = **513 registros**

**❓ ¿Para qué sirve un CROSS JOIN?**
> Para generar combinaciones, como: todas las combinaciones posibles de turnos con empleados, o matrices de compatibilidad.

---

## 2.6 UNION y UNION ALL (Actividad 10)

### Consulta 10.1
```sql
SELECT CONCAT(first_name, ' ', last_name) AS nombre
FROM employees
UNION
SELECT department_name AS nombre
FROM departments;
```

**🔍 ¿Qué hace?**
Combina los nombres de empleados y departamentos en una sola lista, eliminando duplicados.

**❓ Diferencia entre UNION y UNION ALL:**
> UNION elimina duplicados (más lento, hace ordenamiento interno).
> UNION ALL conserva duplicados (más rápido).

---

## 2.7 INTERSECT emulado (Actividad 11)

### Consulta 11.2 - Empleados en ambas tablas
```sql
SELECT DISTINCT CONCAT(e.first_name, ' ', e.last_name) AS empleado
FROM employees e
INNER JOIN job_history h ON e.employee_id = h.employee_id;
```

**🔍 ¿Qué hace?**
Encuentra empleados que están TANTO en employees COMO en job_history (intersección).

**❓ ¿Por qué no se usa INTERSECT directamente?**
> MySQL/MariaDB no soporta INTERSECT de forma nativa. Se emula con INNER JOIN.

---

## 2.8 EXCEPT emulado (Actividad 12)

### Consulta 12.1 - Empleados sin historial laboral
```sql
SELECT e.employee_id,
       CONCAT(e.first_name, ' ', e.last_name) AS empleado
FROM employees e
LEFT JOIN job_history h ON e.employee_id = h.employee_id
WHERE h.employee_id IS NULL
ORDER BY e.employee_id;
```

**🔍 ¿Qué hace?**
Encuentra empleados que están en employees PERO NO en job_history (diferencia de conjuntos).

**❓ ¿Cómo se emula EXCEPT?**
> Con LEFT JOIN + WHERE ... IS NULL. Se trae todos los de la izquierda, y se filtran los que NO tienen correspondencia en la derecha.

---

## 2.9 Consultas Integradoras (Actividad 13)

### 13.1 - Empleado con mayor salario por departamento
```sql
SELECT d.department_id, d.department_name,
       e.employee_id,
       CONCAT(e.first_name, ' ', e.last_name) AS empleado,
       e.salary
FROM employees e
INNER JOIN departments d ON e.department_id = d.department_id
INNER JOIN (
  SELECT department_id, MAX(salary) AS salario_maximo
  FROM employees WHERE department_id IS NOT NULL
  GROUP BY department_id
) ms ON e.department_id = ms.department_id AND e.salary = ms.salario_maximo
ORDER BY d.department_id;
```

**🔍 ¿Qué hace?**
Usa una subconsulta para encontrar el salario máximo por departamento, luego la une con employees para obtener el nombre del empleado que lo gana.

---

# 3. TALLER 14 - SUBCONSULTAS, TABLAS DERIVADAS Y CTE

## 3.1 Subconsultas Simples (Actividad 9)

### Consulta 9.1 - Empleados con salario superior al promedio
```sql
SELECT employee_id, first_name, last_name, salary, department_id
FROM employees
WHERE salary > (SELECT AVG(salary) FROM employees)
ORDER BY salary DESC, employee_id;
```

**🔍 ¿Qué hace?**
Compara el salario de cada empleado contra el promedio general de la empresa.

**❓ ¿Qué es una subconsulta escalar?**
> Es una subconsulta que devuelve UN SOLO valor (una fila, una columna). Aquí, `SELECT AVG(salary) FROM employees` devuelve un número, que se usa en la comparación `salary > X`.

**❓ ¿Cuánto es el salario promedio?**
> Se calcula dinámicamente. En este caso, alrededor de 7,741.59.

**❓ ¿Cuántos empleados ganan por encima del promedio?**
> 51 empleados de 107.

---

### Consulta 9.2 - Empleado con el salario más alto
```sql
SELECT employee_id, first_name, last_name, salary, department_id
FROM employees
WHERE salary = (SELECT MAX(salary) FROM employees);
```

**🔍 ¿Qué hace?**
Encuentra al empleado que más gana.

**❓ ¿Quién es?**
> Steven King, con $24,000 (Presidente).

---

### Consulta 9.5 - Empleados con salario menor al promedio de su departamento
```sql
SELECT e.employee_id, e.first_name, e.last_name, e.salary, e.department_id
FROM employees e
WHERE e.department_id IS NOT NULL
  AND e.salary < (SELECT AVG(e2.salary) FROM employees e2 WHERE e2.department_id = e.department_id)
ORDER BY e.department_id, e.salary, e.employee_id;
```

**❓ ¿Qué es una subconsulta correlacionada?**
> Es una subconsulta que referencia a la consulta externa. Aquí, `e2.department_id = e.department_id` hace que el promedio se calcule PARA CADA departamento por separado. Sin la correlación, tomaría el promedio general.

---

## 3.2 Subconsultas con Operadores (Actividad 10)

### Consulta 10.1 - IN
```sql
SELECT e.employee_id, e.first_name, e.last_name, e.salary, e.department_id
FROM employees e
WHERE e.department_id IN (
  SELECT d.department_id
  FROM departments d
  INNER JOIN locations l ON d.location_id = l.location_id
  INNER JOIN countries c ON l.country_id = c.country_id
  WHERE c.country_id = 'US'
)
ORDER BY e.employee_id;
```

**🔍 ¿Qué hace?**
Encuentra empleados en departamentos ubicados en Estados Unidos.

**❓ ¿Cuándo usar IN vs JOIN?**
> IN es más intuitivo cuando solo necesitas filtrar por una condición de otra tabla. JOIN es mejor cuando necesitas columnas de ambas tablas en el resultado.

---

### Consulta 10.2 - EXISTS
```sql
SELECT d.department_id, d.department_name
FROM departments d
WHERE EXISTS (
  SELECT 1 FROM employees e WHERE e.department_id = d.department_id
)
ORDER BY d.department_id;
```

**🔍 ¿Qué hace?**
Muestra solo departamentos que TIENEN al menos un empleado.

**❓ ¿Diferencia entre IN y EXISTS?**
> EXISTS es más eficiente cuando la subconsulta puede devolver muchas filas, porque se detiene en cuanto encuentra una coincidencia. Además, EXISTS puede hacer subconsultas correlacionadas más complejas.

**❓ ¿Por qué se usa `SELECT 1` en lugar de `SELECT *`?**
> Porque no importa QUÉ columnas se seleccionen, solo importa si existen filas. `SELECT 1` es más eficiente.

---

### Consulta 10.3 - ANY
```sql
SELECT employee_id, first_name, last_name, salary, department_id
FROM employees
WHERE salary > ANY (
  SELECT salary FROM employees WHERE department_id = 60
)
ORDER BY salary DESC, employee_id;
```

**🔍 ¿Qué hace?**
Muestra empleados cuyo salario es mayor que AL MENOS UNO de los salarios del departamento 60 (IT).

**❓ ¿Cómo funciona ANY?**
> `salary > ANY (...)` significa: "salary es mayor que cualquiera de los valores de la lista". Es equivalente a: salary > (SELECT MIN(salary) FROM ...).

---

### Consulta 10.4 - ALL
```sql
SELECT employee_id, first_name, last_name, salary, department_id
FROM employees
WHERE salary > ALL (
  SELECT salary FROM employees WHERE department_id = 60
)
ORDER BY salary DESC, employee_id;
```

**🔍 ¿Qué hace?**
Muestra empleados cuyo salario es mayor que TODOS los salarios del departamento 60.

**❓ ¿Cómo funciona ALL?**
> `salary > ALL (...)` significa: "salary es mayor que TODOS los valores de la lista". Es equivalente a: salary > (SELECT MAX(salary) FROM ...).

---

## 3.3 Subconsultas Correlacionadas (Actividad 11)

### Consulta 11.1 - Mayor salario por departamento
```sql
SELECT e.employee_id, e.first_name, e.last_name, e.salary, e.department_id
FROM employees e
WHERE e.salary = (
  SELECT MAX(e2.salary) FROM employees e2 WHERE e2.department_id = e.department_id
)
ORDER BY e.department_id, e.employee_id;
```

**🔍 Explicación detallada:**
Para CADA empleado (e), la subconsulta calcula el salario máximo de su departamento. Si el salario del empleado es igual a ese máximo, entonces es el que más gana en su departamento.

**Ejemplo visual:**
```
Departamento 60 (IT):
  Alexander Hunold: $9,000 → ¿Es = MAX(9000,6000,4800,4800,4200)? Sí ✅
  Bruce Ernst: $6,000 → ¿Es = MAX(9000,6000,4800,4800,4200)? No ❌
```

---

## 3.4 Tablas Derivadas (Actividad 12)

### Consulta 12.1 - Promedio salarial por ciudad
```sql
SELECT t.city, ROUND(AVG(t.salary), 2) AS salario_promedio
FROM (
  SELECT l.city, e.salary
  FROM employees e
  INNER JOIN departments d ON e.department_id = d.department_id
  INNER JOIN locations l ON d.location_id = l.location_id
) t
GROUP BY t.city
ORDER BY t.city;
```

**🔍 ¿Qué es una tabla derivada?**
> Es una subconsulta en el FROM que actúa como una tabla virtual. Aquí, la subconsulta `t` contiene (city, salary) de todos los empleados, y la consulta principal agrupa por ciudad para calcular el promedio.

**❓ ¿Por qué usarla en vez de agrupar directamente?**
> Porque necesitamos primero "aplanar" los datos (unir las tablas) y luego agruparlos. Es más claro y modular.

---

### Consulta 12.2 - Ranking salarial por departamento
```sql
SELECT t.department_id, t.first_name, t.last_name, t.salary, t.ranking_salarial
FROM (
  SELECT e.department_id, e.first_name, e.last_name, e.salary,
         ROW_NUMBER() OVER (
           PARTITION BY e.department_id
           ORDER BY e.salary DESC, e.employee_id
         ) AS ranking_salarial
  FROM employees e
  WHERE e.department_id IS NOT NULL
) t
ORDER BY t.department_id, t.ranking_salarial;
```

**🔍 Explicación de ROW_NUMBER():**
`ROW_NUMBER()` asigna un número consecutivo a cada fila dentro de cada "partición" (PARTITION BY department_id), ordenadas por salario descendente.

**❓ ¿Qué significa PARTITION BY?**
> Divide los datos en grupos (particiones) según department_id, y reinicia la numeración en cada grupo. Es como "GROUP BY pero para numerar".

---

### Consulta 12.3 - Departamentos con gasto superior al promedio
```sql
SELECT d.department_id, d.department_name, d.gasto_salarial_total
FROM (
  SELECT dept.department_id, dept.department_name,
         SUM(salary) AS gasto_salarial_total
  FROM employees
  INNER JOIN departments dept ON employees.department_id = dept.department_id
  WHERE employees.department_id IS NOT NULL
  GROUP BY dept.department_id, dept.department_name
) d
WHERE d.gasto_salarial_total > (
  SELECT AVG(gasto_salarial_total)
  FROM (
    SELECT SUM(salary) AS gasto_salarial_total
    FROM employees WHERE department_id IS NOT NULL
    GROUP BY department_id
  ) x
)
ORDER BY d.gasto_salarial_total DESC, d.department_id;
```

**🔍 Explicación:**
Esta consulta tiene DOS niveles de tablas derivadas:
1. La tabla `d` calcula el gasto salarial por departamento
2. La tabla `x` calcula el gasto de CADA departamento
3. La subconsulta calcula el PROMEDIO de esos gastos
4. Filtra solo los departamentos cuyo gasto supera ese promedio

**❓ Posible pregunta del profesor: Desglosa esta consulta**
> 1. Primero se agrupan los salarios por departamento (tabla `x`): [90: 58000], [60: 28800], etc.
> 2. Se calcula el promedio de esos totales: AVG(58000, 28800, ...) ≈ 37000
> 3. Se obtienen los departamentos cuyo gasto total > 37000

---

## 3.5 CTE (Common Table Expressions) - Actividad 13

### Consulta 13.3 - Análisis de salarios por región
```sql
WITH salarios_region AS (
  SELECT r.region_id, r.region_name, e.salary
  FROM employees e
  INNER JOIN departments d ON e.department_id = d.department_id
  INNER JOIN locations l ON d.location_id = l.location_id
  INNER JOIN countries c ON l.country_id = c.country_id
  INNER JOIN regions r ON c.region_id = r.region_id
)
SELECT region_id, region_name,
       COUNT(*) AS total_empleados,
       ROUND(AVG(salary), 2) AS salario_promedio,
       SUM(salary) AS gasto_salarial_total
FROM salarios_region
GROUP BY region_id, region_name
ORDER BY gasto_salarial_total DESC, region_name;
```

**🔍 ¿Qué es una CTE?**
> Una CTE (WITH) es como una tabla derivada pero más legible. Se define al inicio con `WITH nombre_cte AS (...)` y se puede referenciar varias veces en la consulta principal.

**❓ Ventaja de CTE sobre tabla derivada:**
> - Más legible (la definición está al inicio)
> - Se puede referenciar MÚLTIPLES veces en la misma consulta
> - Se pueden encadenar: `WITH a AS (...), b AS (...) SELECT ...`

---

## 3.6 Funciones Avanzadas (Actividad 14)

```sql
-- Mayúsculas
SELECT UPPER(first_name) FROM employees;

-- Longitud de texto
SELECT first_name, LENGTH(first_name) FROM employees;

-- Años trabajados
SELECT first_name, last_name, hire_date,
       TIMESTAMPDIFF(YEAR, hire_date, CURDATE()) AS anios_trabajados
FROM employees;
```

---

## 3.7 Informe Gerencial (Actividad 15)

### Top 5 salarios más altos
```sql
SELECT employee_id, CONCAT(first_name, ' ', last_name) AS empleado, salary
FROM employees ORDER BY salary DESC, employee_id LIMIT 5;
```

**❓ Posible pregunta: ¿Cómo funciona LIMIT?**
> LIMIT 5 restringe el resultado a solo las primeras 5 filas. Combinado con ORDER BY DESC, obtenemos los 5 salarios más altos.

---

# 4. TALLER 15 Y 16 - OPTIMIZACIÓN DE CONSULTAS

## 4.1 Actividad 1 - Exploración inicial

```sql
SELECT COUNT(*) AS total_empleados FROM employees;
SELECT ROUND(AVG(salary), 2) AS salario_promedio FROM employees;
SELECT MAX(salary) AS maximo, MIN(salary) AS minimo FROM employees;
SELECT COUNT(*) AS total_departamentos FROM departments;
SELECT COUNT(commission_pct) AS empleados_con_comision FROM employees;
```

**❓ Posible pregunta: ¿Por qué COUNT(commission_pct) cuenta solo 35?**
> Porque COUNT(columna) ignoran los valores NULL. commission_pct es NULL para empleados que no están en ventas. COUNT(*) contaría todos (107).

---

## 4.2 Actividad 2 - Agrupamiento

```sql
-- Empleados por departamento
SELECT department_id, COUNT(*) AS cantidad
FROM employees WHERE department_id IS NOT NULL
GROUP BY department_id;

-- Salario promedio por cargo
SELECT job_id, ROUND(AVG(salary), 2) AS salario_promedio
FROM employees GROUP BY job_id;

-- Departamento con mayor gasto salarial
SELECT d.department_name, SUM(e.salary) AS gasto_salarial
FROM employees e
JOIN departments d ON e.department_id = d.department_id
GROUP BY d.department_name
ORDER BY gasto_salarial DESC
LIMIT 1;
```

**❓ Posible pregunta: ¿Qué departamento gasta más en salarios?**
> Shipping (departamento 50) con gasto total de aproximadamente $147,000.

---

## 4.3 Actividad 4 - EXPLAIN

```sql
EXPLAIN SELECT d.department_name, SUM(e.salary) AS gasto_salarial
FROM employees e
JOIN departments d ON e.department_id = d.department_id
GROUP BY d.department_name
ORDER BY gasto_salarial DESC
LIMIT 1;
```

**🔍 ¿Qué muestra EXPLAIN?**
El plan de ejecución: cómo MySQL va a ejecutar la consulta, qué índices usa, cuántas filas estima examinar, etc.

**❓ Columnas importantes de EXPLAIN:**
| Columna | Significado |
|---------|-------------|
| **type** | Cómo se unen las tablas (ALL = scan completo, ref = usa índice) |
| **possible_keys** | Índices que podría usar |
| **key** | Índice que realmente usa |
| **rows** | Filas estimadas a examinar |
| **Extra** | Información adicional (Using index, Using temporary, etc.) |

**❓ ¿Qué significa "Using temporary"?**
> Que MySQL necesita crear una tabla temporal para ordenar o agrupar. Es ineficiente y se puede optimizar con índices.

---

## 4.4 Actividad 5 - Creación de índices

```sql
CREATE INDEX idx_hire_date ON employees(hire_date);
CREATE INDEX idx_salary ON employees(salary);
CREATE INDEX idx_commission ON employees(commission_pct);
```

**❓ ¿Qué es un índice?**
> Como un índice de un libro: acelera la búsqueda de datos. En lugar de leer toda la tabla, MySQL busca directamente en el índice.

**❓ ¿Cuándo crear un índice?**
> - Columnas usadas en WHERE, JOIN, ORDER BY
> - Columnas con muchos valores distintos (alta cardinalidad)
> - No crear en columnas que se actualizan frecuentemente

**❓ ¿Desventajas de los índices?**
> Ocupan espacio en disco y ralentizan INSERT/UPDATE/DELETE porque deben actualizarse también.

---

## 4.5 Actividad 6 - Vista Materializada Simulada

```sql
-- Crear tabla resumen
CREATE TABLE mv_salarios_depto_ciudad (
  department_name VARCHAR(30) NOT NULL,
  city VARCHAR(30) NOT NULL,
  salario_promedio DECIMAL(10, 2),
  gasto_total DECIMAL(10, 2),
  cantidad_empleados INT,
  ultima_actualizacion TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  PRIMARY KEY (department_name, city)
);

-- Refrescar datos
REPLACE INTO mv_salarios_depto_ciudad (department_name, city, salario_promedio, gasto_total, cantidad_empleados)
SELECT d.department_name, l.city,
  ROUND(AVG(e.salary), 2), ROUND(SUM(e.salary), 2), COUNT(e.employee_id)
FROM employees e
JOIN departments d ON e.department_id = d.department_id
JOIN locations l ON d.location_id = l.location_id
GROUP BY d.department_name, l.city;
```

**🔍 ¿Qué es una vista materializada?**
> MySQL no tiene vistas materializadas nativas, así que se simulan con una tabla física que almacena resultados precalculados. Es más rápida que la consulta original porque los datos ya están agregados.

**❓ Ventajas sobre la consulta original:**
> - No necesita hacer JOINs cada vez
> - No necesita agrupar cada vez
> - Los datos ya están calculados y listos

---

## 4.6 Actividad 7 - Comparación optimizada vs no optimizada

```sql
-- Consulta ORIGINAL (hace JOINs y GROUP BY cada vez)
SELECT d.department_name, l.city,
  ROUND(AVG(e.salary), 2), ROUND(SUM(e.salary), 2), COUNT(e.employee_id)
FROM employees e
JOIN departments d ON e.department_id = d.department_id
JOIN locations l ON d.location_id = l.location_id
GROUP BY d.department_name, l.city;

-- VS Vista materializada (datos ya precalculados)
SELECT * FROM mv_salarios_depto_ciudad ORDER BY department_name, city;
```

**❓ ¿Cuál es más rápida?**
> La vista materializada, porque ya tiene los datos calculados. La consulta original necesita unir 3 tablas y agrupar cada vez que se ejecuta.

---

# 5. POSIBLES PREGUNTAS DEL PROFESOR

## 5.1 Preguntas teóricas generales

### Q1: ¿Cuál es la diferencia entre INNER JOIN y LEFT JOIN?
> **INNER JOIN** devuelve SOLO las filas que coinciden en ambas tablas. Si un empleado no tiene departamento, no aparece.
> **LEFT JOIN** devuelve TODAS las filas de la tabla izquierda, y NULL donde no hay coincidencia en la derecha.

### Q2: ¿Qué es una clave primaria (PK) y una clave foránea (FK)?
> **PK**: Identifica de forma única cada fila. Ej: `employee_id` en employees.
> **FK**: Referencia a la PK de otra tabla. Ej: `department_id` en employees referencia a `department_id` en departments.

### Q3: ¿Qué tipos de relaciones existen en el modelo HR?
> - **1:N**: regions → countries, countries → locations, locations → departments, departments → employees, jobs → employees
> - **N:N**: No hay directamente, pero job_history actúa como tabla puente
> - **Auto-referencia**: employees.manager_id → employees.employee_id (empleado se reporta a otro empleado)

### Q4: ¿Qué es una subconsulta correlacionada?
> Es una subconsulta que depende de la consulta externa. Se evalúa para CADA fila de la consulta principal. Ejemplo: encontrar empleados con salario mayor al promedio de SU departamento.

### Q5: ¿Diferencia entre WHERE y HAVING?
> - **WHERE**: Filtra filas ANTES de agrupar (GROUP BY)
> - **HAVING**: Filtra grupos DESPUÉS de agrupar
> Ej: `WHERE salary > 5000` (filtra empleados) vs `HAVING AVG(salary) > 5000` (filtra departamentos)

### Q6: ¿Qué función cumple GROUP BY?
> Agrupa filas con el mismo valor en las columnas especificadas, permitiendo usar funciones de agregación (COUNT, SUM, AVG, MAX, MIN) sobre cada grupo.

### Q7: ¿Qué es un índice y para qué sirve?
> Estructura de datos que acelera la búsqueda de filas. Como el índice de un libro. Sin índice, MySQL hace un "full table scan" (lee toda la tabla).

### Q8: ¿Qué información da EXPLAIN?
> Muestra el plan de ejecución: qué índices usa la consulta, cuántas filas estima examinar, cómo se unen las tablas, etc.

### Q9: ¿Qué es una CTE y ventajas sobre subconsultas?
> CTE (WITH) es una consulta nombrada que se define al inicio. Ventajas: más legible, se puede reutilizar múltiples veces, se pueden encadenar.

### Q10: ¿Qué es ROW_NUMBER() y cómo funciona?
> Función de ventana que asigna un número secuencial a cada fila dentro de una partición. `PARTITION BY` divide los datos en grupos, y dentro de cada grupo se numera según el `ORDER BY`.

### Q11: ¿Diferencia entre UNION y UNION ALL?
> UNION elimina duplicados, UNION ALL los conserva. UNION es más lento porque necesita ordenar y comparar.

### Q12: ¿Cómo se emula INTERSECT y EXCEPT en MySQL?
> - **INTERSECT**: Con INNER JOIN y DISTINCT
> - **EXCEPT**: Con LEFT JOIN y WHERE ... IS NULL

### Q13: ¿Qué es el producto cartesiano (CROSS JOIN)?
> Combina cada fila de una tabla con cada fila de la otra. Si tabla A tiene 10 filas y tabla B 20, produce 200 filas.

### Q14: ¿Qué hace la función COALESCE?
> Devuelve el primer valor NO NULL de la lista. Ej: `COALESCE(commission_pct, 0)` devuelve 0 si commission_pct es NULL.

### Q15: ¿Para qué sirve TIMESTAMPDIFF?
> Calcula la diferencia entre dos fechas en una unidad específica. Ej: `TIMESTAMPDIFF(YEAR, hire_date, CURDATE())` calcula los años trabajados.

## 5.2 Preguntas prácticas (que te pueden pedir en vivo)

### P: "Muéstrame todos los empleados del departamento IT"
```sql
SELECT e.* FROM employees e
JOIN departments d ON e.department_id = d.department_id
WHERE d.department_name = 'IT';
```

### P: "¿Cuántos empleados hay por cada cargo?"
```sql
SELECT j.job_title, COUNT(*) AS total
FROM employees e
JOIN jobs j ON e.job_id = j.job_id
GROUP BY j.job_title
ORDER BY total DESC;
```

### P: "¿Qué empleados ganan más que su jefe?"
```sql
SELECT CONCAT(e.first_name, ' ', e.last_name) AS empleado, e.salary,
       CONCAT(m.first_name, ' ', m.last_name) AS jefe, m.salary
FROM employees e
JOIN employees m ON e.manager_id = m.employee_id
WHERE e.salary > m.salary;
```

### P: "¿Cuál es el salario promedio por ciudad?"
```sql
SELECT l.city, ROUND(AVG(e.salary), 2) AS promedio
FROM employees e
JOIN departments d ON e.department_id = d.department_id
JOIN locations l ON d.location_id = l.location_id
GROUP BY l.city
ORDER BY promedio DESC;
```

### P: "Agrega un nuevo empleado"
```sql
INSERT INTO employees (employee_id, first_name, last_name, email, phone_number,
                       hire_date, job_id, salary, manager_id, department_id)
VALUES (207, 'Juan', 'Perez', 'JPEREZ', '555.123.4567',
        CURDATE(), 'IT_PROG', 7500, 103, 60);
```

### P: "Crea un índice y muestra cómo mejora la consulta"
```sql
-- Antes del índice
EXPLAIN SELECT * FROM employees WHERE salary > 10000;

CREATE INDEX idx_salary_test ON employees(salary);

-- Después del índice (el type cambiará de ALL a ref o range)
EXPLAIN SELECT * FROM employees WHERE salary > 10000;
```

### P: "Modifica el salario de un empleado"
```sql
UPDATE employees SET salary = 26000 WHERE employee_id = 100;
-- Verificar
SELECT first_name, last_name, salary FROM employees WHERE employee_id = 100;
```

### P: "Elimina un empleado (si no tiene restricciones)"
```sql
DELETE FROM employees WHERE employee_id = 207;
```

---

# 6. COMANDOS RÁPIDOS PARA LA EXPOSICIÓN

## Conectarse a MySQL80
```bash
& "C:\Program Files\MySQL\MySQL Server 8.0\bin\mysql.exe" -u root -p9312 -h localhost -P 3307 hr
```

## Comandos útiles dentro de MySQL

```sql
-- Ver todas las tablas
SHOW TABLES;

-- Estructura de una tabla
DESCRIBE employees;

-- Ver índices
SHOW INDEX FROM employees;

-- Ver las bases de datos
SHOW DATABASES;

-- Seleccionar BD
USE hr;

-- Salir
EXIT;
```

## Si el profesor pide ver los datos rápidamente:

```sql
-- Ver todo de una tabla (limitado)
SELECT * FROM departments;
SELECT * FROM jobs;
SELECT * FROM employees LIMIT 10;

-- Ver cantidad de registros
SELECT 'regions' AS tabla, COUNT(*) FROM regions
UNION ALL SELECT 'countries', COUNT(*) FROM countries
UNION ALL SELECT 'locations', COUNT(*) FROM locations
UNION ALL SELECT 'departments', COUNT(*) FROM departments
UNION ALL SELECT 'jobs', COUNT(*) FROM jobs
UNION ALL SELECT 'employees', COUNT(*) FROM employees;
```

---

## 🎯 RESUMEN FINAL

| Concepto | Explicación breve |
|----------|------------------|
| **PK** | Identifica cada fila (ej: employee_id) |
| **FK** | Conecta tablas (ej: department_id en employees) |
| **INNER JOIN** | Solo filas que coinciden en ambas tablas |
| **LEFT JOIN** | Todas las filas de la izquierda + coincidencias |
| **SELF JOIN** | Una tabla unida consigo misma |
| **CROSS JOIN** | Producto cartesiano (todas las combinaciones) |
| **Subconsulta escalar** | Devuelve un solo valor |
| **Subconsulta correlacionada** | Depende de la consulta externa |
| **CTE** | Consulta nombrada (WITH) más legible |
| **Tabla derivada** | Subconsulta en el FROM |
| **ROW_NUMBER()** | Numera filas dentro de grupos |
| **EXPLAIN** | Muestra el plan de ejecución |
| **Índice** | Acelera búsquedas |
| **Vista materializada** | Datos precalculados en una tabla |

---

> **Documentación generada para la exposición de los Talleres 13, 14, 15 y 16**
> Base de datos HR - MySQL 8.0.46 - Puerto 3307

---

# 📎 ANEXOS

## ANEXO A - TUTORIAL: CÓMO DESARROLLAR CUALQUIER CONSULTA EN 5 PASOS

Si el profesor te pide crear una consulta **nunca antes vista**, sigue esta receta:

### Paso 1: Identificar las tablas
Pregúntate: ¿Qué datos me está pidiendo?

**Ejemplo del profesor:** *"Nombre del empleado, su jefe y la ciudad donde trabaja"*

| Dato necesario | Tabla |
|----------------|-------|
| Nombre del empleado | `employees` (alias `e`) |
| Nombre del jefe | `employees` (alias `m`) - SELF JOIN |
| Ciudad | `locations` (via `departments`) |

### Paso 2: Identificar las relaciones
Busca las FK que conectan las tablas:
```
employees.department_id  →  departments.department_id
departments.location_id  →  locations.location_id
employees.manager_id     →  employees.employee_id (SELF JOIN)
```

### Paso 3: Elegir el tipo de JOIN
| Situación | JOIN |
|-----------|------|
| Solo los que coinciden | `INNER JOIN` |
| Todos aunque no tengan correspondencia | `LEFT JOIN` |
| Es la misma tabla | SELF JOIN (alias diferentes) |

### Paso 4: Armar la consulta (Plantilla Universal)

```sql
SELECT [columnas a mostrar]
FROM [tabla principal]
JOIN [segunda tabla] ON [condición]
JOIN [tercera tabla] ON [condición]
WHERE [filtros]
GROUP BY [agrupación]
HAVING [filtro de grupos]
ORDER BY [orden]
LIMIT [cantidad];
```

### Paso 5: Verificar
- Ejecuta y revisa si el número de filas tiene sentido
- Agrega `LIMIT 10` si hay muchas filas

### Ejemplo práctico completo

> **Profesor:** *"Necesito una consulta que muestre el nombre del empleado, el nombre de su jefe, su salario, y la ciudad donde trabaja, solo para empleados que ganan más de $10,000"*

```sql
SELECT CONCAT(e.first_name, ' ', e.last_name) AS empleado,
       CONCAT(m.first_name, ' ', m.last_name) AS jefe,
       e.salary, l.city
FROM employees e
LEFT JOIN employees m ON e.manager_id = m.employee_id
JOIN departments d ON e.department_id = d.department_id
JOIN locations l ON d.location_id = l.location_id
WHERE e.salary > 10000
ORDER BY e.salary DESC;
```

**Explicación para el profesor:**
1. `employees e` es la tabla principal (los empleados)
2. `employees m` es el SELF JOIN para obtener el jefe
3. `departments d` conecta empleados con ubicaciones
4. `locations l` tiene la ciudad
5. `WHERE e.salary > 10000` filtra solo los que ganan más de 10K
6. `ORDER BY e.salary DESC` ordena de mayor a menor salario

---

## ANEXO B - MIGRACIÓN Y MONTAJE DE LA BASE DE DATOS

### B.1 ¿Qué significa "montar la base de datos"?
Es el proceso de crear la base de datos desde cero: ejecutar los scripts SQL que crean las tablas, insertan los datos y configuran las relaciones.

### B.2 Archivos del modelo HR
Los scripts originales están en `base_datos/modelo_HR/`:

| Archivo | Contenido |
|---------|-----------|
| `1_usuario.sql` | Crea el usuario `hr` y la BD `hr` |
| `2_Tablas.sql` | Crea las 7 tablas con PKs, FKs e índices |
| `3_datos.sql` | Inserta todos los registros |
| `4_otros.sql` | Vistas, procedimientos y triggers |

**⚠️ Orden de ejecución estricto:** `1 → 2 → 3 → 4`

> **Posible pregunta:** *¿Qué pasa si ejecuto 2_Tablas.sql antes de 1_usuario.sql?*
> **Respuesta:** Falla porque la base de datos `hr` no existe todavía.

### B.3 Método 1: Con MySQL CLI (recomendado para exponer)

```sql
mysql> SOURCE ruta/completa/1_usuario.sql;
mysql> SOURCE ruta/completa/2_Tablas.sql;
mysql> SOURCE ruta/completa/3_datos.sql;
mysql> SOURCE ruta/completa/4_otros.sql;
```

O desde PowerShell:
```powershell
& "C:\Program Files\MySQL\MySQL Server 8.0\bin\mysql.exe" -u root -p9312 -h localhost -P 3307 < "ruta\1_usuario.sql"
```

### B.4 Verificar que la BD está montada

```sql
USE hr;
SHOW TABLES;           -- 7 tablas
SELECT COUNT(*) FROM employees;   -- 107
SELECT COUNT(*) FROM departments; -- 27
```

### B.6 Backup de la base de datos

Con `mysqldump`:
```powershell
& "C:\Program Files\MySQL\MySQL Server 8.0\bin\mysqldump.exe" -u root -p9312 -h localhost -P 3307 hr > hr_backup.sql
```

### B.6 Posibles preguntas sobre migración

**Q:** *¿Por qué se usa `SET FOREIGN_KEY_CHECKS=0` en los scripts?*
> **R:** Para desactivar temporalmente la validación de claves foráneas mientras se crean las tablas. Así podemos crear tablas en cualquier orden sin que las FKs bloqueen el proceso. Luego se reactiva con `SET FOREIGN_KEY_CHECKS=1`.

**Q:** *¿Qué problemas encontraron al migrar de MariaDB a MySQL 8.0?*
> **R:** Los scripts originales eran para MariaDB. Tuvimos que:
> - Cambiar el puerto de 3306 (MariaDB) a 3307 (MySQL80)
> - Adaptar la autenticación (MariaDB usaba `auth_gssapi_client`, MySQL80 usa `caching_sha2_password`)
> - Verificar que `STR_TO_DATE()` y las funciones de ventana funcionaran igual

**Q:** *¿Cómo verifican que todos los datos se migraron correctamente?*
> **R:** Comparamos el conteo de registros contra los scripts originales y ejecutamos el SQL_Maestro completo (38 consultas, 0 errores).

---

## ANEXO C - CONEXIÓN CON DBeaver

### C.1 ¿Qué es DBeaver?
Es una herramienta GUI gratuita para administrar bases de datos. Soporta MySQL, MariaDB, PostgreSQL, Oracle, SQL Server, etc.

### C.2 Pasos para conectar DBeaver a MySQL80

| Paso | Acción |
|------|--------|
| 1 | Abrir DBeaver |
| 2 | Click en "New Database Connection" (ícono ⛁ + ➕) o menú `Database → New Database Connection` |
| 3 | Seleccionar **MySQL** de la lista |
| 4 | Llenar los datos: |

```
Host:     localhost
Port:     3307
Database: hr
Username: root
Password: 9312
```

| 5 | Click en **"Test Connection"** (botón inferior izquierdo) |
| 6 | Si pide descargar driver, aceptar |
| 7 | Click **"Finish"** |

### C.3 Qué mostrarle al profesor en DBeaver

#### 1. Navegación (Panel izquierdo)
Expandir: `MySQL → hr → Tables`
Ahí aparecen las 7 tablas del modelo HR.

#### 2. Ver datos de una tabla
Click derecho en `employees` → **"View Data"** → Muestra los 107 empleados en cuadrícula.

#### 3. Ejecutar consultas
Click en **"SQL Editor"** (ícono de hoja + lápiz).
Pegar: `SELECT * FROM departments;`
Presionar: `Ctrl + Enter`

#### 4. Ver diagrama ER
Click derecho en `hr` → **"View Diagram"** → Muestra el diagrama entidad-relación completo con todas las tablas y sus relaciones.

#### 5. Exportar resultados
Ejecutar una consulta → Click derecho en resultado → **"Export"** → Elegir formato (CSV, Excel, HTML, etc.)

### C.4 Ventajas de DBeaver (para mencionar en exposición)

1. **Visual:** No necesitas escribir SQL para ver datos básicos
2. **Diagramas:** Muestra las relaciones entre tablas gráficamente
3. **Exportación:** Puedes sacar los resultados a Excel o CSV
4. **Multi-base:** Un solo programa para MySQL, MariaDB, PostgreSQL
5. **Gratuito:** Open source

---

## ANEXO D - PREGUNTAS AVANZADAS DEL PROFESOR (Y RESPUESTAS)

### D.1 Sobre el diseño de la base de datos

**Q:** *¿Por qué job_history tiene una PK compuesta (employee_id, start_date)?*
> **R:** Porque un empleado puede cambiar de cargo varias veces, pero no puede tener dos cambios en la misma fecha. La combinación empleado+fecha es única.

**Q:** *¿Por qué la tabla departments tiene manager_id si ya employees tiene department_id?*
> **R:** Es una relación bidireccional: un empleado pertenece a un departamento, y un departamento tiene un gerente. El gerente también es un empleado, por eso manager_id es FK hacia employees.

**Q:** *¿Qué pasa si intento eliminar una región que tiene países?*
> **R:** MySQL no lo permite por la FK. Primero debo eliminar los países de esa región, o usar `ON DELETE CASCADE` (que no está configurado aquí).

**Q:** *¿Por qué el salario de Steven King (employee_id=100) tiene manager_id=NULL?*
> **R:** Porque es el presidente (AD_PRES), el máximo cargo. No reporta a nadie.

### D.2 Sobre rendimiento y optimización

**Q:** *¿Cómo se mide si una consulta es lenta?*
> **R:** Con `EXPLAIN` vemos cuántas filas examina (columna `rows`). Si examina más filas de las que devuelve, hay que optimizar con índices.

**Q:** *¿Qué significa "Using temporary" en EXPLAIN?*
> **R:** Que MySQL creó una tabla temporal en memoria o disco para resolver la consulta (típico de `GROUP BY` sin índices). Es ineficiente y se debe evitar.

**Q:** *¿Qué diferencia hay entre clave primaria e índice?*
> **R:** La PK es única, no permite NULL, y automáticamente crea un índice. Un índice puede tener valores duplicados y permitir NULL. Toda PK es un índice, pero no todo índice es PK.

### D.3 Sobre la exposición

**Q:** *¿Por qué usaron MySQL 8.0 y no MariaDB?*
> **R:** El taller pedía específicamente MySQL 8.0 para usar funciones de ventana como `ROW_NUMBER()`.

**Q:** *¿Cuantas veces probaron las consultas?*
> **R:** Ejecutamos el SQL_Maestro completo con 38 consultas, 0 errores. Cada consulta se probó en MySQL CLI y DBeaver.

### D.4 Preguntas TRAMPA (las que más hace el profesor)

**TRAMPA 1:** *"¿Cuántos registros tiene employees?"*
> **R:** 107, pero si cuento con `COUNT(commission_pct)` da 35 porque `COUNT(columna)` ignora NULLs.

**TRAMPA 2:** *"¿Qué pasa si hago SELECT * sin WHERE en una tabla grande?"*
> **R:** Devuelve todas las filas. Si la tabla tuviera millones de registros, podría saturar la memoria y la red.

**TRAMPA 3:** *"Diferencia entre DELETE y TRUNCATE"*
> **R:** `DELETE` elimina fila por fila (lento), respeta triggers y se puede usar con WHERE. `TRUNCATE` elimina todo de golpe (rápido), no respeta triggers, y no acepta WHERE. TRUNCATE además reinicia los AUTO_INCREMENT.

**TRAMPA 4:** *"¿Qué es más rápido, subconsulta o JOIN?"*
> **R:** Generalmente JOIN es más rápido porque el optimizador de MySQL maneja mejor los JOINs que las subconsultas, especialmente las correlacionadas.

**TRAMPA 5:** *"¿Por qué usan LEFT JOIN en vez de RIGHT JOIN?"*
> **R:** Por convención, la mayoría usamos LEFT JOIN porque es más legible (la tabla "principal" va primero). RIGHT JOIN existe pero se usa menos.

**TRAMPA 6:** *"Regla de oro de GROUP BY"*
> **R:** Toda columna en el SELECT que NO sea función de agregación (COUNT, SUM, AVG, MAX, MIN) debe estar en GROUP BY. Si no, MySQL da error o resultados inesperados.

**TRAMPA 7:** *"¿Se puede tener una FK sin una PK?"*
> **R:** No. Una FK siempre referencia a una PK o columna UNIQUE de otra tabla.

**TRAMPA 8:** *"¿Qué es más eficiente, EXISTS o IN?"*
> **R:** `EXISTS` es más eficiente cuando la subconsulta devuelve muchas filas, porque se detiene en la primera coincidencia. `IN` evalúa todas las filas.

**TRAMPA 9:** *"¿Para qué sirve COALESCE?"*
> **R:** Devuelve el primer valor NO NULL de la lista. Ej: `COALESCE(commission_pct, 0)` devuelve 0 cuando no hay comisión, evitando errores en cálculos.

**TRAMPA 10:** *"¿Cuál es la diferencia entre CHAR y VARCHAR?"*
> **R:** `CHAR` es de longitud fija (más rápido, ocupa siempre el mismo espacio). `VARCHAR` es de longitud variable (ahorra espacio). Para el modelo HR, `country_id CHAR(2)` tiene sentido porque siempre son 2 caracteres.

---

## ANEXO E - MINI-TUTORIALES RÁPIDOS

### E.1 "Hazme una consulta que muestre X"

**Plantilla universal** (funciona para casi cualquier consulta):

```sql
SELECT [qué columnas mostrar]
FROM [tabla principal]
JOIN [otra tabla] ON [condición]
WHERE [filtro]
GROUP BY [columna para agrupar]
HAVING [filtro sobre grupos]
ORDER BY [columna para ordenar]
LIMIT [número de filas];
```

Para llenar los corchetes solo pregúntate:
- ¿Qué datos me pide? → las columnas del SELECT
- ¿De dónde vienen? → las tablas del FROM/JOIN
- ¿Cómo se relacionan? → las condiciones del ON
- ¿Qué filtros? → el WHERE
- ¿Necesita agrupar? → GROUP BY

### E.2 "Explícalo paso a paso"

**Ejemplo:** *"Empleados que ganan más que el promedio de su cargo"*

```sql
SELECT employee_id, first_name, last_name, salary, job_id
FROM employees e
WHERE salary > (
    SELECT AVG(e2.salary) FROM employees e2
    WHERE e2.job_id = e.job_id
);
```

**Explicación oral:** *"Primero tomo cada empleado (e). Para cada uno, calculo el salario promedio de todos los empleados que tienen el mismo cargo (e2.job_id = e.job_id). Si su salario es mayor que ese promedio, lo muestro en el resultado."*

### E.3 "Modifica esta consulta para que..."

| Si el profesor pide... | Haces... |
|------------------------|----------|
| "Agrega el nombre del departamento" | Agregas `JOIN departments` |
| "Filtra por salario > 5000" | Agregas `WHERE salary > 5000` |
| "Ordena por apellido" | Agregas `ORDER BY last_name` |
| "Muestra solo los 10 primeros" | Agregas `LIMIT 10` |
| "Cuenta cuántos hay por grupo" | Cambias a `COUNT(*)` y agregas `GROUP BY` |

### E.4 Explicación completa de la BD en 2 minutos

1. *"Esta es la base de datos HR, un modelo empresarial típico de recursos humanos"*
2. *"Tiene 7 tablas: regions, countries, locations, departments, jobs, employees, job_history"*
3. *"La tabla central es employees (107 empleados), que se relaciona con departments (27 departamentos) y jobs (19 cargos)"*
4. *"Cada empleado puede tener un jefe (auto-referencia: manager_id → employee_id)"*
5. *"job_history guarda el historial de cambios de cargo o departamento"*
6. *"Los empleados trabajan en 23 ciudades distribuidas en 25 países y 4 regiones"*

---

## ANEXO F - CHECKLIST PARA LA EXPOSICIÓN

### Antes de empezar
- [ ] Verificar que MySQL80 está corriendo: `Get-Service MySQL80`
- [ ] Conectarse a la BD hr
- [ ] Tener DBeaver abierto y conectado
- [ ] Tener la terminal MySQL abierta y lista
- [ ] Tener esta guía a mano

### Durante la exposición
- [ ] Mostrar las 7 tablas (`SHOW TABLES`)
- [ ] Mostrar cuántos registros tiene cada tabla
- [ ] Explicar las relaciones (PK y FK)
- [ ] Ejecutar 2-3 consultas clave (INNER JOIN, LEFT JOIN, subconsulta)
- [ ] Mostrar `EXPLAIN` de una consulta
- [ ] Mostrar DBeaver conectado con el diagrama ER

### Posibles preguntas preparadas
- [ ] ¿Cuántos empleados hay? (107)
- [ ] ¿Qué departamentos no tienen empleados? (16)
- [ ] ¿Quién gana más? (Steven King, $24,000)
- [ ] ¿Cuál es el salario promedio? (~$7,741)
- [ ] ¿Cuántos empleados por departamento?
- [ ] ¿Qué índices crearon y por qué?
- [ ] ¿Cómo migraron la base de datos?
- [ ] ¿Qué problemas tuvieron?
- [ ] ¿Pueden mostrar la conexión en DBeaver?

