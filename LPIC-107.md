# Lpic-107

## 107-2

>**Use systemd that runs a touch command after two minutes ?**

```bash
sudo systemd-run --on-active="2minute" touch /tmp/test.md
```

## 107.3

>**Display current time of system in the format 20230602-1408?**

```bash
date +'%Y%m%d-%H%M
```
