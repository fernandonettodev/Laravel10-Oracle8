# Configuração do Projeto Laravel com DB Oracle, Sail, Redis e PostgreSQL

Este guia irá ajudá-lo a configurar seu projeto Laravel 10 para usar o banco de dados Oracle com PHP 8.1, Sail, Redis e PostgreSQL.

## Pré-requisitos

- [Composer](https://getcomposer.org/) instalado
- [Docker](https://www.docker.com/) e [Docker Compose](https://docs.docker.com/compose/) instalados

## Passos para Configuração

### 1. Baixar o Projeto

Primeiro, baixe o seu projeto e acesse a pasta do projeto:

```bash
git clone <URL_DO_SEU_REPOSITORIO>
cd <NOME_DO_SEU_PROJETO>
```

### 2. Instalar o Sail

Instale o Sail usando o Composer:

```bash
composer require laravel/sail --dev
```

### 3. Baixar a Pasta do Docker que tem o Instant Client

Baixe a pasta docker ([Aqui](https://drive.google.com/file/d/1iVGDYebnmYxK9If1GL9lKdiUzFm9HI4-/view?usp=sharing)) e coloque na raiz do projeto. A estrutura deve ficar assim:

```
projeto/
├── docker/
│   └── instantclient_21_7/
└── ...
```

(Fique a vontade para usar a versão que melhor quiser do instant client, estou usando a 21_7 pois ser a mais estavel pra configurar)

### 4. Criar o `docker-compose.yml`

Crie um arquivo `docker-compose.yml` na raiz do projeto com o seguinte conteúdo:

```yaml
services:
    laravel.test:
        build:
            context: ./docker
            dockerfile: Dockerfile
            args:
                WWWGROUP: '${WWWGROUP}'
        image: sail-8.1/app
        extra_hosts:
            - 'host.docker.internal:host-gateway'
        ports:
            - '${APP_PORT:-80}:80'
            - '${VITE_PORT:-5173}:${VITE_PORT:-5173}'
        environment:
            WWWUSER: '${WWWUSER}'
            LARAVEL_SAIL: 1
            XDEBUG_MODE: '${SAIL_XDEBUG_MODE:-off}'
            XDEBUG_CONFIG: '${SAIL_XDEBUG_CONFIG:-client_host=host.docker.internal}'
        volumes:
            - '.:/var/www/html'
        networks:
            - sail
        depends_on:
            - pgsql
            - redis

    pgsql:
        image: 'postgres:${POSTGRES_VERSION:-15}'
        ports:
            - '${FORWARD_DB_PORT:-5432}:5432'
        environment:
            POSTGRES_DB: '${DB_DATABASE}'
            POSTGRES_USER: '${DB_USERNAME}'
            POSTGRES_PASSWORD: '${DB_PASSWORD}'
        volumes:
            - 'sail-pgsql:/var/lib/postgresql/data'
        networks:
            - sail
        healthcheck:
            test: ["CMD-SHELL", "pg_isready -U ${DB_USERNAME}"]
            retries: 3
            timeout: 5s

    redis:
        image: 'redis:alpine'
        ports:
            - '${FORWARD_REDIS_PORT:-6379}:6379'
        volumes:
            - 'sail-redis:/data'
        networks:
            - sail
        healthcheck:
            test: ["CMD", "redis-cli", "ping"]
            retries: 3
            timeout: 5s

networks:
    sail:
        driver: bridge

volumes:
    sail-pgsql:
        driver: local
    sail-redis:
        driver: local
```

Sinta-se à vontade para adicionar outros serviços que você precise, como Memcached, Selenium, etc.

### 5. Configurar o Acesso ao Banco Oracle

Coloque os dados de acesso do seu Banco Oracle na pasta `docker/instantclient_21_7/network/admin`.

### 6. Subir os Containers

Na raiz do projeto, execute o comando:

```bash
./vendor/bin/sail up -d
```

Espere o processo de build ser concluído.

### 7. Instale os pacotes do Yajra pra identificar o banco oracle:
```bash
composer require yajra/laravel-datatables-oracle:"^11"
```
e
```bash
composer require yajra/laravel-oci8": "^10.0
```

após isso adicione no config/app.php:

providers:
```bash
Yajra\DataTables\DataTablesServiceProvider::class,
Yajra\Oci8\Oci8ServiceProvider::class,
```

aliases: 
```bash
'DataTables' => Yajra\DataTables\Facades\DataTables::class,
```

### 8. Usar o Sail

Agora, para executar comandos Artisan e outros comandos do Laravel, você deve preceder os comandos com `./vendor/bin/sail`. Por exemplo, para rodar as migrações, use:

```bash
./vendor/bin/sail artisan migrate
```

### 8. Criar um Alias para o Sail (Opcional)

Para facilitar o uso, você pode criar um alias para o Sail. Abra o seu terminal e adicione a seguinte linha ao seu arquivo de configuração do shell (como `~/.bashrc` ou `~/.zshrc`):

```bash
alias sail='[ -f sail ] && ./vendor/bin/sail || die "Sail not found"'
```

Depois, execute:

```bash
source ~/.bashrc   # ou source ~/.zshrc
```

Agora você pode usar o comando `sail` diretamente.

---

Com estas instruções, você deve estar pronto para usar o banco de dados Oracle com o Laravel 10 em um ambiente Dockerizado. Aproveite!
