# Iteen Python Image

This repo is aimed to keep and build a base Python image for Iteen project.

## What does the image provide?

| Trait  | Version  | Details |
|--------|----------|--------|
| OS     | Bookworm |  |
| UID    | `9999`     |  |
| GID    | `9999`     |  |
| User   | `iteen`    |  |
| Root | ⛔️ |
| Sudo | ⛔️ |
| Home   | `/home/iteen/` |  |
| Packaging directory | `/opt/iteen/` | Used by this image |
| Service directory | `/srv/iteen/` | Use it to copy or mount your app |
| [Task](https://taskfile.dev) | ✅ |  |
| `wait-for` | ✅ | A script to wait for host:port available |
| Python | 3.11.8   |  |
| Pip | 24.0 | |
| [Virtualenv](https://docs.python.org/3.11/library/venv.html) | ⛔️ |
| [Poetry](https://python-poetry.org) | ⛔️ |
| [Pipenv](https://pipenv.pypa.io/en/latest/) | ⛔️ |
| [Conda](https://docs.conda.io/en/latest/) | ⛔️ |

## How to use?

```Dockerfile
FROM alexandersidorov/iteen-python:latest

RUN do your stuff
```

## How to develop?

1. Clone the repo
2. Run `docker compose build`
3. Run `docker compose up`

## How to deploy?

`task deploy`

## How to update packages?

1. set the target=system
2. build image
3. run container: `docker compose run --rm -it --entrypoint /bin/bash python`
4. navigate to `/opt/iteen`
5. run `poetry lock`
6. copy new lock file to the host: `cp poetry.lock /mnt/shared/`
7. exit container
8. replace lock file: `mv ./.local/shared/poetry.lock ./src/dependencies/`
9. clear target / set to final
10. rebuild image
