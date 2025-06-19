Таблица матеериалы 
-- Создание базы данных
CREATE DATABASE IF NOT EXISTS materials_db 
CHARACTER SET utf8mb4 
COLLATE utf8mb4_unicode_ci;

-- Использование созданной базы данных
USE materials_db;

-- Создание таблицы "Материалы" (без generated column для максимальной совместимости)
CREATE TABLE IF NOT EXISTS materials (
    id INT AUTO_INCREMENT PRIMARY KEY,
    name VARCHAR(100) NOT NULL COMMENT 'Название материала',
    type VARCHAR(50) NOT NULL COMMENT 'Тип материала',
    cost_per_pack DECIMAL(10, 2) NOT NULL COMMENT 'Стоимость за упаковку',
    unit VARCHAR(20) NOT NULL COMMENT 'Единица измерения',
    quantity_per_pack INT NOT NULL COMMENT 'Количество в упаковке',
    min_stock_level INT NOT NULL COMMENT 'Минимальное количество на складе',
    current_stock INT NOT NULL COMMENT 'Текущее количество на складе',
    batch_cost DECIMAL(12, 2) COMMENT 'Стоимость партии (рассчитывается триггером)'
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COMMENT='Таблица материалов на складе';

-- Создание триггера для автоматического расчета batch_cost
DELIMITER //
CREATE TRIGGER calculate_batch_cost 
BEFORE INSERT ON materials 
FOR EACH ROW 
BEGIN
    SET NEW.batch_cost = NEW.cost_per_pack * (NEW.current_stock / NEW.quantity_per_pack);
END//
CREATE TRIGGER calculate_batch_cost_update 
BEFORE UPDATE ON materials 
FOR EACH ROW 
BEGIN
    SET NEW.batch_cost = NEW.cost_per_pack * (NEW.current_stock / NEW.quantity_per_pack);
END//
DELIMITER ;

-- Создание индексов для ускорения поиска
CREATE INDEX idx_material_name ON materials(name);
CREATE INDEX idx_material_type ON materials(type);

-- Пример заполнения данными
INSERT INTO materials (name, type, cost_per_pack, unit, quantity_per_pack, min_stock_level, current_stock)
VALUES 
    ('Гвозди 50мм', 'Крепеж', 150.00, 'шт', 100, 5000, 12500),
    ('Доска сосна 40x100x6000', 'Пиломатериалы', 450.00, 'м3', 1, 10, 25),
    ('Краска белая', 'ЛКМ', 1200.00, 'кг', 10, 100, 350),
    ('Плитка керамическая', 'Отделочные', 2800.00, 'м2', 5, 50, 120);



 таблица поставщика 
 CREATE TABLE supplies (
    id INT AUTO_INCREMENT PRIMARY KEY,
    material_id INT NOT NULL,
    supplier_id INT NOT NULL,
    supply_date DATE NOT NULL,
    quantity INT NOT NULL,
    unit_price DECIMAL(10,2) NOT NULL,
    total_cost DECIMAL(12,2) GENERATED ALWAYS AS (quantity * unit_price) STORED,
    invoice_number VARCHAR(50),
    FOREIGN KEY (material_id) REFERENCES materials(id),
    FOREIGN KEY (supplier_id) REFERENCES suppliers(id)
);

таблица проект 
CREATE TABLE projects (
    id INT AUTO_INCREMENT PRIMARY KEY,
    name VARCHAR(100) NOT NULL,
    start_date DATE,
    end_date DATE,
    manager VARCHAR(100),
    status ENUM('planning', 'in_progress', 'completed', 'cancelled')
);

таблица Расход материалов
CREATE TABLE material_usage (
    id INT AUTO_INCREMENT PRIMARY KEY,
    material_id INT NOT NULL,
    project_id INT NOT NULL,
    usage_date DATE NOT NULL,
    quantity INT NOT NULL,
    purpose TEXT,
    FOREIGN KEY (material_id) REFERENCES materials(id),
    FOREIGN KEY (project_id) REFERENCES projects(id)
);

таблица Категории материалов
CREATE TABLE material_categories (
    id INT AUTO_INCREMENT PRIMARY KEY,
    name VARCHAR(50) NOT NULL,
    description TEXT
);

Таблица Местоположения на складе
CREATE TABLE storage_locations (
    id INT AUTO_INCREMENT PRIMARY KEY,
    name VARCHAR(50) NOT NULL,
    description TEXT,
    capacity INT
);
 Таблица Поставщики-Материалы
 CREATE TABLE supplier_materials (
    supplier_id INT NOT NULL,
    material_id INT NOT NULL,
    lead_time INT COMMENT 'Время поставки в днях',
    PRIMARY KEY (supplier_id, material_id),
    FOREIGN KEY (supplier_id) REFERENCES suppliers(id),
    FOREIGN KEY (material_id) REFERENCES materials(id)
);
