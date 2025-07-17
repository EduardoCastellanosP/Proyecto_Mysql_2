# 📊 Proyecto de Base de Datos - Gestión Empresarial

Este proyecto consiste en el diseño e implementación de una base de datos relacional para una plataforma de gestión empresarial. El objetivo es almacenar, consultar y analizar información sobre empresas, productos, clientes y sus interacciones.

## 🧱 Estructura de la Base de Datos

La base de datos está compuesta por las siguientes tablas principales:

- `companies`: Información de las empresas registradas.
- `products`: Detalles de los productos disponibles.
- `clients`: Datos de los clientes.
- `companyproducts`: Relación entre empresas y productos ofrecidos.
- `favorites`: Productos marcados como favoritos por los clientes.
- `ratings`: Calificaciones dadas a productos.
- `unitmeasures`: Unidades de medida utilizadas para los productos.
- `memberships`: Planes de membresía contratados por los clientes.
- `categories`: Clasificación de los productos.

## 🛠️ Tecnologías

- **MySQL** 8.x
- **Workbench** o **terminal de Linux** para gestión de la base de datos
- Scripts `.sql` para creación e inserción de datos

## 📂 Contenido del Repositorio

- `schema.sql`: Script para crear las tablas y relaciones.
- `inserts.sql`: Datos de prueba para poblar la base de datos.
- `queries.sql`: Consultas SQL útiles para análisis.
- `README.md`: Documentación del proyecto.

## 📌 Ejemplos de Consultas

Algunas consultas desarrolladas:

- Total de clientes por ciudad
- Promedio de precios por unidad de medida
- Calificación promedio de productos favoritos
- Porcentaje de productos evaluados
- Desviación estándar de precios por categoría

