# Sistema_reserva_hotel
Sistema de Reserva de Hotel automatizado em SQL Server, com gerenciamento de hóspedes, quartos e reservas, incluindo validação automática de disponibilidade de quartos.

-- Banco de Dados
CREATE DATABASE HotelDB;
GO

USE HotelDB;
GO

-- Tabela Hospedes
CREATE TABLE Hospedes (
    HospedeID INT IDENTITY(1,1) PRIMARY KEY,
    Nome NVARCHAR(100) NOT NULL,
    CPF NVARCHAR(14) UNIQUE,
    Email NVARCHAR(100)
);
GO

-- Tabela Quartos
CREATE TABLE Quartos (
    QuartoID INT IDENTITY(1,1) PRIMARY KEY,
    Numero INT NOT NULL,
    Tipo NVARCHAR(50),
    Preco DECIMAL(10,2)
);
GO

-- Tabela Reservas
CREATE TABLE Reservas (
    ReservaID INT IDENTITY(1,1) PRIMARY KEY,
    HospedeID INT NOT NULL,
    QuartoID INT NOT NULL,
    DataEntrada DATE,
    DataSaida DATE,
    FOREIGN KEY (HospedeID) REFERENCES Hospedes(HospedeID),
    FOREIGN KEY (QuartoID) REFERENCES Quartos(QuartoID)
);
GO

-- Procedure CheckDisponibilidade
CREATE PROCEDURE CheckDisponibilidade
    @QuartoID INT,
    @DataEntrada DATE,
    @DataSaida DATE
AS
BEGIN
    SET NOCOUNT ON;

    SELECT *
    FROM Reservas
    WHERE QuartoID = @QuartoID
      AND (DataEntrada <= @DataSaida AND DataSaida >= @DataEntrada);
END;
GO

-- Trigger AntesInserirReserva
CREATE TRIGGER AntesInserirReserva
ON Reservas
INSTEAD OF INSERT
AS
BEGIN
    DECLARE @QuartoID INT, @DataEntrada DATE, @DataSaida DATE;

    SELECT TOP 1 @QuartoID = QuartoID, @DataEntrada = DataEntrada, @DataSaida = DataSaida
    FROM inserted;

    IF EXISTS (
        SELECT 1
        FROM Reservas
        WHERE QuartoID = @QuartoID
          AND (DataEntrada <= @DataSaida AND DataSaida >= @DataEntrada)
    )
    BEGIN
        RAISERROR('Quarto indisponível nesta data!', 16, 1);
    END
    ELSE
    BEGIN
        INSERT INTO Reservas (HospedeID, QuartoID, DataEntrada, DataSaida)
        SELECT HospedeID, QuartoID, DataEntrada, DataSaida
        FROM inserted;
    END
END;
GO

-- Inserção de dados de exemplo
INSERT INTO Hospedes (Nome, CPF, Email)
VALUES 
('João Silva', '123.456.789-00', 'joao@email.com'),
('Maria Souza', '987.654.321-00', 'maria@email.com');
GO

INSERT INTO Quartos (Numero, Tipo, Preco)
VALUES
(101, 'Solteiro', 150.00),
(102, 'Casal', 250.00);
GO

INSERT INTO Reservas (HospedeID, QuartoID, DataEntrada, DataSaida)
VALUES (1, 1, '2025-10-01', '2025-10-05');
GO

-- Consultas úteis
SELECT h.Nome, r.DataEntrada, r.DataSaida, q.Numero, q.Tipo
FROM Reservas r
JOIN Hospedes h ON r.HospedeID = h.HospedeID
JOIN Quartos q ON r.QuartoID = q.QuartoID
WHERE h.Nome = 'João Silva';
GO

DECLARE @DataEntrada DATE = '2025-10-01';
DECLARE @DataSaida DATE = '2025-10-05';

SELECT * 
FROM Quartos q
WHERE NOT EXISTS (
    SELECT 1 
    FROM Reservas r
    WHERE r.QuartoID = q.QuartoID
      AND r.DataEntrada <= @DataSaida
      AND r.DataSaida >= @DataEntrada
);
GO

