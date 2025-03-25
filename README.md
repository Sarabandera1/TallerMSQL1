# TallerMSQL1
Base de datos
-- Creación de la base de datos
CREATE DATABASE vtaszfs;
USE vtaszfs;
-- Tabla Clientes
CREATE TABLE Clientes (
id INT PRIMARY KEY AUTO_INCREMENT,
nombre VARCHAR(100),
email VARCHAR(100) UNIQUE
puesto VARCHAR(50),
salario DECIMAL(10, 2),
fecha_contratacion DATE
);
-- Tabla Proveedores
CREATE TABLE Proveedores (
id INT PRIMARY KEY AUTO_INCREMENT,
nombre VARCHAR(100),
contacto VARCHAR(100),
telefono VARCHAR(20),
direccion VARCHAR(255)
);
-- Tabla TiposProductos
CREATE TABLE TiposProductos (
id INT PRIMARY KEY AUTO_INCREMENT,
tipo_nombre VARCHAR(100),
descripcion TEXT
);
-- Tabla Productos
CREATE TABLE Productos (
id INT PRIMARY KEY AUTO_INCREMENT,
nombre VARCHAR(100),
precio DECIMAL(10, 2),
proveedor_id INT,
tipo_id INT,
FOREIGN KEY (proveedor_id) REFERENCES Proveedores(id),
FOREIGN KEY (tipo_id) REFERENCES TiposProductos(id)
);
-- Tabla Pedidos
CREATE TABLE Pedidos (
id INT PRIMARY KEY AUTO_INCREMENT,
cliente_id INT,
fecha DATE,
total DECIMAL(10, 2),
FOREIGN KEY (cliente_id) REFERENCES Clientes(id)
);
-- Tabla DetallesPedido
CREATE TABLE DetallesPedido ();
ATE TABLE DetallesPedido (
id INT PRIMARY KEY AUTO_INCREMENT,
pedido_id INT,
producto_id INT,
cantidad INT,
precio DECIMAL(10, 2),
FOREIGN KEY (pedido_id) REFERENCES Pedidos(id),
FOREIGN KEY (producto_id) REFERENCES Productos(id)
);-- Tabla UbicacionCliente


# 1. Normalización
1. Crear una tabla HistorialPedidos que almacene cambios en los pedidos.
2. Evaluar la tabla Clientes para eliminar datos redundantes y normalizar hasta 3NF.
3. Separar la tabla Empleados en una tabla de DatosEmpleados y otra para Puestos .
4. Revisar la relación Clientes y UbicacionCliente para evitar duplicación de datos.
5. Normalizar Proveedores para tener ContactoProveedores en otra tabla.
6. Crear una tabla de Telefonos para almacenar múltiples números por cliente.
7. Transformar TiposProductos en una relación categórica jerárquica.
8. Normalizar Pedidos y DetallesPedido para evitar inconsistencias de precios.
9. Usar una relación de muchos a muchos para Empleados y Proveedores .
10. Convertir la tabla UbicacionCliente en una relación genérica de Ubicaciones .

CREATE TABLE HistorialPedidos (
    id INT PRIMARY KEY AUTO_INCREMENT,
    pedido_id INT,
    cambio_fecha TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    descripcion TEXT,
    FOREIGN KEY (pedido_id) REFERENCES Pedidos(id)
);

CREATE TABLE Direcciones (
    id INT PRIMARY KEY AUTO_INCREMENT,
    cliente_id INT,
    direccion VARCHAR(255),
    ciudad VARCHAR(100),
    estado VARCHAR(50),
    codigo_postal VARCHAR(10),
    pais VARCHAR(50),
    FOREIGN KEY (cliente_id) REFERENCES Clientes(id)
);
CREATE TABLE Puestos (
    id INT PRIMARY KEY AUTO_INCREMENT,
    nombre VARCHAR(50)
);
CREATE TABLE DatosEmpleados (
    id INT PRIMARY KEY AUTO_INCREMENT,
    nombre VARCHAR(100),
    salario DECIMAL(10, 2),
    fecha_contratacion DATE,
    puesto_id INT,
    FOREIGN KEY (puesto_id) REFERENCES Puestos(id)
);
CREATE TABLE Ubicaciones (
    id INT PRIMARY KEY AUTO_INCREMENT,
    cliente_id INT,
    tipo_ubicacion VARCHAR(50),  -- "envío", "facturación", etc.
    direccion VARCHAR(255),
    ciudad VARCHAR(100),
    estado VARCHAR(50),
    codigo_postal VARCHAR(10),
    pais VARCHAR(50),
    FOREIGN KEY (cliente_id) REFERENCES Clientes(id)
);
CREATE TABLE ContactoProveedores (
    id INT PRIMARY KEY AUTO_INCREMENT,
    proveedor_id INT,
    nombre_contacto VARCHAR(100),
    telefono VARCHAR(20),
    correo VARCHAR(100),
    FOREIGN KEY (proveedor_id) REFERENCES Proveedores(id)
);
CREATE TABLE Telefonos (
    id INT PRIMARY KEY AUTO_INCREMENT,
    cliente_id INT,
    numero_telefono VARCHAR(20),
    tipo_telefono VARCHAR(50),  -- Ejemplo: "móvil", "fijo", "oficina"
    FOREIGN KEY (cliente_id) REFERENCES Clientes(id)
);
CREATE TABLE TiposProductos (
    id INT PRIMARY KEY AUTO_INCREMENT,
    tipo_nombre VARCHAR(100),
    descripcion TEXT,
    categoria_padre_id INT,
    FOREIGN KEY (categoria_padre_id) REFERENCES TiposProductos(id)  -- Auto-relación
);
CREATE TABLE DetallesPedido (
    id INT PRIMARY KEY AUTO_INCREMENT,
    pedido_id INT,
    producto_id INT,
    cantidad INT,
    precio DECIMAL(10, 2),
    FOREIGN KEY (pedido_id) REFERENCES Pedidos(id),
    FOREIGN KEY (producto_id) REFERENCES Productos(id)
);
CREATE TABLE EmpleadoProveedor (
    empleado_id INT,
    proveedor_id INT,
    PRIMARY KEY (empleado_id, proveedor_id),
    FOREIGN KEY (empleado_id) REFERENCES DatosEmpleados(id),
    FOREIGN KEY (proveedor_id) REFERENCES Proveedores(id)
);



# 2. Joins
1. Obtener la lista de todos los pedidos con los nombres de clientes usando INNER JOIN .
2. Listar los productos y proveedores que los suministran con INNER JOIN .
3. Mostrar los pedidos y las ubicaciones de los clientes con LEFT JOIN .
4. Consultar los empleados que han registrado pedidos, incluyendo empleados sin pedidos
( LEFT JOIN ).
5. Obtener el tipo de producto y los productos asociados con INNER JOIN .
6. Listar todos los clientes y el número de pedidos realizados con COUNT y GROUP BY .
7. Combinar Pedidos y Empleados para mostrar qué empleados gestionaron pedidos
específicos.
8. Mostrar productos que no han sido pedidos ( RIGHT JOIN ).
9. Mostrar el total de pedidos y ubicación de clientes usando múltiples JOIN .
10. Unir Proveedores , Productos , y TiposProductos para un listado completo de inventario.


SELECT Pedidos.id AS pedido_id, Clientes.nombre AS cliente_nombre
FROM Pedidos
INNER JOIN Clientes ON Pedidos.cliente_id = Clientes.id;

SELECT Productos.nombre AS producto_nombre, Proveedores.nombre AS proveedor_nombre
FROM Productos
INNER JOIN Proveedores ON Productos.proveedor_id = Proveedores.id;

SELECT Pedidos.id AS pedido_id, Clientes.nombre AS cliente_nombre, Ubicaciones.direccion
FROM Pedidos
LEFT JOIN Clientes ON Pedidos.cliente_id = Clientes.id
LEFT JOIN Ubicaciones ON Clientes.id = Ubicaciones.cliente_id;

SELECT DatosEmpleados.nombre AS empleado_nombre, Pedidos.id AS pedido_id
FROM DatosEmpleados
LEFT JOIN Pedidos ON DatosEmpleados.id = Pedidos.empleado_id;  -- Suponiendo que la relación con los empleados es a través de una columna 'empleado_id' en Pedidos.

SELECT TiposProductos.tipo_nombre, Productos.nombre AS producto_nombre
FROM Productos
INNER JOIN TiposProductos ON Productos.tipo_id = TiposProductos.id;

SELECT Clientes.nombre AS cliente_nombre, COUNT(Pedidos.id) AS num_pedidos
FROM Clientes
LEFT JOIN Pedidos ON Clientes.id = Pedidos.cliente_id
GROUP BY Clientes.id;

SELECT Pedidos.id AS pedido_id, DatosEmpleados.nombre AS empleado_nombre
FROM Pedidos
INNER JOIN DatosEmpleados ON Pedidos.empleado_id = DatosEmpleados.id;  -- Asegúrate de tener un campo 'empleado_id' en la tabla Pedidos.

SELECT Productos.nombre AS producto_nombre
FROM Productos
RIGHT JOIN DetallesPedido ON Productos.id = DetallesPedido.producto_id
WHERE DetallesPedido.producto_id IS NULL;

SELECT SUM(Pedidos.total) AS total_pedidos, Ubicaciones.direccion
FROM Pedidos
INNER JOIN Clientes ON Pedidos.cliente_id = Clientes.id
INNER JOIN Ubicaciones ON Clientes.id = Ubicaciones.cliente_id
GROUP BY Ubicaciones.id;

SELECT Proveedores.nombre AS proveedor_nombre, Productos.nombre AS producto_nombre, TiposProductos.tipo_nombre
FROM Productos
INNER JOIN Proveedores ON Productos.proveedor_id = Proveedores.id
INNER JOIN TiposProductos ON Productos.tipo_id = TiposProductos.id;


# 3. Consultas Simples
1. Seleccionar todos los productos con precio mayor a $50.
2. Consultar clientes registrados en una ciudad específica.
3. Mostrar empleados contratados en los últimos 2 años.
4. Seleccionar proveedores que suministran más de 5 productos.
5. Listar clientes que no tienen dirección registrada en UbicacionCliente .
6. Calcular el total de ventas por cada cliente.7. Mostrar el salario promedio de los empleados.
8. Consultar el tipo de productos disponibles en TiposProductos .
9. Seleccionar los 3 productos más caros.
10. Consultar el cliente con el mayor número de pedidos.

SELECT nombre, precio
FROM Productos
WHERE precio > 50;

SELECT Clientes.nombre, Clientes.email
FROM Clientes
INNER JOIN Ubicaciones ON Clientes.id = Ubicaciones.cliente_id
WHERE Ubicaciones.ciudad = 'CiudadEspecifica';  -- Reemplaza 'CiudadEspecifica' con el nombre de la ciudad

SELECT nombre, fecha_contratacion
FROM DatosEmpleados
WHERE fecha_contratacion >= CURDATE() - INTERVAL 2 YEAR;

SELECT Proveedores.nombre, COUNT(Productos.id) AS num_productos
FROM Proveedores
INNER JOIN Productos ON Proveedores.id = Productos.proveedor_id
GROUP BY Proveedores.id
HAVING COUNT(Productos.id) > 5;

SELECT Clientes.nombre
FROM Clientes
LEFT JOIN Ubicaciones ON Clientes.id = Ubicaciones.cliente_id
WHERE Ubicaciones.id IS NULL;

SELECT Clientes.nombre, SUM(Pedidos.total) AS total_ventas
FROM Clientes
INNER JOIN Pedidos ON Clientes.id = Pedidos.cliente_id
GROUP BY Clientes.id;

SELECT AVG(salario) AS salario_promedio
FROM DatosEmpleados;


SELECT tipo_nombre
FROM TiposProductos;

SELECT nombre, precio
FROM Productos
ORDER BY precio DESC
LIMIT 3;

SELECT Clientes.nombre, COUNT(Pedidos.id) AS num_pedidos
FROM Clientes
INNER JOIN Pedidos ON Clientes.id = Pedidos.cliente_id
GROUP BY Clientes.id
ORDER BY num_pedidos DESC
LIMIT 1;

4. Consultas Multitabla
1. Listar todos los pedidos y el cliente asociado.
2. Mostrar la ubicación de cada cliente en sus pedidos.
3. Listar productos junto con el proveedor y tipo de producto.
4. Consultar todos los empleados que gestionan pedidos de clientes en una ciudad específica.
5. Consultar los 5 productos más vendidos.
6. Obtener la cantidad total de pedidos por cliente y ciudad.
7. Listar clientes y proveedores en la misma ciudad.
8. Mostrar el total de ventas agrupado por tipo de producto.
9. Listar empleados que gestionan pedidos de productos de un proveedor específico.
10. Obtener el ingreso total de cada proveedor a partir de los productos vendidos.

SELECT Pedidos.id AS pedido_id, Clientes.nombre AS cliente_nombre
FROM Pedidos
INNER JOIN Clientes ON Pedidos.cliente_id = Clientes.id;

SELECT Pedidos.id AS pedido_id, Clientes.nombre AS cliente_nombre, Ubicaciones.direccion, Ubicaciones.ciudad
FROM Pedidos
INNER JOIN Clientes ON Pedidos.cliente_id = Clientes.id
INNER JOIN Ubicaciones ON Clientes.id = Ubicaciones.cliente_id;

SELECT Productos.nombre AS producto_nombre, Proveedores.nombre AS proveedor_nombre, TiposProductos.tipo_nombre
FROM Productos
INNER JOIN Proveedores ON Productos.proveedor_id = Proveedores.id
INNER JOIN TiposProductos ON Productos.tipo_id = TiposProductos.id;

SELECT DISTINCT DatosEmpleados.nombre AS empleado_nombre
FROM DatosEmpleados
INNER JOIN Pedidos ON DatosEmpleados.id = Pedidos.empleado_id
INNER JOIN Clientes ON Pedidos.cliente_id = Clientes.id
INNER JOIN Ubicaciones ON Clientes.id = Ubicaciones.cliente_id
WHERE Ubicaciones.ciudad = 'CiudadEspecifica';  -- Reemplaza 'CiudadEspecifica' con la ciudad deseada

SELECT Productos.nombre AS producto_nombre, SUM(DetallesPedido.cantidad) AS cantidad_vendida
FROM DetallesPedido
INNER JOIN Productos ON DetallesPedido.producto_id = Productos.id
GROUP BY Productos.id
ORDER BY cantidad_vendida DESC
LIMIT 5;

SELECT Clientes.nombre AS cliente_nombre, Ubicaciones.ciudad, COUNT(Pedidos.id) AS num_pedidos
FROM Pedidos
INNER JOIN Clientes ON Pedidos.cliente_id = Clientes.id
INNER JOIN Ubicaciones ON Clientes.id = Ubicaciones.cliente_id
GROUP BY Clientes.id, Ubicaciones.ciudad;

SELECT Clientes.nombre AS cliente_nombre, Proveedores.nombre AS proveedor_nombre, Ubicaciones.ciudad
FROM Clientes
INNER JOIN Ubicaciones ON Clientes.id = Ubicaciones.cliente_id
INNER JOIN Productos ON Ubicaciones.cliente_id = Productos.proveedor_id
INNER JOIN Proveedores ON Productos.proveedor_id = Proveedores.id
WHERE Ubicaciones.ciudad = 'CiudadEspecifica';  -- Reemplaza 'CiudadEspecifica' con la ciudad deseada

SELECT TiposProductos.tipo_nombre, SUM(DetallesPedido.precio * DetallesPedido.cantidad) AS total_ventas
FROM DetallesPedido
INNER JOIN Productos ON DetallesPedido.producto_id = Productos.id
INNER JOIN TiposProductos ON Productos.tipo_id = TiposProductos.id
GROUP BY TiposProductos.id;

SELECT DISTINCT DatosEmpleados.nombre AS empleado_nombre
FROM DatosEmpleados
INNER JOIN Pedidos ON DatosEmpleados.id = Pedidos.empleado_id
INNER JOIN DetallesPedido ON Pedidos.id = DetallesPedido.pedido_id
INNER JOIN Productos ON DetallesPedido.producto_id = Productos.id
WHERE Productos.proveedor_id = 'ProveedorIdEspecifico';  -- Reemplaza 'ProveedorIdEspecifico' con el ID del proveedor

SELECT Proveedores.nombre AS proveedor_nombre, SUM(DetallesPedido.precio * DetallesPedido.cantidad) AS ingreso_total
FROM DetallesPedido
INNER JOIN Productos ON DetallesPedido.producto_id = Productos.id
INNER JOIN Proveedores ON Productos.proveedor_id = Proveedores.id
GROUP BY Proveedores.id;

5. Subconsultas
1. Consultar el producto más caro en cada categoría.
2. Encontrar el cliente con mayor total en pedidos.
3. Listar empleados que ganan más que el salario promedio.
4. Consultar productos que han sido pedidos más de 5 veces.
5. Listar pedidos cuyo total es mayor al promedio de todos los pedidos.
6. Seleccionar los 3 proveedores con más productos.
7. Consultar productos con precio superior al promedio en su tipo.
8. Mostrar clientes que han realizado más pedidos que la media.
9. Encontrar productos cuyo precio es mayor que el promedio de todos los productos.
10. Mostrar empleados cuyo salario es menor al promedio del departamento.

SELECT p.nombre, p.precio, p.tipo_id
FROM Productos p
WHERE p.precio = (
    SELECT MAX(precio)
    FROM Productos
    WHERE tipo_id = p.tipo_id
);
SELECT nombre
FROM Clientes
WHERE id = (
    SELECT cliente_id
    FROM Pedidos
    GROUP BY cliente_id
    ORDER BY SUM(total) DESC
    LIMIT 1
);
SELECT nombre, salario
FROM DatosEmpleados
WHERE salario > (
    SELECT AVG(salario)
    FROM DatosEmpleados
);
SELECT nombre
FROM Productos
WHERE id IN (
    SELECT producto_id
    FROM DetallesPedido
    GROUP BY producto_id
    HAVING COUNT(*) > 5
);
SELECT id, total
FROM Pedidos
WHERE total > (
    SELECT AVG(total)
    FROM Pedidos
);
SELECT nombre
FROM Proveedores
WHERE id IN (
    SELECT proveedor_id
    FROM Productos
    GROUP BY proveedor_id
    ORDER BY COUNT(*) DESC
    LIMIT 3
);
SELECT nombre, precio, tipo_id
FROM Productos
WHERE precio > (
    SELECT AVG(precio)
    FROM Productos
    WHERE tipo_id = Productos.tipo_id
);
SELECT nombre
FROM Clientes
WHERE id IN (
    SELECT cliente_id
    FROM Pedidos
    GROUP BY cliente_id
    HAVING COUNT(*) > (
        SELECT AVG(num_pedidos)
        FROM (
            SELECT cliente_id, COUNT(*) AS num_pedidos
            FROM Pedidos
            GROUP BY cliente_id
        ) AS subquery
    )
);
SELECT nombre, precio
FROM Productos
WHERE precio > (
    SELECT AVG(precio)
    FROM Productos
);
SELECT nombre, salario
FROM DatosEmpleados
WHERE salario < (
    SELECT AVG(salario)
    FROM DatosEmpleados
    WHERE puesto_id = DatosEmpleados.puesto_id
);



6. Procedimientos Almacenados
1. Crear un procedimiento para actualizar el precio de todos los productos de un proveedor.
2. Un procedimiento que devuelva la dirección de un cliente por ID.
3. Crear un procedimiento que registre un pedido nuevo y sus detalles.
4. Un procedimiento para calcular el total de ventas de un cliente.
5. Crear un procedimiento para obtener los empleados por puesto.
6. Un procedimiento que actualice el salario de empleados por puesto.
7. Crear un procedimiento que liste los pedidos entre dos fechas.8. Un procedimiento para aplicar un descuento a productos de una categoría.
9. Crear un procedimiento que liste todos los proveedores de un tipo de producto.
10. Un procedimiento que devuelva el pedido de mayor valor.

DELIMITER $$

CREATE PROCEDURE actualizar_precio_proveedor(
    IN proveedor_id INT,
    IN nuevo_precio DECIMAL(10, 2)
)
BEGIN
    UPDATE Productos
    SET precio = nuevo_precio
    WHERE proveedor_id = proveedor_id;
END $$

DELIMITER ;
DELIMITER $$

CREATE PROCEDURE obtener_direccion_cliente(
    IN cliente_id INT
)
BEGIN
    SELECT direccion
    FROM Ubicaciones
    WHERE cliente_id = cliente_id;
END $$

DELIMITER ;
DELIMITER $$

CREATE PROCEDURE registrar_pedido(
    IN cliente_id INT,
    IN total DECIMAL(10, 2),
    IN productos JSON  -- Suponiendo que los productos son pasados como un JSON: [{"producto_id": 1, "cantidad": 2, "precio": 10.00}, ...]
)
BEGIN
    DECLARE pedido_id INT;
    INSERT INTO Pedidos (cliente_id, fecha, total)
    VALUES (cliente_id, CURDATE(), total);
    SET pedido_id = LAST_INSERT_ID();
    INSERT INTO DetallesPedido (pedido_id, producto_id, cantidad, precio)
    SELECT pedido_id, p.producto_id, p.cantidad, p.precio
    FROM JSON_TABLE(productos, '$[*]' COLUMNS (
        producto_id INT PATH '$.producto_id',
        cantidad INT PATH '$.cantidad',
        precio DECIMAL(10, 2) PATH '$.precio'
    )) AS p;
END $$

DELIMITER ;
DELIMITER $$

CREATE PROCEDURE total_ventas_cliente(
    IN cliente_id INT
)
BEGIN
    SELECT SUM(total) AS total_ventas
    FROM Pedidos
    WHERE cliente_id = cliente_id;
END $$

DELIMITER ;
DELIMITER $$

CREATE PROCEDURE empleados_por_puesto(
    IN puesto_id INT
)
BEGIN
    SELECT nombre
    FROM DatosEmpleados
    WHERE puesto_id = puesto_id;
END $$

DELIMITER ;
DELIMITER $$

CREATE PROCEDURE actualizar_salario_por_puesto(
    IN puesto_id INT,
    IN nuevo_salario DECIMAL(10, 2)
)
BEGIN
    UPDATE DatosEmpleados
    SET salario = nuevo_salario
    WHERE puesto_id = puesto_id;
END $$

DELIMITER ;
DELIMITER $$

CREATE PROCEDURE listar_pedidos_por_fecha(
    IN fecha_inicio DATE,
    IN fecha_fin DATE
)
BEGIN
    SELECT *
    FROM Pedidos
    WHERE fecha BETWEEN fecha_inicio AND fecha_fin;
END $$

DELIMITER ;
DELIMITER $$

CREATE PROCEDURE aplicar_descuento_categoria(
    IN tipo_id INT,
    IN descuento DECIMAL(5, 2)  -- Porcentaje de descuento (por ejemplo, 10 para un 10%)
)
BEGIN
    UPDATE Productos
    SET precio = precio - (precio * descuento / 100)
    WHERE tipo_id = tipo_id;
END $$

DELIMITER ;
DELIMITER $$

CREATE PROCEDURE proveedores_por_tipo_producto(
    IN tipo_id INT
)
BEGIN
    SELECT DISTINCT Proveedores.nombre
    FROM Productos
    INNER JOIN Proveedores ON Productos.proveedor_id = Proveedores.id
    WHERE Productos.tipo_id = tipo_id;
END $$

DELIMITER ;
DELIMITER $$

CREATE PROCEDURE pedido_mayor_valor()
BEGIN
    SELECT *
    FROM Pedidos
    ORDER BY total DESC
    LIMIT 1;
END $$

DELIMITER ;


7. Funciones Definidas por el Usuario
1. Crear una función que reciba una fecha y devuelva los días transcurridos.
2. Crear una función para calcular el total con impuesto de un monto.
3. Una función que devuelva el total de pedidos de un cliente específico.
4. Crear una función para aplicar un descuento a un producto.
5. Una función que indique si un cliente tiene dirección registrada.
6. Crear una función que devuelva el salario anual de un empleado.
7. Una función para calcular el total de ventas de un tipo de producto.
8. Crear una función para devolver el nombre de un cliente por ID.
9. Una función que reciba el ID de un pedido y devuelva su total.
10. Crear una función que indique si un producto está en inventario.


DELIMITER $$

CREATE FUNCTION dias_transcurridos(fecha_inicial DATE) 
RETURNS INT
DETERMINISTIC
BEGIN
    RETURN DATEDIFF(CURDATE(), fecha_inicial);
END $$

DELIMITER ;
DELIMITER $$

CREATE FUNCTION total_con_impuesto(monto DECIMAL(10, 2), impuesto DECIMAL(5, 2))
RETURNS DECIMAL(10, 2)
DETERMINISTIC
BEGIN
    RETURN monto + (monto * impuesto / 100);
END $$

DELIMITER ;
DELIMITER $$

CREATE FUNCTION total_pedidos_cliente(cliente_id INT)
RETURNS DECIMAL(10, 2)
DETERMINISTIC
BEGIN
    DECLARE total DECIMAL(10, 2);
    SELECT SUM(total) INTO total
    FROM Pedidos
    WHERE cliente_id = cliente_id;
    RETURN total;
END $$

DELIMITER ;
DELIMITER $$

CREATE FUNCTION aplicar_descuento_producto(producto_id INT, descuento DECIMAL(5, 2))
RETURNS DECIMAL(10, 2)
DETERMINISTIC
BEGIN
    DECLARE precio DECIMAL(10, 2);
    SELECT precio INTO precio
    FROM Productos
    WHERE id = producto_id;
    RETURN precio - (precio * descuento / 100);
END $$

DELIMITER ;
DELIMITER $$

CREATE FUNCTION cliente_con_direccion(cliente_id INT)
RETURNS BOOLEAN
DETERMINISTIC
BEGIN
    DECLARE direccion_count INT;
    SELECT COUNT(*) INTO direccion_count
    FROM Ubicaciones
    WHERE cliente_id = cliente_id;
    RETURN direccion_count > 0;
END $$

DELIMITER ;
DELIMITER $$

CREATE FUNCTION salario_anual(empleado_id INT)
RETURNS DECIMAL(10, 2)
DETERMINISTIC
BEGIN
    DECLARE salario DECIMAL(10, 2);
    SELECT salario INTO salario
    FROM DatosEmpleados
    WHERE id = empleado_id;
    RETURN salario * 12;
END $$

DELIMITER ;
DELIMITER $$

CREATE FUNCTION total_ventas_tipo_producto(tipo_id INT)
RETURNS DECIMAL(10, 2)
DETERMINISTIC
BEGIN
    DECLARE total DECIMAL(10, 2);
    SELECT SUM(DetallesPedido.precio * DetallesPedido.cantidad) INTO total
    FROM DetallesPedido
    INNER JOIN Productos ON DetallesPedido.producto_id = Productos.id
    WHERE Productos.tipo_id = tipo_id;
    RETURN total;
END $$

DELIMITER ;
DELIMITER $$

CREATE FUNCTION nombre_cliente(cliente_id INT)
RETURNS VARCHAR(100)
DETERMINISTIC
BEGIN
    DECLARE cliente_nombre VARCHAR(100);
    SELECT nombre INTO cliente_nombre
    FROM Clientes
    WHERE id = cliente_id;
    RETURN cliente_nombre;
END $$

DELIMITER ;
DELIMITER $$

CREATE FUNCTION total_pedido(pedido_id INT)
RETURNS DECIMAL(10, 2)
DETERMINISTIC
BEGIN
    DECLARE total DECIMAL(10, 2);
    SELECT total INTO total
    FROM Pedidos
    WHERE id = pedido_id;
    RETURN total;
END $$

DELIMITER ;
DELIMITER $$

CREATE FUNCTION producto_en_inventario(producto_id INT)
RETURNS BOOLEAN
DETERMINISTIC
BEGIN
    DECLARE cantidad INT;
    SELECT SUM(cantidad) INTO cantidad
    FROM DetallesPedido
    WHERE producto_id = producto_id;
    RETURN cantidad > 0;
END $$

DELIMITER ;

8. Triggers
1. Crear un trigger que registre en HistorialSalarios cada cambio de salario de empleados.
2. Crear un trigger que evite borrar productos con pedidos activos.
3. Un trigger que registre en HistorialPedidos cada actualización en Pedidos .
4. Crear un trigger que actualice el inventario al registrar un pedido.
5. Un trigger que evite actualizaciones de precio a menos de $1.
6. Crear un trigger que registre la fecha de creación de un pedido en HistorialPedidos .
7. Un trigger que mantenga el precio total de cada pedido en Pedidos .
8. Crear un trigger para validar que UbicacionCliente no esté vacío al crear un cliente.
9. Un trigger que registre en LogActividades cada modificación en Proveedores .
10. Crear un trigger que registre en HistorialContratos cada cambio en Empleados .

DELIMITER $$

CREATE TRIGGER after_salario_update
AFTER UPDATE ON DatosEmpleados
FOR EACH ROW
BEGIN
    IF OLD.salario != NEW.salario THEN
        INSERT INTO HistorialSalarios (empleado_id, salario_anterior, salario_nuevo, fecha_cambio)
        VALUES (NEW.id, OLD.salario, NEW.salario, NOW());
    END IF;
END $$

DELIMITER ;
DELIMITER $$

CREATE TRIGGER before_producto_delete
BEFORE DELETE ON Productos
FOR EACH ROW
BEGIN
    DECLARE cantidad_pedidos INT;
    SELECT COUNT(*) INTO cantidad_pedidos
    FROM DetallesPedido
    WHERE producto_id = OLD.id;
END $$

DELIMITER ;
DELIMITER $$

CREATE TRIGGER after_pedido_update
AFTER UPDATE ON Pedidos
FOR EACH ROW
BEGIN
    INSERT INTO HistorialPedidos (pedido_id, cambio_fecha, descripcion)
    VALUES (NEW.id, NOW(), CONCAT('Pedido actualizado: Total anterior: ', OLD.total, ', Total nuevo: ', NEW.total));
END $$

DELIMITER ;
DELIMITER $$

CREATE TRIGGER after_pedido_insert
AFTER INSERT ON Pedidos
FOR EACH ROW
BEGIN
    DECLARE p_id INT;
    DECLARE cantidad INT;
    DECLARE precio DECIMAL(10, 2);

END $$

DELIMITER ;
DELIMITER $$

CREATE TRIGGER before_precio_update
BEFORE UPDATE ON Productos
FOR EACH ROW
BEGIN
    IF NEW.precio < 1 THEN
        SIGNAL SQLSTATE '45000' SET MESSAGE_TEXT = 'El precio no puede ser menor a $1.';
    END IF;
END $$

DELIMITER ;

DELIMITER $$

CREATE TRIGGER before_pedido_insert
BEFORE INSERT ON Pedidos
FOR EACH ROW
BEGIN
    SET NEW.fecha = IFNULL(NEW.fecha, CURDATE()); -- Si no se especifica fecha, asignar la fecha actual
    INSERT INTO HistorialPedidos (pedido_id, cambio_fecha, descripcion)
    VALUES (NEW.id, NEW.fecha, 'Pedido creado.');
END $$

DELIMITER ;
DELIMITER $$

CREATE TRIGGER after_detallepedido_insert
AFTER INSERT ON DetallesPedido
FOR EACH ROW
BEGIN
    DECLARE total DECIMAL(10, 2);
    SELECT SUM(precio * cantidad) INTO total
    FROM DetallesPedido
    WHERE pedido_id = NEW.pedido_id;
    UPDATE Pedidos
    SET total = total
    WHERE id = NEW.pedido_id;
END $$

DELIMITER ;
DELIMITER $$

CREATE TRIGGER before_cliente_insert
BEFORE INSERT ON Clientes
FOR EACH ROW
BEGIN
    DECLARE direccion_count INT;
    SELECT COUNT(*) INTO direccion_count
    FROM Ubicaciones
    WHERE cliente_id = NEW.id;

  IF direccion_count = 0 THEN
        SIGNAL SQLSTATE '45000' SET MESSAGE_TEXT = 'El cliente debe tener al menos una dirección registrada.';
    END IF;
END $$

DELIMITER ;
DELIMITER $$

CREATE TRIGGER after_proveedor_update
AFTER UPDATE ON Proveedores
FOR EACH ROW
BEGIN
    INSERT INTO LogActividades (tabla, actividad, fecha)
    VALUES ('Proveedores', CONCAT('Proveedor actualizado: ', OLD.nombre, ' a ', NEW.nombre), NOW());
END $$

DELIMITER ;



Ejercicios Combinados de Funciones y Consultas
Función de Descuento por Categoría de Producto
Objetivo: Crear una función que aplique un descuento sobre el precio de un producto si
pertenece a una categoría específica.
Pasos:
1. Crear una función CalcularDescuento que reciba el tipo_id del producto y el precio
original, y aplique un descuento del 10% si el tipo es "Electrónica".
2. Realizar una consulta para mostrar el nombre del producto, el precio original y el precio
con descuento.Función para Obtener la Edad de un Cliente y Filtrar Clientes Mayores de
Edad
Objetivo: Crear una función que calcule la edad de un cliente en función de su fecha de
nacimiento y luego usarla para listar solo los clientes mayores de 18 años.
Pasos:
1. Crear la función CalcularEdad que reciba la fecha de nacimiento y calcule la edad.
2. Consultar todos los clientes y mostrar solo aquellos que sean mayores de 18 años.
Función de Cálculo de Impuesto y Consulta de Productos con Precio Final
Objetivo: Crear una función que calcule el precio final de un producto aplicando un
impuesto del 15% y luego mostrar una lista de productos con el precio final incluido.
Pasos:
1. Crear la función CalcularImpuesto que reciba el precio del producto y aplique el
impuesto.
2. Mostrar el nombre del producto, el precio original y el precio final con impuesto.
DELIMITER $$

-- Función para aplicar descuento si el tipo es "Electrónica"
CREATE FUNCTION CalcularDescuento(tipo_id INT, precio DECIMAL(10, 2))
RETURNS DECIMAL(10, 2)
DETERMINISTIC
BEGIN
    DECLARE descuento DECIMAL(5, 2);
    DECLARE nuevo_precio DECIMAL(10, 2);
    SELECT tipo_nombre INTO descuento
    FROM TiposProductos
    WHERE id = tipo_id;
    IF descuento = 'Electrónica' THEN
        SET nuevo_precio = precio * 0.90; -- Aplicar 10% de descuento
    ELSE
        SET nuevo_precio = precio; -- Sin descuento
    END IF;
    RETURN nuevo_precio;
END $$

DELIMITER ;
SELECT 
    P.nombre AS Producto,
    P.precio AS Precio_Original,
    CalcularDescuento(P.tipo_id, P.precio) AS Precio_Con_Descuento
FROM Productos P;
DELIMITER $$

-- Función para calcular la edad de un cliente
CREATE FUNCTION CalcularEdad(fecha_nacimiento DATE)
RETURNS INT
DETERMINISTIC
BEGIN
    DECLARE edad INT;
    SET edad = TIMESTAMPDIFF(YEAR, fecha_nacimiento, CURDATE());
    IF (MONTH(fecha_nacimiento) > MONTH(CURDATE())) OR (MONTH(fecha_nacimiento) = MONTH(CURDATE()) AND DAY(fecha_nacimiento) > DAY(CURDATE())) THEN
        SET edad = edad - 1;
    END IF;
    RETURN edad;
END $$

DELIMITER ;
SELECT 
    C.nombre AS Cliente,
    CalcularEdad(C.fecha_nacimiento) AS Edad
FROM Clientes C
WHERE CalcularEdad(C.fecha_nacimiento) >= 18;
DELIMITER $$

-- Función para calcular el precio con impuesto
CREATE FUNCTION CalcularImpuesto(precio DECIMAL(10, 2))
RETURNS DECIMAL(10, 2)
DETERMINISTIC
BEGIN
    DECLARE precio_final DECIMAL(10, 2);
    SET precio_final = precio * 1.15;
    RETURN precio_final;
END $$

DELIMITER ;
SELECT 
    P.nombre AS Producto,
    P.precio AS Precio_Original,
    CalcularImpuesto(P.precio) AS Precio_Final_Con_Impuesto
FROM Productos P;


Función para Calcular el Total de Pedidos de un Cliente
Objetivo: Crear una función que calcule el total de los pedidos de un cliente y usarla para
mostrar los clientes con total de pedidos mayor a $1000.
Pasos:
1. Crear la función TotalPedidosCliente que reciba el ID de un cliente y calcule el total
de todos sus pedidos.
2. Realizar una consulta que muestre el nombre del cliente y su total de pedidos, y filtrar
clientes con un total mayor a $1000.



Función para Calcular el Salario Anual de un Empleado
Objetivo: Crear una función que calcule el salario anual de un empleado y usarla para listar
todos los empleados con un salario anual mayor a $50,000.
Pasos:
1. Crear la función SalarioAnual que reciba el salario mensual y lo multiplique por 12.
2. Realizar una consulta que muestre el nombre del empleado y su salario anual, filtrando
empleados con salario mayor a $50,000.
Función de Bonificación y Consulta de Salarios Ajustados
Objetivo: Crear una función que calcule la bonificación de un empleado (10% de su salario) y
mostrar el salario ajustado de cada empleado.
Pasos:
1. Crear una función Bonificacion que reciba el salario y calcule el 10%.
2. Realizar una consulta que muestre el salario ajustado (salario + bonificación).
   

DELIMITER $$

-- Función para calcular el total de los pedidos de un cliente
CREATE FUNCTION TotalPedidosCliente(cliente_id INT)
RETURNS DECIMAL(10, 2)
DETERMINISTIC
BEGIN
    DECLARE total DECIMAL(10, 2);
    SELECT SUM(total) INTO total
    FROM Pedidos
    WHERE cliente_id = cliente_id;
    IF total IS NULL THEN
        SET total = 0;
    END IF;
    RETURN total;
END $$

DELIMITER ;
SELECT 
    C.nombre AS Cliente,
    TotalPedidosCliente(C.id) AS Total_Pedidos
FROM Clientes C
HAVING TotalPedidosCliente(C.id) > 1000;


Función para Calcular Días Transcurridos Desde el Último Pedido
Objetivo: Crear una función que calcule los días desde el último pedido de un cliente y
mostrar clientes que hayan hecho un pedido en los últimos 30 días.
Pasos:
1. Crear la función DiasDesdeUltimoPedido que reciba el ID de un cliente y calcule los
días desde su último pedido.
2. Realizar una consulta que muestre solo a los clientes con pedidos en los últimos 30 días.
Función para Calcular el Total en Inventario de un Producto
Objetivo: Crear una función que calcule el total en inventario (cantidad x precio) de cada
producto y listar productos con inventario superior a $500.
Pasos:
1. Crear la función TotalInventarioProducto que multiplique cantidad y precio de un
producto.
2. Realizar una consulta que muestre el nombre del producto y su total en inventario,
filtrando los productos con inventario superior a $500.

DELIMITER $$

-- Función para calcular los días desde el último pedido
CREATE FUNCTION DiasDesdeUltimoPedido(cliente_id INT)
RETURNS INT
DETERMINISTIC
BEGIN
    DECLARE dias_transcurridos INT;
    SELECT DATEDIFF(CURDATE(), MAX(fecha)) INTO dias_transcurridos
    FROM Pedidos
    WHERE cliente_id = cliente_id;
    IF dias_transcurridos IS NULL THEN
        RETURN NULL;
    END IF;
    RETURN dias_transcurridos;
END $$

DELIMITER ;
SELECT 
    C.nombre AS Cliente,
    DiasDesdeUltimoPedido(C.id) AS Dias_Ultimo_Pedido
FROM Clientes C
WHERE DiasDesdeUltimoPedido(C.id) <= 30;
DELIMITER $$

-- Función para calcular el total en inventario de un producto
CREATE FUNCTION TotalInventarioProducto(producto_id INT)
RETURNS DECIMAL(10, 2)
DETERMINISTIC
BEGIN
    DECLARE total_inventario DECIMAL(10, 2);
    SELECT SUM(D.cantidad * P.precio) INTO total_inventario
    FROM DetallesPedido D
    JOIN Productos P ON D.producto_id = P.id
    WHERE P.id = producto_id;
    IF total_inventario IS NULL THEN
        SET total_inventario = 0;
    END IF;
    RETURN total_inventario;
END $$

DELIMITER ;
SELECT 
    P.nombre AS Producto,
    TotalInventarioProducto(P.id) AS Total_En_Inventario
FROM Productos P
HAVING TotalInventarioProducto(P.id) > 500;



Creación de un Historial de Precios de Productos
Descripción: Crear un trigger y una tabla para mantener un historial de precios de
productos. Cada vez que el precio de un producto cambia, el trigger debe guardar el ID del
producto, el precio antiguo, el nuevo precio y la fecha de cambio.
Pasos:
1. Crear la tabla HistorialPrecios .
2. Crear el trigger RegistroCambioPrecio en la tabla Productos para registrar los
cambios de precio.

CREATE TABLE HistorialPrecios (
    id INT PRIMARY KEY AUTO_INCREMENT,
    producto_id INT,
    precio_antiguo DECIMAL(10, 2),
    precio_nuevo DECIMAL(10, 2),
    fecha_cambio TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (producto_id) REFERENCES Productos(id)
);
DELIMITER $$

-- Trigger para registrar el cambio de precio de productos
CREATE TRIGGER RegistroCambioPrecio
AFTER UPDATE ON Productos
FOR EACH ROW
BEGIN
    -- Verificar si el precio ha cambiado
    IF OLD.precio <> NEW.precio THEN
        -- Insertar el cambio en la tabla HistorialPrecios
        INSERT INTO HistorialPrecios (producto_id, precio_antiguo, precio_nuevo, fecha_cambio)
        VALUES (NEW.id, OLD.precio, NEW.precio, NOW());
    END IF;
END $$

DELIMITER ;


Procedimiento para Generar Reporte de Ventas Mensuales por Empleado
Descripción: Crear un procedimiento almacenado que genere un reporte de ventas mensual
para cada empleado. El procedimiento debe recibir como parámetros el mes y el año, y
devolver una lista de empleados con el total de ventas que gestionaron en ese periodo.
Pasos:
1. Crear el procedimiento ReporteVentasMensuales .
2. Usar una subconsulta que agrupe las ventas por empleado y que filtre por el mes y el
año.

DELIMITER $$

-- Procedimiento almacenado para generar reporte de ventas mensuales por empleado
CREATE PROCEDURE ReporteVentasMensuales(IN mes INT, IN anio INT)
BEGIN
    -- Selecciona los empleados y el total de ventas en el mes y año especificados
    SELECT 
        E.nombre AS Empleado,
        SUM(P.total) AS Total_Ventas
    FROM 
        DatosEmpleados E
    JOIN 
        Pedidos P ON E.id = P.empleado_id
    WHERE 
        MONTH(P.fecha) = mes AND YEAR(P.fecha) = anio
    GROUP BY 
        E.id
    ORDER BY 
        Total_Ventas DESC;
END $$

DELIMITER ;



Subconsulta para Obtener el Producto Más Vendido por Cada Proveedor
Descripción: Realizar una consulta compleja que devuelva el producto más vendido para
cada proveedor, mostrando el nombre del proveedor, el nombre del producto y la cantidad
vendida.
Pasos:
1. Utilizar una subconsulta para calcular la cantidad total de ventas por producto.
2. Filtrar en la consulta principal para obtener el producto más vendido de cada
proveedor.

SELECT 
    PR.nombre AS Proveedor, 
    P.nombre AS Producto, 
    SUM(DP.cantidad) AS CantidadVendida
FROM 
    Proveedores PR
JOIN 
    Productos P ON PR.id = P.proveedor_id
JOIN 
    DetallesPedido DP ON P.id = DP.producto_id
GROUP BY 
    PR.id, P.id
HAVING 
    SUM(DP.cantidad) = (
        SELECT MAX(SUM(DP2.cantidad)) 
        FROM DetallesPedido DP2 
        JOIN Productos P2 ON DP2.producto_id = P2.id 
        WHERE P2.proveedor_id = PR.id
        GROUP BY P2.id
    )
ORDER BY 
    PR.nombre;


Función para Calcular el Estado de Stock de un Producto
Descripción: Crear una función que calcule el estado de stock de un producto y lo clasifique
en “Alto”, “Medio” o “Bajo” en función de su cantidad. Usar esta función en una consulta para
listar todos los productos y su estado de stock.
Pasos:
1. Crear la función EstadoStock que reciba la cantidad de un producto.
2. En la consulta principal, utilizar la función para clasificar el estado de stock de cada
producto.

DELIMITER $$

CREATE FUNCTION EstadoStock(cantidad INT)
RETURNS VARCHAR(20)
DETERMINISTIC
BEGIN
    DECLARE estado VARCHAR(20);
    IF cantidad >= 100 THEN
        SET estado = 'Alto';
    ELSEIF cantidad >= 50 THEN
        SET estado = 'Medio';
    ELSE
        SET estado = 'Bajo';
    END IF;
    RETURN estado;
END $$

DELIMITER ;
SELECT 
    nombre AS Producto,
    cantidad AS CantidadDisponible,
    EstadoStock(cantidad) AS EstadoStock
FROM 
    Productos;
SELECT 
    nombre AS Producto,
    cantidad AS CantidadDisponible,
    EstadoStock(cantidad) AS EstadoStock
FROM 
    Productos;


Trigger para Control de Inventario en Pedidos
Descripción: Crear un trigger que, al insertar un nuevo pedido, disminuya automáticamente
la cantidad en stock del producto. El trigger debe también prevenir que se inserte el pedido si
el stock es insuficiente.
Pasos:
1. Crear el trigger ActualizarInventario en la tabla DetallesPedido .
2. Controlar que no se permita la inserción si la cantidad es mayor que el stock disponible.

DELIMITER $$

CREATE TRIGGER ActualizarInventario
BEFORE INSERT ON DetallesPedido
FOR EACH ROW
BEGIN
    -- Verificar que haya suficiente stock disponible
    DECLARE stock_disponible INT;
    SELECT cantidad INTO stock_disponible
    FROM Productos
    WHERE id = NEW.producto_id;
    IF stock_disponible < NEW.cantidad THEN
        SIGNAL SQLSTATE '45000' SET MESSAGE_TEXT = 'No hay suficiente stock para este producto.';
    END IF;
    UPDATE Productos
    SET cantidad = cantidad - NEW.cantidad
    WHERE id = NEW.producto_id;

END $$

DELIMITER ;






