# LANCache Prefill

[LANCache Prefill](https://github.com/Bastika07/docker-lancache-prefill) downloads and installs [BattleNetPrefill](https://github.com/tpill90/battlenet-lancache-prefill), [EpicPrefill](https://github.com/tpill90/epic-lancache-prefill) and [SteamPrefill](https://github.com/tpill90/steam-lancache-prefill), and runs them on a nightly cron schedule to keep your LanCache instance warm.

After installation each enabled prefill tool has to be configured once (login and app selection). Open a shell on the TrueNAS host and run:

```
docker exec -u prefill -it ix-lancache-prefill-lancache-prefill-1 bash
cd /lancacheprefill/SteamPrefill
./SteamPrefill select-apps
```

Repeat with `EpicPrefill`/`BattleNetPrefill` if enabled, then restart the app.

Make sure the container resolves DNS through your LanCache DNS (DNS Servers option in the network configuration), otherwise the prefill downloads will bypass the cache.
