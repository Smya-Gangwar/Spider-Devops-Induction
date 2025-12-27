# ---------- Builder ----------
FROM rust:1.86 AS builder

WORKDIR /app

# Install build deps
RUN apt-get update && apt-get install -y \
    libpq-dev \
    pkg-config \
    && rm -rf /var/lib/apt/lists/*

COPY Cargo.toml Cargo.lock ./
COPY src ./src

RUN cargo build --release


# ---------- Runtime ----------
FROM debian:bookworm-slim

# Install runtime dependency ONLY
RUN apt-get update && apt-get install -y \
    libpq5 \
    ca-certificates \
    && rm -rf /var/lib/apt/lists/*

WORKDIR /app

COPY --from=builder /app/target/release/login-app-backend .

EXPOSE 8080

CMD ["./login-app-backend"]
