
****Hardware:****

  

- ****Control Plane / Workstation**** — MacBook Air M4 (16GB) running macOS

- ****Worker Node**** — AMD Ryzen 5 1600x, Asrock AB350 Pro4, 32GB RAM, Fractal Design Define R4, Noctua cooler — currently on the floor with side panel off, ready for OS install

- ****Storage**** — 2TB spinner (Tunes drive) sitting on network shelf, ready to mount in AMD tower

  

****Network:****

  

- TP-Link Omada ecosystem — OC200 hardware controller, ER605 router, TL-SG1210P PoE switch, 2x EAP245 access points

- Recommend static DHCP reservation for worker node by MAC address in Omada controller

  

****OS:****

  

- Ubuntu Server 24.04 LTS for worker node

- OrbStack on MacBook for local container/VM management

  

****Next Steps:****

  

- Install Ubuntu Server 24.04 on AMD tower

- Assign static DHCP reservation in Omada

- Configure SSH access from MacBook

- Mount 2TB drive as data volume

- Install k3s to bootstrap cluster

  

****First Project — Audio Server:****

  

- ****Navidrome**** — Subsonic API compatible music server, lightweight, excellent Kubernetes support

- ****Source library**** — ~1.3TB mix of FLAC and 320 VBR MP3s

- ****Desktop client**** — Strawberry Music Player (Linux), explore MacOS build

- ****Mobile client**** — Symfonium on Razr 2023, potentially also on retired LG V40 over WiFi