---
title: "Planning Analytics"
date: 2026-07-24
tags:
  - tm1
  - learning
  - work
  - qubefini
publish: false
---

# IBM Planning Analytics Workspace — Workshop

## Planning Analytics Engine (TM1 Engine)

- Planning Analytics is built on the **Applix TM1 engine**, created in **1999**
- Originally designed as a **financial allocation engine**
- It is a **pure in-memory OLAP engine**
- All calculations are resolved dynamically at query time

Core principle:

> Planning Analytics calculates relationships, not stored totals.

---

## Allocation Concept

**Allocation** refers to distributing values from a parent (C-level) element to child (N-level) elements.

Types of allocation:

- Equal distribution
- Driver-based distribution
	- Fixed drivers
	- Variable drivers (e.g., revenue share, volume, headcount)

This allocation capability is the foundation of TM1.

---

## Data Storage Model

- Data is stored as **intersections of dimension elements**
	- Example: `(Product=P1, Month=Jan, Measure=Sales)`
- Dimensions contain:
	- **N-level elements** → store data
	- **C-level elements** → calculated dynamically
- Consolidations are **not stored**, only computed when queried

Memory optimization:

- Only populated intersections exist in memory
- Cube compression can reduce memory usage by up to **85%**
- Enables fast read performance even with large cubes

---

## In-Memory Engine: Trade-offs

### Advantages

- Extremely fast read and write
- Real-time recalculation
- High concurrency support

### Disadvantages

- Memory fragmentation over time
- Garbage collection is not continuous

Important behavior:

- **Memory cleanup requires a server restart**
- Servers are typically restarted before major forecast cycles
- On restart, memory is reloaded cleanly

---

## Hierarchy, Element Weight, and Basic Calculations

- Dimensions define **parent–child hierarchies**
- Calculations such as addition and subtraction should be handled via **hierarchy consolidation**
- **Element weight** controls aggregation logic
	- Default weight = `1`
	- Negative values (e.g., `-1`) enable subtraction
- Best practice:
	- Use hierarchy + weights for simple math
	- Avoid rules for basic additions

---

## Rules and Rule Editor

- Complex calculations (multiplication, division, conditional logic) require **rules**
- Rules are written in the **rule editor**, which has its own execution engine
- Rules operate only on relevant intersections in memory

Performance optimization:

- Use `SKIPCHECK;` to avoid unnecessary consolidation checks
- Required when rules rely on feeders

---

## Feeders

- Feeders inform the engine **which cells should be calculated**
- Without feeders, rule-driven cells may return zero

Thumb rules:

- Addition/subtraction rules → tend to overfeed
- Multiplication/division rules → must be driver-based
- Underfeeding = incorrect results
- Overfeeding = excessive memory usage

Division rule best practice:

- Avoid `/` (forward slash)
- Use `\` (TM1-safe division)
- Prevents `NA` errors when denominator is zero

---

## Feeder Testing

- Feeders are validated by:
	- Restarting the server
	- Checking calculated values post-restart
- Feeders may appear to work until memory refresh occurs
- Restart ensures true validation

---

## Platform and Administration Overview

Planning Analytics is positioned as a **platform**, not just a tool.

Main sections:

- Applications & Plans
- Reports & Analysis
- Data Model
- Administration

---

## Database Services (Not SQL)

- "Databases" in PA are **services**, not SQL databases
- Entirely **file-based and encrypted**
- Each service hosts a model
- Services are separated for:
	- Security
	- Departmental isolation
	- Performance control

---

## Service Monitoring

Service properties include:

- Health status
- CPU usage
- Memory consumption
- IP and port details for API access

Administrators can view:

- Thread activity
- Active user sessions
- Running processes

---

## Authentication and Configuration

- Authentication methods:
	- CAM ID
	- HTTP Sub ID
	- Email / LDAP (cloud setups)
- Configuration parameters control:
	- Feature toggles
	- Data spread
	- Startup TI processes

---

## Log Files and Server Optimization

Important logs:

- `server_starter.log`
- Main server log
- Event log (generated on crash)
- Action logs (cloud environments)

Uses:

- Performance troubleshooting
- Crash analysis
- Object load timing
- Error diagnosis

---

## TM1S File and Data Integrity

- `TM1S` file ensures **data persistence**
- Writes happen in parallel during runtime
- Prevents data loss in case of crashes
- Backup TM1S files are created periodically (e.g., midnight)

This mechanism ensures **millisecond-level data loss at worst** and is a core patented feature.

---

## Security Roles

Three primary roles:

- **Analyst** → Consumer / end-user
- **Modeler** → Development, rules, security
- **Administrator** → Full server control

Key distinction:

- Modelers cannot add server-level users
- Administrators manage services and system configuration

---

## Data Model Components

Inside a service:

- Cubes
- Dimensions
- TI Processes
- Control Objects

**Control Objects** act like a master database:

- Track all objects
- Maintain system metadata
- Essential for system stability

---

## Dimensions

- Represent grouped elements (similar to `GROUP BY` in SQL)
- Minimum **two dimensions** required per cube
- Recommended:
	- 6 dimensions (ideal)
	- 8 dimensions (upper practical limit)

Types of dimensions:

- Time
- Measure
- Version
- Hierarchy
- Simple (lists / placeholders)

---

## Placeholder Dimension

- Always include **one placeholder dimension**
- Allows future restructuring without cube recreation
- Commonly used for adjustments or scenario extensions

---

## Virtual Hierarchy

- Replaces older parallel hierarchy approach
- Built using attributes
- Created on-demand in memory
- Uses temporary memory only
- Improves flexibility and memory efficiency

---

## Dimension Order and Sparsity

- Measures are always **dense**
- Time and version are usually less sparse
- Product and entity are highly sparse
- Dimension order impacts memory layout and performance

---

## Cube Creation and Reordering

- Initial cube design is based on assumptions
- Real data reveals true sparsity
- **Cube reordering**:
	- Rearranges dimension order
	- Reduces memory footprint
	- Improves performance
- Should be done periodically after data stabilizes

---

## TI Processes (Turbo Integrator)

Used for:

- Data loading
- Transformations
- P&L generation
- Heavy computations

Characteristics:

- CPU-intensive
- Low object memory usage
- Faster for bulk calculations than rules

---

## Chores (Schedulers)

- Used to schedule TI processes
- Contain no business logic
- Can run processes sequentially or in parallel

Best practice:

- Keep chores simple
- Keep logic inside TI processes

---

## Dimension Creation

Two methods:

- Manual
	- Copy & paste
	- Insert elements
- TI-based
	- CSV-driven
	- Automated
	- Preferred for dynamic dimensions

Rule:

> If it matters tomorrow, automate it today.

---

## Naming Conventions

### Measure Dimensions

- Use technical names
- Suffix `_m` recommended
- Example: `price_quantity_m`

### Time Dimensions

- Start name with `t`
- Example: `t_months`
- Built-in time dimension generator can be used for year-wise tracking

### Element Naming

- Technical names for rules and TI
- Aliases for business-friendly display
- Element names cannot be edited once created

---

## Workshop Next Steps

- New access link to be shared
- Workshop data files will be provided
- Participants will:
	- Choose a case study
	- Design a model
	- Build it during the second half
- Control Objects and lifecycle management to be covered later

## Related

- [[mdx-queries]]
- [[scheduler-optimization]]
- [[qubefini]]
