# OPOL-API Deployment

This repository contains only the necessary files to deploy the OPOL-API service using Docker.

## Prerequisites

- Docker
- Docker Compose

## Quick Start

1. **Clone this repository** (or copy these files) to your server.
2. **Setup environment variables**:
   ```bash
   cp .env.example .env
   ```
3. **Configure the `.env` file**: 
   Open `.env` and fill in your database credentials and other settings.
4. **Deploy the service**:
   ```bash
   docker-compose up -d
   ```

## Registry Details

- **Image**: `ghcr.io/duanganucha/opol-api:latest`
- **Owner**: duanganucha

The service will automatically pull the latest image from the GitHub Container Registry.
