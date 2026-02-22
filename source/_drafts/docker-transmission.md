version: '3.6'

services:
  torrent:
    image: linuxserver/transmission:3.00-r0-ls63
    container_name: torrent
    environment:
      - PUID=1000
      - PGID=1000
      - TZ=Asia/Seoul
      - USER=hyunsub
      - PASS=hyunsub1234
    volumes:
      - /archive/torrent/downloads:/downloads
      - /archive/torrent/config:/config
    ports:
      - "8082:9091"
    restart: always
