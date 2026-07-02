# MERN Stack Implementation: To-Do Application

## Overview

This documentation walks through the implementation of a full-stack **To-Do application** using the **MERN stack**. It covers backend setup, database configuration, API testing, and frontend development, resulting in a complete web application deployed on an AWS EC2 instance.

### What is the MERN Stack?

The MERN stack consists of four core technologies:

| Component | Role |
|---|---|
| **M**ongoDB | A NoSQL database that stores data in JSON-like documents, offering flexibility and scalability for web applications. |
| **E**xpress.js | A web framework for Node.js that provides a robust set of features for building server-side applications and APIs. |
| **R**eact.js | A front-end JavaScript library (developed by Facebook) used to build dynamic, single-page user interfaces without full page reloads. |
| **N**ode.js | A JavaScript runtime built on Chrome's V8 engine, enabling server-side execution of JavaScript for full-stack development in a single language. |

Together, these four technologies form a complete framework for building dynamic web applications.

### Project Scope

We will build a simple **To-Do application** that allows users to:
- Create tasks
- View (retrieve) tasks
- Delete tasks

- **Frontend:** React.js
- **Backend:** Node.js + Express.js
- **Database:** MongoDB

The project is divided into two major parts:
1. **Backend configuration** (server, database, API routes)
2. **Frontend configuration** (React UI)

---

## Prerequisites

- An AWS EC2 instance (Ubuntu) with SSH access
- A terminal/SSH client
- A [Postman](https://www.postman.com/) account (for API testing)
- A [MongoDB Atlas](https://www.mongodb.com/atlas) account (referred to as "MLab" in the walkthrough, but the modern equivalent is MongoDB Atlas)
