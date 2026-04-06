# Smart Hostel Management System - Database Entities & Relationships

## Table of Contents

1. [Entity Classification](#1-entity-classification)
2. [Strong Entities](#2-strong-entities)
3. [Weak Entities](#3-weak-entities)
4. [Relationship Types & Cardinality](#4-relationship-types--cardinality)
5. [Foreign Key Hierarchy](#5-foreign-key-hierarchy)
6. [Functional Clusters](#6-functional-clusters)
7. [Constraints Summary](#7-constraints-summary)

---

## 1. Entity Classification

The database contains **26 entities** in total:

| Category | Count |
|---|---|
| Strong Entities | 4 |
| Weak Entities | 22 |
| 1:1 Relationships | 2 |
| 1:N Relationships | 45+ |
| M:N Relationships | 1 |

---

## 2. Strong Entities

Strong entities exist independently and do not depend on any other entity for their identification.

| Entity | Primary Key | Description |
|---|---|---|
| **hostel_block** | block_id | Physical hostel blocks/buildings |
| **staff** | staff_id | Staff members (warden, admin, housekeeping, security, maintenance) |
| **student** | student_id | Student residents of the hostel |
| **found_items** | found_id | Found items reported by staff (can exist without a corresponding lost item) |

---

## 3. Weak Entities

Weak entities **cannot exist without their parent (owner) entity**. If the parent is deleted, the weak entity is also removed (via ON DELETE CASCADE) or its reference is nullified (via ON DELETE SET NULL).

### 3.1 Direct Weak Entities (Depend on Strong Entities)

| Weak Entity | Owner Entity | Identifying Relationship | On Delete |
|---|---|---|---|
| **room** | hostel_block | A block contains rooms | CASCADE |
| **allocation** | student, room | A student is allocated to a room | CASCADE |
| **emergency_contact** | student | A student has emergency contacts | CASCADE |
| **fees** | student | A student owes fee records | CASCADE |
| **inquiry** | student, staff | A disciplinary inquiry involves a student | CASCADE / SET NULL |
| **notifications** | student, staff | A notification is sent to a user | CASCADE |
| **room_complaints** | room, student | A student files a complaint about a room | CASCADE |
| **room_keeping** | room, staff | A housekeeping schedule for a room by staff | CASCADE |
| **biometric_log** | student | Entry/exit log for a student | CASCADE |
| **parcels** | student, staff | A parcel arrives for a student | CASCADE / SET NULL |
| **visitor_log** | student, staff | A visitor visits a student | CASCADE / SET NULL |
| **lost_items** | student | A student reports a lost item | CASCADE |
| **room_change_request** | student, room, hostel_block | A student requests a room change | CASCADE / SET NULL |
| **waitlist** | student, hostel_block | A student waits for a room in a block | CASCADE / SET NULL |
| **roommate_preference** | student | A student's roommate preferences | CASCADE |
| **roommate_compatibility** | student, student | Compatibility score between two students | CASCADE |
| **hostel_feedback** | student | A student submits feedback | CASCADE |
| **laundry_order** | student | A student places a laundry order | CASCADE |
| **audit_logs** | staff / student | An action performed by a user | No FK constraint |

### 3.2 Multi-Level Weak Entities (Depend on Other Weak Entities)

| Weak Entity | Owner (also Weak) | Chain | On Delete |
|---|---|---|---|
| **payments** | fees, student | student -> fees -> payments | CASCADE |
| **laundry_items** | laundry_order | student -> laundry_order -> laundry_items | CASCADE |
| **item_claim** | lost_items, found_items | student -> lost_items -> item_claim | CASCADE |

**Multi-level dependency chain example:**
```
student (Strong)
  └── laundry_order (Weak - Level 1)
        └── laundry_items (Weak - Level 2)
```

### 3.3 Partial Keys

| Weak Entity | Partial Key | Full Identification |
|---|---|---|
| **laundry_items** | item_type | order_id + item_type |
| **item_claim** | claim context | lost_id and/or found_id |
| **allocation** | check_in_date | student_id + check_in_date |

---

## 4. Relationship Types & Cardinality

### 4.1 One-to-One (1:1) Relationships

| Entity A | Entity B | Description | Enforced By |
|---|---|---|---|
| hostel_block | staff | Each block has exactly one warden | FK warden_id |
| student | roommate_preference | Each student has one preference set | UNIQUE constraint on student_id |

### 4.2 One-to-Many (1:N) Relationships

#### From hostel_block (1) to Many (N)

| Many Side | FK Column | Description |
|---|---|---|
| room | block_id | One block has many rooms |
| room_change_request | requested_block | Many requests target one block |
| waitlist | block_id | Many students waitlisted for one block |

#### From student (1) to Many (N)

| Many Side | FK Column | Description |
|---|---|---|
| allocation | student_id | One student has many allocations over time |
| emergency_contact | student_id | One student has many emergency contacts |
| fees | student_id | One student has many fee records |
| payments | student_id | One student makes many payments |
| inquiry | student_id | One student may face many inquiries |
| notifications | student_id | One student receives many notifications |
| room_complaints | student_id | One student files many complaints |
| biometric_log | student_id | One student has many entry/exit logs |
| parcels | student_id | One student receives many parcels |
| visitor_log | student_id | One student has many visitors |
| lost_items | student_id | One student reports many lost items |
| room_change_request | student_id | One student makes many room change requests |
| waitlist | student_id | One student can be on many waitlists |
| hostel_feedback | student_id | One student submits many feedback forms |
| laundry_order | student_id | One student places many laundry orders |

#### From staff (1) to Many (N)

| Many Side | FK Column | Description |
|---|---|---|
| inquiry | disciplinary_officer | One officer handles many inquiries |
| notifications | staff_id | One staff sends many notifications |
| room_keeping | staff_id | One staff has many cleaning schedules |
| parcels | received_by | One staff receives many parcels |
| visitor_log | approved_by_staff_id | One staff approves many visitors |
| found_items | found_by_staff | One staff reports many found items |
| room_change_request | reviewed_by | One staff reviews many requests |

#### From room (1) to Many (N)

| Many Side | FK Column | Description |
|---|---|---|
| allocation | room_id | One room has many allocations |
| room_complaints | room_id | One room has many complaints |
| room_keeping | room_id | One room has many cleaning schedules |
| room_change_request | current_room_id | Many requests reference one current room |

#### From fees (1) to Many (N)

| Many Side | FK Column | Description |
|---|---|---|
| payments | fee_id | One fee record has many payments |

#### From laundry_order (1) to Many (N)

| Many Side | FK Column | Description |
|---|---|---|
| laundry_items | order_id | One order contains many items |

#### From lost_items / found_items (1) to Many (N)

| Many Side | FK Column | Description |
|---|---|---|
| item_claim | lost_id | One lost item can have many claims |
| item_claim | found_id | One found item can have many claims |

### 4.3 Many-to-Many (M:N) Relationships

| Entity A | Entity B | Junction Table | Description |
|---|---|---|---|
| student | student | **roommate_compatibility** | Self-referencing M:N - any two students can have a compatibility score |

**Implementation:**
```sql
roommate_compatibility (
    compatibility_id  -- PK
    student1_id       -- FK -> student.student_id
    student2_id       -- FK -> student.student_id
    compatibility_percentage  -- 0 to 100
)
```

---

## 5. Foreign Key Hierarchy

### Level 0 - Root Entities (No Dependencies)

```
hostel_block
staff
student
```

### Level 1 - Direct Dependencies on Root Entities

```
room              -> hostel_block
allocation        -> student, room
emergency_contact -> student
fees              -> student
inquiry           -> student, staff
notifications     -> student, staff
room_complaints   -> room, student
room_keeping      -> room, staff
biometric_log     -> student
parcels           -> student, staff
visitor_log       -> student, staff
lost_items        -> student
found_items       -> staff
room_change_request -> student, room, hostel_block, staff
waitlist          -> student, hostel_block
roommate_preference -> student
roommate_compatibility -> student, student
hostel_feedback   -> student
laundry_order     -> student
audit_logs        -> (references staff/student without FK)
```

### Level 2 - Dependencies on Level 1 Entities

```
payments      -> fees, student
laundry_items -> laundry_order
item_claim    -> lost_items, found_items
```

---

## 6. Functional Clusters

The 26 entities are organized into **8 functional clusters**:

### Cluster 1: Core Management
| Entity | Role |
|---|---|
| hostel_block | Physical buildings |
| staff | Personnel |
| room | Individual rooms |
| student | Residents |
| allocation | Room assignments |

### Cluster 2: Security & Discipline
| Entity | Role |
|---|---|
| emergency_contact | Student safety contacts |
| inquiry | Disciplinary proceedings |
| notifications | System alerts |
| biometric_log | Entry/exit tracking |

### Cluster 3: Financial
| Entity | Role |
|---|---|
| fees | Fee records |
| payments | Payment transactions |

### Cluster 4: Maintenance
| Entity | Role |
|---|---|
| room_complaints | Maintenance requests |
| room_keeping | Housekeeping schedules |

### Cluster 5: Logistics
| Entity | Role |
|---|---|
| visitor_log | Visitor management |
| parcels | Parcel tracking |

### Cluster 6: Property Recovery (Lost & Found)
| Entity | Role |
|---|---|
| lost_items | Lost item reports |
| found_items | Found item records |
| item_claim | Claims linking lost and found |

### Cluster 7: Room Compatibility & Assignment
| Entity | Role |
|---|---|
| roommate_preference | Student preferences |
| roommate_compatibility | Pairwise compatibility scores |
| waitlist | Room allocation queue |
| room_change_request | Transfer requests |

### Cluster 8: Services & Audit
| Entity | Role |
|---|---|
| hostel_feedback | Student feedback |
| laundry_order | Laundry service orders |
| laundry_items | Individual laundry items |
| audit_logs | System audit trail |

---

## 7. Constraints Summary

### Unique Constraints
| Table | Column | Purpose |
|---|---|---|
| hostel_block | block_name | No duplicate block names |
| staff | email | No duplicate staff emails |
| student | email | No duplicate student emails |
| roommate_preference | student_id | Enforces 1:1 with student |

### Check Constraints (Key Examples)

| Table | Constraint |
|---|---|
| room | capacity > 0, current_occupancy >= 0, current_occupancy <= capacity |
| room | room_type IN ('single', 'double', 'triple', 'dormitory') |
| student | gender IN ('male', 'female', 'other') |
| student | year_of_study BETWEEN 1 AND 6, cgpa BETWEEN 0 AND 10 |
| fees | amount > 0, status IN ('pending', 'paid', 'overdue', 'waived') |
| payments | payment_status IN ('completed', 'pending', 'failed', 'refunded') |
| roommate_compatibility | compatibility_percentage BETWEEN 0 AND 100 |
| hostel_feedback | all ratings BETWEEN 1 AND 5 |

### ON DELETE Behavior

| Behavior | Applied To |
|---|---|
| **CASCADE** | Most parent-child relationships (fees, payments, allocations, contacts, complaints, logs, orders, etc.) |
| **SET NULL** | Optional staff references (inquiry.disciplinary_officer, parcels.received_by, visitor_log.approved_by, room_change_request.reviewed_by) |

---

## Visual Summary: Entity-Relationship Overview

```
                          ┌──────────────┐
                          │ hostel_block  │ (Strong)
                          └──────┬───────┘
                     1:1 warden │          1:N
              ┌─────────────────┤──────────────────┐
              v                 v                   v
       ┌──────────┐      ┌──────────┐        ┌──────────┐
       │  staff   │      │   room   │        │ waitlist  │
       │ (Strong) │      │  (Weak)  │        │  (Weak)  │
       └────┬─────┘      └────┬─────┘        └──────────┘
            │                  │
     ┌──────┴──────┐    ┌─────┴─────────┐
     v             v    v               v
 ┌────────┐  ┌────────┐ ┌─────────┐ ┌───────────┐
 │inquiry │  │room_   │ │room_    │ │allocation │
 │        │  │keeping │ │complaints│ │           │
 └────────┘  └────────┘ └─────────┘ └─────┬─────┘
                                          │
                                          v
                                   ┌──────────┐
                                   │ student  │ (Strong)
                                   └────┬─────┘
                                        │ 1:N to many weak entities
          ┌────────┬────────┬───────┬───┴───┬────────┬──────────┐
          v        v        v       v       v        v          v
     ┌────────┐┌──────┐┌───────┐┌─────┐┌───────┐┌────────┐┌────────┐
     │  fees  ││bio_  ││parcels││lost_││visitor││feedback││laundry_│
     │        ││log   ││       ││items││_log   ││        ││order   │
     └───┬────┘└──────┘└───────┘└──┬──┘└───────┘└────────┘└───┬────┘
         v                         v                           v
     ┌────────┐              ┌──────────┐               ┌──────────┐
     │payments│              │item_claim│               │laundry_  │
     │(Lvl 2) │              │ (Lvl 2)  │               │items     │
     └────────┘              └──────────┘               │(Lvl 2)  │
                                   ^                    └──────────┘
                             ┌─────┘
                        ┌────────────┐
                        │found_items │ (Strong)
                        └────────────┘

     M:N (Self-referencing):
     student ◄──── roommate_compatibility ────► student
```

---

*Generated from schema analysis of `/database/schema.sql`*
