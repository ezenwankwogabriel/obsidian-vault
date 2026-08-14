**╭─ ⚕ Hermes ─────────────────────────────────────────────────────────────────────╮**

First step: don't touch the VPS — check the tunnel, then relaunch the desktop app. That single move fixes it most of the time, because the desktop caches a dead connection across reboots.

  

Concretely, in order:

  

1. Is the SSH tunnel up?

  

lsof -nP -iTCP:9119 -sTCP:LISTEN

  

No output → tunnel died with the reboot. Restart it:

  

ssh -N -L 9119:127.0.0.1:9119 root@188.68.34.254

  

  

2. Is the backend answering through it? This one command tells you whether the problem is server-side or client-side:

  

curl -s -o /dev/null -w "%{http_code}\n" \

  -H "X-Hermes-Session-Token: aX7qP2vL9mK3wR5tY8nB1cD4eF6gH0jU" \

  http://localhost:9119/api/status

  

- 200 → backend is fine, it's purely the desktop app. Skip to step 3.

- connection refused → tunnel or dashboard is down.

- 401 → token mismatch; the dashboard restarted without HERMES_DASHBOARD_SESSION_TOKEN.

  

3. Hard-relaunch the desktop.

  

pkill -f "Hermes.app/Contents/MacOS/Hermes"; sleep 3; ~/bin/hermes-desktop

  

Always launch via that script — it exports the URL and token env vars, which is the only reliable path since the token is stored encrypted via OS safeStorage.

  

Two things I'd carry over from today so you don't repeat my detour:

  

- Use /api/status as your probe, not /api/auth/me. I burned several steps chasing 401s from /api/auth/me and a 404 from /api/auth/ws-ticket — those routes belong to a different auth gate and don't exist in build 0.19.1. They will lie to you and make a healthy server look broken.

- If you do see a real 401 on /api/status, check for a dashboard.basic_auth block in /root/.hermes/config.yaml and remove it, then restart the dashboard with the token exported. That plugin path is defective on this build.





What's in place:

- /etc/systemd/system/hermes-dashboard.service — enabled, so it starts automatically on VPS boot

- /etc/hermes-dashboard.env — holds the session token, chmod 600, root-only (kept out of the unit file so the secret isn't world-readable via systemctl cat)

- Restart=always with a 5s delay

- Logs to /var/log/hermes-dashboard.log

  

Verified, not assumed:

- is-enabled → enabled, is-active → active

- Single process owning 127.0.0.1:9119, no orphans from the old nohup runs

- kill -9 on the main PID → systemd respawned it (56959 → 57032) and the API returned 200 again

- REST through your tunnel from the Mac → 200, desktop renderer alive

  

One thing worth noting: partway through, my pkill -f "hermes dashboard" matched its own SSH command string and killed my connection (exit 255). Harmless here, but that pattern is worth avoiding on that box — systemctl stop hermes-dashboard is the safe way now.

  

Your recovery procedure is now shorter. After a VPS reboot the dashboard comes back on its own, so next time it's really just:

  

  

lsof -nP -iTCP:9119 -sTCP:LISTEN          # tunnel alive?

ssh -N -L 9119:127.0.0.1:9119 root@188.68.34.254   # if not

pkill -f "Hermes.app/Contents/MacOS/Hermes"; sleep 3; ~/bin/hermes-desktop

  

  

The SSH tunnel on your Mac is still the one manual piece — it won't survive a local reboot. If you want, I can set that up as a LaunchAgent with autossh so it reconnects by itself, which would make the whole thing hands-off.