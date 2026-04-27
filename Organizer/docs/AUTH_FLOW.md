# Authentication Flow

The API uses JWT bearer tokens passed via the `Authorization` header for all authenticated requests. A short-lived cookie named `user_token` is issued only when a guest session is created via `/auth/session`.

Guest accounts use the cookie so the backend can automatically resume the anonymous session without requiring storage on the client. When a user signs up or logs in, the cookie is removed and the client should store the returned JWT (e.g. in `localStorage`) and include it in the `Authorization` header for subsequent requests.

Regular accounts do **not** rely on the cookie.
