<!--
Multi-Monitor-Shortcuts:
Ctrl-O: Move Window to next screen
Mod4 + Control + j/k: Focus next/previous screen

reveal.js-Shortcuts:
o: Öffne Übersicht
s: Öffne Vortragsmonitor
-->

# Systemd
## System- und Servicemanager

**Tmux:** ssh tmux@turingmachine.ghcq.ml

**Live-Präsentation:** http://turingmachine.ghcq.ml/systemd-ta/

**Code:** http://github.com/Mic92/systemd-ta

Terminalmitschnitt folgt

Note:
- Thema: Systemd - System- und Servicemanager für Linux
- Nur lesbare Tmuxsession per SSH
- Codeschnipsel auf Github
- Kopierbarer Terminalmitschnitt später verlinkt


## Systemd - Ablauf
- kurze Einführung/Konzepte
- Praxisteil

Note:
- Bei Fragen: fragen
- Fragen:
  - Wer benutzt systemd irgendwo?
  - Wer hat Unit Files geschrieben


## Systemd - PID 1

<img src="img/boot.png" alt="Boot process">

Note:

- 1. Prozess nach dem Kernel gestartet -(init= Kernel-Parameter, /bin/init, /bin/bash)
- initialisiert System, startet Nicht-Betriebsystem-Dienste, Dateisysteme (/,
  /tmp, swap), Netzwerk, Zeit, Hostname, ...


## Systemd - Konzepte

- Units
- Abhängigkeiten
- Überwachung von Prozessen
- Beenden von Diensten
- minimale Bootzeit
- Debugbarkeit

Note:
- organisiert in Units anstatt nur Deamons/Dieste
- Abhängigkeiten zwischen Units (zirkuläre Abhängigkeiten werden aufgebrochen)
- Statt Start/Stop -> Überwachung von Prozessen
  -> Cgroups (cd /sys/fs/cgroup/systemd/)
  -> system.slice/user.slice/..., tasks
  -> faire Ressourcenverteilung (apache vs. mysql)
  -> systemd-cgls
- Systemd ordnet alle Prozesse zu ihren Diensten hinzu
  (Vorteil beim Beenden)
  $ ps xawf -eo pid,user,cgroup,args | less
- Start parallelisiert unter Berücksichtung von Abhängigkeiten (Desktop schnell,
  Server -> Container)
- Nachrichten beim Boot gehen nicht verloren, Ausgaben aller Programme in
  journald


## Systemd - Units

|                     |                   |
| --------------------| ------------------|
| **service.service** | **target.target** |
| **socket.socket**   | **path.path**     |
| **device.device**   | **timer.timer**   |
| **mount.mount**     | snapshot.snapshot |
| automount.automount | **slice.slice**   |
| swap.swap           | scope.scope       |

Note:
- Jede Unit eine Datei
- Endung -> Typ
- service: Daemons
- target:
  - vergleichbar runlevel, aber mehrere
  - zum Gruppieren von units
  - multi-user.target.wants, sleep.target.wants
- socket: Socket Activation
- device: Gerät, das von udev mit TAG+="systemd" getaggt wurde -> Service
  starten, wenn bestimmte Geräte angeschlossen werden
- mount: Systemd Generators -> parst fstab -> mount.targets -> sinnvoll als Abhängigkeit
- timer: Zeitevents -> Cron
- path: Änderungen im Dateisystem
- slice: Gruppieren von Diensten zur Resourcenverwaltung


## Systemd - Praxisteil

```bash
archlinux> systemctl --version
systemd 216
```

```bash
debian> systemctl --version
systemd 215
```

- Code: http://github.com/Mic92/systemd-ta

Note:
- falls gezeigte Beispiele nicht funktionieren -> systemd-Version überprüfen
- System-Units:
  - /usr/lib/systemd/system
  - /etc/systemd/system <-- Höhere Priorität
- pacman -Ql systemd | grep /usr/share/man | wc -l

<!--
[Service]
# keine Shell! -> Volle Pfade, Redirects oder Pipe werden NICHT unterstützt
ExecStart=/usr/bin/socat TCP-LISTEN:8888 'SYSTEM:echo Hello World'

host> systemctl start socat.service
host> systemctl status socat.service
● socat.service
   Loaded: loaded (/etc/systemd/system/socat.service; static) <- Pfad
   Active: active (running) since Sun 2014-10-26 10:40:41 CET; 1min 46s ago <- Startzeit, Laufzeit
 Main PID: 20362 (socat) <- Vaterprozess
   CGroup: /system.slice/socat.service
           └─20362 /usr/bin/socat TCP-LISTEN:8888 SYSTEM:echo Hello World <- Prozess
)
sudo ss -tlnp | grep -C3 8888
nc localhost 8888
host> systemctl status socat.service
-->

<!--
[Service]
ExecStart=/usr/bin/socat TCP-LISTEN:8888,reuseaddr 'SYSTEM:echo Hello World'
Restart=on-success # or always
-->

<!--
[Unit]
Description=Socat Greeting Service
Documentation=man:socat(1)

[Service]
ExecStart=/usr/bin/socat TCP-LISTEN:8888,reuseaddr 'SYSTEM:echo Hello World'
Restart=on-success # or always

[Install]
WantedBy=multi-user.target

host> systemctl enable socat
host> ls -la /etc/systemd/system/multi-user.target.wants/socat.service
host> systemctl status socat
host> sudo systemadm
-->

<!--
host> pacstrap -d arch base vim git htop dnsutils
host> tree ~/arch
host> systemd-nspawn -D ~/arch
host> systemd-nspawn -D ~/arch -b
-->

<!--
host> debootstrap --include=vim,locales,htop,git,curl,dnsutils testing ~/debian
host> tree ~/debian
host> systemd-nspawn -D ~/debian
$ passwd
host> systemd-nspawn -D ~/debian -b
$ dpkg-reconfigure locales
$ locale-gen en_US.UTF-8
$ apt-get install dbus  # logind
$ # restart
$ systemd-nspawn -D debian --network-veth
$ systemd-nspawn -D debian --private-network
$ ip a
host> ip a
-->