# LibreBooking Docker

This repository is a simple way to run [LibreBooking](https://github.com/librebooking/app) in Docker.  
Basically, it’s a **wrapper** that puts LibreBooking into a container with Apache/PHP and works with MariaDB.

> ⚠️ The repo **does not include LibreBooking itself** — you need to fetch it as a submodule.

---

## Repository Structure

```

.
├── docker/           # Dockerfile and scripts
├── src/              # LibreBooking (submodule)
├── docker-compose.yml
├── bin/              # helper scripts
├── README.md
└── LICENSE

````

- `docker/Dockerfile` — builds the container with PHP, Apache, LibreBooking, and all dependencies.  
- `docker/bin` — startup scripts and cron jobs.  
- `src` — the LibreBooking submodule.  
- `docker-compose.yml` — example setup with MariaDB.

---

## Installation and Run

1. Clone the repository **with submodules**:

```bash
git clone --recurse-submodules git@github.com:santa-bremor/librebooking-docker.git
cd librebooking-docker
````

If you cloned without submodules:

```bash
git submodule update --init --recursive
```

2. Build the Docker image:

```bash
docker build -t librebooking docker/
```

Or use `docker-compose`:

```bash
docker-compose up -d --build
```

3. LibreBooking will be available at `http://localhost:8080` (default, can be changed in `docker-compose.yml`).

---

## Updating LibreBooking

To pull the latest changes from the original repository:

```bash
cd src
git fetch origin
git checkout develop      # or the branch you need
git pull
cd ..
docker-compose build     # rebuild the container with the updated version
```

---

## Configuration

* Configuration is stored in `VOLUME /config`, including cron jobs and logs.
* PHP/Apache are preconfigured to run LibreBooking out-of-the-box.
* Installed PHP extensions: `gd`, `mysqli`, `ldap`, `timezonedb`.

---

## Useful Commands

* View Apache logs:

```bash
docker logs -f <container_id>
```

* Access the container shell:

```bash
docker exec -it <container_id> bash
```

---

## License

GPL-3.0 (same as LibreBooking)
