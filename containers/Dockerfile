FROM node:26.9.0-bookworm-slim@sha256:582460f614631b59b824ac6020533b9bf339c7fdf3a6d7db31abb6b4065f0212 AS node
FROM rust:1.94.0-bookworm@sha256:365468470075493dc4583f47387001854321c5a8583ea9604b297e67f01c5a4f AS build
WORKDIR /source
COPY Cargo.toml Cargo.lock ./
COPY crates ./crates
COPY xtask ./xtask
RUN --mount=type=cache,target=/usr/local/cargo/registry --mount=type=cache,target=/source/target cargo test --locked -p yoneda-runtime -- --test-threads=1 && cargo build --locked --release -p yoneda-runtime && cp target/release/yoneda-runtime /yoneda-runtime

FROM rust:1.94.0-bookworm@sha256:365468470075493dc4583f47387001854321c5a8583ea9604b297e67f01c5a4f
COPY --from=node /usr/local/bin/ /usr/local/bin/
COPY --from=node /usr/local/lib/node_modules/ /usr/local/lib/node_modules/
RUN apt-get update && apt-get install -y --no-install-recommends git ca-certificates python3 && rm -rf /var/lib/apt/lists/* \
    && npm install -g pnpm@11.16.0 @openai/codex@0.161.0 @anthropic-ai/claude-code@2.1.293 @google/gemini-cli@0.63.0 \
    && useradd --uid 1000 --create-home agent && mkdir -p /work /home/agent/.cargo && chmod 700 /root
COPY --from=build /yoneda-runtime /usr/local/bin/yoneda-runtime
EXPOSE 8080
ENTRYPOINT ["/usr/local/bin/yoneda-runtime"]
