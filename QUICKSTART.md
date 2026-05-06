# CUSAR Bot — Quick Start

This repo contains:

- The CUSAR rebranding patch (`cusar-rebrand.patch`)
- Rebranded `.env.example`, `package.json`, `.gitignore`

The full source code is the upstream **The-Soviet-Union-bot** repo with this patch applied. The patch only changes branding strings — no runtime code paths.

## On a DigitalOcean droplet (Ubuntu 24.04)

```bash
# 1. Install Node 20 + Postgres + PM2 + git
curl -fsSL https://deb.nodesource.com/setup_20.x | sudo -E bash -
sudo apt install -y nodejs postgresql postgresql-contrib git
sudo npm install -g pm2

# 2. Create database
sudo -u postgres psql <<'EOF'
CREATE DATABASE cusarbot;
CREATE USER cusarbot WITH ENCRYPTED PASSWORD 'CHANGE_ME';
GRANT ALL PRIVILEGES ON DATABASE cusarbot TO cusarbot;
\c cusarbot
GRANT ALL ON SCHEMA public TO cusarbot;
EOF

# 3. Clone upstream and apply the CUSAR patch
git clone https://github.com/Gtamin1/The-Soviet-Union-bot.git Connor-USA-bot
cd Connor-USA-bot
curl -O https://raw.githubusercontent.com/Gtamin1/Connor-USA-bot/claude/customize-bot-to-cusar-7R8dF/cusar-rebrand.patch
git apply cusar-rebrand.patch

# 4. Install + configure
npm install
cp .env.example .env
nano .env   # fill in DISCORD_TOKEN, DISCORD_CLIENT_ID, DATABASE_URL, ENCRYPTION_KEY, API_KEY

# 5. Build + run
npm run prisma:generate
npm run prisma:deploy
npm run build
pm2 start dist/index.js --name cusar-bot
pm2 save && pm2 startup

# 6. Open API port
sudo ufw allow OpenSSH
sudo ufw allow 3000/tcp
sudo ufw enable
```

## Generate secrets

```bash
# 32-char ENCRYPTION_KEY
node -e "console.log(require('crypto').randomBytes(32).toString('base64').slice(0,32))"

# Long API_KEY for Roblox -> bot
node -e "console.log(require('crypto').randomBytes(32).toString('base64'))"
```

## First-run Discord configuration

```
/setapikey
/setadminrole @Admin
/setlogchannel #bot-logs
/setverifiedrole @Verified
/setunverifiedrole @Unverified
/setprimarygroup YOUR_ROBLOX_GROUP_ID
/setofficerrank 100
/setroblosecurity   (paste your bot account's .ROBLOSECURITY cookie)
```

See `SETUP.md` and `IMPLEMENTATION_GUIDE.md` (in upstream after `git apply`) for full details.
