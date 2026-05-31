![[Pasted image 20260531104349.png]]

Data has no meaning by itself. It has to be processed first.

Data- Collection of raw unorganised bytes

Information- Processed, organised and structured data

Database- System where data is stored in a way it can be easily accessed, managed and updated

DBMS (Database management system)- Collection of interrelated data (Database) and a set of programs to manage that data

![[Pasted image 20260531105715.png]]


## Why not just use File systems? (or Why use DBMS?)

1. Data redundancy- Multiple files may store the same information of users
2. Data inconsistency- Change to some data in one file creates inconsistent copies in other files
3. Difficulty in accessing data
4. Data isolation
5. Integrity problems- Suppose we have a constraint that savings ac balance cannot go less than 10k. With file systems, we have to change the source code to implement this functionality. Ok, but if these changes are needed frequently or if we want to add more features, then we have to do more work
6. Atomicity problems- Some transactions should be atomic. This is easy to implement with DBMS but not file systems, we need to add extra checks
7. Concurrent access anomalies- We need functionality to handle concurrent transactions. These functionalites need to be manually added in File systems
8. Security problems- We may want to restrict access to specific data. This functionality comes with DBMS, but has to be added with FS.

TLDR: DBMS comes with extra functionality to manage data. FS needs such things to be manually implemented

# DBMS Architecture

## View of data (3 schema architecture)

![[Pasted image 20260531133921.png]]

DBMS uses abstraction to provide a 'view' of the database to different types of Users.

For example, There could be a Logistics dept, Customers service dept, etc. They need access to only some specific data (like Logistics dept may need only Name, Addr, Phno of users, while CSDept may need some others columns too). DBMS provides a different view to each of them. 

![[Pasted image 20260531134411.png]]

## (PLV)

![[Pasted image 20260531135740.png]]

### Physical level: The actual data

- **i.** The lowest level of abstraction, describes how the data are stored.
    
- **ii.** Low-level data structures used.
    
- **iii.** It has **Physical schema** which describes physical storage structure of DB.
    
- **iv.** Deals with: Storage allocation (N-ary tree etc), Data compression & encryption etc.
    
- **v.** **Goal:** We must define algorithms that allow efficient access to data.

**Physical data independence is achieved, ie, change in physical schema should NOT affect the logical schema or upper levels
### Logical level / Conceptual level: Describes what the data means

- **i.** The **conceptual/logical schema** describes the design of a database at the conceptual level, describes _what_ data are stored in DB, and what _relationships_ exist among those data (like Students DB and Courses DB are related, as students apply to courses). We map the data from physical -> logical level using the logical schema. 
    
- **ii.** User at logical level does not need to be aware about physical-level structures.
    
- **iii.** **DBA**, who must decide what information to keep in the DB use the logical level of abstraction.
    
- **iv.** **Goal:** ease to use.
    
Contains- attributes of tables, consistency constraints, relationships
Logical schema is the most important for programmers
### View level / External level:

- **i.** Highest level of abstraction, aims to simplify users' interaction with the system by providing different view to different **end-user**s.
    
- **ii.** Each **view schema** describes the database part that a particular user group is interested and hides the remaining database from that user group.
    
- **iii.** At the external level, a database contains several schemas that sometimes called as **subschema**. The subschema is used to describe the different view of the database. (Like one for Logistics and one for Customer service)
    
- **iv.** Views also provide a **security** mechanism to prevent users from accessing certain parts of DB.

![[Pasted image 20260531141433.png]]

## Instance of DB

Describes the data in a DB at a point of time (snapshot of DB)

## Data Models (design at the logical level)

![[Pasted image 20260531142150.png]]

## DBMS languages (How to use DBMS)

![[Pasted image 20260531142444.png]]

DDL- schema (design) of different tables of a DB (ex- HOW a student looks like, what attributes they have)
DML- modify the data (Insert, Delete, Update, Retrieval- RUDI)

![[Pasted image 20260531143053.png]]

![[Pasted image 20260531143115.png]]

![[Pasted image 20260531143324.png]]

**DBA works at the logical level

## DBMS application architecture

![[Pasted image 20260531143738.png]]
![[Pasted image 20260531143905.png]]

T1 example- localhost, where DB and server are both present

T2

![[Pasted image 20260531144052.png]]

problem- What if client sends a stupid query that fucks up the DB?

T3- Client communicates thru an application server

