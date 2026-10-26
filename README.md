# Soitinlainaus

Soitinlainaus ("instrument lending" in Finnish) is a web-based peer-to-peer rental service for musical instruments. Instrument owners can list their instruments for rent, and other users can send a rental request for them. The owner accepts the request, and when the rental period ends, the renter returns the instrument through the service.

This was a group project (Group-5) built during my studies at Savonia University of Applied Sciences, and it was my last school project.

**Authors:** Elias Salo, Kalle Karvinen, Elias Niskanen and Joni Kourunen

> The user interface of the application is in Finnish.

---

## Contents

- [What the application does](#what-the-application-does)
- [Technologies](#technologies)
- [Project structure](#project-structure)
- [Installation and running](#installation-and-running)
- [User guide](#user-guide)
- [API](#api)
- [My contribution](#my-contribution)
- [Use of AI](#use-of-ai)
- [Known limitations](#known-limitations)

---

## What the application does

The service has two roles, and the same user can act in both:

- **Owner (lender)** adds an instrument with its details (type, model, model year, price, description and picture), sees incoming rental requests and accepts them.
- **Renter** browses available instruments, sends a rental request for a chosen period and returns the instrument when the rental ends.

Rental lifecycle:

```
Instrument added  →  Rental request sent  →  Owner accepts  →  Rented out  →  Returned
                     (Lainauspyynto = true)                   (Lainattu = true)  (Lainattu = false)
```

The service also includes user registration and login, a personal profile (name, nickname, locality, introduction, profile picture) and a member list showing the other users of the service.

A visitor who is not logged in can see the front page and the list of available instruments, but renting requires registration.

## Technologies

| Part | Technology |
|---|---|
| Front end | Blazor WebAssembly (.NET 6) |
| Back end | ASP.NET Core Web API (.NET 6) |
| Database | SQLite + Entity Framework Core |
| API documentation | Swagger / OpenAPI |
| Styling | Bootstrap, Open Iconic icons, custom CSS |

## Project structure

```
Soitinlainaus/
├── Soitinlainaus.sln
├── src/
│   ├── BlazorPeople/     Blazor WebAssembly front end (pages, navigation, styles)
│   ├── PeopleApi/        Web API: controllers, database context, seed data, Rentals.db
│   └── PeopleLib/        Shared data models (Renter, RentalInstrument, Rental, ...)
└── UnitTests/            Unit test project (see Known limitations)
```

Key data models (`src/PeopleLib`):

- `Renter` – a user (name, nickname, locality, introduction, picture, email, password)
- `RentalInstrument` – an instrument (owner, type, model, model year, price, description, picture, status)
- `Rental` – a rental (start and end date, message, instrument, owner, renter)

## Installation and running

### Requirements

- [.NET 6 SDK](https://dotnet.microsoft.com/download/dotnet/6.0)
- Visual Studio 2022 (optional)

### With Visual Studio

1. Open `Soitinlainaus.sln`.
2. In the solution properties, choose **Multiple startup projects** and set both `PeopleApi` and `BlazorPeople` to start.
3. Run (F5).

### From the command line

Run both projects in separate terminals from the `src` folder:

```sh
# 1. Web API – starts at https://localhost:5000
dotnet run --project PeopleApi/PeopleApi.csproj

# 2. Front end – starts at https://localhost:3000
dotnet run --project BlazorPeople/BlazorPeople.csproj
```

Open `https://localhost:3000` in your browser. The Swagger view of the API is at `https://localhost:5000/swagger`.

> **Note:** The front end expects the API at `https://localhost:5000` (set in `src/BlazorPeople/Program.cs`). If you change the API port, update the address there as well.

### Database

The API uses the SQLite database `src/PeopleApi/Rentals.db`. If the database does not exist, it is created on startup and filled with test data (`SeedData.cs`). To start from a clean slate, delete `Rentals.db` and restart the API.

## User guide

### 1. Log in or register

From the front page you can log in or register. For testing you can use the ready-made test accounts:

| Username (email) | Password |
|---|---|
| `salasana` | `salasana` |
| `salasana2` | `salasana` |

A new user registers by filling in their name, nickname, email and password. The same email cannot be registered twice.

### 2. Navigation

After logging in, the menu on the left contains (UI labels in Finnish):

| Menu item | What it does |
|---|---|
| **Lainattavat soittimet** (Available instruments) | All instruments in the service; you can send a rental request from here |
| **Omat tiedot** (My details) | Edit your profile |
| **Jäsenet** (Members) | Other users of the service |
| **Lainatut soittimet** (Borrowed instruments) | Instruments you have rented from others |
| **Lainassa olevat soittimet** (Rented-out instruments) | Your own instruments that are currently rented out |
| **Lisää uusi soitin** (Add new instrument) | Add a new instrument to the service |
| **Lainauspyynnöt** (Rental requests) | Requests for your instruments |
| **Omat soittimet** (My instruments) | List and edit your own instruments |

### 3. Adding an instrument (owner)

1. Select **Lisää uusi soitin** (Add new instrument).
2. Fill in the type, model, model year, price and description, and add a picture.
3. Save. The instrument now appears to others under *Available instruments*.

You can edit the instrument later under **My instruments → Muokkaa soitinta** (Edit instrument).

### 4. Renting an instrument (renter)

1. Open **Available instruments** and choose an instrument with **Varaa soitin** (Reserve instrument).
2. Pick the start and end date of the rental and optionally write a message to the owner.
3. Send the request. The owner sees it in their *Rental requests* view.

### 5. Accepting a request (owner)

1. Open **Rental requests**.
2. Select **Siirry hyväksymään lainauspyyntö** (Go to accept request) and accept it.
3. The instrument's status changes to *rented out*.

### 6. Returning an instrument (renter)

1. Open the instrument you have rented and select **Palauta soitin** (Return instrument).
2. Confirm the return. The instrument becomes available for rent again.

### 7. Logging out

Log out from the menu. Login details are kept only for the browser session.

## API

The base address of the API is `https://localhost:5000`. All routes are listed and can be tried out in Swagger.

| Controller | Main routes |
|---|---|
| `/Renter` | `GET` all users, `GET /{id}`, `GET /login/{email}/{password}`, `GET /register/{email}`, `POST`, `PUT /{id}` |
| `/RentalInstruments` | `GET` all, `GET /{id}`, `GET /getinstrument/{id}`, `GET /update/{id}`, `POST`, `PUT /{id}`, `DELETE /{id}` |
| `/Rental` | `GET` all, `GET /{id}`, `GET /customer/{id}`, `GET /renteri/{id}`, `POST`, `PUT /{id}`, `DELETE /{id}` |
| `/RentalPicture` | `GET`, `POST` |
| `/ProfileDefaultPicture` | `GET` |

## My contribution

The project was built by a group of four, and I worked on almost every part of it:

- **Design:** the service logic and the rental flow (request → acceptance → rental → return)
- **Data models and database:** the `PeopleLib` classes, the Entity Framework context and the SQLite seed data
- **Web API:** controllers for users, instruments, rentals and pictures
- **Front end:** Blazor pages for registration, login, instrument management, renting and returning
- **Styling:** page styles, navigation and background images
- **Testing and documentation**

## Use of AI

The application code was written entirely without AI. AI (Claude) was used only to write this README file and polish its layout.

## Known limitations

This is a school project and is not intended for production use. Known limitations:

- Passwords are stored in plain text in the database, and login is done with a GET request that puts the credentials in the URL.
- The login state is kept in a static class (`LoginInfo`) on the front end rather than real authentication (e.g. JWT or cookies).
- The API allows CORS requests from any origin.
- The `UnitTests` project comes from the project template (*Kuopio Hikers*) and refers to classes that do not exist in this project, so the tests do not compile as they are.

## Useful links

- [ASP.NET Core Blazor](https://learn.microsoft.com/aspnet/core/blazor/)
- [Call a web API from ASP.NET Core Blazor](https://learn.microsoft.com/aspnet/core/blazor/call-web-api)
- [Entity Framework Core with SQLite](https://learn.microsoft.com/ef/core/providers/sqlite/)
