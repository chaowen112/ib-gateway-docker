---
name: ib-gateway-deployer
description: Deploy, configure, and troubleshoot the Interactive Brokers Gateway using the ib-gateway-docker repository. Use this skill when the user wants to set up a trading environment (live or paper), connect an API client, or manage the IB Gateway container.
---

# IB Gateway Deployer Skill

This skill provides specialized procedures for deploying and managing the [Interactive Brokers Gateway Docker](https://github.com/gnzsnz/ib-gateway-docker) repository.

## Deployment Workflow

### 1. Research & Strategy
Before deploying, verify the user's intent:
-   **Trading Mode**: `paper` (default) or `live`.
-   **Service Type**: `ib-gateway` (headless API) or `tws-rdesktop` (GUI-based TWS).
-   **Credentials**: Ensure the user has their IB `TWS_USERID` and `TWS_PASSWORD` ready (do NOT store these in code, use `.env` or secrets).

### 2. Implementation (Plan -> Act -> Validate)

#### Plan
-   Determine which `docker-compose` file to use (`docker-compose.yml` for gateway, `tws-docker-compose.yml` for TWS).
-   Identify the necessary environment variables for the chosen mode.

#### Act
-   **Setup .env**: Copy `.env-dist` to `.env`.
-   **Configure**: Update `.env` with the following key variables:
    -   `TWS_USERID` / `TWS_PASSWORD`
    -   `TRADING_MODE` (`live`, `paper`, or `both`)
    -   `VNC_SERVER_PASSWORD` (if VNC access is needed)
-   **Build & Run**:
    ```bash
    docker compose up -d
    ```

#### Validate
-   Check container status: `docker compose ps`
-   Verify logs for successful login: `docker compose logs -f`
-   Test API connectivity: Use `socat` or a simple script to ping the API ports (4001 for live, 4002 for paper).

## Key Configuration Reference

| Mode | Internal Port | Host Port (Default) |
| --- | --- | --- |
| Live Gateway | 4003 (mapped to 4001 via socat) | 4001 |
| Paper Gateway | 4004 (mapped to 4002 via socat) | 4002 |
| Live TWS | 7498 (mapped to 7496 via socat) | 7496 |
| Paper TWS | 7499 (mapped to 7497 via socat) | 7497 |

## Troubleshooting
-   **Login Failures**: Check `docker compose logs` for IBC error messages.
-   **Connectivity**: If `localhost` doesn't work, verify the `socat` process is running inside the container (`ps aux | grep socat`).
-   **2FA**: If the account has 2FA, the gateway may require manual intervention via VNC.

## Best Practices
-   **Security**: Never commit the `.env` file with real credentials.
-   **Persistence**: Use Docker volumes for `jts.ini` and settings if configuration must persist across restarts.
-   **Updates**: Use the `latest` or `stable` tags as appropriate for the environment.
