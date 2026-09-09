# Hardware

Intel NUC, i3 1220p

- 1x Sandisk SD8SB8U256G1122
- 1x i226 NIC

Intel NUC slim, i5 1240p

- 1x TS256GMTS430S
- 1x Samsung SSD 970 EVO Plus 1TB

Intel NUC slim, i5 1240p

- 1x TS256GMTS430S
- 1x Samsung SSD 970 EVO Plus 1TB

Intel NUC slim, i5 1240p

- 1x TS256GMTS430S
- 1x WD Black SN850x 2TB

Synology 218play

- 2x Seagate ironwolf pro 22TB

MX6200 Routers

Mokerlink 2.5GbE 8 port unmanaged switch

6 port PDU

Anker Power Supply

# Maintainence

All available revisions stored on the NAS & Object storage.

## BIOS/UEFI **DO NOT UPDATE**

Prefer corebooting devices and running tianocore.

Intel NUCS

- Updating to Asus based firmwares will lock them down according to readme.

Synology

- KISS

## Firmware

Intel NUCS

- On-board 2.5GbE I225-V NICS.
    - The NIC's are currently flashed with the firmware 1.94 of the I225-LM with
      specific bits edited. The eetrackID is #? Flashing is done with the intel
      official nvmupdate64 tool.
