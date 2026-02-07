# Product Requirements Document: PartnerTools CRM

## 1. Executive Summary

PartnerTools CRM is a high-performance, cost-efficient Customer Relationship Management platform. It aims to replace expensive Salesforce per-user licensing with a JAMstack architecture powered by a Rust API.

By leveraging the Salesforce -like SuiteCRM SQL table structure aligned with the Microsoft Common Data Model (CDM), the system ensures data interoperability across agencies while reducing infrastructure costs. The platform focuses on speed, type safety, and "write-once, deploy-everywhere" web standards.

## 2. Problem Statement

High Costs: Licensing fees for Salesforce and Dynamics are a significant recurring drain.
Data Silos: Without a shared data language, integrating data from materials, manufacturing, and sales (or agency equivalents) requires custom, expensive implementations.

Performance: Legacy CRM architectures (monolithic PHP/Java) often suffer from slow load times and security vulnerabilities compared to modern static-first approaches.

## 3. Goals & Success Metrics

Cost Reduction: Reduce CRM operating costs by eliminating per-user fees.
Performance: API response times under 50ms using Rust.
Interoperability: Achieve schema compliance with the Common Data Model (CDM) for core entities (accounts, contacts).

## 4. Technical Architecture

4.1. The Stack (JAMstack)

Frontend (Markup/JS): A Single Page Application (SPA) served via CDN. 
This decouples the web experience from the business logic.
API (The "A"): A REST/GraphQL API built in Rust.
Database: PostgreSQL, structured to mirror SuiteCRM. (See model.earth/profile/crm)

4.2. Rust API Specification

See: https://github.com/ModelEarth/team/blob/main/Cargo.toml

Framework: Actix-web for high-performance, async handling.

Need to confirm these are the ones used in Cargo.toml link above:
ORM Strategy: SeaORM is recommended over Diesel or SQLx.Rationale: SeaORM is async-native and inspired by SQLAlchemy, allowing for dynamic query construction which is essential for handling complex CRM schemas that may not have 1:1 struct mappings.

Security: Rust’s memory safety ensures robust handling of sensitive government data.

## 5. Data Model & Database Schema

The database must be similar to the Salesforce-like SuiteCRM naming conventions while mapping conceptually to the Microsoft Common Data Model (CDM) to ensure standard shapes for "accounts" etc.

5.1. Core Identity & UUIDs

ID Format: All primary keys must use UUIDs (e.g., 46c35607-bcad-c7f1-1745-558d6b858b27) rather than auto-incrementing integers to facilitate data imports and merging.

Naming Convention: Table names must be lowercase.

5.2. Core Tables (CDM Mapped)

SuiteCRM Table NameCDM Entity EquivalentDescriptionaccountsAccountOrganizations/Agencies. Columns: id (UUID), name, billing\_address\_street, deleted (bool).contactsContactIndividuals. Columns: id, first\_name, last\_name, phone\_mobile.opportunitiesOpportunityGrants/Contracts. Columns: id, amount, sales\_stage.campaignsCampaignOutreach http://initiatives.email\_addressesEmailNormalized email storage.

5.3. Relationship Schema

Relationships are managed via distinct join tables rather than database-level foreign key constraints, mirroring SuiteCRM’s logic to allow for application-level handling of "soft deletes".
Many-to-Many Implementation:Table: accounts\_contacts
Query Logic: Rust API must handle joins manually.
Example SQL: SELECT http://accounts.name, contacts.last\_name FROM accounts INNER JOIN accounts\_contacts ON http://accounts.id = accounts\_contacts.account\_id.

Flex Relate:Fields parent\_type and parent\_id are used to link a record (e.g., a Call) to multiple potential entities (Account, Contact, or Lead).

5.4. Custom Fields (\_cstm) - TO BE DETERMINED

To support government-specific data without altering core schemas, custom fields are stored in \_cstm tables joined by id\_c.
Example: If an agency needs an "Age" field for a contact, it is stored in contacts\_cstm.age\_c.
Rust Implementation: The API must automatically perform LEFT JOIN on \_cstm tables when fetching detail views.

## 6. Feature Requirements

6.1. CRM Core
Entity Management: CRUD operations for Accounts, Contacts, Leads, and Opportunities.
Logic Hooks: Rust-based implementation of "Logic Hooks" to trigger actions (e.g., email notifications) on save/update, replacing PHP logic hooks.
Search: ElasticSearch integration for high-speed retrieval of UUID-based records.
6.2. Contextual View: Automatically pulls the CRM Contact record based on the sender\'s email address using the Rust API.

One-Click Archiving: Button to save email content to the emails table in the CRM.
Manifest Type: Configure strictly as a Web Add-in to prevent installation of legacy COM counterparts.

## 7. Migration & Integration Strategy

7.1. Data Migration
ETL Process: Extract data from Salesforce/Dynamics, transform IDs to UUIDs, and load into the Postgres accounts and contacts tables.

Schema Extension: Use the CDM's extensibility to map specific government verticals (e.g., Budget, Currency) into the standard model.
7.2. Interoperability
CDM Compliance: By adhering to the CDM metadata system, PartnerTools CRM data can be consumed by Microsoft PowerBI and Azure Data Lake without complex transformation.

## 8. Security & Compliance

Authentication: Better-auth OAuth2 implementation.

Authorization: Role-based access control (RBAC) mirroring SuiteCRM's "Security Groups" to ensure agencies can only view their own data.

Audit Logging: All relationship changes (e.g., removing a contact from an account) must be logged, as the DB does not enforce cascading deletes.

## 9. Roadmap

Phase 1 (Core): Rust API development; implementation of accounts and contacts schemas; UUID generation logic.

Phase 2 (Frontend): JAMstack UI deployment; basic CRUD features.

Phase 3 (Analytics): Integration of FinOps dashboards for real-time financial transparency.