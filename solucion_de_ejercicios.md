## Consultas SQL Especializadas**

1. Como analista, quiero listar todos los productos con su empresa asociada y el precio más bajo por ciudad.

```sql
SELECT 
    cm.name AS ciudad,
    p.name AS producto,
    c.name AS empresa,
    cpp.price AS precio_mas_bajo
FROM companyproduct_prices cpp
JOIN companies c ON cpp.company_id = c.id
JOIN citiesormunicipalities cm ON c.city_id = cm.code
JOIN products p ON cpp.product_id = p.id
JOIN (
    SELECT 
        c2.city_id,
        cpp2.product_id,
        MIN(cpp2.price) AS min_price
    FROM companyproduct_prices cpp2
    JOIN companies c2 ON cpp2.company_id = c2.id
    GROUP BY c2.city_id, cpp2.product_id
) precios_minimos ON precios_minimos.product_id = cpp.product_id
                   AND precios_minimos.city_id = c.city_id
                   AND precios_minimos.min_price = cpp.price
ORDER BY cm.name, p.name, cpp.price;

```

2. Como administrador, deseo obtener el top 5 de clientes que más productos han calificado en los últimos 6 meses.

```sql
SELECT
    c.name AS cliente,
    COUNT(qp.product_id) AS cantidad_calificaciones
FROM quality_products qp
JOIN customers c ON qp.customer_id = c.id
WHERE qp.daterating >= DATE_SUB(CURDATE(), INTERVAL 6 MONTH)
GROUP BY c.id, c.name
ORDER BY cantidad_calificaciones DESC
LIMIT 5;

```

3. Como gerente de ventas, quiero ver la distribución de productos por categoría y unidad de medida.

```sql
SELECT 
    cat.description AS categoria,
    um.description AS unidad_medida,
    COUNT(DISTINCT cp.product_id) AS cantidad_productos
FROM companyproducts cp
JOIN products p ON cp.product_id = p.id
JOIN categories cat ON p.category_id = cat.id
JOIN unitofmeasure um ON cp.unitmeasure_id = um.id
GROUP BY cat.description, um.description
ORDER BY categoria, unidad_medida;

```

4. Como cliente, quiero saber qué productos tienen calificaciones superiores al promedio general.

   ```sql
   SELECT 
       p.name AS producto,
       AVG(qp.rating) AS promedio_producto
   FROM quality_products qp
   JOIN products p ON qp.product_id = p.id
   GROUP BY qp.product_id, p.name
   HAVING AVG(qp.rating) > (
       SELECT AVG(rating) FROM quality_products
   )
   ORDER BY promedio_producto DESC;
   
   ```

5. Como auditor, quiero conocer todas las empresas que no han recibido ninguna calificación.

```sql
SELECT c.id, c.name
FROM companies c
LEFT JOIN quality_products qp ON c.id = qp.company_id
WHERE qp.company_id IS NULL;

```

*Todas las empresas tiene calificaciones: Empty set (0.00 sec)*

6. Como operador, deseo obtener los productos que han sido añadidos como favoritos por más de 10 clientes distintos.

```sql
SELECT 
    p.name AS producto,
    COUNT(DISTINCT f.customer_id) AS cantidad_clientes
FROM details_favorites df
JOIN favorites f ON df.favorite_id = f.id
JOIN products p ON df.product_id = p.id
GROUP BY df.product_id, p.name
HAVING COUNT(DISTINCT f.customer_id) > 10;

```

*Devuelve 0 filas, significa que ningún producto ha sido marcado como favorito por más de 10 clientes distintos hasta ahora.*

7. Como gerente regional, quiero obtener todas las empresas activas por ciudad y categoría.

```sql
SELECT 
    cm.name AS ciudad,
    cat.description AS categoria,
    c.name AS empresa
FROM companies c
JOIN citiesormunicipalities cm ON c.city_id = cm.code
JOIN categories cat ON c.category_id = cat.id
ORDER BY cm.name, cat.description, c.name;

```

8. Como especialista en marketing, deseo obtener los 10 productos más calificados en cada ciudad.

```sql
SELECT 
    ciudad,
    producto,
    empresa,
    promedio_rating
FROM (
    SELECT 
        cm.name AS ciudad,
        p.name AS producto,
        c.name AS empresa,
        AVG(qp.rating) AS promedio_rating
    FROM quality_products qp
    JOIN products p ON qp.product_id = p.id
    JOIN companies c ON qp.company_id = c.id
    JOIN citiesormunicipalities cm ON c.city_id = cm.code
    GROUP BY cm.name, p.name, c.name
) AS sub
ORDER BY ciudad, promedio_rating DESC;

```



9. Como técnico, quiero identificar productos sin unidad de medida asignada.

   ```sql
   SELECT p.id, p.name
   FROM products p
   JOIN companyproducts cp ON p.id = cp.product_id
   WHERE cp.unitmeasure_id IS NULL;
   
   ```

   *Todos los productos tienen unidad de medida asignada, por lo tanto sale vacio*

10. Como gestor de beneficios, deseo ver los planes de membresía sin beneficios registrados.

```sql
SELECT 
    mp.membership_id,
    m.name AS nombre_membresia,
    mp.period_id
FROM membershipperiods mp
JOIN memberships m ON mp.membership_id = m.id
LEFT JOIN membershipbenefits mb ON 
    mp.membership_id = mb.membership_id AND 
    mp.period_id = mb.period_id
WHERE mb.membership_id IS NULL;

```

11. Como supervisor, quiero obtener los productos de una categoría específica con su promedio de calificación.

```sql
SELECT 
    p.name AS producto,
    AVG(qp.rating) AS promedio_calificacion
FROM products p
JOIN quality_products qp ON p.id = qp.product_id
WHERE p.category_id = 3  
GROUP BY p.id, p.name
ORDER BY promedio_calificacion DESC;

```

12. Como asesor, deseo obtener los clientes que han comprado productos de más de una empresa.

```sql
SELECT 
    c.id,
    c.name AS cliente,
    COUNT(DISTINCT qp.company_id) AS empresas_distintas
FROM quality_products qp
JOIN customers c ON qp.customer_id = c.id
GROUP BY c.id, c.name
HAVING COUNT(DISTINCT qp.company_id) > 1
ORDER BY empresas_distintas DESC;

```

13. Como director, quiero identificar las ciudades con más clientes activos.

```sql
SELECT 
    cm.name AS ciudad,
    COUNT(c.id) AS cantidad_clientes
FROM customers c
JOIN citiesormunicipalities cm ON c.city_id = cm.code
GROUP BY cm.name
ORDER BY cantidad_clientes DESC;

```

14. Como analista de calidad, deseo obtener el ranking de productos por empresa basado en la media de `quality_products`.

```sql
SELECT 
    comp.name AS empresa,
    p.name AS producto,
    AVG(qp.rating) AS promedio_rating
FROM quality_products qp
JOIN products p ON qp.product_id = p.id
JOIN companies comp ON qp.company_id = comp.id
GROUP BY comp.id, comp.name, p.id, p.name
ORDER BY comp.name, promedio_rating DESC;

```

15. Como administrador, quiero listar empresas que ofrecen más de cinco productos distintos.

```sql
SELECT 
    c.id,
    c.name AS empresa,
    COUNT(DISTINCT cp.product_id) AS productos_distintos
FROM companyproducts cp
JOIN companies c ON cp.company_id = c.id
GROUP BY c.id, c.name
HAVING COUNT(DISTINCT cp.product_id) > 5
ORDER BY productos_distintos DESC;

```

*Genera empty porque ninguna empresa ofrece mas de 5 productos*

16. Como cliente, deseo visualizar los productos favoritos que aún no han sido calificados.

```sql
SELECT 
    p.id AS producto_id,
    p.name AS producto,
    f.customer_id
FROM details_favorites df
JOIN favorites f ON df.favorite_id = f.id
JOIN products p ON df.product_id = p.id
LEFT JOIN quality_products qp 
    ON qp.product_id = df.product_id 
   AND qp.customer_id = f.customer_id
WHERE qp.product_id IS NULL;

```

17. Como desarrollador, deseo consultar los beneficios asignados a cada audiencia junto con su descripción.

```sql
SELECT 
    a.id AS audiencia_id,
    a.description AS audiencia,
    b.description AS beneficio,
    b.detail AS detalle
FROM audiencebenefits ab
JOIN audiences a ON ab.audience_id = a.id
JOIN benefits b ON ab.benefit_id = b.id
ORDER BY a.description, b.description;

```

18. Como operador logístico, quiero saber en qué ciudades hay empresas sin productos asociados.

```sql
SELECT DISTINCT cm.name AS ciudad
FROM companies c
JOIN citiesormunicipalities cm ON c.city_id = cm.code
LEFT JOIN companyproducts cp ON c.id = cp.company_id
WHERE cp.company_id IS NULL
ORDER BY cm.name;

```

*Genera empty porque las empresas tienen productos asociados*

19. Como técnico, deseo obtener todas las empresas con productos duplicados por nombre.

```sql
SELECT 
    c.id AS empresa_id,
    c.name AS empresa,
    p.name AS producto,
    COUNT(*) AS veces_registrado
FROM companyproducts cp
JOIN products p ON cp.product_id = p.id
JOIN companies c ON cp.company_id = c.id
GROUP BY c.id, p.name
HAVING COUNT(*) > 1
ORDER BY c.name, veces_registrado DESC;

```

20. Como analista, quiero una vista resumen de clientes, productos favoritos y promedio de calificación recibido.

```sql
SELECT 
    c.id AS cliente_id,
    c.name AS cliente,
    p.id AS producto_id,
    p.name AS producto,
    ROUND(AVG(qp.rating), 2) AS promedio_calificacion
FROM favorites f
JOIN customers c ON f.customer_id = c.id
JOIN details_favorites df ON df.favorite_id = f.id
JOIN products p ON df.product_id = p.id
LEFT JOIN quality_products qp ON p.id = qp.product_id
GROUP BY c.id, c.name, p.id, p.name
ORDER BY c.name, p.name;

```

  Subconsultas

1. Como gerente, quiero ver los productos cuyo precio esté por encima del promedio de su categoría.

```sql
SELECT 
    p.id,
    p.name AS producto,
    c.description AS categoria,
    p.price AS precio
FROM products p
JOIN categories c ON p.category_id = c.id
WHERE p.price > (
    SELECT AVG(p2.price)
    FROM products p2
    WHERE p2.category_id = p.category_id
)
ORDER BY c.description, p.price DESC;

```

*Retorna empty porque no hay productos por encima del promedio*

2.Como administrador, deseo listar las empresas que tienen más productos que la media de empresas.

```sql
SELECT 
    c.id,
    c.name AS empresa,
    COUNT(DISTINCT cp.product_id) AS cantidad_productos
FROM companies c
JOIN companyproducts cp ON c.id = cp.company_id
GROUP BY c.id, c.name
HAVING COUNT(DISTINCT cp.product_id) > (
    SELECT AVG(productos_por_empresa) FROM (
        SELECT COUNT(DISTINCT cp2.product_id) AS productos_por_empresa
        FROM companyproducts cp2
        GROUP BY cp2.company_id
    ) AS sub
)
ORDER BY cantidad_productos DESC;

```

3. Como cliente, quiero ver mis productos favoritos que han sido calificados por otros clientes.

```sql
SELECT 
    c.id AS cliente_id,
    c.name AS cliente,
    p.id AS producto_id,
    p.name AS producto,
    ROUND(AVG(qp.rating), 2) AS promedio_rating
FROM customers c
JOIN favorites f ON f.customer_id = c.id
JOIN details_favorites df ON df.favorite_id = f.id
JOIN products p ON df.product_id = p.id
JOIN quality_products qp 
    ON qp.product_id = p.id AND qp.customer_id <> c.id -- calificado por otros
GROUP BY c.id, c.name, p.id, p.name
ORDER BY c.name, promedio_rating DESC;

```

4. Como supervisor, deseo obtener los productos con el mayor número de veces añadidos como favoritos.

```sql
SELECT 
    p.id AS producto_id,
    p.name AS producto,
    COUNT(*) AS veces_favorito
FROM details_favorites df
JOIN products p ON df.product_id = p.id
GROUP BY p.id, p.name
HAVING COUNT(*) = (
    SELECT MAX(favs.total)
    FROM (
        SELECT COUNT(*) AS total
        FROM details_favorites
        GROUP BY product_id
    ) AS favs
);

```

5. Como técnico, quiero listar los clientes cuyo correo no aparece en la tabla `rates` ni en `quality_products`.

```sql
SELECT 
    c.id AS cliente_id,
    c.name AS cliente,
    ce.email
FROM customers c
JOIN customer_emails ce ON ce.customer_id = c.id
WHERE c.id NOT IN (
    SELECT customer_id FROM rates
)
AND c.id NOT IN (
    SELECT customer_id FROM quality_products
)
ORDER BY c.name;

```

6. Como gestor de calidad, quiero obtener los productos con una calificación inferior al mínimo de su categoría.

```sql
SELECT 
    p.id AS producto_id,
    p.name AS producto,
    p.category_id,
    ROUND(AVG(qp.rating), 2) AS promedio_producto
FROM products p
JOIN quality_products qp ON p.id = qp.product_id
GROUP BY p.id, p.name, p.category_id
HAVING AVG(qp.rating) < (
    SELECT MIN(prom_categ)
    FROM (
        SELECT 
            p2.category_id,
            AVG(qp2.rating) AS prom_categ
        FROM products p2
        JOIN quality_products qp2 ON p2.id = qp2.product_id
        GROUP BY p2.id, p2.category_id
        HAVING p2.category_id = p.category_id
    ) AS sub
);

```

7. Como desarrollador, deseo listar las ciudades que no tienen clientes registrados.

```sql
SELECT 
    cm.code AS ciudad_codigo,
    cm.name AS ciudad
FROM citiesormunicipalities cm
WHERE cm.code NOT IN (
    SELECT DISTINCT city_id FROM customers
)
ORDER BY cm.name;

```

8. Como administrador, quiero ver los productos que no han sido evaluados en ninguna encuesta.

```sql
SELECT 
    p.id,
    p.name AS producto
FROM products p
WHERE p.id NOT IN (
    SELECT product_id FROM quality_products
)
ORDER BY p.name;

```

9. Como auditor, quiero listar los beneficios que no están asignados a ninguna audiencia.

```sql
SELECT 
    b.id,
    b.description,
    b.detail
FROM benefits b
WHERE b.id NOT IN (
    SELECT DISTINCT benefit_id FROM audiencebenefits
)
ORDER BY b.description;

```

10. Como cliente, deseo obtener mis productos favoritos que no están disponibles actualmente en ninguna empresa.

```sql
SELECT 
    c.id AS cliente_id,
    c.name AS cliente,
    p.id AS producto_id,
    p.name AS producto
FROM customers c
JOIN favorites f ON f.customer_id = c.id
JOIN details_favorites df ON df.favorite_id = f.id
JOIN products p ON df.product_id = p.id
WHERE p.id NOT IN (
    SELECT DISTINCT product_id FROM companyproducts
)
ORDER BY c.name, p.name;

```

11. Como director, deseo consultar los productos vendidos en empresas cuya ciudad tenga menos de tres empresas registradas.

```sql
SELECT 
    p.id AS producto_id,
    p.name AS producto,
    c.name AS empresa,
    cm.name AS ciudad
FROM companyproducts cp
JOIN products p ON cp.product_id = p.id
JOIN companies c ON cp.company_id = c.id
JOIN citiesormunicipalities cm ON c.city_id = cm.code
WHERE c.city_id IN (
    SELECT city_id
    FROM companies
    GROUP BY city_id
    HAVING COUNT(*) < 3
)
ORDER BY cm.name, p.name;

```

12. Como analista, quiero ver los productos con calidad superior al promedio de todos los productos.

```sql
SELECT 
    p.id AS producto_id,
    p.name AS producto,
    ROUND(AVG(qp.rating), 2) AS promedio_producto
FROM products p
JOIN quality_products qp ON p.id = qp.product_id
GROUP BY p.id, p.name
HAVING AVG(qp.rating) > (
    SELECT AVG(rating) FROM quality_products
)
ORDER BY promedio_producto DESC;

```

13. Como gestor, quiero ver empresas que sólo venden productos de una única categoría.

```sql
SELECT 
    c.id AS empresa_id,
    c.name AS empresa,
    (
        SELECT cat.description
        FROM companyproducts cp2
        JOIN products p2 ON cp2.product_id = p2.id
        JOIN categories cat ON p2.category_id = cat.id
        WHERE cp2.company_id = c.id
        GROUP BY p2.category_id
        LIMIT 1
    ) AS categoria
FROM companies c
WHERE c.id IN (
    SELECT company_id
    FROM companyproducts cp
    JOIN products p ON cp.product_id = p.id
    GROUP BY cp.company_id
    HAVING COUNT(DISTINCT p.category_id) = 1
)
ORDER BY empresa;

```

14. Como gerente comercial, quiero consultar los productos con el mayor precio entre todas las empresas.

```sql
SELECT 
    p.id AS producto_id,
    p.name AS producto,
    c.name AS empresa,
    cm.name AS ciudad,
    cpp.price
FROM companyproduct_prices cpp
JOIN products p ON cpp.product_id = p.id
JOIN companies c ON cpp.company_id = c.id
JOIN citiesormunicipalities cm ON c.city_id = cm.code
WHERE (cpp.product_id, cpp.price) IN (
    SELECT product_id, MAX(price)
    FROM companyproduct_prices
    GROUP BY product_id
)
ORDER BY cpp.price DESC;

```

15. Como cliente, quiero saber si algún producto de mis favoritos ha sido calificado por otro cliente con más de 4 estrellas.

```sql
SELECT 
    c.id AS cliente_id,
    c.name AS cliente,
    p.id AS producto_id,
    p.name AS producto
FROM customers c
JOIN favorites f ON f.customer_id = c.id
JOIN details_favorites df ON df.favorite_id = f.id
JOIN products p ON df.product_id = p.id
WHERE EXISTS (
    SELECT 1
    FROM quality_products qp
    WHERE qp.product_id = p.id
      AND qp.customer_id <> c.id
      AND qp.rating > 4
)
ORDER BY c.name, p.name;


```

16. Como operador, quiero saber qué productos no tienen imagen asignada pero sí han sido calificados.

```sql
SELECT 
    p.id AS producto_id,
    p.name AS producto
FROM products p
WHERE (p.image IS NULL OR p.image = '')
AND EXISTS (
    SELECT 1
    FROM quality_products qp
    WHERE qp.product_id = p.id
);

```

17. Como auditor, quiero ver los planes de membresía sin periodo vigente.

```sql
SELECT
    m.id AS membership_id,
    m.name AS membresía
FROM memberships m
WHERE m.id NOT IN (
    SELECT DISTINCT membership_id
    FROM membershipperiods
);

```

18. Como especialista, quiero identificar los beneficios compartidos por más de una audiencia.

```sql
SELECT 
    b.id AS benefit_id,
    b.description AS beneficio
FROM benefits b
WHERE (
    SELECT COUNT(DISTINCT ab.audience_id)
    FROM audiencebenefits ab
    WHERE ab.benefit_id = b.id
) > 1
ORDER BY b.description;

```

19. Como técnico, quiero encontrar empresas cuyos productos no tengan unidad de medida definida.

```sql
SELECT 
    c.id AS empresa_id,
    c.name AS empresa
FROM companies c
WHERE c.id IN (
    SELECT cp.company_id
    FROM companyproducts cp
    WHERE cp.unitmeasure_id IS NULL
);

```

20. Como gestor de campañas, deseo obtener los clientes con membresía activa y sin productos favoritos.

```sql
SELECT 
    c.id AS cliente_id,
    c.name AS cliente
FROM customers c
WHERE c.audience_id IN (
    SELECT DISTINCT mb.audience_id
    FROM membershipbenefits mb
)
AND c.id NOT IN (
    SELECT DISTINCT f.customer_id
    FROM favorites f
)
ORDER BY c.name;

```

## **Funciones Agregadas**

1. Obtener el promedio de calificación por producto

```sql
SELECT 
    p.id AS producto_id,
    p.name AS producto,
    ROUND(AVG(qp.rating), 2) AS promedio_calificacion
FROM products p
JOIN quality_products qp ON p.id = qp.product_id
GROUP BY p.id, p.name
ORDER BY promedio_calificacion DESC;

```

2. Contar cuántos productos ha calificado cada cliente

```sql
SELECT 
    c.id AS cliente_id,
    c.name AS cliente,
    COUNT(qp.product_id) AS cantidad_calificaciones
FROM customers c
JOIN quality_products qp ON c.id = qp.customer_id
GROUP BY c.id, c.name
ORDER BY cantidad_calificaciones DESC;

```

3. Sumar el total de beneficios asignados por audiencia

```sql
SELECT 
    a.id AS audiencia_id,
    a.description AS audiencia,
    COUNT(ab.benefit_id) AS total_beneficios
FROM audiences a
LEFT JOIN audiencebenefits ab ON a.id = ab.audience_id
GROUP BY a.id, a.description
ORDER BY total_beneficios DESC;

```

4. Calcular la media de productos por empresa

   ```sql
   SELECT 
       ROUND(AVG(productos_por_empresa), 2) AS media_productos_por_empresa
   FROM (
       SELECT 
           company_id,
           COUNT(DISTINCT product_id) AS productos_por_empresa
       FROM companyproducts
       GROUP BY company_id
   ) AS sub;
   
   ```

5. Contar el total de empresas por ciudad

```sql
SELECT 
    cm.name AS ciudad,
    COUNT(c.id) AS total_empresas
FROM citiesormunicipalities cm
LEFT JOIN companies c ON cm.code = c.city_id
GROUP BY cm.code, cm.name
ORDER BY total_empresas DESC;

```

6. Calcular el promedio de precios por unidad de medida

```sql
SELECT 
    u.description AS unidad_medida,
    ROUND(AVG(cpp.price), 2) AS promedio_precio
FROM companyproducts cp
JOIN companyproduct_prices cpp 
    ON cp.company_id = cpp.company_id AND cp.product_id = cpp.product_id
JOIN unitofmeasure u ON cp.unitmeasure_id = u.id
GROUP BY u.id, u.description
ORDER BY promedio_precio DESC;

```

7. Contar cuántos clientes hay por ciudad

```sql
SELECT 
    cm.name AS ciudad,
    COUNT(c.id) AS total_clientes
FROM citiesormunicipalities cm
LEFT JOIN customers c ON cm.code = c.city_id
GROUP BY cm.code, cm.name
ORDER BY total_clientes DESC;

```

8.  Calcular planes de membresía por periodo

```sql
SELECT 
    p.name AS periodo,
    COUNT(mp.membership_id) AS total_planes
FROM periods p
LEFT JOIN membershipperiods mp ON p.id = mp.period_id
GROUP BY p.id, p.name
ORDER BY total_planes DESC;

```

9. Ver el promedio de calificaciones dadas por un cliente a sus favoritos

```sql
SELECT 
    c.id AS cliente_id,
    c.name AS cliente,
    ROUND(AVG(qp.rating), 2) AS promedio_favoritos
FROM customers c
JOIN favorites f ON c.id = f.customer_id
JOIN details_favorites df ON df.favorite_id = f.id
JOIN quality_products qp 
  ON qp.product_id = df.product_id AND qp.customer_id = c.id
GROUP BY c.id, c.name
ORDER BY promedio_favoritos DESC;

```

10. Consultar la fecha más reciente en que se calificó un producto

```sql
SELECT 
    p.id AS producto_id,
    p.name AS producto,
    MAX(qp.daterating) AS fecha_ultima_calificacion
FROM products p
JOIN quality_products qp ON p.id = qp.product_id
GROUP BY p.id, p.name
ORDER BY fecha_ultima_calificacion DESC
LIMIT 1;

```

11. Obtener la desviación estándar de precios por categoría

```sql
SELECT 
    cat.description AS categoria,
    ROUND(STDDEV(p.price), 2) AS desviacion_estandar_precio
FROM products p
JOIN categories cat ON p.category_id = cat.id
GROUP BY cat.id, cat.description
ORDER BY desviacion_estandar_precio DESC;

```

12.  Contar cuántas veces un producto fue favorito

```sql
SELECT 
    p.id AS producto_id,
    p.name AS producto,
    COUNT(df.favorite_id) AS veces_favorito
FROM products p
LEFT JOIN details_favorites df ON p.id = df.product_id
GROUP BY p.id, p.name
ORDER BY veces_favorito DESC;

```

13. Calcular el porcentaje de productos evaluados

```sql
SELECT
    (COUNT(DISTINCT cp.product_id) / COUNT(p.id)) * 100 AS porcentaje_productos_evaluados
FROM
    products p
LEFT JOIN
    companyproducts cp ON p.id = cp.product_id
LEFT JOIN
    rates r ON cp.company_id = r.company_id;
```

14. Ver el promedio de rating por encuesta

    ```sql
    SELECT 
        p.id AS poll_id,
        p.name AS encuesta,
        AVG(r.rating) AS promedio_rating
    FROM 
        polls p
    LEFT JOIN 
        rates r ON p.id = r.poll_id
    GROUP BY 
        p.id, p.name
    ORDER BY 
        promedio_rating DESC;
    ```

15. Calcular el promedio y total de beneficios por plan

```sql
SELECT 
    m.id AS membership_id,
    m.name AS nombre_plan,
    COUNT(mb.benefit_id) AS total_beneficios
FROM 
    memberships m
LEFT JOIN 
    membershipbenefits mb ON m.id = mb.membership_id
GROUP BY 
    m.id, m.name
ORDER BY 
    total_beneficios DESC;
```

16. Obtener media y varianza de precios por empresa

```sql
SELECT 
    c.id AS company_id,
    c.name AS company_name,
    COUNT(*) AS product_count,
    ROUND(AVG(cpp.price), 2) AS average_price,
    ROUND(VARIANCE(cpp.price), 2) AS price_variance,
    ROUND(STDDEV(cpp.price), 2) AS price_standard_deviation
FROM 
    companies c
JOIN 
    companyproduct_prices cpp ON c.id = cpp.company_id
WHERE 
    cpp.end_date IS NULL OR cpp.end_date > NOW() 
GROUP BY 
    c.id, c.name
ORDER BY 
    price_variance DESC;
```

17. Ver total de productos disponibles en la ciudad del cliente

```sql
SELECT 
    c.id AS customer_id,
    c.name AS customer_name,
    c.city_id,
    cm.name AS city_name,
    COUNT(DISTINCT cp.product_id) AS available_products,
    IFNULL(GROUP_CONCAT(DISTINCT CONCAT(p.name, ' ($', FORMAT(p.price_usd, 2), ')') SEPARATOR ' | '), 'No products available') AS product_list_with_prices
FROM 
    customers c
JOIN 
    citiesormunicipalities cm ON c.city_id = cm.code
LEFT JOIN 
    companies co ON co.city_id = c.city_id
LEFT JOIN 
    companyproducts cp ON co.id = cp.company_id
LEFT JOIN 
    products p ON cp.product_id = p.id
WHERE 
    c.id <= 60
GROUP BY 
    c.id, c.name, c.city_id, cm.name
ORDER BY 
    c.id;
```

18. Contar productos únicos por tipo de empresa

```sql
SELECT 
    sc.description AS company_type,
    COUNT(DISTINCT cp.product_id) AS unique_products_count
FROM 
    companies co
JOIN 
    companyproducts cp ON co.id = cp.company_id
JOIN 
    subdivisioncategories sc ON co.type_id = sc.id
GROUP BY 
    sc.description
ORDER BY 
    unique_products_count DESC;
```

19. Ver total de clientes sin correo electrónico registrado

```sql
SELECT 
    COUNT(*) AS total_clientes_sin_email
FROM 
    customers c
WHERE 
    NOT EXISTS (
        SELECT 1 
        FROM customer_emails ce 
        WHERE ce.customer_id = c.id
    );
```

20. Empresa con más productos calificados

```sql
xxxxxxxxxx 	SELECT     co.id AS company_id,    co.name AS company_name,    COUNT(*) AS rating_countFROM     companies coJOIN     rates r ON co.id = r.company_idGROUP BY     co.id, co.nameORDER BY     rating_count DESCLIMIT 1;
```



## **Procedimientos Almacenados**

1. Registrar una nueva calificación y actualizar el promedio

```sql
DELIMITER //

CREATE PROCEDURE registrar_calificacion(
    IN p_customer_id INT,
    IN p_company_id INT,
    IN p_rating DOUBLE,
    IN p_poll_id INT
)
BEGIN
    DECLARE EXIT HANDLER FOR SQLEXCEPTION
    BEGIN
        ROLLBACK;
        SELECT 'Error al registrar la calificación' AS message;
    END;
    
    START TRANSACTION;
    
    
    INSERT INTO rates (customer_id, company_id, poll_id, rating, daterating)
    VALUES (p_customer_id, p_company_id, p_poll_id, p_rating, NOW());
    
    COMMIT;
    SELECT 'Calificación registrada correctamente' AS message;
END //

DELIMITER ;
```

```sql
CALL registrar_calificacion(8, 3, 4.5, 6);
```

2.  Insertar empresa y asociar productos por defecto

```SQL
DELIMITER //

CREATE PROCEDURE crear_empresa_con_productos_fijos(
    IN p_nombre_empresa VARCHAR(80),
    IN p_tipo_id INT,
    IN p_categoria_id INT,
    IN p_ciudad_id VARCHAR(6),
    IN p_audiencia_id INT
)
BEGIN
    DECLARE v_empresa_id INT;
    
   
    INSERT INTO companies (name, type_id, category_id, city_id, audience_id)
    VALUES (p_nombre_empresa, p_tipo_id, p_categoria_id, p_ciudad_id, p_audiencia_id);
    
    SET v_empresa_id = LAST_INSERT_ID();
    
    INSERT INTO companyproducts (company_id, product_id)
    VALUES 
        (v_empresa_id, 1), 
        (v_empresa_id, 3),  
        (v_empresa_id, 5); 
    SELECT CONCAT('Empresa creada con 3 productos básicos') AS resultado;
END //

DELIMITER ;
```

```sql
CALL crear_empresa_con_productos_fijos(
    'Tienda Musical Ejemplo',  
    1,                        
    3,                       
    '05001',                  
    1                         
);
```

3. Añadir producto favorito validando duplicados

```sql
DELIMITER //

CREATE PROCEDURE agregar_producto_favorito(
    IN p_customer_id INT,
    IN p_product_id INT
)
BEGIN
    DECLARE v_favorite_id INT;
    DECLARE v_product_name VARCHAR(80);
    DECLARE v_product_category INT;
    
    SELECT name, category_id INTO v_product_name, v_product_category
    FROM products WHERE id = p_product_id;
    
    IF v_product_name IS NULL THEN
        SELECT 'Error: El producto no existe' AS resultado;
    ELSE
        
        SELECT id INTO v_favorite_id FROM favorites 
        WHERE customer_id = p_customer_id 
        LIMIT 1;
        
        IF v_favorite_id IS NULL THEN
            INSERT INTO favorites (customer_id) VALUES (p_customer_id);
            SET v_favorite_id = LAST_INSERT_ID();
        END IF;
        
        INSERT INTO details_favorites (favorite_id, product_id, category_id, name, detail)
        SELECT 
            v_favorite_id, 
            p_product_id, 
            v_product_category, 
            v_product_name,
            (SELECT detail FROM products WHERE id = p_product_id)
        ON DUPLICATE KEY UPDATE
            category_id = v_product_category,
            name = v_product_name,
            detail = (SELECT detail FROM products WHERE id = p_product_id);
        
        SELECT CONCAT('Producto "', v_product_name, '" añadido a favoritos') AS resultado;
    END IF;
END //

DELIMITER ;
```

```sql
CALL agregar_producto_favorito(1, 3);
```

4. Generar resumen mensual de calificaciones por empresa

```sql
DELIMITER //

CREATE PROCEDURE generar_resumen_calificaciones_mensual(
    IN p_mes INT,
    IN p_anio INT
)
BEGIN
    CREATE TABLE IF NOT EXISTS resumen_calificaciones (
        id INT AUTO_INCREMENT PRIMARY KEY,
        empresa_id INT NOT NULL,
        empresa_nombre VARCHAR(80) NOT NULL,
        mes INT NOT NULL,
        anio INT NOT NULL,
        promedio_calificacion DECIMAL(3,2) NOT NULL,
        total_calificaciones INT NOT NULL,
        fecha_generacion TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
        UNIQUE KEY (empresa_id, mes, anio)
    );
    
    DELETE FROM resumen_calificaciones 
    WHERE mes = p_mes AND anio = p_anio;
    
    INSERT INTO resumen_calificaciones (
        empresa_id,
        empresa_nombre,
        mes,
        anio,
        promedio_calificacion,
        total_calificaciones
    )
    SELECT 
        r.company_id,
        co.name,
        p_mes,
        p_anio,
        ROUND(AVG(r.rating), 2) AS promedio,
        COUNT(*) AS total
    FROM 
        rates r
    JOIN 
        companies co ON r.company_id = co.id
    WHERE 
        MONTH(r.daterating) = p_mes
        AND YEAR(r.daterating) = p_anio
    GROUP BY 
        r.company_id, co.name;
    
    SELECT CONCAT('Resumen generado para ', p_mes, '-', p_anio, 
                 ': ', ROW_COUNT(), ' empresas procesadas') AS resultado;
END //

DELIMITER ;
```

```sql
CALL generar_resumen_calificaciones_mensual(1, 2023);

SET @mes_actual = MONTH(CURRENT_DATE());
SET @anio_actual = YEAR(CURRENT_DATE());
CALL generar_resumen_calificaciones_mensual(@mes_actual, @anio_actual);
```

6. Eliminar productos huérfanos

```sql
DELIMITER //

CREATE PROCEDURE eliminar_productos_huerfanos()
BEGIN
    DELETE FROM products
    WHERE id NOT IN (
        SELECT DISTINCT cp.product_id FROM companyproducts cp
    );
END;
//

DELIMITER ;

```

```sql
CALL eliminar_productos_huerfanos();

```

7. Actualizar precios de productos por categoría

```sql
DELIMITER //

CREATE PROCEDURE actualizar_precios_por_categoria(
    IN categoria INT,
    IN factor DECIMAL(10,2)
)
BEGIN
    UPDATE companyproduct_prices cpp
    JOIN products p ON cpp.product_id = p.id
    SET cpp.price = cpp.price * factor
    WHERE p.category_id = categoria
      AND CURDATE() BETWEEN cpp.start_date AND IFNULL(cpp.end_date, CURDATE());
END;
//

DELIMITER ;

```

```sql
CALL actualizar_precios_por_categoria(2, 1.10); 
CALL actualizar_precios_por_categoria(5, 0.95); -

```

8. Validar inconsistencia entre `rates` y `quality_products`

```sql
CREATE TABLE errores_log (
    id INT AUTO_INCREMENT PRIMARY KEY,
    origen VARCHAR(100),
    detalle TEXT,
    fecha TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

```

```sql
DELIMITER //

CREATE PROCEDURE validar_inconsistencias_rates()
BEGIN
    INSERT INTO errores_log (origen, detalle)
    SELECT 
        'rates_vs_quality_products',
        CONCAT('Sin quality_products → customer_id: ', r.customer_id,
               ', company_id: ', r.company_id,
               ', poll_id: ', r.poll_id)
    FROM rates r
    LEFT JOIN quality_products qp
      ON r.customer_id = qp.customer_id
      AND r.company_id = qp.company_id
      AND r.poll_id = qp.poll_id
    WHERE qp.customer_id IS NULL;
END;
//

DELIMITER ;

```

```sql
CALL validar_inconsistencias_rates();

```

9. Asignar beneficios a nuevas audiencias

```sql
DELIMITER //

CREATE PROCEDURE asignar_beneficio_a_audiencia(
    IN p_audience_id INT,
    IN p_benefit_id INT
)
BEGIN
    IF EXISTS (
        SELECT 1 FROM audiences WHERE id = p_audience_id
    ) AND EXISTS (
        SELECT 1 FROM benefits WHERE id = p_benefit_id
    ) THEN
        
        IF NOT EXISTS (
            SELECT 1 FROM audiencebenefits 
            WHERE audience_id = p_audience_id 
              AND benefit_id = p_benefit_id
        ) THEN
            INSERT INTO audiencebenefits (audience_id, benefit_id)
            VALUES (p_audience_id, p_benefit_id);
        END IF;
    END IF;
END;
//

DELIMITER ;


```

```sql
CALL asignar_beneficio_a_audiencia(5, 12);

```

10. Activar planes de membresía vencidos con pago confirmado

```sql
DELIMITER //

CREATE PROCEDURE activar_planes_vencidos_con_pago()
BEGIN
    UPDATE membershipperiods
    SET status = 'ACTIVA'
    WHERE pago_confirmado = TRUE
      AND status != 'ACTIVA'
      AND end_date < CURDATE();
END;
//

DELIMITER ;

```

```sql
CALL activar_planes_vencidos_con_pago();

```

11. Listar productos favoritos del cliente con su calificación

```sql
DELIMITER //

CREATE PROCEDURE listar_favoritos_con_rating(IN p_customer_id INT)
BEGIN
    SELECT 
        df.product_id,
        p.name AS nombre_producto,
        ROUND(AVG(r.rating), 2) AS promedio_rating
    FROM favorites f
    JOIN details_favorites df ON f.id = df.favorite_id
    JOIN products p ON df.product_id = p.id
    LEFT JOIN rates r 
        ON f.customer_id = r.customer_id 
        AND f.company_id = r.company_id
    WHERE f.customer_id = p_customer_id
    GROUP BY df.product_id, p.name;
END;
//

DELIMITER ;

```

```sql
CALL listar_favoritos_con_rating(1);

```

12. Registrar encuesta y sus preguntas asociadas

```sql
CREATE TABLE IF NOT EXISTS poll_questions (
    id INT AUTO_INCREMENT PRIMARY KEY,
    poll_id INT NOT NULL,
    question_text TEXT NOT NULL,
    FOREIGN KEY (poll_id) REFERENCES polls(id)
);

```

```sql
DELIMITER //

CREATE PROCEDURE registrar_encuesta_con_preguntas(
    IN p_name VARCHAR(80),
    IN p_description TEXT,
    IN p_question1 TEXT,
    IN p_question2 TEXT,
    IN p_question3 TEXT
)
BEGIN
    DECLARE new_poll_id INT;

    INSERT INTO polls (name, description, isactive)
    VALUES (p_name, p_description, 1);

    SET new_poll_id = LAST_INSERT_ID();

    INSERT INTO poll_questions (poll_id, question_text)
    VALUES 
        (new_poll_id, p_question1),
        (new_poll_id, p_question2),
        (new_poll_id, p_question3);
END;
//

DELIMITER ;

```

```sql
CALL registrar_encuesta_con_preguntas(
    'Encuesta de satisfacción',
    'Esta encuesta evalúa la experiencia del cliente',
    '¿Qué tan satisfecho está con el servicio?',
    '¿Recomendaría nuestro producto?',
    '¿Qué aspectos mejorarías?'
);

```

13. Eliminar favoritos antiguos sin calificaciones

```sql
DELIMITER //

CREATE PROCEDURE eliminar_favoritos_antiguos()
BEGIN
    DELETE df
    FROM details_favorites df
    JOIN favorites f ON df.favorite_id = f.id
    LEFT JOIN rates r 
        ON r.customer_id = f.customer_id 
        AND r.company_id = f.company_id
        AND r.daterating >= CURDATE() - INTERVAL 12 MONTH
    WHERE f.fecha_agregado < CURDATE() - INTERVAL 12 MONTH
      AND r.customer_id IS NULL;
END;
//

DELIMITER ;

```

```sql
CALL eliminar_favoritos_antiguos();

```

14. Asociar beneficios automáticamente por audiencia

```sql
DELIMITER //

CREATE PROCEDURE asociar_beneficios_automaticos()
BEGIN
    INSERT IGNORE INTO audiencebenefits (audience_id, benefit_id)
    VALUES 
        (1, 1),
        (1, 2);

    INSERT IGNORE INTO audiencebenefits (audience_id, benefit_id)
    VALUES 
        (2, 2),
        (2, 3);

    INSERT IGNORE INTO audiencebenefits (audience_id, benefit_id)
    VALUES 
        (3, 1),
        (3, 3),
        (3, 4);
END;
//

DELIMITER ;

```

```sql
CALL asociar_beneficios_automaticos();

```

15. Historial de cambios de precio

```sql
CREATE TABLE IF NOT EXISTS historial_precios (
    id INT AUTO_INCREMENT PRIMARY KEY,
    company_id INT NOT NULL,
    product_id INT NOT NULL,
    precio_anterior DOUBLE,
    precio_nuevo DOUBLE,
    fecha_cambio DATETIME DEFAULT NOW(),
    FOREIGN KEY (company_id) REFERENCES companies(id),
    FOREIGN KEY (product_id) REFERENCES products(id)
);

```

```sql
DELIMITER //

CREATE PROCEDURE actualizar_precio(
    IN p_company_id INT,
    IN p_product_id INT,
    IN p_nuevo_precio DOUBLE
)
BEGIN
    DECLARE v_precio_actual DOUBLE;

    SELECT price INTO v_precio_actual
    FROM companyproduct_prices
    WHERE company_id = p_company_id
      AND product_id = p_product_id
    ORDER BY start_date DESC
    LIMIT 1;

    -
    IF v_precio_actual IS NULL OR v_precio_actual != p_nuevo_precio THEN
       
        INSERT INTO companyproduct_prices (
            company_id, product_id, price, start_date
        ) VALUES (
            p_company_id, p_product_id, p_nuevo_precio, NOW()
        );
       
        INSERT INTO historial_precios (
            company_id, product_id, precio_anterior, precio_nuevo
        ) VALUES (
            p_company_id, p_product_id, v_precio_actual, p_nuevo_precio
        );
    END IF;
END;
//

DELIMITER ;

```

```sql
CALL actualizar_precio(2, 5, 29900);

```

16. Registrar encuesta activa automáticamente

```sql
DELIMITER //

CREATE PROCEDURE registrar_encuesta_activa(
    IN p_nombre VARCHAR(80),
    IN p_descripcion TEXT,
    IN p_categoria INT
)
BEGIN
    
    UPDATE polls SET isactive = 0 WHERE isactive = 1;

    INSERT INTO polls (name, description, isactive, categorypoll_id)
    VALUES (p_nombre, p_descripcion, 1, p_categoria);
END //

DELIMITER ;


```

```sql
CALL registrar_encuesta_activa('Encuesta de satisfacción', 'Opinión sobre el servicio al cliente', 2);

```

17. Actualizar unidad de medida de productos sin afectar ventas

```SQL
DELIMITER //

CREATE PROCEDURE actualizar_unidad_companyproduct(
    IN p_company_id INT,
    IN p_product_id INT,
    IN p_nueva_unidad INT
)
BEGIN
    UPDATE companyproducts
    SET unitmeasure_id = p_nueva_unidad
    WHERE company_id = p_company_id
      AND product_id = p_product_id;

    SELECT 'Unidad de medida actualizada en companyproducts.' AS mensaje;
END //

DELIMITER ;

```

```SQL
CALL actualizar_unidad_companyproduct(1, 3, 2);

```

20. Validar claves foráneas entre calificaciones y encuestas

```sql
DELIMITER //

CREATE PROCEDURE validar_claves_foraneas_rates_polls()
BEGIN
    SELECT r.customer_id, r.company_id, r.poll_id, r.rating, r.daterating
    FROM rates r
    LEFT JOIN polls p ON r.poll_id = p.id
    WHERE p.id IS NULL;
END //

DELIMITER ;

```

```sql
CALL validar_claves_foraneas_rates_polls();

```

## **Triggers**

1. Actualizar la fecha de modificación de un producto

```sql
DELIMITER //

CREATE TRIGGER before_update_products
BEFORE UPDATE ON products
FOR EACH ROW
BEGIN
    SET NEW.updated_at = NOW();
END //

DELIMITER ;

```

2. Registrar log cuando un cliente califica un producto

```sql
CREATE TABLE IF NOT EXISTS log_acciones (
    id INT AUTO_INCREMENT PRIMARY KEY,
    cliente_id INT,
    empresa_id INT,
    poll_id INT,
    descripcion TEXT,
    fecha DATETIME DEFAULT CURRENT_TIMESTAMP
);
```

```sql
DELIMITER //

CREATE TRIGGER after_insert_rates
AFTER INSERT ON rates
FOR EACH ROW
BEGIN
    INSERT INTO log_acciones (cliente_id, empresa_id, poll_id, descripcion, fecha)
    VALUES (
        NEW.customer_id,
        NEW.company_id,
        NEW.poll_id,
        CONCAT('El cliente ', NEW.customer_id, ' calificó la encuesta ', NEW.poll_id, ' para la empresa ', NEW.company_id),
        NOW()
    );
END //

DELIMITER ;

```

3. Impedir insertar productos sin unidad de medida

   ```sql
   DELIMITER //
   
   CREATE TRIGGER before_insert_companyproducts
   BEFORE INSERT ON companyproducts
   FOR EACH ROW
   BEGIN
       IF NEW.unitmeasure_id IS NULL THEN
           SIGNAL SQLSTATE '45000'
           SET MESSAGE_TEXT = ' Error: No se puede insertar un producto sin unidad de medida.';
       END IF;
   END //
   
   DELIMITER ;
   
   ```

   4. Validar calificaciones no mayores a 5

   ```sql
   DELIMITER //
   
   CREATE TRIGGER before_insert_validar_rating
   BEFORE INSERT ON rates
   FOR EACH ROW
   BEGIN
       IF NEW.rating > 5 THEN
           SIGNAL SQLSTATE '45000'
           SET MESSAGE_TEXT = ' Error: La calificación no puede ser mayor a 5.';
       END IF;
   END //
   
   DELIMITER ;
   
   ```

   5. Actualizar estado de membresía cuando vence

```sql
DELIMITER //

CREATE TRIGGER after_update_membershipperiods
AFTER UPDATE ON membershipperiods
FOR EACH ROW
BEGIN
    IF NEW.end_date < CURDATE() THEN
        UPDATE memberships
        SET status = 'INACTIVA'
        WHERE id = NEW.membership_id;
    END IF;
END //

DELIMITER ;

```

7. Enviar notificación al añadir un favorito

```sql
DELIMITER //

CREATE TRIGGER after_insert_favorite_notification
AFTER INSERT ON details_favorites
FOR EACH ROW
BEGIN
    DECLARE v_customer_id INT;
    SELECT customer_id INTO v_customer_id
    FROM favorites
    WHERE id = NEW.favorite_id;

    INSERT INTO notificaciones (customer_id, product_id, mensaje, fecha)
    VALUES (
        v_customer_id,
        NEW.product_id,
        CONCAT('Has añadido el producto ID ', NEW.product_id, ' a tus favoritos.'),
        NOW()
    );
END //

DELIMITER ;

```

8. Insertar fila en `quality_products` tras calificación

```sql
DELIMITER //

CREATE TRIGGER after_insert_rate_quality_product
AFTER INSERT ON rates
FOR EACH ROW
BEGIN
    INSERT INTO quality_products (
        product_id,
        customer_id,
        poll_id,
        company_id,
        daterating,
        rating
    )
    VALUES (
        NEW.product_id,
        NEW.customer_id,
        NEW.poll_id,
        NEW.company_id,
        NEW.daterating,
        NEW.rating
    );
END //

DELIMITER ;

```

9. Eliminar favoritos si se elimina el producto

```sql
DELIMITER //

CREATE TRIGGER after_delete_product_eliminar_favoritos
AFTER DELETE ON products
FOR EACH ROW
BEGIN
    DELETE FROM details_favorites
    WHERE product_id = OLD.id;
END //

DELIMITER ;

```



10. Bloquear modificación de audiencias activas

```sql
DELIMITER //

CREATE TRIGGER before_update_audiences_bloquear_activas
BEFORE UPDATE ON audiences
FOR EACH ROW
BEGIN
    IF OLD.isactive = 1 THEN
        SIGNAL SQLSTATE '45000'
        SET MESSAGE_TEXT = 'Error: No se puede modificar una audiencia activa.';
    END IF;
END //

DELIMITER ;

```

12 . Registrar asignación de nuevo beneficio

```sql
DELIMITER //

CREATE TRIGGER after_insert_membershipbenefits_log
AFTER INSERT ON membershipbenefits
FOR EACH ROW
BEGIN
    INSERT INTO bitacora (tabla, accion, detalle)
    VALUES (
        'membershipbenefits',
        'INSERT',
        CONCAT('Beneficio asignado: membership_id=', NEW.membership_id,
               ', period_id=', NEW.period_id,
               ', audience_id=', NEW.audience_id,
               ', benefit_id=', NEW.benefit_id)
    );
END //

CREATE TRIGGER after_insert_audiencebenefits_log
AFTER INSERT ON audiencebenefits
FOR EACH ROW
BEGIN
    INSERT INTO bitacora (tabla, accion, detalle)
    VALUES (
        'audiencebenefits',
        'INSERT',
        CONCAT('Beneficio asignado: audience_id=', NEW.audience_id,
               ', benefit_id=', NEW.benefit_id)
    );
END //

DELIMITER ;

```

13. Impedir doble calificación por parte del cliente

```sql
DELIMITER //

CREATE TRIGGER before_insert_rates_no_duplicado
BEFORE INSERT ON rates
FOR EACH ROW
BEGIN
    DECLARE ya_existe INT;

    SELECT COUNT(*) INTO ya_existe
    FROM rates
    WHERE customer_id = NEW.customer_id
      AND company_id = NEW.company_id
      AND poll_id = NEW.poll_id;

    IF ya_existe > 0 THEN
        SIGNAL SQLSTATE '45000'
        SET MESSAGE_TEXT = ' Ya existe una calificación para este cliente, empresa y encuesta.';
    END IF;
END //

DELIMITER ;

```

14. Validar correos duplicados en clientes

```sql
DELIMITER //

CREATE TRIGGER before_insert_email_unico
BEFORE INSERT ON customer_emails
FOR EACH ROW
BEGIN
    DECLARE existe INT;

    SELECT COUNT(*) INTO existe
    FROM customer_emails
    WHERE email = NEW.email;

    IF existe > 0 THEN
        SIGNAL SQLSTATE '45000'
        SET MESSAGE_TEXT = ' Error: El correo ya está registrado para otro cliente.';
    END IF;
END //

DELIMITER ;

```

15. Eliminar detalles de favoritos huérfanos

```sql
DELIMITER //

CREATE TRIGGER eliminar_detalles_favoritos_huerfanos
AFTER DELETE ON favorites
FOR EACH ROW
BEGIN
    DELETE FROM details_favorites
    WHERE favorite_id = OLD.id;
END //

DELIMITER ;

```

16. Actualizar campo `updated_at` en `companies`

```sql
DELIMITER //

CREATE TRIGGER actualizar_updated_at_companies
BEFORE UPDATE ON companies
FOR EACH ROW
BEGIN
    SET NEW.updated_at = NOW();
END //

DELIMITER ;

```

17. Impedir borrar ciudad si hay empresas activas

```sql
DELIMITER //

CREATE TRIGGER bloquear_borrado_ciudad_con_empresas
BEFORE DELETE ON citiesormunicipalities
FOR EACH ROW
BEGIN
    DECLARE total_empresas INT;

    SELECT COUNT(*) INTO total_empresas
    FROM companies
    WHERE city_id = OLD.code;

    IF total_empresas > 0 THEN
        SIGNAL SQLSTATE '45000'
        SET MESSAGE_TEXT = ' No se puede eliminar la ciudad: existen empresas registradas en ella.';
    END IF;
END //

DELIMITER ;

```

18. Registrar cambios de estado en encuestas

    ```sql
    CREATE TABLE log_cambios_encuesta (
        id INT AUTO_INCREMENT PRIMARY KEY,
        poll_id INT NOT NULL,
        nuevo_estado TINYINT(1),
        fecha DATETIME DEFAULT NOW(),
        observacion VARCHAR(255)
    );
    ```

    ```sql
    DELIMITER //
    
    CREATE TRIGGER log_estado_encuesta
    AFTER UPDATE ON polls
    FOR EACH ROW
    BEGIN
        IF (OLD.isactive != NEW.isactive OR (OLD.isactive IS NULL AND NEW.isactive IS NOT NULL) OR (OLD.isactive IS NOT NULL AND NEW.isactive IS NULL)) THEN
            INSERT INTO log_cambios_encuesta (poll_id, nuevo_estado, observacion)
            VALUES (NEW.id, NEW.isactive, 'Cambio de estado de la encuesta');
        END IF;
    END //
    
    DELIMITER ;
    
    ```

19. Eliminar productos sin relación a empresas

```sql
DELIMITER //

CREATE TRIGGER after_delete_companyproducts_eliminar_producto
AFTER DELETE ON companyproducts
FOR EACH ROW
BEGIN
    -- Verifica si el producto eliminado aún tiene otras relaciones con empresas
    IF NOT EXISTS (
        SELECT 1
        FROM companyproducts
        WHERE product_id = OLD.product_id
    ) THEN
        -- Si ya no hay relaciones, elimina el producto
        DELETE FROM products
        WHERE id = OLD.product_id;
    END IF;
END //

DELIMITER ;

```

## **EVENTOS**

  1. Borrar productos sin actividad cada 6 meses

```sql
DELIMITER //

CREATE PROCEDURE eliminar_productos_inactivos()
BEGIN
    DELETE FROM products
    WHERE id NOT IN (SELECT product_id FROM rates)
      AND id NOT IN (SELECT product_id FROM favorites)
      AND id NOT IN (SELECT product_id FROM companyproducts);
END //

DELIMITER ;

```

```sql
CREATE EVENT IF NOT EXISTS borrar_productos_inactivos
ON SCHEDULE EVERY 6 MONTH
STARTS CURRENT_TIMESTAMP
DO
    CALL eliminar_productos_inactivos();

```

2. Recalcular el promedio de calificaciones semanalmente

```sql
CREATE TABLE IF NOT EXISTS product_metrics (
    product_id INT PRIMARY KEY,
    average_rating DECIMAL(4,2) DEFAULT 0
);

```

```sql
DELIMITER //

CREATE PROCEDURE actualizar_promedios_productos()
BEGIN
    
    UPDATE product_metrics pm
    JOIN (
        SELECT cp.product_id, AVG(r.rating) AS promedio
        FROM rates r
        JOIN companyproducts cp ON r.company_id = cp.company_id
        GROUP BY cp.product_id
    ) calc ON pm.product_id = calc.product_id
    SET pm.average_rating = calc.promedio;

    INSERT INTO product_metrics (product_id, average_rating)
    SELECT calc.product_id, calc.promedio
    FROM (
        SELECT cp.product_id, AVG(r.rating) AS promedio
        FROM rates r
        JOIN companyproducts cp ON r.company_id = cp.company_id
        GROUP BY cp.product_id
    ) calc
    WHERE calc.product_id NOT IN (SELECT product_id FROM product_metrics);
END //

DELIMITER ;

```

```sql
CREATE EVENT IF NOT EXISTS recalcular_promedios_semanal
ON SCHEDULE EVERY 1 WEEK
STARTS CURRENT_TIMESTAMP
DO
    CALL actualizar_promedios_productos();

```

3. Actualizar precios según inflación mensual

```sql
DELIMITER //

CREATE PROCEDURE ajustar_precios_inflacion()
BEGIN
    UPDATE companyproduct_prices
    SET price = price * 1.03;
END //

DELIMITER ;

```

```sql
CREATE EVENT IF NOT EXISTS evento_ajuste_inflacion
ON SCHEDULE EVERY 1 MONTH
STARTS CURRENT_TIMESTAMP
DO
    CALL ajustar_precios_inflacion();

```

4. Crear backups lógicos diariamente

```sql
CREATE TABLE IF NOT EXISTS products_backup LIKE products;
CREATE TABLE IF NOT EXISTS rates_backup LIKE rates;

```

```sql
DELIMITER //

CREATE PROCEDURE realizar_backup_logico()
BEGIN
    DELETE FROM products_backup;
    DELETE FROM rates_backup;

    INSERT INTO products_backup SELECT * FROM products;
    INSERT INTO rates_backup SELECT * FROM rates;
END //

DELIMITER ;

```

```sql
CREATE EVENT IF NOT EXISTS evento_backup_diario
ON SCHEDULE
    EVERY 1 DAY
    STARTS TIMESTAMP(CURRENT_DATE, '00:00:00')
DO
    CALL realizar_backup_logico();

```



5. Notificar sobre productos favoritos sin calificar

```sql
CREATE TABLE IF NOT EXISTS recordatorios (
    id INT AUTO_INCREMENT PRIMARY KEY,
    customer_id INT,
    product_id INT,
    fecha DATETIME DEFAULT NOW(),
    mensaje VARCHAR(255)
);

```

```sql
DELIMITER //

CREATE PROCEDURE generar_recordatorios_favoritos()
BEGIN
    DELETE FROM recordatorios;

    INSERT INTO recordatorios (customer_id, product_id, mensaje)
    SELECT 
        f.customer_id,
        df.product_id,
        CONCAT(' Recuerda calificar el producto ', df.name)
    FROM favorites f
    JOIN details_favorites df ON f.id = df.favorite_id
    LEFT JOIN rates r 
        ON r.customer_id = f.customer_id
        AND r.company_id = f.company_id
        AND r.poll_id IN (
            SELECT id FROM polls 
            WHERE isactive = 1
        )
      
    WHERE r.rating IS NULL;
END //

DELIMITER ;

```

```sql
CREATE EVENT IF NOT EXISTS evento_recordatorios_favoritos
ON SCHEDULE
    EVERY 1 DAY
    STARTS TIMESTAMP(CURRENT_DATE, '08:00:00')
DO
    CALL generar_recordatorios_favoritos();

```

6. Revisar inconsistencias entre empresa y productos

```sql
CREATE TABLE IF NOT EXISTS errores_log (
    id INT AUTO_INCREMENT PRIMARY KEY,
    tipo_error VARCHAR(100),
    descripcion TEXT,
    fecha DATETIME DEFAULT NOW()
);

```

```sql
DELIMITER //

CREATE PROCEDURE revisar_inconsistencias_empresas_productos()
BEGIN
    
    INSERT INTO errores_log (tipo_error, descripcion)
    SELECT 
        'Producto sin empresa',
        CONCAT('🔍 El producto "', p.name, '" (ID ', p.id, ') no está relacionado con ninguna empresa.')
    FROM products p
    WHERE NOT EXISTS (
        SELECT 1 FROM companyproducts cp WHERE cp.product_id = p.id
    );

    
    INSERT INTO errores_log (tipo_error, descripcion)
    SELECT 
        'Empresa sin productos',
        CONCAT(' La empresa "', c.name, '" (ID ', c.id, ') no tiene productos asociados.')
    FROM companies c
    WHERE NOT EXISTS (
        SELECT 1 FROM companyproducts cp WHERE cp.company_id = c.id
    );
END //

DELIMITER ;

```

```sql
CREATE EVENT IF NOT EXISTS evento_revisar_inconsistencias
ON SCHEDULE
    EVERY 1 WEEK
    STARTS TIMESTAMP(CURRENT_DATE + INTERVAL ((7 - WEEKDAY(CURRENT_DATE)) % 7) DAY, '04:00:00')
DO
    CALL revisar_inconsistencias_empresas_productos();

```

7. Archivar membresías vencidas diariamente

```sql
ALTER TABLE membershipperiods
ADD COLUMN status VARCHAR(20) DEFAULT 'ACTIVA';

```

```sql
DELIMITER //

CREATE PROCEDURE archivar_membresias_vencidas()
BEGIN
    UPDATE membershipperiods
    SET status = 'INACTIVA'
    WHERE status != 'INACTIVA' AND end_date < CURDATE();
END //

DELIMITER ;

```

```sql
CREATE EVENT IF NOT EXISTS evento_archivar_membresias
ON SCHEDULE EVERY 1 DAY
STARTS CURRENT_DATE + INTERVAL 1 DAY
DO
    CALL archivar_membresias_vencidas();

```

8. Notificar beneficios nuevos a usuarios semanalmente

```sql
ALTER TABLE benefits
ADD COLUMN created_at DATETIME DEFAULT CURRENT_TIMESTAMP;

```

```sql
DELIMITER //

CREATE PROCEDURE notificar_beneficios_nuevos()
BEGIN
    INSERT INTO notificaciones (mensaje, fecha)
    SELECT 
        CONCAT(' Nuevo beneficio disponible: ', name),
        NOW()
    FROM benefits
    WHERE created_at >= NOW() - INTERVAL 7 DAY;
END //

DELIMITER ;

```

```sql
CREATE EVENT IF NOT EXISTS evento_notificar_beneficios
ON SCHEDULE EVERY 1 WEEK
STARTS CURRENT_DATE + INTERVAL 1 WEEK
DO
    CALL notificar_beneficios_nuevos();

```

9. Calcular cantidad de favoritos por cliente mensualmente

```sql
CREATE TABLE IF NOT EXISTS favoritos_resumen (
    customer_id INT NOT NULL,
    cantidad_favoritos INT NOT NULL,
    mes_resumen DATE NOT NULL,
    PRIMARY KEY (customer_id, mes_resumen)
);

```

```sql
DELIMITER //

CREATE PROCEDURE calcular_favoritos_mensuales()
BEGIN
    INSERT INTO favoritos_resumen (customer_id, cantidad_favoritos, mes_resumen)
    SELECT 
        f.customer_id,
        COUNT(df.product_id),
        DATE_FORMAT(CURRENT_DATE, '%Y-%m-01') 
    FROM favorites f
    JOIN details_favorites df ON df.favorite_id = f.id
    GROUP BY f.customer_id;
END //

DELIMITER ;

```

```sql
CREATE EVENT IF NOT EXISTS evento_favoritos_mensuales
ON SCHEDULE EVERY 1 MONTH
STARTS DATE_FORMAT(NOW(), '%Y-%m-01') + INTERVAL 1 MONTH
DO
    CALL calcular_favoritos_mensuales();

```

10.  Validar claves foráneas semanalmente

```sql
CREATE TABLE IF NOT EXISTS inconsistencias_fk (
    id INT AUTO_INCREMENT PRIMARY KEY,
    tabla_origen VARCHAR(50),
    campo VARCHAR(50),
    valor_invalido VARCHAR(50),
    fecha_registro DATETIME DEFAULT CURRENT_TIMESTAMP,
    descripcion TEXT
);

```

```sql
DELIMITER //

CREATE PROCEDURE validar_claves_foraneas()
BEGIN
 
    DELETE FROM inconsistencias_fk WHERE DATE(fecha_registro) = CURDATE();

 
    INSERT INTO inconsistencias_fk (tabla_origen, campo, valor_invalido, descripcion)
    SELECT 'rates', 'customer_id', r.customer_id, 'Cliente no existe'
    FROM rates r
    LEFT JOIN customers c ON r.customer_id = c.id
    WHERE c.id IS NULL;

   
    INSERT INTO inconsistencias_fk (tabla_origen, campo, valor_invalido, descripcion)
    SELECT 'rates', 'company_id', r.company_id, 'Empresa no existe'
    FROM rates r
    LEFT JOIN companies co ON r.company_id = co.id
    WHERE co.id IS NULL;

  
    INSERT INTO inconsistencias_fk (tabla_origen, campo, valor_invalido, descripcion)
    SELECT 'rates', 'poll_id', r.poll_id, 'Encuesta no existe'
    FROM rates r
    LEFT JOIN polls p ON r.poll_id = p.id
    WHERE p.id IS NULL;


END //

DELIMITER ;

```

```sql
CREATE EVENT IF NOT EXISTS evento_validar_fk
ON SCHEDULE EVERY 1 WEEK
STARTS CURRENT_DATE + INTERVAL 1 WEEK
DO
    CALL validar_claves_foraneas();

```

12. Cambiar estado de encuestas inactivas automáticamente

```sql
DELIMITER //

CREATE PROCEDURE actualizar_encuestas_inactivas()
BEGIN
    UPDATE polls
    SET isactive = 0
    WHERE id NOT IN (
        SELECT DISTINCT r.poll_id
        FROM rates r
        WHERE r.daterating >= CURDATE() - INTERVAL 6 MONTH
    );
END //

DELIMITER ;

```

```sql
CREATE EVENT IF NOT EXISTS evento_encuestas_inactivas
ON SCHEDULE EVERY 1 MONTH
STARTS CURRENT_DATE + INTERVAL 1 MONTH
DO
    CALL actualizar_encuestas_inactivas();

```

13. Registrar auditorías de forma periódica

```sql
CREATE TABLE IF NOT EXISTS auditorias_diarias (
    id INT AUTO_INCREMENT PRIMARY KEY,
    fecha DATE NOT NULL,
    total_productos INT,
    total_clientes INT,
    total_empresas INT,
    total_encuestas INT
);

```

```sql
DELIMITER //

CREATE PROCEDURE registrar_auditoria_diaria()
BEGIN
    INSERT INTO auditorias_diarias (
        fecha, total_productos, total_clientes, total_empresas, total_encuestas
    )
    VALUES (
        CURDATE(),
        (SELECT COUNT(*) FROM products),
        (SELECT COUNT(*) FROM customers),
        (SELECT COUNT(*) FROM companies),
        (SELECT COUNT(*) FROM polls)
    );
END //

DELIMITER ;

```

```sql
CREATE EVENT IF NOT EXISTS evento_auditoria_diaria
ON SCHEDULE EVERY 1 DAY
STARTS CURRENT_DATE + INTERVAL 1 DAY
DO
    CALL registrar_auditoria_diaria();

```

14. Notificar métricas de calidad a empresas

```sql
CREATE TABLE IF NOT EXISTS notificaciones_empresa (
    id INT AUTO_INCREMENT PRIMARY KEY,
    company_id INT,
    product_id INT,
    promedio_rating DECIMAL(4,2),
    fecha TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

```

```sql
DELIMITER //

CREATE PROCEDURE registrar_metricas_calidad_empresas()
BEGIN
    INSERT INTO notificaciones_empresa (company_id, product_id, promedio_rating)
    SELECT 
        r.company_id,
        cp.product_id,
        AVG(r.rating)
    FROM rates r
    JOIN companyproducts cp 
        ON r.company_id = cp.company_id
    WHERE r.rating IS NOT NULL
    GROUP BY r.company_id, cp.product_id;
END //

DELIMITER ;

```

```sql
CREATE EVENT IF NOT EXISTS evento_metricas_calidad_empresas
ON SCHEDULE EVERY 1 WEEK
STARTS DATE_ADD(CURRENT_DATE, INTERVAL 1 - WEEKDAY(CURRENT_DATE) DAY) 
DO
    CALL registrar_metricas_calidad_empresas();

```

15. Recordar renovación de membresías

```sql
CREATE TABLE IF NOT EXISTS recordatorios_membresias (
    id INT AUTO_INCREMENT PRIMARY KEY,
    membership_id INT,
    customer_id INT,
    mensaje TEXT,
    fecha_recordatorio DATETIME DEFAULT CURRENT_TIMESTAMP
);

```

```sql
DELIMITER //

CREATE PROCEDURE recordar_renovacion_membresias()
BEGIN
    INSERT INTO recordatorios_membresias (membership_id, customer_id, mensaje)
    SELECT 
        mp.membership_id,
        m.customer_id,
        CONCAT('Tu membresía vencerá el ', p.end_date, '. Por favor, considera renovarla.')
    FROM membershipperiods mp
    JOIN memberships m ON mp.membership_id = m.id
    JOIN periods p ON mp.period_id = p.id
    WHERE p.end_date BETWEEN CURDATE() AND CURDATE() + INTERVAL 7 DAY;
END //

DELIMITER ;

```

```sql
CREATE EVENT IF NOT EXISTS evento_recordar_renovacion
ON SCHEDULE EVERY 1 DAY
STARTS CURRENT_DATE + INTERVAL 1 DAY
DO
    CALL recordar_renovacion_membresias();

```

16. Reordenar estadísticas generales cada semana

```sql
CREATE TABLE IF NOT EXISTS estadisticas (
    id INT PRIMARY KEY,
    total_productos INT DEFAULT 0,
    total_clientes INT DEFAULT 0,
    total_empresas INT DEFAULT 0,
    total_calificaciones INT DEFAULT 0,
    total_favoritos INT DEFAULT 0,
    fecha_actualizacion DATETIME DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP
);

```

```sql
DELIMITER //

CREATE PROCEDURE actualizar_estadisticas()
BEGIN
    INSERT INTO estadisticas (id, total_productos, total_clientes, total_empresas, total_calificaciones, total_favoritos)
    VALUES (
        1,
        (SELECT COUNT(*) FROM products),
        (SELECT COUNT(*) FROM customers),
        (SELECT COUNT(*) FROM companies),
        (SELECT COUNT(*) FROM rates),
        (SELECT COUNT(*) FROM favorites)
    )
    ON DUPLICATE KEY UPDATE
        total_productos = VALUES(total_productos),
        total_clientes = VALUES(total_clientes),
        total_empresas = VALUES(total_empresas),
        total_calificaciones = VALUES(total_calificaciones),
        total_favoritos = VALUES(total_favoritos),
        fecha_actualizacion = NOW();
END //

DELIMITER ;

```

```sql

CREATE EVENT IF NOT EXISTS evento_estadisticas_semanal
ON SCHEDULE EVERY 1 WEEK
STARTS CURRENT_DATE + INTERVAL 1 WEEK
DO
    CALL actualizar_estadisticas();

```

17. Crear resúmenes temporales de uso por categoría


```sql
CREATE TABLE IF NOT EXISTS resumen_categoria_uso (
    categoria_id INT,
    nombre_categoria VARCHAR(80),
    total_productos_calificados INT,
    fecha DATETIME DEFAULT CURRENT_TIMESTAMP
);

```

```sql
DELIMITER //

CREATE PROCEDURE generar_resumen_categoria_uso()
BEGIN
   
    DELETE FROM resumen_categoria_uso;

    INSERT INTO resumen_categoria_uso (categoria_id, nombre_categoria, total_productos_calificados)
    SELECT 
        p.category_id,
        c.name,
        COUNT(DISTINCT r.customer_id, r.company_id, r.poll_id) AS total_productos_calificados
    FROM rates r
    JOIN companyproducts cp ON r.company_id = cp.company_id AND cp.product_id = r.product_id
    JOIN products p ON cp.product_id = p.id
    JOIN categories c ON p.category_id = c.id
    GROUP BY p.category_id, c.name;
END //

DELIMITER ;

```

```sql
CREATE EVENT IF NOT EXISTS evento_resumen_categoria
ON SCHEDULE EVERY 1 WEEK
STARTS CURRENT_DATE + INTERVAL 1 WEEK
DO
    CALL generar_resumen_categoria_uso();

```

18. Actualizar beneficios caducados

```sql
DELIMITER //

CREATE PROCEDURE desactivar_beneficios_expirados()
BEGIN
    UPDATE benefits
    SET isactive = 0
    WHERE expires_at IS NOT NULL
      AND expires_at < CURDATE()
      AND isactive = 1;
END //

DELIMITER ;

```

```sql
CREATE EVENT IF NOT EXISTS evento_desactivar_beneficios
ON SCHEDULE EVERY 1 DAY
STARTS CURRENT_DATE + INTERVAL 1 DAY
DO
    CALL desactivar_beneficios_expirados();

```

19. Alertar productos sin evaluación anual

```sql
CREATE TABLE IF NOT EXISTS alertas_productos (
    id INT AUTO_INCREMENT PRIMARY KEY,
    product_id INT NOT NULL,
    fecha_alerta DATETIME DEFAULT NOW(),
    motivo VARCHAR(255),
    FOREIGN KEY (product_id) REFERENCES products(id)
);

```

```sql
DELIMITER //

CREATE PROCEDURE alertar_productos_sin_evaluacion()
BEGIN
    INSERT INTO alertas_productos (product_id, motivo)
    SELECT p.id, 'Sin evaluación en el último año'
    FROM products p
    WHERE NOT EXISTS (
        SELECT 1
        FROM rates r
        WHERE r.company_id IN (
            SELECT company_id FROM companyproducts WHERE product_id = p.id
        )
        AND r.daterating >= CURDATE() - INTERVAL 1 YEAR
    );
END //

DELIMITER ;

```

```sql
CREATE EVENT IF NOT EXISTS evento_alertas_productos_sin_eval
ON SCHEDULE EVERY 1 MONTH
DO
    CALL alertar_productos_sin_evaluacion();

```

20. Actualizar precios con índice externo

```sql
CREATE TABLE IF NOT EXISTS inflacion_indice (
    id INT AUTO_INCREMENT PRIMARY KEY,
    fecha DATE NOT NULL DEFAULT CURDATE(),
    porcentaje DECIMAL(5,4) NOT NULL COMMENT 'Ejemplo: 0.0300 para 3%'
);

```

```sql
DELIMITER //

CREATE PROCEDURE actualizar_precios_por_indice()
BEGIN
    DECLARE multiplicador DECIMAL(5,4);

    SELECT porcentaje INTO multiplicador
    FROM inflacion_indice
    ORDER BY fecha DESC
    LIMIT 1;

   
    UPDATE companyproducts cp
    JOIN products p ON cp.product_id = p.id
    SET cp.price = cp.price * (1 + multiplicador)
    WHERE cp.price IS NOT NULL;
END //

DELIMITER ;

```

```sql
CREATE EVENT IF NOT EXISTS evento_actualizar_precios_indice
ON SCHEDULE EVERY 1 MONTH
DO
    CALL actualizar_precios_por_indice();

```

## JOINS

1. Ver productos con la empresa que los vende

```sql
SELECT 
    co.name AS empresa,
    p.name AS producto,
    p.price AS precio
FROM companyproducts cp
INNER JOIN companies co ON cp.company_id = co.id
INNER JOIN products p ON cp.product_id = p.id;

```

2. Mostrar productos favoritos con su empresa y categoría

```sql
SELECT
    df.name AS producto,
    cat.description AS categoria,
    co.name AS empresa
FROM favorites f
JOIN details_favorites df ON f.id = df.favorite_id
JOIN products p ON df.product_id = p.id
JOIN categories cat ON p.category_id = cat.id
JOIN companyproducts cp ON p.id = cp.product_id
JOIN companies co ON cp.company_id = co.id
WHERE f.customer_id = 1;

```

3.  Ver empresas aunque no tengan productos

```sql
SELECT 
    co.name AS empresa,
    p.name AS producto
FROM companies co
LEFT JOIN companyproducts cp ON co.id = cp.company_id
LEFT JOIN products p ON cp.product_id = p.id;

```

4. Ver productos que fueron calificados (o no)

```sql
SELECT 
    p.name AS producto,
    co.name AS empresa,
    AVG(r.rating) AS promedio_calificacion
FROM rates r
JOIN companies co ON r.company_id = co.id
JOIN companyproducts cp ON cp.company_id = co.id
JOIN products p ON cp.product_id = r.poll_id  
GROUP BY p.id, co.name;

```

5. Ver productos con promedio de calificación y empresa

```sql
SELECT 
    p.name AS producto,
    co.name AS empresa,
    ROUND(AVG(r.rating), 2) AS promedio_calificacion
FROM products p
JOIN companyproducts cp ON p.id = cp.product_id
JOIN companies co ON cp.company_id = co.id
LEFT JOIN rates r ON r.poll_id = p.id  
GROUP BY p.id, co.name;

```

6. Ver clientes y sus calificaciones (si las tienen)

```sql
SELECT 
    cu.id AS cliente_id,
    cu.name AS cliente,
    r.rating,
    r.daterating
FROM customers cu
LEFT JOIN rates r ON r.customer_id = cu.id;

```

7. Ver favoritos con la última calificación del cliente

```sql
SELECT 
    df.name AS producto,
    r.rating AS ultima_calificacion,
    r.daterating AS fecha
FROM favorites f
JOIN details_favorites df ON f.id = df.favorite_id
LEFT JOIN rates r ON r.customer_id = f.customer_id 
                  AND r.poll_id = df.product_id
WHERE f.customer_id = 1
  AND r.daterating = (
      SELECT MAX(r2.daterating)
      FROM rates r2
      WHERE r2.customer_id = f.customer_id
        AND r2.poll_id = df.product_id
  );

```

 8. Ver beneficios incluidos en cada plan de membresía

```sql
SELECT
    m.name AS plan,
    b.description AS beneficio
FROM membershipbenefits mb
JOIN memberships m ON mb.membership_id = m.id
JOIN benefits b ON mb.benefit_id = b.id
ORDER BY m.name, b.description;

```

9. Ver clientes con membresía activa y sus beneficios

```sql
mysql> SELECT
    ->     cu.name AS cliente,
    ->     m.name AS membresia,
    ->     b.description AS beneficio
    -> FROM customers cu
    -> JOIN membershipbenefits mb ON cu.audience_id = mb.audience_id
    -> JOIN memberships m ON mb.membership_id = m.id
    -> JOIN benefits b ON mb.benefit_id = b.id
    -> JOIN membershipperiods mp ON mb.membership_id = mp.membership_id
    -> WHERE CURDATE() <= mp.end_date
    ->   AND mp.status = 'ACTIVA';
Empty set (0.00 sec)

```

 10. Ver ciudades con cantidad de empresas

```sql
SELECT
    c.name AS ciudad,
    COUNT(co.id) AS cantidad_empresas
FROM citiesormunicipalities c
LEFT JOIN companies co ON co.city_id = c.code
GROUP BY c.code, c.name
ORDER BY cantidad_empresas DESC;

```

11. Ver encuestas con calificaciones

```sql
SELECT
    p.id AS encuesta_id,
    p.name AS encuesta,
    r.customer_id,
    r.company_id,
    r.poll_id,
    r.rating,
    r.daterating
FROM polls p
JOIN rates r ON r.poll_id = p.id
ORDER BY p.name, r.daterating DESC;

```

13 . Ver productos con audiencia de la empresa

```sql
SELECT
    pr.id AS producto_id,
    pr.name AS nombre_producto,
    co.id AS empresa_id,
    co.name AS nombre_empresa,
    a.description AS audiencia_objetivo
FROM products pr
JOIN companyproducts cp ON cp.product_id = pr.id
JOIN companies co ON co.id = cp.company_id
JOIN audiences a ON a.id = co.audience_id
ORDER BY co.name, pr.name;

```

14. Ver clientes con sus productos favoritos

```sql
SELECT
    c.id AS cliente_id,
    c.name AS nombre_cliente,
    p.id AS producto_id,
    p.name AS nombre_producto
FROM customers c
JOIN favorites f ON f.customer_id = c.id
JOIN details_favorites df ON df.favorite_id = f.id
JOIN products p ON p.id = df.product_id
ORDER BY c.name, p.name;

```

15. Ver planes, periodos, precios y beneficios

```sql
SELECT
    m.id AS plan_id,
    m.name AS nombre_plan,
    p.name AS periodo,
    mp.price AS precio,
    b.description AS beneficio,
    b.detail AS detalle_beneficio
FROM memberships m
JOIN membershipperiods mp
  ON mp.membership_id = m.id
JOIN periods p
  ON p.id = mp.period_id
JOIN membershipbenefits mb
  ON mb.membership_id = m.id AND mb.period_id = mp.period_id
JOIN benefits b
  ON b.id = mb.benefit_id
ORDER BY m.name, p.name, b.description;

```

16. Ver combinaciones empresa-producto-cliente calificados

```sql
SELECT
    c.name AS nombre_empresa,
    cu.name AS nombre_cliente,
    r.rating AS calificacion,
    r.daterating AS fecha_calificacion,
    po.name AS encuesta
FROM rates r
JOIN customers cu ON r.customer_id = cu.id
JOIN companies c ON r.company_id = c.id
JOIN polls po ON r.poll_id = po.id
ORDER BY r.daterating DESC;

```

18. Ver productos ordenados por categoría

```sql
SELECT
    p.name AS nombre_producto,
    c.description AS categoria
FROM products p
JOIN categories c ON p.category_id = c.id
ORDER BY c.description, p.name;

```

19. Ver beneficios por audiencia, incluso vacíos

```sql
SELECT
    a.description AS audiencia,
    COALESCE(b.description, 'Sin beneficio') AS beneficio
FROM audiences a
LEFT JOIN audiencebenefits ab ON a.id = ab.audience_id
LEFT JOIN benefits b ON ab.benefit_id = b.id
ORDER BY a.description, beneficio;

```

20. Ver datos cruzados entre calificaciones, encuestas, productos y clientes

```sql
SELECT
    c.name AS cliente,
    p.name AS producto,
    pl.name AS encuesta,
    r.rating AS calificacion,
    r.daterating AS fecha_calificacion
FROM rates r
JOIN customers c ON r.customer_id = c.id
JOIN companyproducts cp ON r.company_id = cp.company_id
JOIN products p ON cp.product_id = p.id
JOIN polls pl ON r.poll_id = pl.id
ORDER BY r.daterating DESC;

```

## **Historias de Usuario con Funciones Definidas por el Usuario (UDF)**

1. Como analista, quiero una función que calcule el **promedio ponderado de calidad** de un producto basado en sus calificaciones y fecha de evaluación.

```sql
DELIMITER //

CREATE FUNCTION calcular_promedio_ponderado(p_id INT)
RETURNS DECIMAL(5,2)
DETERMINISTIC
BEGIN
    DECLARE promedio_ponderado DECIMAL(5,2);

    SELECT
        IFNULL(SUM(r.rating * (1 / (DATEDIFF(NOW(), r.daterating) + 1))) /
               SUM(1 / (DATEDIFF(NOW(), r.daterating) + 1)), 0)
    INTO promedio_ponderado
    FROM rates r
    JOIN companyproducts cp ON r.company_id = cp.company_id
    WHERE cp.product_id = p_id;

    RETURN promedio_ponderado;
END;
//

DELIMITER ;

```

```sql
SELECT calcular_promedio_ponderado(3) AS promedio_ponderado;

```

2. Como auditor, deseo una función que determine si un producto ha sido **calificado recientemente** (últimos 30 días)

```sql
DELIMITER //

CREATE FUNCTION es_calificacion_reciente(fecha DATE)
RETURNS BOOLEAN
DETERMINISTIC
BEGIN
    RETURN fecha >= CURDATE() - INTERVAL 30 DAY;
END;
//

DELIMITER ;

```

```sql
SELECT es_calificacion_reciente(CURDATE() - INTERVAL 10 DAY) AS reciente_1;

SELECT es_calificacion_reciente(CURDATE() - INTERVAL 50 DAY) AS reciente_2;

```

3. Como desarrollador, quiero una función que reciba un `product_id` y devuelva el **nombre completo de la empresa** que lo vende.

```sql
DELIMITER //

CREATE FUNCTION obtener_empresa_producto(p_id INT)
RETURNS VARCHAR(255)
DETERMINISTIC
BEGIN
    DECLARE nombre_empresa VARCHAR(255);

    SELECT c.name
    INTO nombre_empresa
    FROM companyproducts cp
    JOIN companies c ON cp.company_id = c.id
    WHERE cp.product_id = p_id
    LIMIT 1;

    RETURN nombre_empresa;
END;
//

DELIMITER ;

```

```sql
SELECT obtener_empresa_producto(3) AS empresa;

```

5. Como administrador, quiero una función que valide si una ciudad tiene **más de X empresas registradas**, recibiendo la ciudad y el número como parámetros.

```sql
DELIMITER //

CREATE FUNCTION ciudad_supera_empresas(p_city_id INT, p_limite INT)
RETURNS BOOLEAN
DETERMINISTIC
BEGIN
    DECLARE total_empresas INT;

    SELECT COUNT(*) INTO total_empresas
    FROM companies
    WHERE city_id = p_city_id;

    RETURN total_empresas > p_limite;
END;
//

DELIMITER ;

```

```sql
SELECT ciudad_supera_empresas(5, 10) AS resultado;

```

6. Como gerente, deseo una función que, dado un `rate_id`, me devuelva una **descripción textual de la calificación** (por ejemplo, “Muy bueno”, “Regular”).

```sql
DELIMITER //

CREATE FUNCTION descripcion_calificacion(valor INT)
RETURNS VARCHAR(20)
DETERMINISTIC
BEGIN
    DECLARE descripcion VARCHAR(20);

    CASE valor
        WHEN 5 THEN SET descripcion = 'Excelente';
        WHEN 4 THEN SET descripcion = 'Muy bueno';
        WHEN 3 THEN SET descripcion = 'Regular';
        WHEN 2 THEN SET descripcion = 'Malo';
        WHEN 1 THEN SET descripcion = 'Pésimo';
        ELSE SET descripcion = 'Desconocido';
    END CASE;

    RETURN descripcion;
END;
//

DELIMITER ;

```

```sql
SELECT descripcion_calificacion(4) AS resultado;
```



