# 107
## Lpic-107-2

>**Use systemd that runs a touch command after two minutes ?**

```bash
sudo systemd-run --on-active="2minute" touch /tmp/test.md
```

## 107.3