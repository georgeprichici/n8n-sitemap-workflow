# Sitemap Crawler with n8n and crawl4ai

This project sets up a containerized environment for crawling website sitemaps using n8n workflows and crawl4ai, running behind an nginx proxy. The system processes sitemap XML files, crawls the URLs, and stores the results in a Supabase vector database.

## Prerequisites

- Docker and Docker Compose
- Supabase account and project
- Ollama installed locally (for embedding generation)
- SSL certificates for nginx (if using HTTPS)

## Setup

1. Clone this repository:
```bash
git clone <repository-url>
cd sitemap-crawler
```

2. Create a `.env` file with the following variables:
```env
# Supabase Configuration
SUPABASE_DB_HOST=your_supabase_db_host
SUPABASE_DB_NAME=your_db_name
SUPABASE_DB_USER=your_db_user
SUPABASE_DB_PASSWORD=your_db_password

# n8n Configuration
N8N_HOST=your_domain
N8N_PROTOCOL=http  # or https

# crawl4ai Configuration
CRAWL4AI_API_TOKEN=your_api_token
```

3. Configure nginx:
   - Place your SSL certificates in `./nginx/certs/`
   - Review and adjust `./nginx/nginx.conf` as needed

4. Start the containers:
```bash
docker-compose up -d
```

This will start:
- nginx proxy on ports 80, 443, and 5678
- n8n container with Supabase PostgreSQL connection
- crawl4ai container with fixed IP in the Docker network

## Network Configuration

The setup uses a custom Docker network (`n8n-network`) with:
- Subnet: 172.20.0.0/16
- crawl4ai fixed IP: 172.20.0.10
- Host machine access via host.docker.internal

### Platform-Specific Configuration

For Ollama connectivity:
- Mac/Windows: Uses `http://host.docker.internal:11434`
- Linux: Uses `http://172.17.0.1:11434` (requires host-gateway in extra_hosts)

## Usage

1. Ensure Ollama is running locally with the required model:
```bash
ollama pull deepseek-r1:7b
```

2. Access n8n at http://your_domain:5678 (or https://your_domain:5678)

3. Import and configure the workflow:
   - Import the provided template (`Sitemap To Supabase.json`)
   - Configure Supabase credentials
   - Configure HTTP Header Auth for crawl4ai
   - Verify Ollama connection

4. The workflow will:
   - Fetch and parse sitemap XML
   - Send URLs to crawl4ai for processing
   - Generate embeddings using Ollama
   - Store results in Supabase

## Container Management

### View logs:
```bash
docker logs n8n-workflow
docker logs crawl4ai
docker logs nginx-proxy
```

### Check container health:
```bash
docker ps --format "table {{.Names}}\t{{.Status}}\t{{.Health}}"
```

### Restart services:
```bash
docker-compose restart n8n
docker-compose restart crawl4ai
docker-compose restart nginx
```

## Troubleshooting

1. Network Issues:
   - Verify containers can reach each other: `docker exec n8n-workflow ping crawl4ai`
   - Check Ollama accessibility from n8n container
   - Verify nginx proxy configurations

2. Database Connection:
   - Confirm Supabase credentials
   - Check SSL settings for PostgreSQL connection
   - Verify database migrations and tables

3. Common Problems:
   - If Ollama connection fails, verify the correct host address for your platform
   - For SSL certificate issues, check nginx certificate paths and permissions
   - If crawl4ai is unreachable, verify the fixed IP configuration

## License

MIT
