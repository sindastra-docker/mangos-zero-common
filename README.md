# mangos-zero-common
MaNGOS Zero realmd and mangosd common build for Docker

# Setting things up

Run these two commands once to generate the volumes and place the config files

    docker run -tid --restart always --name mangos0-realmd --mount type=volume,src=mangos-zero-config,dst=/mangos/zero/etc -p3724:3724 sindastra/mangos-zero-realmd:latest
    docker run -tid --restart always --name mangos0-mangosd --mount type=volume,src=mangos-zero-config,dst=/mangos/zero/etc --mount type=volume,src=mangos-zero-data,dst=/mangos/zero/data -p8085:8085 sindastra/mangos-zero-mangosd:latest

Now stop them to edit the config files and to generate and install the data

    docker stop mangos0-realmd
    docker stop mangos0-mangosd

# Configuring the server

Now edit the .conf files at ```/var/lib/docker/volumes/mangos-zero-config/_data/``` (or wherever your docker volumes are stored).

The important files are realmd.conf and mangosd.conf where in both cases you have to specify your MySQL/MariaDB credentials and database info.

Since this docker image is designed to run the database on the host, you'll have to specify 172.17.0.1 as MySQL/MariaDB address. Make sure your MySQL/MariaDB server is listening to that address. You can bind the MySQL/MariaDB server to 0.0.0.0 but make sure MySQL/MariaDB is firewalled so it cannot be accessed from outside! And make sure 172.17.0.0/16 can access 172.17.0.1 in your firewall.

Make sure that in mangosd.conf DataDir = "/mangos/zero/data"

You will have to obtain a database elsewhere and set it up yourself. For the database server, MariaDB is recommended.

# Extracting and generating data

Note that you need to extract these in the following order.

Assuming your client is stored at ```/Applications/World of Warcraft/1.12.1```:

### Extracting maps and dbc

    docker run -ti --rm --mount type=bind,src="/Applications/World of Warcraft/1.12.1",dst=/mangos/zero/client -w /mangos/zero/client sindastra/mangos-zero-common /mangos/zero/bin/tools/map-extractor

### Extracting buildings and generating vmaps

    docker run -ti --rm --mount type=bind,src="/Applications/World of Warcraft/1.12.1",dst=/mangos/zero/client -w /mangos/zero/client sindastra/mangos-zero-common /mangos/zero/bin/tools/vmap-extractor

You can delete the "Buildings" directory after it's done.

### Building movemaps (mmaps)

    docker run -ti --rm --mount type=bind,src="/Applications/World of Warcraft/1.12.1",dst=/mangos/zero/client -w /mangos/zero/client sindastra/mangos-zero-common /mangos/zero/bin/tools/movemap-generator

# Installing data

After running mangosd at least once to generate the volume, move dbc, maps, vmaps and mmaps to ```/var/lib/docker/volumes/mangos-zero-data/_data/``` (or wherever your docker volumes are stored).

# Running

After you set up the database, config and map data, you can start the server with:

    docker start mangos0-realmd
    docker start mangos0-mangosd

Don't forget to edit your realmlist (both on the server and the client)!
