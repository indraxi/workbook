# workbook
CREATE DATABASE IF NOT EXISTS library_db;
USE library_db;


CREATE TABLE members (
    member_id INT AUTO_INCREMENT PRIMARY KEY,
    member_name VARCHAR(100) NOT NULL,
    join_date DATE DEFAULT CURRENT_DATE
);


CREATE TABLE books (
    book_id INT AUTO_INCREMENT PRIMARY KEY,
    title VARCHAR(150) NOT NULL,
    author VARCHAR(100),
    available_copies INT DEFAULT 1 CHECK (available_copies >= 0)
);


CREATE TABLE borrow_records (
    record_id INT AUTO_INCREMENT PRIMARY KEY,
    member_id INT,
    book_id INT,
    borrow_date DATE DEFAULT CURRENT_DATE,
    return_date DATE,
    FOREIGN KEY (member_id) REFERENCES members(member_id),
    FOREIGN KEY (book_id) REFERENCES books(book_id)
);


INSERT INTO members (member_name) VALUES
('Arjun'),
('Riya'),
('Neha');


INSERT INTO books (title, author, available_copies) VALUES
('Java Programming', 'E. Balagurusamy', 2),
('DBMS Concepts', 'Korth', 1),
('Operating System Principles', 'Silberschatz', 3);



CREATE PROCEDURE borrow_book(IN m_id INT, IN b_id INT)
BEGIN
    IF (SELECT available_copies FROM books WHERE book_id = b_id) > 0 THEN
        
        UPDATE books
        SET available_copies = available_copies - 1
        WHERE book_id = b_id;

        INSERT INTO borrow_records(member_id, book_id)
        VALUES (m_id, b_id);

    ELSE
        SIGNAL SQLSTATE '45000'
        SET MESSAGE_TEXT = 'No copies available to borrow.';
    END IF;

DELIMITER ;



CREATE TRIGGER return_book_update
AFTER UPDATE ON borrow_records
FOR EACH ROW
BEGIN
    IF NEW.return_date IS NOT NULL THEN
        UPDATE books
        SET available_copies = available_copies + 1
        WHERE book_id = NEW.book_id;
    END IF;
END //
DELIMITER ;


CREATE VIEW borrowed_details AS
SELECT 
    m.member_name AS Borrower,
    b.title AS Book_Title,
    r.borrow_date AS Date_Borrowed,
    r.return_date AS Date_Returned
FROM borrow_records r
JOIN members m ON r.member_id = m.member_id
JOIN books b ON r.book_id = b.book_id;
