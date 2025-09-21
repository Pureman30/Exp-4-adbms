-- Full SQL Code to Demonstrate ACID Properties in FeePayments Table

-- Step 1: Create the FeePayments Table with Constraints
-- This ensures payment_id is unique (Primary Key), amount is positive, and student_name is not NULL.

CREATE TABLE IF NOT EXISTS FeePayments (
    payment_id INT PRIMARY KEY,
    student_name VARCHAR(100) NOT NULL,
    amount DECIMAL(10,2) CHECK (amount > 0),
    payment_date DATE
);

-- Clear any existing data (for demonstration purposes)
DELETE FROM FeePayments;

-- Part A: Insert Multiple Fee Payments in a Transaction (Demonstrating Atomicity)
-- All inserts succeed and are committed as a unit.

BEGIN TRANSACTION;

INSERT INTO FeePayments (payment_id, student_name, amount, payment_date) VALUES
    (1, 'Ashish', 5000.00, '2024-06-01'),
    (2, 'Smaran', 4500.00, '2024-06-02'),
    (3, 'Vaibhav', 5500.00, '2024-06-03');

COMMIT;

-- Verify Part A (Output should show the 3 records)
SELECT * FROM FeePayments;

-- Part B: Demonstrate ROLLBACK for Failed Payment Insertion (Atomicity and Consistency)
-- Attempt to insert with duplicate payment_id and negative amount (violates constraints).
-- The entire transaction is rolled back.

BEGIN TRANSACTION;

INSERT INTO FeePayments (payment_id, student_name, amount, payment_date) VALUES
    (4, 'Kiran', 4600.00, '2024-06-04'),
    (1, 'Ashish', -5000.00, '2024-06-05');  -- This will fail: duplicate ID and negative amount

-- If an error occurs (as it will), rollback
ROLLBACK;

-- Verify Part B (Output should still show only the original 3 records)
SELECT * FROM FeePayments;

-- Part C: Simulate Partial Failure and Ensure Consistent State (Atomicity and Consistency)
-- One valid insert, one invalid (NULL student_name violates NOT NULL constraint).
-- Entire transaction rolled back.

BEGIN TRANSACTION;

INSERT INTO FeePayments (payment_id, student_name, amount, payment_date) VALUES
    (5, 'Meera', 4700.00, '2024-06-06'),
    (6, NULL, 4800.00, '2024-06-07');  -- This will fail: NULL student_name

-- If an error occurs (as it will), rollback
ROLLBACK;

-- Verify Part C (Output should still show only the original 3 records)
SELECT * FROM FeePayments;

-- Part D: Verify ACID Compliance with Transaction Flow
-- Demonstrates all ACID properties:
-- - Atomicity: All-or-nothing inserts.
-- - Consistency: Constraints enforced.
-- - Isolation: Changes not visible until commit (simulated via comments; in practice, use multiple sessions).
-- - Durability: Committed changes persist (verify with SELECT after commit).

-- Successful transaction (Atomicity and Durability)
BEGIN TRANSACTION;

INSERT INTO FeePayments (payment_id, student_name, amount, payment_date) VALUES
    (8, 'Arjun', 4000.00, '2024-06-09');

COMMIT;

-- Simulate Isolation: In another session, during the above transaction (before commit), 
-- a SELECT would not see the new record. After commit, it would.

-- Failed transaction (rolls back, maintaining Consistency)
BEGIN TRANSACTION;

INSERT INTO FeePayments (payment_id, student_name, amount, payment_date) VALUES
    (9, 'Priya', 5000.00, '2024-06-10'),
    (8, 'DuplicateArjun', 4800.00, '2024-06-11');  -- Fails: duplicate payment_id

-- If an error occurs (as it will), rollback
ROLLBACK;

-- Verify Part D (Final state: original 3 + the 1 successful from this part, matching the sample image)
SELECT * FROM FeePayments;


