## SearXNG on Local

```bash
mkdir -p ./searxng/core-config/
cd ./searxng/

curl -fsSL \
    -O https://raw.githubusercontent.com/searxng/searxng/master/container/docker-compose.yml \
    -O https://raw.githubusercontent.com/searxng/searxng/master/container/.env.example

cp -i .env.example .env

# edit .env to set the host
vim .env

docker compose up -d
docker compose down

# support json format
sudo vim core-config/settings.yml
    search:
        formats:
        - html
        - json # important!

docker compose up -d
```

[searxng docker setup](https://docs.searxng.org/admin/installation-docker.html#compose-instancing)


## References

* [OpenClaw github repo](https://github.com/openclaw/openclaw)

* [huggingface unsloth/gemma-4-E2B-it-GGUF](https://huggingface.co/unsloth/gemma-4-E2B-it-GGUF)

* [huggingface unsloth/gemma-4-E2B-it-GGUF MTP](https://huggingface.co/unsloth/gemma-4-E2B-it-GGUF/blob/main/MTP/README.md)

* [SearXNG installation](https://docs.searxng.org/admin/installation-docker.html#compose-instancing)
