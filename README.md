# Bank Transaction System Backend

A Node.js and Express backend for user authentication, bank accounts, ledger-based balances, and idempotent account-to-account transactions.

## Features

- User registration, login, and logout
- JWT authentication through cookies or Bearer tokens
- MongoDB persistence with Mongoose
- Account creation and balance lookup
- Ledger-based credit and debit tracking
- Idempotent transactions using an `idempotencyKey`
- System-user initial funds transactions
- Registration email delivery through Gmail OAuth2

## Requirements

- Node.js 18+
- MongoDB database
- Gmail OAuth2 credentials if registration emails are enabled

## Setup

1. Install dependencies:

   ```bash
   npm install
   ```

2. Create a local environment file:

   ```bash
   copy .env.example .env
   ```

   On macOS/Linux, use `cp .env.example .env`.

3. Fill in the values in `.env`.

4. Start the development server:

   ```bash
   npm run dev
   ```

The API runs on `http://localhost:4000`.

## Environment Variables

| Variable | Description |
| --- | --- |
| `MONGO_URI` | MongoDB connection string |
| `JWT_SECRET` | Secret used to sign JWTs |
| `EMAIL_USER` | Gmail address used as the sender |
| `CLIENT_ID` | Google OAuth2 client ID |
| `CLIENT_SECRET` | Google OAuth2 client secret |
| `REFRESH_TOKEN` | Google OAuth2 refresh token |

Never commit `.env` or real credentials.

## API Endpoints

### Authentication

#### Register

```http
POST /api/auth/register
Content-Type: application/json
```

```json
{
  "email": "user@example.com",
  "name": "Example User",
  "password": "strong-password"
}
```

#### Login

```http
POST /api/auth/login
Content-Type: application/json
```

```json
{
  "email": "user@example.com",
  "password": "strong-password"
}
```

The response includes a JWT and the server also sets an authentication cookie.

#### Logout

```http
POST /api/auth/logout
```

### Accounts

All account endpoints require authentication using the `token` cookie or:

```http
Authorization: Bearer YOUR_TOKEN
```

| Method | Endpoint | Description |
| --- | --- | --- |
| `POST` | `/api/accounts` | Create an account for the logged-in user |
| `GET` | `/api/accounts` | List the logged-in user's accounts |
| `GET` | `/api/accounts/balance/:accountId` | Get an account balance |

### Transactions

#### Transfer funds

```http
POST /api/transactions
Authorization: Bearer YOUR_TOKEN
Content-Type: application/json
```

```json
{
  "fromAccount": "SOURCE_ACCOUNT_ID",
  "toAccount": "TARGET_ACCOUNT_ID",
  "amount": 100,
  "idempotencyKey": "transfer-unique-key-001"
}
```

The source and target accounts must be active. Reusing an idempotency key prevents the same transaction from being processed twice.

#### Add initial funds

```http
POST /api/transactions/system/initial-funds
Authorization: Bearer SYSTEM_USER_TOKEN
Content-Type: application/json
```

```json
{
  "toAccount": "TARGET_ACCOUNT_ID",
  "amount": 1000,
  "idempotencyKey": "initial-funds-unique-key-001"
}
```

This endpoint requires a user with `systemUser: true` and an account linked to that user. Create the system user's account through `POST /api/accounts` before using this endpoint.

## Project Structure

```text
.
├── server.js
├── src
│   ├── config
│   ├── controllers
│   ├── middleware
│   ├── models
│   ├── routes
│   └── services
├── .env.example
└── package.json
```

## Notes

- The server currently listens on port `4000`.
- MongoDB transactions require a MongoDB deployment that supports sessions and transactions, such as a replica set or MongoDB Atlas.
- Gmail OAuth2 credentials must be valid for the configured sender account.
- Add automated tests before using this backend in production.

## License

This project is currently distributed without a declared open-source license.
