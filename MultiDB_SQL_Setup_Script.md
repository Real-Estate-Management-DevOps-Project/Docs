# 🧩 Multi-Database SQL Initialization Script

This SQL script sets up all the databases and tables for the **Property, Lease, User, Notification, and Audit microservices**.  
Each service maintains its own database schema for scalability and modularity.

---

## 💾 Setup Script

```sql
create database users_db;
use users_db;

-- CREATE TABLE `users` (
--   `id` BIGINT NOT NULL AUTO_INCREMENT,
--   `username` VARCHAR(255) NOT NULL UNIQUE,
--   `email` VARCHAR(255) NOT NULL UNIQUE,
--   `password_hash` VARCHAR(512) NOT NULL,
--   `role` VARCHAR(50) NOT NULL COMMENT 'e.g., admin, user, auditor',
--   `created_at` TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
--   `updated_at` TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
--   PRIMARY KEY (`id`)
-- );

-- =================================================================
-- REVISED DATABASE FOR User/Identity Service (.NET)
-- Suggested Database Name: users_db
-- =================================================================
CREATE TABLE `users` (
  `id` BIGINT NOT NULL AUTO_INCREMENT,
  `asgardio_user_id` VARCHAR(255) NOT NULL UNIQUE COMMENT 'The unique user ID (subject) from the Asgardio JWT. This is the new primary link.',
  `email` VARCHAR(255) NOT NULL UNIQUE,
  `username` VARCHAR(255) NOT NULL,
  `created_at` TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
  `updated_at` TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
  PRIMARY KEY (`id`),
  INDEX `idx_asgardio_user_id` (`asgardio_user_id`)
);

USE users_db;

-- Insert Example Users
INSERT INTO `users` (asgardio_user_id, email, username) VALUES
('a1b2c3d4e5f6g7h8i9j0k1l2m3n4o5p6', 'sys_admin@corp.com', 'sys_admin_profile'),
('p6o5n4m3l2k1j0i9h8g7f6e5d4c3b2a1', 'john.doe@appuser.net', 'john_doe_app'),
('z9y8x7w6v5u4t3s2r1q0p9o8n7m6l5k4', 'auditor.jane@corp.net', 'jane_audits');

-- =================================================================
-- AUDIT DATABASE
-- =================================================================
create database audit_db;
use audit_db;

CREATE TABLE `audit_logs` (
  `id` BIGINT NOT NULL AUTO_INCREMENT,
  `timestamp` TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
  `service_name` VARCHAR(100) NOT NULL COMMENT 'e.g., PropertyService, LeaseService',
  `user_id` BIGINT COMMENT 'User performing the action, NULL for system actions',
  `action` VARCHAR(255) NOT NULL COMMENT 'e.g., PROPERTY_CREATED, LEASE_TERMINATED',
  `details` JSON COMMENT 'Details about the event',
  PRIMARY KEY (`id`)
);

-- Sample Audit Entries
INSERT INTO `audit_logs` (timestamp, service_name, user_id, action, details) VALUES
(NOW(), 'PropertyService', 1, 'PROPERTY_CREATED', '{"property_id": 101, "location": "New York"}'),
(NOW(), 'IdentityService', 2, 'USER_LOGIN_SUCCESS', '{"ip": "192.168.1.5"}'),
(NOW(), 'SystemMaintenance', NULL, 'DB_BACKUP_COMPLETE', '{"backup_path": "/backups/20251013"}'),
(NOW(), 'IdentityService', 1, 'USER_ROLE_UPDATED', '{"target_user_id": 2, "new_role": "premium_user"}');

-- =================================================================
-- NOTIFICATION DATABASE
-- =================================================================
create database notifications_db;
use notifications_db;

CREATE TABLE `sent_notifications` (
  `id` BIGINT NOT NULL AUTO_INCREMENT,
  `recipient_user_id` BIGINT NOT NULL,
  `type` VARCHAR(100) NOT NULL,
  `message` TEXT NOT NULL,
  `status` VARCHAR(50) NOT NULL,
  `sent_at` TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
  PRIMARY KEY (`id`),
  INDEX `idx_recipient_user_id` (`recipient_user_id`)
);

-- Sample Notifications
INSERT INTO `sent_notifications` (recipient_user_id, type, message, status) VALUES
(2, 'RENT_REMINDER', 'Your rent is due in 5 days.', 'sent'),
(3, 'LEASE_EXPIRY', 'Your lease expired 9 months ago.', 'sent'),
(1, 'SYSTEM_ALERT', 'High server load detected.', 'failed');

-- =================================================================
-- PROPERTY DATABASE
-- =================================================================
create database properties_db;
use properties_db;

CREATE TABLE `properties` (
  `id` BIGINT NOT NULL AUTO_INCREMENT,
  `name` VARCHAR(255) NOT NULL,
  `address` TEXT NOT NULL,
  `type` VARCHAR(50) NOT NULL,
  `status` VARCHAR(50) NOT NULL DEFAULT 'available',
  `created_at` TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
  `updated_at` TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
  PRIMARY KEY (`id`)
);

-- Example Property Data
INSERT INTO `properties` (name, address, type, status) VALUES
('Sunshine Residences Unit 4A', '123 Main Street, Apt 4A', 'Apartment', 'occupied'),
('Tech Hub Commercial Center', '456 Enterprise Road', 'Building', 'available'),
('Green Acres Lot 5', 'Rural Route 7', 'Land', 'maintenance');

-- =================================================================
-- LEASE DATABASE
-- =================================================================
create database leases_db;
use leases_db;

CREATE TABLE `leases` (
  `id` BIGINT NOT NULL AUTO_INCREMENT,
  `property_id` BIGINT NOT NULL,
  `tenant_user_id` BIGINT NOT NULL,
  `start_date` DATE NOT NULL,
  `end_date` DATE NOT NULL,
  `rent_amount` DECIMAL(10,2) NOT NULL,
  `status` VARCHAR(50) NOT NULL,
  `created_at` TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
  `updated_at` TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
  PRIMARY KEY (`id`),
  INDEX `idx_property_id` (`property_id`),
  INDEX `idx_tenant_user_id` (`tenant_user_id`)
) COMMENT 'Links a property from properties_db to a user from users_db';

-- Example Lease Data
INSERT INTO `leases` (property_id, tenant_user_id, start_date, end_date, rent_amount, status) VALUES
(1, 2, '2024-09-01', '2025-08-31', 1850.50, 'active'),
(2, 3, '2023-01-15', '2024-01-14', 5500.00, 'expired'),
(1, 1, '2025-11-01', '2026-10-31', 1900.00, 'pending');
```

---

## 🧩 Notes

- Each service has its **own database** for modularity and scalability.
- `users_db` integrates with **Asgardio Identity Provider**.
- `audit_db` centralizes all audit logs from microservices.
- `leases_db` maintains foreign keys to both users and properties.
- `notifications_db` logs sent or failed notification events.
