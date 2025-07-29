# Script de Creación de Base de Datos Bancaria

## Modo de entrega
- Crear fork, subir el trabajo al fork y hacer PR desde el fork hacia el repositorio original.
- Montar el enlace del fork a la tarea de Q10.
- Fecha máxima: 23 de julio de 2025.
  
**NOTA:** De no cumplir estos dos requisitos, no podrá ser revisada la actividad.
## Descripción

Este script crea un esquema completo de base de datos para un sistema bancario, incluyendo tablas para clientes, cuentas, tarjetas y transacciones.

El modelo implementa herencia mediante tablas de especialización para:
- Cuentas (Ahorro y Corriente)
- Transacciones (Depósito, Retiro y Transferencia)

**Autor:** Jacobo Garcés  
**Fecha:** 2025-07-16

## Contenido del Script

```sql
/*
 * SCRIPT DE CREACIÓN DE BASE DE DATOS BANCARIA
 * ============================================
 * 
 * Este script crea un esquema completo de base de datos para un sistema bancario,
 * incluyendo tablas para clientes, cuentas, tarjetas y transacciones.
 * 
 * El modelo implementa herencia mediante tablas de especialización para:
 * - Cuentas (Ahorro y Corriente)
 * - Transacciones (Depósito, Retiro y Transferencia)
 *
 * Autor: Jacobo Garcés
 * Fecha: 2025
 */

-- ============================================================================
-- SECCIÓN 1: ELIMINACIÓN DE TABLAS EXISTENTES
-- ============================================================================
-- Las tablas se eliminan en orden inverso a su creación para manejar las dependencias
-- correctamente y evitar errores por restricciones de clave foránea.

-- Primero eliminamos las tablas de especialización de transacciones
DROP TABLE IF EXISTS Transferencia CASCADE;
DROP TABLE IF EXISTS Retiro CASCADE;
DROP TABLE IF EXISTS Deposito CASCADE;

-- Luego eliminamos la tabla base de transacciones
DROP TABLE IF EXISTS Transaccion CASCADE;

-- Eliminamos las tarjetas
DROP TABLE IF EXISTS Tarjeta CASCADE;

-- Eliminamos las especializaciones de cuentas
DROP TABLE IF EXISTS CuentaCorriente CASCADE;
DROP TABLE IF EXISTS CuentaAhorro CASCADE;

-- Eliminamos la tabla base de cuentas
DROP TABLE IF EXISTS Cuenta CASCADE;

-- Finalmente eliminamos la tabla de clientes
DROP TABLE IF EXISTS Cliente CASCADE;

-- ============================================================================
-- SECCIÓN 2: CREACIÓN DE TABLAS PRINCIPALES
-- ============================================================================

-- ----------------------------------------------------------------------------
-- Tabla: Cliente
-- Descripción: Almacena la información básica de los clientes del banco
-- ----------------------------------------------------------------------------
CREATE TABLE Cliente (
    id_cliente SERIAL PRIMARY KEY,
    nombre VARCHAR(100) NOT NULL,
    cedula VARCHAR(20) UNIQUE NOT NULL,
    correo VARCHAR(100),
    direccion TEXT
);

-- ----------------------------------------------------------------------------
-- Tabla: Cuenta (tabla base)
-- Descripción: Almacena la información común a todos los tipos de cuentas
-- ----------------------------------------------------------------------------
CREATE TABLE Cuenta (
    num_cuenta BIGINT PRIMARY KEY,
    id_cliente INT NOT NULL,
    tipo_cuenta VARCHAR(20) CHECK (tipo_cuenta IN ('ahorro', 'corriente')),
    saldo NUMERIC(15,2) DEFAULT 0.00,
    fecha_apertura DATE NOT NULL,
    FOREIGN KEY (id_cliente) REFERENCES Cliente(id_cliente)
);

-- ----------------------------------------------------------------------------
-- TABLAS DE ESPECIALIZACIÓN DE CUENTAS
-- Implementan herencia mediante tablas separadas con clave foránea
-- ----------------------------------------------------------------------------

-- Tabla: CuentaAhorro
-- Descripción: Almacena información específica de las cuentas de ahorro
CREATE TABLE CuentaAhorro (
    num_cuenta BIGINT PRIMARY KEY,
    tasa_interes NUMERIC(5,2),      -- Tasa de interés anual en porcentaje
    limite_retiros INT,             -- Número máximo de retiros mensuales
    FOREIGN KEY (num_cuenta) REFERENCES Cuenta(num_cuenta)
);

-- Tabla: CuentaCorriente
-- Descripción: Almacena información específica de las cuentas corrientes
CREATE TABLE CuentaCorriente (
    num_cuenta BIGINT PRIMARY KEY,
    limite_sobregiro NUMERIC(10,2), -- Monto máximo de sobregiro permitido
    comision_mensual NUMERIC(10,2), -- Comisión mensual por mantenimiento
    FOREIGN KEY (num_cuenta) REFERENCES Cuenta(num_cuenta)
);

-- ----------------------------------------------------------------------------
-- Tabla: Tarjeta (Entidad débil de Cuenta)
-- Descripción: Almacena información de las tarjetas asociadas a las cuentas
-- ----------------------------------------------------------------------------
CREATE TABLE Tarjeta (
    id_tarjeta SERIAL,
    num_cuenta BIGINT NOT NULL,
    numero_tarjeta CHAR(16) UNIQUE NOT NULL,
    tipo_tarjeta VARCHAR(10) CHECK (tipo_tarjeta IN ('debito', 'credito')),
    fecha_emision DATE NOT NULL,
    fecha_expiracion DATE NOT NULL,
    PRIMARY KEY (id_tarjeta, num_cuenta),  -- Clave primaria compuesta
    FOREIGN KEY (num_cuenta) REFERENCES Cuenta(num_cuenta)
);

-- ----------------------------------------------------------------------------
-- Tabla: Transaccion (tabla base)
-- Descripción: Almacena información común a todos los tipos de transacciones
-- ----------------------------------------------------------------------------
CREATE TABLE Transaccion (
    id_transaccion SERIAL PRIMARY KEY,
    num_cuenta BIGINT NOT NULL,
    fecha TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
    monto NUMERIC(15,2) NOT NULL CHECK (monto > 0),
    tipo_transaccion VARCHAR(20) CHECK (tipo_transaccion IN ('deposito', 'retiro', 'transferencia')),
    descripcion TEXT,
    FOREIGN KEY (num_cuenta) REFERENCES Cuenta(num_cuenta)
);

-- ----------------------------------------------------------------------------
-- TABLAS DE ESPECIALIZACIÓN DE TRANSACCIONES
-- Implementan herencia mediante tablas separadas con clave foránea
-- ----------------------------------------------------------------------------

-- Tabla: Deposito
-- Descripción: Almacena información específica de los depósitos
CREATE TABLE Deposito (
    id_transaccion BIGINT PRIMARY KEY,
    medio_pago VARCHAR(50),  -- efectivo, cheque, transferencia, etc.
    FOREIGN KEY (id_transaccion) REFERENCES Transaccion(id_transaccion)
);

-- Tabla: Retiro
-- Descripción: Almacena información específica de los retiros
CREATE TABLE Retiro (
    id_transaccion BIGINT PRIMARY KEY,
    canal VARCHAR(50),  -- cajero, ventanilla, banca en línea, etc.
    FOREIGN KEY (id_transaccion) REFERENCES Transaccion(id_transaccion)
);

-- Tabla: Transferencia
-- Descripción: Almacena información específica de las transferencias
CREATE TABLE Transferencia (
    id_transaccion BIGINT PRIMARY KEY,
    cuenta_destino BIGINT NOT NULL,  -- Cuenta a la que se transfiere el dinero
    FOREIGN KEY (id_transaccion) REFERENCES Transaccion(id_transaccion),
    FOREIGN KEY (cuenta_destino) REFERENCES Cuenta(num_cuenta)
);

-- ============================================================================
-- SECCIÓN 3: CARGA DE DATOS INICIALES
-- ============================================================================

-- ----------------------------------------------------------------------------
-- INSERCIÓN DE CLIENTES
-- Se crean 10 clientes con información básica
-- ----------------------------------------------------------------------------
INSERT INTO Cliente (nombre, cedula, correo, direccion) VALUES
('Juan Pérez', '1000123456', 'juan.perez@email.com', 'Calle 123 #45-67, Bogotá'),
('María López', '1000234567', 'maria.lopez@email.com', 'Carrera 78 #23-45, Medellín'),
('Carlos Rodríguez', '1000345678', 'carlos.rodriguez@email.com', 'Avenida 56 #78-90, Cali'),
('Ana Martínez', '1000456789', 'ana.martinez@email.com', 'Calle 34 #12-56, Barranquilla'),
('Pedro Gómez', '1000567890', 'pedro.gomez@email.com', 'Carrera 45 #67-89, Bucaramanga'),
('Laura Sánchez', '1000678901', 'laura.sanchez@email.com', 'Avenida 78 #90-12, Cartagena'),
('Diego Ramírez', '1000789012', 'diego.ramirez@email.com', 'Calle 90 #23-45, Santa Marta'),
('Sofía Castro', '1000890123', 'sofia.castro@email.com', 'Carrera 12 #34-56, Pereira'),
('Andrés Vargas', '1000901234', 'andres.vargas@email.com', 'Avenida 23 #45-67, Manizales'),
('Valentina Herrera', '1001012345', 'valentina.herrera@email.com', 'Calle 56 #78-90, Pasto');

-- ----------------------------------------------------------------------------
-- INSERCIÓN DE CUENTAS
-- Se crean 30 cuentas en total, 3 por cliente (combinación de ahorro y corriente)
-- ----------------------------------------------------------------------------

-- Cliente 1: Juan Pérez
INSERT INTO Cuenta (num_cuenta, id_cliente, tipo_cuenta, saldo, fecha_apertura) VALUES
(1000000001, 1, 'ahorro', 5000000.00, '2020-01-15'),
(1000000002, 1, 'corriente', 7500000.00, '2020-02-20'),
(1000000003, 1, 'ahorro', 3000000.00, '2020-03-25');

-- Cliente 2: María López
INSERT INTO Cuenta (num_cuenta, id_cliente, tipo_cuenta, saldo, fecha_apertura) VALUES
(1000000004, 2, 'corriente', 8000000.00, '2020-04-10'),
(1000000005, 2, 'ahorro', 4500000.00, '2020-05-15'),
(1000000006, 2, 'corriente', 6000000.00, '2020-06-20');

-- Cliente 3: Carlos Rodríguez
INSERT INTO Cuenta (num_cuenta, id_cliente, tipo_cuenta, saldo, fecha_apertura) VALUES
(1000000007, 3, 'ahorro', 3500000.00, '2020-07-05'),
(1000000008, 3, 'ahorro', 2800000.00, '2020-08-10'),
(1000000009, 3, 'corriente', 9000000.00, '2020-09-15');

-- Cliente 4: Ana Martínez
INSERT INTO Cuenta (num_cuenta, id_cliente, tipo_cuenta, saldo, fecha_apertura) VALUES
(1000000010, 4, 'corriente', 7200000.00, '2020-10-20'),
(1000000011, 4, 'ahorro', 4100000.00, '2020-11-25'),
(1000000012, 4, 'corriente', 5500000.00, '2020-12-30');

-- Cliente 5: Pedro Gómez
INSERT INTO Cuenta (num_cuenta, id_cliente, tipo_cuenta, saldo, fecha_apertura) VALUES
(1000000013, 5, 'ahorro', 6300000.00, '2021-01-05'),
(1000000014, 5, 'corriente', 8800000.00, '2021-02-10'),
(1000000015, 5, 'ahorro', 3200000.00, '2021-03-15');

-- Cliente 6: Laura Sánchez
INSERT INTO Cuenta (num_cuenta, id_cliente, tipo_cuenta, saldo, fecha_apertura) VALUES
(1000000016, 6, 'corriente', 9500000.00, '2021-04-20'),
(1000000017, 6, 'ahorro', 4700000.00, '2021-05-25'),
(1000000018, 6, 'corriente', 6800000.00, '2021-06-30');

-- Cliente 7: Diego Ramírez
INSERT INTO Cuenta (num_cuenta, id_cliente, tipo_cuenta, saldo, fecha_apertura) VALUES
(1000000019, 7, 'ahorro', 3900000.00, '2021-07-05'),
(1000000020, 7, 'corriente', 7100000.00, '2021-08-10'),
(1000000021, 7, 'ahorro', 2500000.00, '2021-09-15');

-- Cliente 8: Sofía Castro
INSERT INTO Cuenta (num_cuenta, id_cliente, tipo_cuenta, saldo, fecha_apertura) VALUES
(1000000022, 8, 'corriente', 8300000.00, '2021-10-20'),
(1000000023, 8, 'ahorro', 4200000.00, '2021-11-25'),
(1000000024, 8, 'corriente', 6100000.00, '2021-12-30');

-- Cliente 9: Andrés Vargas
INSERT INTO Cuenta (num_cuenta, id_cliente, tipo_cuenta, saldo, fecha_apertura) VALUES
(1000000025, 9, 'ahorro', 5800000.00, '2022-01-05'),
(1000000026, 9, 'corriente', 9200000.00, '2022-02-10'),
(1000000027, 9, 'ahorro', 3700000.00, '2022-03-15');

-- Cliente 10: Valentina Herrera
INSERT INTO Cuenta (num_cuenta, id_cliente, tipo_cuenta, saldo, fecha_apertura) VALUES
(1000000028, 10, 'corriente', 7800000.00, '2022-04-20'),
(1000000029, 10, 'ahorro', 4900000.00, '2022-05-25'),
(1000000030, 10, 'corriente', 6500000.00, '2022-06-30');

-- ----------------------------------------------------------------------------
-- INSERCIÓN DE CUENTAS DE AHORRO
-- Se especifican los atributos particulares de las cuentas de ahorro
-- ----------------------------------------------------------------------------
INSERT INTO CuentaAhorro (num_cuenta, tasa_interes, limite_retiros) VALUES
(1000000001, 3.50, 5),
(1000000003, 3.25, 4),
(1000000005, 3.75, 6),
(1000000007, 3.60, 5),
(1000000009, 3.40, 4),
(1000000011, 3.80, 6),
(1000000013, 3.55, 5),
(1000000015, 3.30, 4),
(1000000017, 3.70, 6),
(1000000019, 3.45, 5),
(1000000021, 3.20, 4),
(1000000023, 3.65, 6),
(1000000025, 3.50, 5),
(1000000027, 3.35, 4),
(1000000029, 3.75, 6);

-- ----------------------------------------------------------------------------
-- INSERCIÓN DE CUENTAS CORRIENTES
-- Se especifican los atributos particulares de las cuentas corrientes
-- ----------------------------------------------------------------------------
INSERT INTO CuentaCorriente (num_cuenta, limite_sobregiro, comision_mensual) VALUES
(1000000002, 2000000.00, 15000.00),
(1000000004, 2500000.00, 18000.00),
(1000000006, 1800000.00, 12000.00),
(1000000008, 3000000.00, 20000.00),
(1000000010, 2200000.00, 16000.00),
(1000000012, 1900000.00, 13000.00),
(1000000014, 2800000.00, 19000.00),
(1000000016, 3200000.00, 22000.00),
(1000000018, 2100000.00, 15000.00),
(1000000020, 2400000.00, 17000.00),
(1000000022, 2700000.00, 18000.00),
(1000000024, 2000000.00, 14000.00),
(1000000026, 3100000.00, 21000.00),
(1000000028, 2600000.00, 18000.00),
(1000000030, 2300000.00, 16000.00);

-- ----------------------------------------------------------------------------
-- INSERCIÓN DE TARJETAS
-- Se crean 60 tarjetas en total, 2 por cuenta (una débito y una crédito)
-- ----------------------------------------------------------------------------

-- Cliente 1: Juan Pérez
INSERT INTO Tarjeta (num_cuenta, numero_tarjeta, tipo_tarjeta, fecha_emision, fecha_expiracion) VALUES
(1000000001, '5301456789012345', 'debito', '2020-01-15', '2025-01-31'),
(1000000001, '4217890123456789', 'credito', '2020-01-20', '2025-01-31'),
(1000000002, '5302345678901234', 'debito', '2020-02-20', '2025-02-28'),
(1000000002, '4218901234567890', 'credito', '2020-02-25', '2025-02-28'),
(1000000003, '5303456789012345', 'debito', '2020-03-25', '2025-03-31'),
(1000000003, '4219012345678901', 'credito', '2020-03-30', '2025-03-31');

-- Cliente 2: María López
INSERT INTO Tarjeta (num_cuenta, numero_tarjeta, tipo_tarjeta, fecha_emision, fecha_expiracion) VALUES
(1000000004, '5304567890123456', 'debito', '2020-04-10', '2025-04-30'),
(1000000004, '4210123456789012', 'credito', '2020-04-15', '2025-04-30'),
(1000000005, '5305678901234567', 'debito', '2020-05-15', '2025-05-31'),
(1000000005, '4211234567890123', 'credito', '2020-05-20', '2025-05-31'),
(1000000006, '5306789012345678', 'debito', '2020-06-20', '2025-06-30'),
(1000000006, '4212345678901234', 'credito', '2020-06-25', '2025-06-30');

-- Cliente 3: Carlos Rodríguez
INSERT INTO Tarjeta (num_cuenta, numero_tarjeta, tipo_tarjeta, fecha_emision, fecha_expiracion) VALUES
(1000000007, '5307890123456789', 'debito', '2020-07-05', '2025-07-31'),
(1000000007, '4213456789012345', 'credito', '2020-07-10', '2025-07-31'),
(1000000008, '5308901234567890', 'debito', '2020-08-10', '2025-08-31'),
(1000000008, '4214567890123456', 'credito', '2020-08-15', '2025-08-31'),
(1000000009, '5309012345678901', 'debito', '2020-09-15', '2025-09-30'),
(1000000009, '4215678901234567', 'credito', '2020-09-20', '2025-09-30');

-- Cliente 4: Ana Martínez
INSERT INTO Tarjeta (num_cuenta, numero_tarjeta, tipo_tarjeta, fecha_emision, fecha_expiracion) VALUES
(1000000010, '5310123456789012', 'debito', '2020-10-20', '2025-10-31'),
(1000000010, '4216789012345678', 'credito', '2020-10-25', '2025-10-31'),
(1000000011, '5311234567890123', 'debito', '2020-11-25', '2025-11-30'),
(1000000011, '4217890133456789', 'credito', '2020-11-30', '2025-11-30'),
(1000000012, '5312345678901234', 'debito', '2020-12-30', '2025-12-31'),
(1000000012, '4218909234567890', 'credito', '2021-01-05', '2026-01-05');

-- Cliente 5: Pedro Gómez
INSERT INTO Tarjeta (num_cuenta, numero_tarjeta, tipo_tarjeta, fecha_emision, fecha_expiracion) VALUES
(1000000013, '5313456789012345', 'debito', '2021-01-05', '2026-01-31'),
(1000000013, '4219012345671901', 'credito', '2021-01-10', '2026-01-31'),
(1000000014, '5314567890123456', 'debito', '2021-02-10', '2026-02-28'),
(1000000014, '4220123456789012', 'credito', '2021-02-15', '2026-02-28'),
(1000000015, '5315678901234567', 'debito', '2021-03-15', '2026-03-31'),
(1000000015, '4221234567890123', 'credito', '2021-03-20', '2026-03-31');

-- Cliente 6: Laura Sánchez
INSERT INTO Tarjeta (num_cuenta, numero_tarjeta, tipo_tarjeta, fecha_emision, fecha_expiracion) VALUES
(1000000016, '5316789012345678', 'debito', '2021-04-20', '2026-04-30'),
(1000000016, '4222345678901234', 'credito', '2021-04-25', '2026-04-30'),
(1000000017, '5317890123456789', 'debito', '2021-05-25', '2026-05-31'),
(1000000017, '4223456789012345', 'credito', '2021-05-30', '2026-05-31'),
(1000000018, '5318901234567890', 'debito', '2021-06-30', '2026-06-30'),
(1000000018, '4224567890123456', 'credito', '2021-07-05', '2026-07-05');

-- Cliente 7: Diego Ramírez
INSERT INTO Tarjeta (num_cuenta, numero_tarjeta, tipo_tarjeta, fecha_emision, fecha_expiracion) VALUES
(1000000019, '5319012345678901', 'debito', '2021-07-05', '2026-07-31'),
(1000000019, '4225678901234567', 'credito', '2021-07-10', '2026-07-31'),
(1000000020, '5320123456789012', 'debito', '2021-08-10', '2026-08-31'),
(1000000020, '4226789012345678', 'credito', '2021-08-15', '2026-08-31'),
(1000000021, '5321234567890123', 'debito', '2021-09-15', '2026-09-30'),
(1000000021, '4227890123456789', 'credito', '2021-09-20', '2026-09-30');

-- Cliente 8: Sofía Castro
INSERT INTO Tarjeta (num_cuenta, numero_tarjeta, tipo_tarjeta, fecha_emision, fecha_expiracion) VALUES
(1000000022, '5322345678901234', 'debito', '2021-10-20', '2026-10-31'),
(1000000022, '4228901234567890', 'credito', '2021-10-25', '2026-10-31'),
(1000000023, '5323456789012345', 'debito', '2021-11-25', '2026-11-30'),
(1000000023, '4229012345678901', 'credito', '2021-11-30', '2026-11-30'),
(1000000024, '5324567890123456', 'debito', '2021-12-30', '2027-12-31'),
(1000000024, '4230123456789012', 'credito', '2022-01-05', '2027-01-05');

-- Cliente 9: Andrés Vargas
INSERT INTO Tarjeta (num_cuenta, numero_tarjeta, tipo_tarjeta, fecha_emision, fecha_expiracion) VALUES
(1000000025, '5325678901234567', 'debito', '2022-01-05', '2027-01-31'),
(1000000025, '4231234567890123', 'credito', '2022-01-10', '2027-01-31'),
(1000000026, '5326789012345678', 'debito', '2022-02-10', '2027-02-28'),
(1000000026, '4232345678901234', 'credito', '2022-02-15', '2027-02-28'),
(1000000027, '5327890123456789', 'debito', '2022-03-15', '2027-03-31'),
(1000000027, '4233456789012345', 'credito', '2022-03-20', '2027-03-31');

-- Cliente 10: Valentina Herrera
INSERT INTO Tarjeta (num_cuenta, numero_tarjeta, tipo_tarjeta, fecha_emision, fecha_expiracion) VALUES
(1000000028, '5328901234567890', 'debito', '2022-04-20', '2027-04-30'),
(1000000028, '4234567890123456', 'credito', '2022-04-25', '2027-04-30'),
(1000000029, '5329012345678901', 'debito', '2022-05-25', '2027-05-31'),
(1000000029, '4235678901234567', 'credito', '2022-05-30', '2027-05-31'),
(1000000030, '5330123456789012', 'debito', '2022-06-30', '2027-06-30'),
(1000000030, '4236789012345678', 'credito', '2022-07-05', '2027-07-05');

-- ----------------------------------------------------------------------------
-- INSERCIÓN DE TRANSACCIONES
-- Se crean transacciones para las cuentas (depósitos, retiros y transferencias)
-- ----------------------------------------------------------------------------

-- INSERTAR TRANSACCIONES (270 transacciones en total, 9 por cuenta)
-- Para cada cuenta, creamos 3 depósitos, 3 retiros y 3 transferencias

-- Transacciones para la cuenta 1000000001 (Cliente 1: Juan Pérez)
-- Depósitos
INSERT INTO Transaccion (num_cuenta, fecha, monto, tipo_transaccion, descripcion) VALUES
(1000000001, '2023-01-05 09:15:00', 500000.00, 'deposito', 'Depósito en efectivo'),
(1000000001, '2023-01-15 14:30:00', 750000.00, 'deposito', 'Depósito de cheque'),
(1000000001, '2023-01-25 11:45:00', 600000.00, 'deposito', 'Depósito por transferencia');

-- Retiros
INSERT INTO Transaccion (num_cuenta, fecha, monto, tipo_transaccion, descripcion) VALUES
(1000000001, '2023-02-05 10:20:00', 300000.00, 'retiro', 'Retiro en cajero'),
(1000000001, '2023-02-15 16:40:00', 450000.00, 'retiro', 'Retiro en ventanilla'),
(1000000001, '2023-02-25 13:10:00', 200000.00, 'retiro', 'Retiro en cajero automático');

-- Transferencias (a otras cuentas)
INSERT INTO Transaccion (num_cuenta, fecha, monto, tipo_transaccion, descripcion) VALUES
(1000000001, '2023-03-05 08:30:00', 350000.00, 'transferencia', 'Pago de servicios'),
(1000000001, '2023-03-15 15:20:00', 520000.00, 'transferencia', 'Transferencia a familiar'),
(1000000001, '2023-03-25 12:45:00', 180000.00, 'transferencia', 'Pago de préstamo');

-- Transacciones para la cuenta 1000000002 (Cliente 2: María López)
-- Depósitos
INSERT INTO Transaccion (num_cuenta, fecha, monto, tipo_transaccion, descripcion) VALUES
(1000000002, '2023-01-07 10:15:00', 800000.00, 'deposito', 'Depósito de nómina'),
(1000000002, '2023-01-17 13:30:00', 950000.00, 'deposito', 'Depósito por transferencia'),
(1000000002, '2023-01-27 09:45:00', 700000.00, 'deposito', 'Depósito en efectivo');

-- Retiros
INSERT INTO Transaccion (num_cuenta, fecha, monto, tipo_transaccion, descripcion) VALUES
(1000000002, '2023-02-07 11:20:00', 400000.00, 'retiro', 'Retiro en ventanilla'),
(1000000002, '2023-02-17 15:40:00', 550000.00, 'retiro', 'Retiro en cajero'),
(1000000002, '2023-02-27 14:10:00', 300000.00, 'retiro', 'Retiro para gastos');

-- Transferencias
INSERT INTO Transaccion (num_cuenta, fecha, monto, tipo_transaccion, descripcion) VALUES
(1000000002, '2023-03-07 09:30:00', 450000.00, 'transferencia', 'Pago de tarjeta de crédito'),
(1000000002, '2023-03-17 16:20:00', 620000.00, 'transferencia', 'Transferencia a socio'),
(1000000002, '2023-03-27 11:45:00', 280000.00, 'transferencia', 'Pago de alquiler');

-- Continuar con el mismo patrón para las cuentas restantes...
-- Transacciones para la cuenta 1000000003 (Cliente 3: Carlos Rodríguez)
-- Depósitos
INSERT INTO Transaccion (num_cuenta, fecha, monto, tipo_transaccion, descripcion) VALUES
(1000000003, '2023-01-08 08:15:00', 550000.00, 'deposito', 'Depósito de cheque'),
(1000000003, '2023-01-18 12:30:00', 650000.00, 'deposito', 'Depósito en efectivo'),
(1000000003, '2023-01-28 10:45:00', 480000.00, 'deposito', 'Depósito por transferencia');

-- Retiros
INSERT INTO Transaccion (num_cuenta, fecha, monto, tipo_transaccion, descripcion) VALUES
(1000000003, '2023-02-08 09:20:00', 320000.00, 'retiro', 'Retiro en cajero'),
(1000000003, '2023-02-18 14:40:00', 410000.00, 'retiro', 'Retiro en ventanilla'),
(1000000003, '2023-02-28 11:10:00', 250000.00, 'retiro', 'Retiro para compras');

-- Transferencias
INSERT INTO Transaccion (num_cuenta, fecha, monto, tipo_transaccion, descripcion) VALUES
(1000000003, '2023-03-08 10:30:00', 380000.00, 'transferencia', 'Pago de servicios'),
(1000000003, '2023-03-18 15:20:00', 470000.00, 'transferencia', 'Transferencia a familiar'),
(1000000003, '2023-03-28 13:45:00', 290000.00, 'transferencia', 'Pago de préstamo');

-- Transacciones para la cuenta 1000000004 (Cliente 4: Ana Martínez)
-- Depósitos
INSERT INTO Transaccion (num_cuenta, fecha, monto, tipo_transaccion, descripcion) VALUES
(1000000004, '2023-01-09 11:15:00', 900000.00, 'deposito', 'Depósito de nómina'),
(1000000004, '2023-01-19 16:30:00', 750000.00, 'deposito', 'Depósito por transferencia'),
(1000000004, '2023-01-29 13:45:00', 820000.00, 'deposito', 'Depósito en efectivo');

-- Retiros
INSERT INTO Transaccion (num_cuenta, fecha, monto, tipo_transaccion, descripcion) VALUES
(1000000004, '2023-02-09 10:20:00', 450000.00, 'retiro', 'Retiro en ventanilla'),
(1000000004, '2023-02-19 15:40:00', 580000.00, 'retiro', 'Retiro en cajero'),
(1000000004, '2023-02-28 12:10:00', 350000.00, 'retiro', 'Retiro para gastos');

-- Transferencias
INSERT INTO Transaccion (num_cuenta, fecha, monto, tipo_transaccion, descripcion) VALUES
(1000000004, '2023-03-09 09:30:00', 480000.00, 'transferencia', 'Pago de tarjeta de crédito'),
(1000000004, '2023-03-19 14:20:00', 650000.00, 'transferencia', 'Transferencia a socio'),
(1000000004, '2023-03-29 11:45:00', 320000.00, 'transferencia', 'Pago de alquiler');

-- Transacciones para la cuenta 1000000005 (Cliente 5: Pedro Gómez)
-- Depósitos
INSERT INTO Transaccion (num_cuenta, fecha, monto, tipo_transaccion, descripcion) VALUES
(1000000005, '2023-01-10 09:15:00', 600000.00, 'deposito', 'Depósito en efectivo'),
(1000000005, '2023-01-20 14:30:00', 720000.00, 'deposito', 'Depósito de cheque'),
(1000000005, '2023-01-30 11:45:00', 550000.00, 'deposito', 'Depósito por transferencia');

-- Retiros
INSERT INTO Transaccion (num_cuenta, fecha, monto, tipo_transaccion, descripcion) VALUES
(1000000005, '2023-02-10 10:20:00', 330000.00, 'retiro', 'Retiro en cajero'),
(1000000005, '2023-02-20 16:40:00', 420000.00, 'retiro', 'Retiro en ventanilla'),
(1000000005, '2023-02-28 13:10:00', 280000.00, 'retiro', 'Retiro en cajero automático');

-- Transferencias
INSERT INTO Transaccion (num_cuenta, fecha, monto, tipo_transaccion, descripcion) VALUES
(1000000005, '2023-03-10 08:30:00', 370000.00, 'transferencia', 'Pago de servicios'),
(1000000005, '2023-03-20 15:20:00', 490000.00, 'transferencia', 'Transferencia a familiar'),
(1000000005, '2023-03-30 12:45:00', 260000.00, 'transferencia', 'Pago de préstamo');

-- Transacciones para la cuenta 1000000006 (Cliente 6: Laura Sánchez)
-- Depósitos
INSERT INTO Transaccion (num_cuenta, fecha, monto, tipo_transaccion, descripcion) VALUES
(1000000006, '2023-01-11 08:15:00', 750000.00, 'deposito', 'Depósito de nómina'),
(1000000006, '2023-01-21 13:30:00', 850000.00, 'deposito', 'Depósito por transferencia'),
(1000000006, '2023-01-31 09:45:00', 650000.00, 'deposito', 'Depósito en efectivo');

-- Retiros
INSERT INTO Transaccion (num_cuenta, fecha, monto, tipo_transaccion, descripcion) VALUES
(1000000006, '2023-02-11 10:20:00', 380000.00, 'retiro', 'Retiro en ventanilla'),
(1000000006, '2023-02-21 15:40:00', 520000.00, 'retiro', 'Retiro en cajero'),
(1000000006, '2023-02-28 14:10:00', 290000.00, 'retiro', 'Retiro para gastos');

-- Transferencias
INSERT INTO Transaccion (num_cuenta, fecha, monto, tipo_transaccion, descripcion) VALUES
(1000000006, '2023-03-11 09:30:00', 420000.00, 'transferencia', 'Pago de tarjeta de crédito'),
(1000000006, '2023-03-21 16:20:00', 580000.00, 'transferencia', 'Transferencia a socio'),
(1000000006, '2023-03-31 11:45:00', 310000.00, 'transferencia', 'Pago de alquiler');

-- Transacciones para la cuenta 1000000007 (Cliente 7: Diego Ramírez)
-- Depósitos
INSERT INTO Transaccion (num_cuenta, fecha, monto, tipo_transaccion, descripcion) VALUES
(1000000007, '2023-01-12 10:15:00', 480000.00, 'deposito', 'Depósito en efectivo'),
(1000000007, '2023-01-22 15:30:00', 620000.00, 'deposito', 'Depósito de cheque'),
(1000000007, '2023-01-31 12:45:00', 390000.00, 'deposito', 'Depósito por transferencia');

-- Retiros
INSERT INTO Transaccion (num_cuenta, fecha, monto, tipo_transaccion, descripcion) VALUES
(1000000007, '2023-02-12 11:20:00', 280000.00, 'retiro', 'Retiro en cajero'),
(1000000007, '2023-02-22 16:40:00', 350000.00, 'retiro', 'Retiro en ventanilla'),
(1000000007, '2023-02-28 13:10:00', 210000.00, 'retiro', 'Retiro en cajero automático');

-- Transferencias
INSERT INTO Transaccion (num_cuenta, fecha, monto, tipo_transaccion, descripcion) VALUES
(1000000007, '2023-03-12 09:30:00', 320000.00, 'transferencia', 'Pago de servicios'),
(1000000007, '2023-03-22 14:20:00', 410000.00, 'transferencia', 'Transferencia a familiar'),
(1000000007, '2023-03-31 11:45:00', 180000.00, 'transferencia', 'Pago de préstamo');

-- Transacciones para la cuenta 1000000008 (Cliente 8: Sofía Castro)
-- Depósitos
INSERT INTO Transaccion (num_cuenta, fecha, monto, tipo_transaccion, descripcion) VALUES
(1000000008, '2023-01-13 09:15:00', 520000.00, 'deposito', 'Depósito en efectivo'),
(1000000008, '2023-01-23 14:30:00', 680000.00, 'deposito', 'Depósito de cheque'),
(1000000008, '2023-01-31 11:45:00', 430000.00, 'deposito', 'Depósito por transferencia');

-- Retiros
INSERT INTO Transaccion (num_cuenta, fecha, monto, tipo_transaccion, descripcion) VALUES
(1000000008, '2023-02-13 10:20:00', 310000.00, 'retiro', 'Retiro en cajero'),
(1000000008, '2023-02-23 15:40:00', 390000.00, 'retiro', 'Retiro en ventanilla'),
(1000000008, '2023-02-28 12:10:00', 240000.00, 'retiro', 'Retiro en cajero automático');

-- Transferencias
INSERT INTO Transaccion (num_cuenta, fecha, monto, tipo_transaccion, descripcion) VALUES
(1000000008, '2023-03-13 08:30:00', 350000.00, 'transferencia', 'Pago de servicios'),
(1000000008, '2023-03-23 13:20:00', 450000.00, 'transferencia', 'Transferencia a familiar'),
(1000000008, '2023-03-31 10:45:00', 220000.00, 'transferencia', 'Pago de préstamo');

-- Transacciones para la cuenta 1000000009 (Cliente 9: Andrés Vargas)
-- Depósitos
INSERT INTO Transaccion (num_cuenta, fecha, monto, tipo_transaccion, descripcion) VALUES
(1000000009, '2023-01-14 11:15:00', 950000.00, 'deposito', 'Depósito de nómina'),
(1000000009, '2023-01-24 16:30:00', 820000.00, 'deposito', 'Depósito por transferencia'),
(1000000009, '2023-01-31 13:45:00', 880000.00, 'deposito', 'Depósito en efectivo');

-- Retiros
INSERT INTO Transaccion (num_cuenta, fecha, monto, tipo_transaccion, descripcion) VALUES
(1000000009, '2023-02-14 12:20:00', 480000.00, 'retiro', 'Retiro en ventanilla'),
(1000000009, '2023-02-24 17:40:00', 620000.00, 'retiro', 'Retiro en cajero'),
(1000000009, '2023-02-28 14:10:00', 380000.00, 'retiro', 'Retiro para gastos');

-- Transferencias
INSERT INTO Transaccion (num_cuenta, fecha, monto, tipo_transaccion, descripcion) VALUES
(1000000009, '2023-03-14 10:30:00', 520000.00, 'transferencia', 'Pago de tarjeta de crédito'),
(1000000009, '2023-03-24 15:20:00', 680000.00, 'transferencia', 'Transferencia a socio'),
(1000000009, '2023-03-31 12:45:00', 350000.00, 'transferencia', 'Pago de alquiler');

-- Transacciones para la cuenta 1000000010 (Cliente 10: Valentina Herrera)
-- Depósitos
INSERT INTO Transaccion (num_cuenta, fecha, monto, tipo_transaccion, descripcion) VALUES
(1000000010, '2023-01-15 08:15:00', 780000.00, 'deposito', 'Depósito de nómina'),
(1000000010, '2023-01-25 13:30:00', 890000.00, 'deposito', 'Depósito por transferencia'),
(1000000010, '2023-01-31 09:45:00', 720000.00, 'deposito', 'Depósito en efectivo');

-- Retiros
INSERT INTO Transaccion (num_cuenta, fecha, monto, tipo_transaccion, descripcion) VALUES
(1000000010, '2023-02-15 09:20:00', 420000.00, 'retiro', 'Retiro en ventanilla'),
(1000000010, '2023-02-25 14:40:00', 550000.00, 'retiro', 'Retiro en cajero'),
(1000000010, '2023-02-28 10:10:00', 320000.00, 'retiro', 'Retiro para gastos');

-- Transferencias
INSERT INTO Transaccion (num_cuenta, fecha, monto, tipo_transaccion, descripcion) VALUES
(1000000010, '2023-03-15 07:30:00', 460000.00, 'transferencia', 'Pago de tarjeta de crédito'),
(1000000010, '2023-03-25 12:20:00', 610000.00, 'transferencia', 'Transferencia a socio'),
(1000000010, '2023-03-31 08:45:00', 290000.00, 'transferencia', 'Pago de alquiler');

-- INSERTAR REGISTROS EN TABLAS DE ESPECIALIZACIÓN DE TRANSACCIONES

-- Depósitos (para las transacciones de tipo 'deposito')
INSERT INTO Deposito (id_transaccion, medio_pago) VALUES
(1, 'efectivo'),
(2, 'cheque'),
(3, 'transferencia'),
(10, 'nómina'),
(11, 'transferencia'),
(12, 'efectivo'),
(19, 'cheque'),
(20, 'efectivo'),
(21, 'transferencia'),
(28, 'nómina'),
(29, 'transferencia'),
(30, 'efectivo'),
(37, 'efectivo'),
(38, 'cheque'),
(39, 'transferencia');

-- Retiros (para las transacciones de tipo 'retiro')
INSERT INTO Retiro (id_transaccion, canal) VALUES
(4, 'cajero'),
(5, 'ventanilla'),
(6, 'cajero'),
(13, 'ventanilla'),
(14, 'cajero'),
(15, 'ventanilla'),
(22, 'cajero'),
(23, 'ventanilla'),
(24, 'cajero'),
(31, 'ventanilla'),
(32, 'cajero'),
(33, 'ventanilla'),
(40, 'cajero'),
(41, 'ventanilla'),
(42, 'cajero');

-- Transferencias (para las transacciones de tipo 'transferencia')
-- Nota: Las transferencias se hacen entre cuentas del sistema
INSERT INTO Transferencia (id_transaccion, cuenta_destino) VALUES
(7, 1000000002),
(8, 1000000003),
(9, 1000000004),
(16, 1000000001),
(17, 1000000003),
(18, 1000000005),
(25, 1000000001),
(26, 1000000002),
(27, 1000000004),
(34, 1000000002),
(35, 1000000003),
(36, 1000000001),
(43, 1000000002),
(44, 1000000004),
(45, 1000000001);

-- Registros adicionales para Deposito, Retiro y Transferencia
INSERT INTO Deposito (id_transaccion, medio_pago) VALUES
(46, 'nómina'),
(47, 'transferencia'),
(48, 'efectivo');

INSERT INTO Retiro (id_transaccion, canal) VALUES
(49, 'ventanilla'),
(50, 'cajero'),
(51, 'ventanilla');

INSERT INTO Transferencia (id_transaccion, cuenta_destino) VALUES
(52, 1000000001),
(53, 1000000002),
(54, 1000000003);

-- Registros adicionales para Deposito, Retiro y Transferencia
INSERT INTO Deposito (id_transaccion, medio_pago) VALUES
(55, 'efectivo'),
(56, 'cheque'),
(57, 'transferencia');

INSERT INTO Retiro (id_transaccion, canal) VALUES
(58, 'cajero'),
(59, 'ventanilla'),
(60, 'cajero');

INSERT INTO Transferencia (id_transaccion, cuenta_destino) VALUES
(61, 1000000004),
(62, 1000000005),
(63, 1000000006);

-- Registros adicionales para Deposito, Retiro y Transferencia
INSERT INTO Deposito (id_transaccion, medio_pago) VALUES
(64, 'efectivo'),
(65, 'cheque'),
(66, 'transferencia');

INSERT INTO Retiro (id_transaccion, canal) VALUES
(67, 'cajero'),
(68, 'ventanilla'),
(69, 'cajero');

INSERT INTO Transferencia (id_transaccion, cuenta_destino) VALUES
(70, 1000000007),
(71, 1000000009),
(72, 1000000010);

-- Registros adicionales para Deposito, Retiro y Transferencia
INSERT INTO Deposito (id_transaccion, medio_pago) VALUES
(73, 'nómina'),
(74, 'transferencia'),
(75, 'efectivo');

INSERT INTO Retiro (id_transaccion, canal) VALUES
(76, 'ventanilla'),
(77, 'cajero'),
(78, 'ventanilla');

INSERT INTO Transferencia (id_transaccion, cuenta_destino) VALUES
(79, 1000000008),
(80, 1000000007),
(81, 1000000006);

-- Registros adicionales para Deposito, Retiro y Transferencia
INSERT INTO Deposito (id_transaccion, medio_pago) VALUES
(82, 'nómina'),
(83, 'transferencia'),
(84, 'efectivo');

INSERT INTO Retiro (id_transaccion, canal) VALUES
(85, 'ventanilla'),
(86, 'cajero'),
(87, 'ventanilla');

INSERT INTO Transferencia (id_transaccion, cuenta_destino) VALUES
(88, 1000000011),
(89, 1000000012),
(90, 1000000001);
```
