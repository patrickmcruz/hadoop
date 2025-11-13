# Hadoop Docker Compose (ambiente de aprendizado)

[![Last Commit](https://img.shields.io/github/last-commit/patrickmcruz/hadoop)](https://github.com/patrickmcruz/hadoop) [![License](https://img.shields.io/github/license/patrickmcruz/hadoop)](https://github.com/patrickmcruz/hadoop/blob/main/LICENSE) [![Docker Pulls](https://img.shields.io/docker/pulls/apache/hadoop)](https://hub.docker.com/r/apache/hadoop) [![Note](https://img.shields.io/badge/environment-learning-blue)](https://github.com/patrickmcruz/hadoop)

Ambiente Hadoop single-node para aprendizado, usando as imagens oficiais `apache/hadoop:3` e Docker Compose. Este repositório é voltado para estudantes de ciência de dados e profissionais de engenharia de dados nível 1 que querem um playground reprodutível para aprender HDFS e YARN.

## Índice

- [Visão Geral](#visão-geral)
- [Pré-requisitos](#pré-requisitos)
- [Instalação](#instalação)
- [Início Rápido (PowerShell)](#início-rápido-powershell)
- [Arquivos e Estrutura](#arquivos-e-estrutura)
- [Portas e UIs Web](#portas-e-uis-web)
- [Dados de Exemplo](#dados-de-exemplo)
- [Solução de Problemas](#solução-de-problemas)
- [Próximos Passos de Aprendizado](#próximos-passos-de-aprendizado)

## Visão Geral

Este projeto inicializa um ambiente mínimo Hadoop usando Docker Compose com quatro serviços: `namenode`, `datanode`, `resourcemanager` e `nodemanager`. Está configurado para aprendizado local e pequenos experimentos — não para produção.

## Pré-requisitos

- Docker Desktop (Windows) com WSL2 ou Hyper-V habilitado
- CLI compatível com Docker Compose (v1/v2)
- PowerShell (os exemplos abaixo usam PowerShell)

## Instalação

Clone o projeto e inicie o ambiente de aprendizado:

```powershell
# clonar repositório
git clone https://github.com/patrickmcruz/hadoop.git
Set-Location -LiteralPath .\hadoop

# parar eventual compose anterior e iniciar a stack (detached)
docker-compose down
docker-compose up -d
```

Se preferir um nome de projeto específico ou iniciar a partir de outra pasta, use `docker-compose -p hadoop up -d`.

## Início Rápido (PowerShell)

Abra o PowerShell na pasta do projeto (`.../hadoop`) e execute:

```powershell
# Parar qualquer stack existente e iniciar em segundo plano
docker-compose down
docker-compose up -d
```

Para acompanhar logs:

```powershell
docker-compose logs -f
```

Para encerrar tudo:

```powershell
docker-compose down
```

Se preferir um nome de projeto explícito (o arquivo Compose já contém `name: hadoop`), você pode executar:

```powershell
docker-compose -p hadoop up -d
```

## Arquivos e Estrutura

- `docker-compose.yml` — configuração do Compose que inicia os quatro contêineres.
- `conf/core-site.xml`, `conf/yarn-site.xml` — configurações mínimas do Hadoop montadas nos contêineres.
- `config` — arquivo de ambiente usado pelos contêineres.
- `sample_data/` — datasets de exemplo (ex.: `california_housing_test.csv`) para experimentos.

Ao modificar montagens ou configurações, tenha cuidado para não quebrar caminhos esperados dentro dos contêineres.

## Portas e UIs Web

- NameNode UI: `http://localhost:9870`
- YARN ResourceManager UI: `http://localhost:8088` (porta do host)

Se a porta `8088` já estiver em uso na sua máquina, você verá um erro de bind ao iniciar a stack. Veja a seção de solução de problemas.

## Dados de Exemplo

Coloque arquivos CSVs ou pequenos datasets em `sample_data/` e copie-os para o HDFS com um comando no contêiner, por exemplo:

```powershell
# Copia um arquivo local para o HDFS (executa dentro do contêiner NameNode)
docker exec -it hadoop-namenode-1 bash -c "hdfs dfs -mkdir -p /user/hduser/data && hdfs dfs -put /opt/hadoop/sample_data/california_housing_test.csv /user/hduser/data/"
```

Ajuste o nome do contêiner caso o Compose tenha gerado outro; use `docker ps` para confirmar.

### Exemplo: criar o diretório `/seu-nome` no HDFS

A seguir há dois exemplos que um aluno pode seguir — (A) executar o comando diretamente no host usando `docker exec`, ou (B) abrir um shell interativo dentro do contêiner e digitar os comandos manualmente.

- A) Executando do host (PowerShell):

```powershell
# Cria o diretório /seu-nome no HDFS executando o comando dentro do contêiner NameNode
docker exec -it hadoop-namenode-1 bash -c "hdfs dfs -mkdir -p /seu-nome"

# Lista o conteúdo da raiz para verificar
docker exec -it hadoop-namenode-1 bash -c "hdfs dfs -ls /"
```

- B) Entrando no contêiner e executando manualmente (fluxo interativo):

```bash
# Abre um shell dentro do contêiner NameNode
docker exec -it hadoop-namenode-1 bash

# Dentro do contêiner (em um shell bash), execute:
hdfs dfs -mkdir -p /seu-nome
hdfs dfs -ls /

# Sai do contêiner
exit
```

Esses exemplos assumem que o contêiner NameNode foi criado com o nome `hadoop-namenode-1` pelo Compose. Use `docker ps` para confirmar o nome, caso seja diferente.

## Solução de Problemas

- Porta 8088 já alocada

  Se você receber um erro como `Bind for 0.0.0.0:8088 failed: port is already allocated`, pare o processo que está usando essa porta ou altere o mapeamento no `docker-compose.yml` para o `resourcemanager`. Exemplo de alteração:

  ```yaml
  resourcemanager:
    ports:
      - "8089:8088"
  ```

  Em seguida rode:

  ```powershell
    docker-compose down
    docker-compose up -d
  ```

- Nomes de contêiner e projeto Compose

  O arquivo Compose agora inclui `name: hadoop` no nível superior; assim os contêineres serão criados com nomes como `hadoop-namenode-1`. Se você remover `name:`, o Compose deriva o nome do projeto a partir da pasta.

## Próximos Passos de Aprendizado

- Prática: coloque CSVs no HDFS e execute MapReduce simples ou jobs YARN.
- Explore: abra as UIs do NameNode e ResourceManager para inspecionar arquivos e aplicações.
- Estenda: monte seu código e execute jobs PySpark contra este cluster.

## Contribuição & Notas

Este repositório é um sandbox de ensino. Se você adicionar configurações ou experimentos, documente-os neste README.

## Licença (GNU GPL v3)

Este projeto é distribuído sob a licença GNU General Public License v3.0 (GPL-3.0). Você tem liberdade para executar, estudar, compartilhar e modificar o projeto sob os termos da GPLv3. O texto completo da licença está incluído no arquivo `LICENSE`.

> GNU General Public License v3.0 — permissão para copiar, distribuir e/ou modificar este software sob os termos da GPLv3.

## Créditos

- **Patrick Motin Cruz** — ML / AI Fullstack Developer

Obrigado ao projeto Apache Hadoop e aos mantenedores open-source cujas imagens e documentação tornam este ambiente de aprendizado possível.
