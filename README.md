# Pokémon Catcher API

This project is a Pokémon Catcher API built using [Hono](https://honojs.dev/), [Prisma](https://www.prisma.io/), and [posgresql](https://www.sqlite.org/). Users can register, log in, and catch Pokémon, which are fetched from the [PokéAPI](https://pokeapi.co/).

## Table of Contents

- [Usage](#usage)
- [API Endpoints](#api-endpoints)
- [Database Schema](#database-schema)
- [Environment Variables](#environment-variables)

## Usage

The API provides endpoints for user authentication, Pokémon data fetching, and managing the Pokémon caught by users. 

## API Endpoints

### Authentication Endpoints

- **Register a new user:**
    ```http
    POST /register
    ```
    **Request Body:**
    ```json
    {
      "email": "user@example.com",
      "password": "securepassword"
    }
    ```
    **Response:**
    ```json
    {
      "message": "user@example.com created successfully"
    }
    ```

- **Log in an existing user:**
    ```http
    POST /login
    ```
    **Request Body:**
    ```json
    {
      "email": "user@example.com",
      "password": "securepassword"
    }
    ```
    **Response:**
    ```json
    {
      "message": "Login successful",
      "token": "jwt_token"
    }
    ```

### Pokémon Data Fetching

- **Fetch Pokémon data by name:**
    ```http
    GET /pokemon/:name
    ```
    **Response:**
    ```json
    {
      "data": {
        "id": 1,
        "name": "bulbasaur",
        "height": 7,
      }
    }
    ```

### Protected User Resource Endpoints

- **Catch a Pokémon:**
    ```http
    POST /protected/catch
    ```
    **Request Body:**
    ```json
    {
      "name": "bulbasaur"
    }
    ```
    **Response:**
    ```json
    {
      "message": "Pokemon caught",
      "data": {
        "id": "uuid",
        "userId": "uuid",
        "pokemonId": "uuid",
        "caughtAt": "timestamp"
      }
    }
    ```

- **Release a caught Pokémon by ID:**
    ```http
    DELETE /protected/release/:id
    ```
    **Response:**
    ```json
    {
      "message": "Pokemon released"
    }
    ```

- **Get all caught Pokémon for the logged-in user:**
    ```http
    GET /protected/caught
    ```
    **Response:**
    ```json
    {
      "data": [
        {
          "id": "uuid",
          "pokemon": {
            "id": "uuid",
            "name": "bulbasaur"
          },
          "caughtAt": "timestamp"
        },
        {
          "id": "uuid",
          "pokemon": {
            "id": "uuid",
            "name": "charmander"
          },
          "caughtAt": "timestamp"
        },
        //more caught pokemons
      ]
    }
    ```

## Database Schema

The database schema is defined using Prisma and includes the following models:

- **User:**
    - `id`: UUID
    - `email`: String (unique)
    - `hashedPassword`: String
    - `caughtPokemon`: Relationship to `CaughtPokemon`

- **Pokemon:**
    - `id`: UUID
    - `name`: String (unique)
    - `caughtPokemon`: Relationship to `CaughtPokemon`

- **CaughtPokemon:**
    - `id`: UUID
    - `userId`: Foreign key to `User`
    - `pokemonId`: Foreign key to `Pokemon`
    - `caughtAt`: DateTime (default to now)

## Environment Variables

The application requires the following environment variables to be set:

- `DATABASE_URL`: postgres://web102cap2db_user:HWMgVAkb6TwvNTm0DvZEWp9mwLNYnyw7@dpg-cppj52tds78s73e9r7ag-a.oregon-postgres.render.com/web102cap2db
