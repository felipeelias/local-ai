FROM denoland/deno:2.1.9

WORKDIR /app

COPY deno.lock .
COPY deno.json .
RUN deno install

COPY . .
RUN deno cache main.ts

CMD ["run", "--allow-read", "--allow-env", "--allow-sys", "--allow-net", "main.ts", "--endpoint", "http://ollama:11434", "--hostname", "0.0.0.0", "--port", "62192", "--cloudflared", "false"]
