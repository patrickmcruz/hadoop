# Hadoop Docker Compose (learning environment)

[![Last Commit](https://img.shields.io/github/last-commit/patrickmcruz/hadoop)](https://github.com/patrickmcruz/hadoop) [![License](https://img.shields.io/github/license/patrickmcruz/hadoop)](https://github.com/patrickmcruz/hadoop/blob/main/LICENSE) [![Docker Pulls](https://img.shields.io/docker/pulls/apache/hadoop)](https://hub.docker.com/r/apache/hadoop) [![Note](https://img.shields.io/badge/environment-learning-blue)](https://github.com/patrickmcruz/hadoop)

Short, hands-on Hadoop single-node cluster using official `apache/hadoop:3` images and Docker Compose. This repository is aimed at data-science students and Level-1 data-engineering professionals who want a reproducible playground to learn HDFS and YARN.

## Index

- [Overview](#overview)
- [Prerequisites](#prerequisites)
- [Installation](#installation)
- [Quick Start (PowerShell)](#quick-start-powershell)
- [Files and Layout](#files-and-layout)
- [Ports and Web UIs](#ports-and-web-uis)
- [Sample Data](#sample-data)
- [Troubleshooting](#troubleshooting)
- [Next Steps for Learning](#next-steps-for-learning)

## Overview

This project boots a minimal Hadoop environment using Docker Compose with four services: `namenode`, `datanode`, `resourcemanager`, and `nodemanager`. It's configured for local learning and small experiments, not production.

## Prerequisites

- Docker Desktop (Windows) with WSL2 or Hyper-V enabled
- Docker Compose v1/v2-compatible CLI
- PowerShell (examples below use PowerShell)

## Installation

Clone the project and start the learning environment:


```powershell
# clone repository
git clone https://github.com/patrickmcruz/hadoop.git
Set-Location -LiteralPath .\hadoop

# stop any previous compose project and start the stack (detached)
docker-compose down
docker-compose up -d
```
docker exec -it hadoop-namenode-1 bash -c "hdfs dfs -ls /"
If you want a specific project name or to run from a different folder, use `docker-compose -p hadoop up -d`.

## Quick Start (PowerShell)

Open PowerShell in the project folder (`.../hadoop`) and run:

```powershell
# Stop any existing stack, then start detached
docker-compose down
docker-compose up -d
```

To tail logs:

```powershell
docker-compose logs -f
```

To bring everything down:

```powershell
docker-compose down
```

If you prefer an explicit project name (the repo includes `name: hadoop`), you can run:

```powershell
docker-compose -p hadoop up -d
```

## Files and Layout

- `docker-compose.yml` — Compose config that launches the four containers.
- `conf/core-site.xml`, `conf/yarn-site.xml` — Minimal Hadoop configs mounted into containers.
- `config` — Environment file used by containers.
- `sample_data/` — Example datasets (e.g., `california_housing_test.csv`) for experiments.

Wrap mounts and config edits carefully when modifying container behavior.

## Ports and Web UIs

- NameNode UI: `http://localhost:9870`
- YARN ResourceManager UI: `http://localhost:8088` (host port)

If `8088` is already in use on your machine, you will see a bind error when starting the stack. See Troubleshooting.

## Sample Data

Drop CSVs or small datasets into `sample_data/` and copy them into HDFS with a container command, for example:

```powershell
# Copy local file into HDFS (runs inside the namenode container)
docker exec -it hadoop-namenode-1 bash -c "hdfs dfs -mkdir -p /user/hduser/data && hdfs dfs -put /opt/hadoop/sample_data/california_housing_test.csv /user/hduser/data/"
```

Adjust the container name if Compose created a different one; use `docker ps` to confirm.

### Example: create the directory `/seu-nome` in HDFS

Here are two examples a student can follow — (A) execute the command directly on the host using `docker exec`, or (B) open an interactive shell inside the container and type the commands manually.

- A) Execute from host (PowerShell):

```powershell
# Create the directory /seu-nome in HDFS executing the command line directly on the host machine NameNode
docker exec -it hadoop-namenode-1 bash -c "hdfs dfs -mkdir -p /seu-nome"

# List the contents of the root directory to verify
docker exec -it hadoop-namenode-1 bash -c "hdfs dfs -ls /"
```
- B) Enter in container and execute manually (interactive flow):

```bash
# open a shell inside the container NameNode
docker exec -it hadoop-namenode-1 bash

# inside in the container, execute: 
hdfs dfs -mkdir -p /seu-nome
hdfs dfs -ls /

# exit the container
exit
```

These examples assume that the NameNode container was created with the name `hadoop-namenode-1` by Compose. Use `docker ps` to confirm the name if it is different.

## Troubleshooting

- Port 8088 already allocated

  If you get an error like `Bind for 0.0.0.0:8088 failed: port is already allocated`, either stop the process using that port or change the host mapping in `docker-compose.yml` for the `resourcemanager` service. Example change:

  ```yaml
  resourcemanager:
    ports:
      - "8089:8088"
  ```

  Then run:

  ```powershell
    docker-compose down
    docker-compose up -d
  ```

- Container names and compose project

  The compose file now includes `name: hadoop` at the top level; containers will be created with names like `hadoop-namenode-1`. If you omit the `name:` entry, Compose derives a project name from the folder.

## Next Steps for Learning

- Practice: put CSVs into HDFS and run simple MapReduce or YARN jobs.
- Explore: open the NameNode and ResourceManager UIs to inspect files and applications.
- Extend: mount your code and run PySpark jobs against this cluster.

## Contributing & Notes

This repository is intended as a teaching sandbox. If you add configs or experiments, please document them in this README.

## License (GNU GPL v3)

This project is released under the GNU General Public License v3.0 (GPL-3.0). You are free to run, study, share and modify the project under the terms of the GPLv3. The full license text is included in the `LICENSE` file; a summary is provided below.

> GNU General Public License v3.0 — permission is granted to copy, distribute and/or modify this software under the terms of the GPLv3.

## Credits

- **Patrick Motin Cruz** — ML / AI Fullstack Developer

Thank you to the Apache Hadoop project and other open-source maintainers whose images and documentation make this learning environment possible.
