# Builds the few-shot Spin component to Wasm and serves it.
# Env vars (valkey_url, inference_api_url, chat_model, inference_api_key, ...)
# are NOT baked in — pass them at runtime as SPIN_VARIABLE_<name>, e.g.:
#   docker run -p 3000:3000 \
#     -e SPIN_VARIABLE_VALKEY_URL="rediss://..." \
#     -e SPIN_VARIABLE_INFERENCE_API_URL="https://api.openai.com/v1/chat/completions" \
#     -e SPIN_VARIABLE_CHAT_MODEL="gpt-4o" \
#     -e SPIN_VARIABLE_INFERENCE_API_KEY="sk-..." \
#     few-shot-inference

FROM node:22-bookworm

# Fermyon Spin CLI (v3, matching mise.toml). The installer drops the binary in
# the working dir; move it onto PATH.
# Must be a full release tag (e.g. v3.6.3) — the installer plugs this straight
# into the download URL, so a bare "v3" 404s.
ARG SPIN_VERSION=v3.6.3
RUN apt-get update && apt-get install -y --no-install-recommends curl ca-certificates \
    && rm -rf /var/lib/apt/lists/* \
    && curl -fsSL https://developer.fermyon.com/downloads/install.sh | bash -s -- -v ${SPIN_VERSION} \
    && mv ./spin /usr/local/bin/spin

WORKDIR /app

# Install deps first so the layer caches across source-only changes.
COPY package.json package-lock.json ./
RUN npm install

# Build the component (esbuild -> j2w -> target/few-shot.wasm).
COPY . .
RUN spin build

EXPOSE 3000

# Spin reads variables from SPIN_VARIABLE_<NAME> in the environment.
CMD ["spin", "up", "--listen", "0.0.0.0:3000"]
