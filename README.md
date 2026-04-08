# BloodAuth

A production-grade authentication plugin for Spigot 1.20+ — a modern, feature-complete
alternative to AuthMe Reloaded built with Java 17 best practices.

---

## Features

- BCrypt password hashing (cost factor 12, never plaintext)
- Session persistence — returning players from same IP skip re-login
- Login timeout with configurable kick
- Temp-ban after configurable maximum failed attempts
- TOTP two-factor authentication (Google Authenticator / Authy compatible)
- Five one-time BCrypt-hashed backup codes per account
- Full player freeze on join — no movement, chat, commands, items, or damage
- Blindness effect until authenticated
- GeoIP country blocking and allowlisting via MaxMind GeoLite2
- New-country login alerts and optional kick
- VPN / proxy detection via configurable REST API
- Rich Discord webhook embeds for every security event
- SQLite (default) and MySQL with HikariCP connection pooling
- Async everywhere — database, GeoIP, Discord, VPN checks never block the main thread
- Hot-reload of config and messages with `/bloodauth reload`
- Automatic inactive account purging
- Rotating auth.log and geoip.log in `plugins/BloodAuth/logs/`

---

## Requirements

| Requirement     | Version              |
|-----------------|----------------------|
| Java            | 17 or newer          |
| Spigot / Paper  | 1.20.1 or newer      |
| Database        | SQLite (bundled) or MySQL 8+ |

---

## Installation

1. Drop `BloodAuth.jar` into your `plugins/` folder.
2. Start the server once to generate `plugins/BloodAuth/config.yml` and `messages.yml`.
3. Stop the server and edit `config.yml` to your needs.
4. *(Optional)* Download the MaxMind GeoLite2 country database — see GeoIP Setup below.
5. Start the server again.

---

## GeoIP Setup

BloodAuth uses the free MaxMind GeoLite2 Country database.

1. Register for a free account at <https://dev.maxmind.com/geoip/geolite2-free-geolocation-data>
2. Download **GeoLite2-Country.mmdb**
3. Place the file in `plugins/BloodAuth/GeoLite2-Country.mmdb`
4. Set `geoip.enabled: true` in `config.yml`
5. Run `/bloodauth reload` or restart the server

The database is updated by MaxMind monthly. Replace the `.mmdb` file and reload to update.

---

## Building from Source

```bash
git clone https://github.com/yourname/bloodauth.git
cd bloodauth
mvn clean package
```

The shaded JAR is at `target/BloodAuth.jar`.

---

## Commands

### Player Commands

| Command | Description |
|---------|-------------|
| `/register <password> <confirmPassword>` | Create a new account |
| `/login <password>` | Log in to your account |
| `/logout` | Log out manually |
| `/changepassword <old> <new>` | Change your password |
| `/2fa <code>` | Enter TOTP code during 2FA login step |

### Admin Commands (`bloodauth.admin`)

| Command | Description |
|---------|-------------|
| `/bloodauth forcelogin <player>` | Force-authenticate an online player |
| `/bloodauth forcelogout <player>` | Force-deauthenticate an online player |
| `/bloodauth unregister <player>` | Delete a player's account |
| `/bloodauth reload` | Hot-reload config.yml and messages.yml |
| `/bloodauth info <player>` | Show account details |
| `/bloodauth tempban <player> <minutes>` | Manually temp-ban a player |

### 2FA Commands

| Command | Description |
|---------|-------------|
| `/bloodauth 2fa enable` | Enable TOTP 2FA (generates QR code and backup codes) |
| `/bloodauth 2fa disable <code>` | Disable 2FA after verifying your current code |
| `/bloodauth 2fa reset <player>` | *(Admin)* Reset a player's 2FA |

---

## Permissions

| Node | Default | Description |
|------|---------|-------------|
| `bloodauth.admin` | op | All admin commands |
| `bloodauth.bypassgeo` | op | Skip GeoIP country checks |
| `bloodauth.bypass2fa` | op | Skip 2FA requirement at login |

---

## Discord Webhook Setup

1. In your Discord server: **Server Settings → Integrations → Webhooks → New Webhook**
2. Copy the webhook URL
3. Set `discord.enabled: true` and paste the URL into `discord.webhook-url` in `config.yml`

Embed colours:
- 🟢 Green — successful login, new registration
- 🔴 Red — failed login, account locked, GeoIP block, VPN detected
- 🟡 Yellow — password changed, 2FA toggled, new-country login
- 🔵 Blue — admin actions

---

## MySQL Setup

In `config.yml`:

```yaml
database:
  type: mysql
  mysql:
    host: localhost
    port: 3306
    database: bloodauth
    username: myuser
    password: mypassword
    pool-size: 10
```

Create the database beforehand:
```sql
CREATE DATABASE bloodauth CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
```

BloodAuth will create all tables automatically on first start.

---

## VPN Detection Setup

BloodAuth supports any REST API that returns a JSON response with a boolean field.

Example using [vpnapi.io](https://vpnapi.io):

```yaml
security:
  vpn-check:
    enabled: true
    api-url: "https://vpnapi.io/api/{ip}?key=YOUR_API_KEY"
    response-field: "security.vpn"
    action: kick   # or "alert" to only send a Discord notification
```

---

## Configuration Reference

All options are documented inline in `config.yml`. Key settings:

| Key | Default | Description |
|-----|---------|-------------|
| `authentication.session-timeout` | `60` | Seconds before unauthenticated player is kicked |
| `authentication.session-persistence` | `300` | Session lifetime in seconds for same-IP rejoin |
| `authentication.max-login-attempts` | `5` | Failed attempts before temp-ban |
| `authentication.temp-ban-duration` | `30` | Temp-ban length in minutes |
| `security.bcrypt-cost` | `12` | BCrypt work factor (do not set below 10) |
| `security.purge-inactive-days` | `90` | Days of inactivity before account deletion (0=off) |
| `geoip.blocked-countries` | `[]` | ISO codes that are always blocked |
| `geoip.allowed-countries` | `[]` | If non-empty, only these countries may join |

---

## License

MIT License. See `LICENSE` for details.
