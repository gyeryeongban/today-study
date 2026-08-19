# Today's Study

A full-stack platform that combines **study-group matching** (create/search/join study groups, free board, to-do list, calendar, non-face-to-face video chat) with a **study-cafe room reservation system** (space owners list rooms, members reserve and review them), plus admin tools for managing the whole service.

Built as a team project during **BIT Academy**, continuing from an earlier prototype into a proper multi-user web application. The goal was to help people studying toward a shared goal (certification exams, job hunting, etc.) find and organize into groups more easily, with video chat and convenience features built specifically for studying, plus a self-contained space-reservation service on the side.

> **Note:** I joined this project on 2021.10.19, after initial prototype work had already begun, and stayed through 2021.11.23, when our team's development concluded. This README reflects the project as of that point.

## Table of Contents

- [Overview](#overview)
- [Features](#features)
- [Tech Stack](#tech-stack)
- [Architecture](#architecture)
- [Getting Started](#getting-started)
- [Project Structure](#project-structure)
- [Team](#team)
- [Retrospective](#retrospective)

## Overview

Internally, the Java code lives under the `com.ogong.pms` package, and the repo's DB scripts and ERD are named with the `ogong` prefix — the codename the team used for the service while building it.

## Features

### Full Feature Set

- **Study groups**: create/edit/delete a study post, search, bookmark, join-request & approval, ongoing/ended study lists, study leader delegation
- **Free Board**: post CRUD with comments, per-study community board
- **To-Do**: per-study to-do list CRUD
- **Calendar**: study schedule calendar (FullCalendar-based)
- **Study-Cafe reservation**: browse cafes/rooms, reserve a room, write/manage reviews; space-owner (CEO) side to register cafes/rooms and manage reservations
- **Ask Board**: 1:1 inquiry board for both members and space owners
- **Admin**: manage members, space owners, studies, cafes, notices, and inquiries
- **Auth**: member / space-owner / admin login, plus Google & Kakao social login

### My Contributions

- **Study**: registration, detail view, search, bookmark, join request/cancellation, ongoing/ended list views
- **Free Board**: post CRUD and comment CRUD within a study
- **To-Do**: add/edit/delete/list to-dos for a study
- **My Study**: study detail/exit, study-leader delegation and member management (Guilder)
- **Member**: initial `MariadbMemberDao` implementation (early stage of the project)
- Contributed to the DB schema/ERD (`ogongmodel.exerd`)

## Tech Stack

| Category | Details |
|---|---|
| Language | Java |
| Build | Gradle (multi-module: `app-client`, `app-server`) |
| Web | Java Servlet + JSP, deployed to a servlet container (Tomcat) — a hand-rolled front-controller/MVC framework (`Command`, `RequestDispatcher`, `*Handler`/`*Controller`) rather than Spring MVC |
| Persistence | MyBatis mapper XML over JDBC, MariaDB |
| Frontend | JSP + Bootstrap, jQuery/Ajax, [FullCalendar.js](https://fullcalendar.io/) for the study calendar |
| Auth | Google / Kakao OAuth social login alongside standard member/CEO(space-owner)/admin login |
| Other | File upload handling (profile photos, board attachments, cafe images), a chatbot feature |

## Architecture

The repository is a two-module Gradle project:

```
app-client/   # earlier DAO/domain layer carried over from the initial prototype
app-server/   # the actual Servlet + JSP web application
```

`app-server` follows a layered structure on top of a custom front-controller pattern:

```
Browser ── HTTP ── app-server (Servlet/JSP)
  web/*Controller      → handles requests per feature (study, member, cafe, admin, …)
  dao / dao/impl        → Mybatis*Dao implementations
  resources/**/mapper   → MyBatis mapper XML (SQL)
  domain / vo            → domain/value objects
  webapp/WEB-INF/jsp      → JSP views
```

`app-client` holds the domain/DAO classes and DB scripts from the project's earlier, pre-web-app stage; most active development after 10/19 happened in `app-server`. The repository root also contains a couple of small standalone folders (`app/`, `to-do/`) left over from early scratch/experimental work, separate from the two main modules.

## Getting Started

### Prerequisites

- JDK 8+
- MariaDB (or MySQL-compatible) instance
- A servlet container such as Tomcat

### Installation

1. Clone the repository

   ```bash
   git clone https://github.com/gyeryeongban/today-study.git
   cd today-study
   ```

2. Build the project

   ```bash
   ./gradlew build
   ```

### Configuration

Create the database and a matching user, then load the schema/sample data from the provided scripts:

```sql
CREATE DATABASE ogongdb;
CREATE USER 'ogong'@'localhost' IDENTIFIED BY '1111';
GRANT ALL PRIVILEGES ON ogongdb.* TO 'ogong'@'localhost';
FLUSH PRIVILEGES;
```

```bash
mysql -u ogong -p ogongdb < docs/pms_dbmodel/ogong-ddl.sql
mysql -u ogong -p ogongdb < docs/pms_dbmodel/ogong-member-data.sql
# load the remaining ogong-*-data.sql scripts as needed (study, cafe, freeboard, todo, ...)
```

The connection details are already set in `app-server/src/main/resources/com/ogong/pms/config/jdbc.properties`:

```properties
jdbc.driver=org.mariadb.jdbc.Driver
jdbc.url=jdbc:mariadb://localhost:3306/ogongdb
jdbc.username=ogong
jdbc.password=1111
```

> **Note:** `app-client` has its own, differently-keyed `jdbc.properties` (`driver`/`url`/`username`/`password` instead of the `jdbc.*`-prefixed keys above) left over from the earlier prototype stage — see [Architecture](#architecture). It points at the same local `ogongdb` database but isn't the one `app-server` actually reads at runtime.
>
> These are local development credentials only (not used anywhere beyond a local MariaDB instance) — swap them out before pointing this at anything beyond your own machine.

### Running the Application

Deploy the `app-server` module's WAR output to your servlet container (e.g. Tomcat) and access it through the configured host/port. MyBatis mappings are wired up via `mybatis-config.xml` in the same config directory as `jdbc.properties`.

## Project Structure

```
app-client/
└── src/main/java/com/ogong/pms/{domain,dao,dao/impl,handler,config}

app-server/
├── src/main/java/com/ogong/pms/
│   ├── web/            # Controllers, grouped by feature (study, member, cafe, ceoCafe, ceoMember, admin, askBoard, myStudy, ...)
│   ├── dao / dao/impl   # DAO interfaces + MyBatis implementations
│   ├── domain / vo      # domain & value objects
│   ├── filter/          # CharacterEncodingFilter
│   ├── listener/        # AppInitListener, FileListener
│   └── config/          # AppConfig, jdbc/mybatis config
├── src/main/resources/com/ogong/pms/mapper/   # MyBatis mapper XML
└── src/main/webapp/
    ├── WEB-INF/jsp/     # JSP views
    ├── css / img / upload
    └── WEB-INF/web.xml

docs/pms_dbmodel/       # ERD (.exerd), DDL, and sample data SQL scripts
```

## Team

| Name | Role | GitHub |
|---|---|---|
| Eunchae Kim | CEO(space-owner) member, Member, Study-Cafe, login/auth incl. Google/Kakao social login | [@Kimeunchaee](https://github.com/Kimeunchaee) |
| Gyeryeong Ban | Study CRUD/search/bookmark/join, Free Board & comments, To-Do, study-leader delegation (Guilder), initial Member DAO | [@gyeryeongban](https://github.com/gyeryeongban) |
| Dahye Song | Study-Cafe reservation & review, CEO(space-owner) member, Member | [@ssongdahye](https://github.com/ssongdahye) |
| Hyeongmin Woo | Study domain, Member, 1:1 Ask Board, CEO(space-owner) account, login/auth | [@woohyeongminn](https://github.com/woohyeongminn) |
| Sol Jo | Admin panel, Study, Member, social login | [@jo-sol](https://github.com/jo-sol) |

## Retrospective

My main takeaway was working across the full stack of a multi-feature service — from the study-matching core (search, bookmarking, membership approval) to a self-contained community board with comments — while coordinating a MyBatis-backed DAO layer with several other developers touching overlapping domains (member, study, cafe) in parallel.
